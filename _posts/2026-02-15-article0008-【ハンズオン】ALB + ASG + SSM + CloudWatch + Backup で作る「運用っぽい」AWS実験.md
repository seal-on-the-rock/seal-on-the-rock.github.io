---
layout: post
title: "article0008-【ハンズオン】ALB + ASG + SSM + CloudWatch + Backup で作る「運用っぽい」AWS実験"
date: 2026-02-15
author: Seal
---

こんにちは。今回は AWS の基本サービスを使って、運用に必要な要素を一通り揃えた小さな実験環境を作りました。

---

# ゴールは以下です：

- ✅ ALB の DNS で Web ページにアクセスできる
  
  ![1]( /assets/images/0008-1.png )
  
- ✅ Target Group に EC2 が 2 台登録され、Healthy になる
  
  ![2]( /assets/images/0008-2.png )
  
- ✅ Session Manager で EC2 に入れる（SSH 不要）
  
  ![3]( /assets/images/0008-3.png )
  
- ✅ CloudWatch Alarm を作成でき、メール通知が届く
  
  ![4]( /assets/images/0008-4.png )
  
- ✅ CloudWatch Logs に nginx の access/error が集約される
  
  ![5]( /assets/images/0008-5.png )
  
- ✅ CloudTrail で操作ログが追える
  
  ![6]( /assets/images/0008-6.png )
  
- ✅ AWS Backup でバックアッププラン作成＋少なくとも 1 回バックアップ実行履歴が残る
  
  ![7]( /assets/images/0008-7.png )
  

---

# 全体アーキテクチャ

構成は以下です（最小構成）：

```text
VPC（2AZ）
Public Subnet ×2
ALB（Internet-facing）
Target Group（Instance）
Auto Scaling Group（Desired=2, Min=2, Max=4）
EC2（Amazon Linux 2023）×2（ASG 管理）
Systems Manager（Session Manager）
CloudWatch（Alarm + Logs）
CloudTrail（操作ログ）
AWS Backup（EBS スナップショット）
```

---

# Step 1. VPC / Subnet を作成

## ポイント

- ALB は 2 つ以上の AZ に跨る Subnet が必要
- 実験のため、今回は EC2 も public subnet に配置（後述）

---

# Step 2. IAM：EC2 用のロールを作成

```
IAM → Roles → Create role
Trusted entity：AWS service → EC2
```

## Attach policies

- AmazonSSMManagedInstanceCore（SSM 必須）
- CloudWatchAgentServerPolicy（CloudWatch Logs 送信用）

```
Role name：ops-case-ec2-role
```

## 知識ポイント

> Session Manager を使うには  
> SSM Agent + IAM role + ネットワーク が必要。  
>  
> SSH を使わない運用は、セキュリティ面でかなり強い。

---

# Step 3. Security Group を作成

## 3.1 ALB 用 SG（ops-case-sg-alb）

| Rule | Setting |
|-------|-----------|
| Inbound | HTTP 80：0.0.0.0/0 |
| Outbound | All traffic：0.0.0.0/0 |

## 3.2 EC2 用 SG（ops-case-sg-ec2）

| Rule | Setting |
|-------|-----------|
| Inbound | HTTP 80：Source = ALB SG（sg-xxxx） |
| Outbound | All traffic：0.0.0.0/0 |

## 知識ポイント

> EC2 側の Inbound で 0.0.0.0/0 を許可するのではなく、  
> ALB の SG からのみ許可するのが定番。

---

# Step 4. Launch Template を作成（ここが重要）

```
EC2 → Launch Templates → Create
Name：ops-case-lt-web
AMI：Amazon Linux 2023
Instance type：t3.micro
Key pair：None（SSH 不要）
IAM role：ops-case-ec2-role
Security group：ops-case-sg-ec2
```

User data（起動時に nginx をインストールして起動）

> ここは改行ミスで失敗しやすいので、必ず以下のように正しく書きます。

```bash
#!/bin/bash
set -euxo pipefail

dnf update -y
dnf install -y nginx amazon-cloudwatch-agent

systemctl enable nginx
systemctl start nginx

HOST=$(hostname)
cat > /usr/share/nginx/html/index.html <<EOF
<h1>ops-case web</h1>
<p>host: ${HOST}</p>
<p>time: $(date)</p>
EOF

cat > /opt/aws/amazon-cloudwatch-agent/bin/config.json <<'EOF'
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          { "file_path": "/var/log/nginx/access.log", "log_group_name": "ops-case-nginx-access", "log_stream_name": "{instance_id}" },
          { "file_path": "/var/log/nginx/error.log",  "log_group_name": "ops-case-nginx-error",  "log_stream_name": "{instance_id}" }
        ]
      }
    }
  }
}
EOF

/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s
```

---

# 注意点（超重要）

User data の失敗は以下で確認できます：

```bash
/var/log/cloud-init-output.log
/var/log/cloud-init.log
```

特に以下が出た場合は user data が実行されていません：

```
Failed to run module scripts-user
part-001 No such file or directory
```

---

# Step 5. Target Group を作成

```
EC2 → Target Groups → Create
Type：Instances
Protocol：HTTP
Port：80
Health check path：/
```

※ ASG が自動登録するので、この時点ではインスタンス登録は不要。

---

# Step 6. ALB を作成

```
EC2 → Load Balancers → Create → Application Load Balancer
Scheme：Internet-facing
IP type：IPv4
Subnets：public subnet ×2（2AZ）
SG：ops-case-sg-alb
Listener：HTTP 80 → Target Group（ops-case-tg）
```

作成後、ALB の DNS 名が発行されます：

```
ops-case-alb-xxxx.us-east-1.elb.amazonaws.com
```

---

# Step 7. Auto Scaling Group を作成

```
EC2 → Auto Scaling Groups → Create
Name：ops-case-asg
Launch template：ops-case-lt-web
Subnets：public subnet ×2
Load balancing：Attach to ALB → ops-case-tg
Health checks：ELB
Desired：2 / Min：2 / Max：4
```

---

# Step 8. SSM（Session Manager）で SSH なしログイン

```
Systems Manager → Fleet Manager → Managed nodes
or
Systems Manager → Session Manager
```

EC2 が Online になれば成功。

## 知識ポイント

- IAM role（AmazonSSMManagedInstanceCore）
- SSM Agent が動作している
- SSM endpoint に到達できるネットワーク

---

# Step 9. CloudWatch Alarm（ALB 5XX）

```
CloudWatch → Alarms → Create alarm
ApplicationELB → HTTPCode_Target_5XX_Count
Threshold：5分間で > 0
SNS topic → Email subscription（Confirm 必須）
```

---

# Step 10. CloudWatch Logs（nginx access/error）

自動作成される Log group：

- ops-case-nginx-access
- ops-case-nginx-error

確認場所：

```
CloudWatch → Logs → Log groups
```

---

# Step 11. CloudTrail（操作ログ確認）

```
CloudTrail → Event history
```

例：

- CreateLoadBalancer
- CreateAutoScalingGroup
- CreateLaunchTemplate
- CreateAlarm
- CreateBackupPlan

## 知識ポイント

> 「誰が」「いつ」「何をしたか」が追えるので、運用・監査に必須。

---

# Step 12. AWS Backup（EBS スナップショット）

```
AWS Backup → Create backup plan
Name：ops-case-backup-plan
Daily
Retention：7 days
```

Backup selection：

- Assign by tags（推奨）
- Select resources（学習では簡単）

---

# 最終チェック（今回の達成条件）

- ✅ ALB DNS で Web が表示される
- ✅ Target Group に 2 台 Healthy
- ✅ Session Manager でログインできる
- ✅ Alarm 作成成功、SNS メール通知が届く
- ✅ Logs に nginx access/error が入る
- ✅ CloudTrail に操作履歴が残る
- ✅ AWS Backup の Plan と実行履歴が確認できる

---

# よくあるハマりポイント（今回実際に踏んだ）

## 1) User data の改行ミス

```bash
#!/bin/bashdnf update -y
```

くっつくと即死します。

## 2) Subnet の Auto-assign public IPv4 が片方 OFF

- 片方だけ public DNS が付く
- 片方は付かない

## 3) Target Group が Unhealthy のときはまず EC2 内を確認

```bash
systemctl status nginx
ss -lntp
curl http://localhost/
```
