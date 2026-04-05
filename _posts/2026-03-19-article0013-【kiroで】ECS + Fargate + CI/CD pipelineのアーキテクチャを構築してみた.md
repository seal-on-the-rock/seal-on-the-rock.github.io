---
layout: post
title: "article0013-【kiroで】ECS + Fargate + CI/CD pipelineのアーキテクチャを構築してみた"
date: 2026-03-19
tags: [kiro, ECS, Fargate, CI/CD]
author: Seal
---


## 📌 はじめに

今回はAIコーディングツール「Kiro」を使って、AWS上に3層Webアーキテクチャを構築してみました。

従来のEC2ベース構成ではなく、コンテナ × サーバーレスの構成を採用し、よりモダンなクラウドアーキテクチャを体験することが目的です。

---

## 🏗️ アーキテクチャ概要

今回構築した構成は以下の通りです。

```
Internet
   ↓
ALB（Public Subnet）
   ↓
ECS Service（Private Subnet）
   ↓
Fargate Tasks
```

---

---

下記のpromptで簡単に構築リソースを作成してくれました

```
Create an AWS CDK TypeScript project that deploys:
- A VPC with public and private subnets
- An ECS cluster
- A Fargate service running nginx
- An Application Load Balancer to expose the service
Generate the full project structure and CDK code.Create the project files in my folder.
```

しかも、下記のpromptで構築図も生成できます。

```
Generate an AWS architecture diagram for my CDK stack.
```

一旦に下記のような構築図になりました。

![AWS1]( /assets/images/0013-1.svg)

## 🔥 ポイント①：EC2を使わない構成

従来の構成：

```
ALB → EC2（Auto Scaling）
```

今回の構成：

```
ALB → ECS（Fargate Tasks）
```

つまり、サーバー（EC2）を管理する必要がなくなり、

* OS管理不要
* パッチ適用不要
* スケーリング簡略化

といったメリットがあります。


## ⚖️ ポイント②：Auto Scalingの対象が変わる

EC2構成では：

```
2〜10台のEC2
```

Fargateでは：

```
2〜10個のTask
```

つまりスケーリング単位が「サーバー」から「コンテナ」に変わります。

---

## 🔐 ポイント③：Private Subnetでもアクセス可能な理由

ECS TaskはPrivate Subnetに配置していますが、外部からアクセス可能です。

その理由は：

👉 ALBがリクエストを転送しているため

```
ユーザー → ALB → ECS Task
```

直接Taskにアクセスしているわけではありません。

また、Security Groupで以下のように制御されています：

* ALB：0.0.0.0/0 からアクセス許可
* ECS：ALBからの通信のみ許可

これによりセキュリティが向上します。

---

## 🧠 ポイント④：IAM Task Role

ECSでは2種類のIAM Roleがあります：

### ① Task Role

アプリケーション用

例：

* S3アクセス
* DynamoDBアクセス

### ② Execution Role

ECS実行用

例：

* ECRからイメージ取得
* CloudWatch Logs出力

---

## 🌐 ポイント⑤：NAT Gatewayの役割

Private Subnet内のTaskが外部通信するためには：

* Dockerイメージ取得（ECR）
* API通信

などが必要になります。

そのために：

👉 NAT Gateway を利用します

（コスト削減のため1つだけ配置）

---

## 🔒 ポイント⑥：HTTPのみ構成について

今回の構成ではALBはPort 80（HTTP）のみ使用しています。

本番環境では：

* ACMで証明書発行
* HTTPS（443）を有効化

するのが一般的です。

---


しかし、ACMとかCloudFrontなどが表現していない気がします。

これから、promptを追加します。

ACMの追加：

```
Please modify the CDK stack to support HTTPS.
Requirements:
- Request an SSL certificate using AWS Certificate Manager (ACM)
- Add an HTTPS listener on port 443 to the Application Load Balancer
- Attach the ACM certificate to the listener
- Redirect HTTP (port 80) to HTTPS
- Keep the ECS Fargate service unchanged
Explain what you added.
```

次はauto scalingの具体な規則を明記：

```
Add auto scaling to the ECS Fargate
service.Requirements:
- Configure ECS Service Auto Scaling- Minimum tasks: 2- Maximum tasks: 6
- Scale out when average CPU utilization > 70%
- Scale in when CPU utilization < 30%
- Use AWS best practicesExplain the scaling policy you configured.
```

後はもっと本番のようなECRも登場します：

```
Update the ECS task definition to use a Docker image stored in Amazon ECR instead of pulling nginx from Docker Hub.
Requirements:
- Create an Amazon ECR repository
- Reference the ECR image in the task definition
- Keep the rest of the architecture unchanged
Explain how to push a Docker image to ECR.
```

次にCloudFrontを追加します（これはEdge Tierに重要な存在ですよね）：

```
Update the ECS task definition to use a Docker image stored in Amazon ECR instead of pulling nginx from Docker Hub.
Requirements:
- Create an Amazon ECR repository
- Reference the ECR image in the task definition
- Keep the rest of the architecture unchanged
Explain how to push a Docker image to ECR.
```

最後に、CI/CD　pipeline を追加します。

```
Add a CI/CD pipeline for deploying this CDK application.
Requirements:
- Use AWS CodePipeline
- Source from GitHub
- Build using CodeBuild
- Deploy using CDK
- Follow AWS best practices
Explain the pipeline stages.
```

リソースを修正してもらったら、もう一回下記のpromptで構築図生成しました。

```
Generate an AWS architecture diagram for my CDK stack.
```

こういう感じです。

![AWS2]( /assets/images/0013-2.jpg)

とても凄いでしょう？！

でもね、二つのAZは分けて表示すればいいと思います。

後は、Route53がちゃんと表示すればもっといいと思います。

そうしたら、下記のpromptでRoute53を追加します。

```
Enhance the existing ECS + Fargate 3-tier architecture by adding a custom domain with Route 53 and HTTPS support.
Requirements:
1. Create a Route 53 hosted zone for a custom domain (e.g., example.com).
2. Create an A record (Alias) pointing to the Application Load Balancer.
3. Request an ACM certificate for the domain (e.g., www.example.com).
4. Validate the certificate using DNS validation in Route 53.
5. Update the ALB to support HTTPS (port 443) using the ACM certificate.
6. Keep HTTP (port 80) but redirect all traffic to HTTPS.
7. Ensure security groups allow HTTPS traffic.
```

もう一回構築図を生成すれば：

![AWS3]( /assets/images/0013-3.svg)

めちゃくちゃいいですよね！！

本アーキテクチャは以下の特徴を持つ：

- フルマネージドなコンテナ実行環境（Fargate）
- 高可用性（Multi-AZ）
- セキュアな通信（HTTPS + Secrets管理）
- CI/CDによる自動デプロイ
- ログ・監視の一元管理

---

# 全体構成

本構成は大きく以下の5レイヤーで構成される：

1. エッジレイヤー（Route53 / CloudFront）
2. ネットワーク（VPC / Subnet）
3. コンピュート（ECS Fargate）
4. ストレージ & ログ
5. CI/CDパイプライン

---

# ① エッジレイヤー

## Route53（DNS）
- `example.com` の名前解決を担当
- CloudFrontへルーティング

### 設計意図
- CDNを経由することでパフォーマンス向上
- セキュリティ強化

---

## ACM（証明書管理）
- TLS証明書を発行
- CloudFront用は us-east-1 で管理

---

## CloudFront（CDN）
- HTTPS終端
- HTTP/2・HTTP/3対応
- コンテンツキャッシュ
- 圧縮（gzip / brotli）

### メリット
- レイテンシ削減
- DDoS耐性向上
- ALBを直接公開しない設計

---

## CloudFront Logs
- S3にアクセスログを保存
- Athena等で分析可能

---

# ② ネットワーク設計（VPC）

## Multi-AZ構成
- AZ-a / AZ-b に分散配置
- 高可用性を確保

---

## Subnet構成

### Public Subnet
- Application Load Balancer配置

### Private Subnet
- ECS Fargate配置（Public IPなし）

### 設計ポイント
- アプリケーションは外部公開しない
- ALB経由のみアクセス可能

---

# ③ ALB（Application Load Balancer）

## 主な設定
- HTTP → HTTPS リダイレクト
- HTTPS（443）
- TLSポリシー適用
- 無効ヘッダの除外

---

## ヘルスチェック
- `GET /` → 200

---

## 証明書
- ACMで管理（DNS検証）
- 自動更新対応

---

# ④ ECS Fargate（コンテナ実行環境）

## ECS Cluster
- Container Insights有効化
- CloudWatchと連携

---

## Fargate Task

### コンテナ
- nginx
- ポート: 80

### リソース
- CPU: 256
- Memory: 512MB

---

## セキュリティ
- Security GroupでALBからの通信のみ許可
- Public IPなし

---

## ログ
- CloudWatch Logsに送信
- 保持期間：30日

---

## Auto Scaling

### スケールアウト
- CPU > 70% → +2タスク

### スケールイン
- CPU < 30% → -1タスク

---

## デプロイ戦略
- minimumHealthyPercent: 100%
- maximumPercent: 200%
- ローリングアップデート

---

# ⑤ ECR（コンテナレジストリ）

## 設定
- イメージpush時スキャン
- タグ保持数：10
- 未使用イメージ：7日後削除

---

# ⑥ ログ設計

## CloudWatch Logs
- アプリケーションログ保存

## S3
- ALB / CloudFrontアクセスログ保存

---

## ログ使い分け

| 種類 | 保存先 |
|------|--------|
| アプリログ | CloudWatch |
| アクセスログ | S3 |

---

# ⑦ Secrets管理

## Secrets Manager
- GitHub OAuthトークン管理

### ポイント
- 認証情報のハードコードを防止
- セキュリティ向上

---

# ⑧ CI/CD パイプライン

## フロー

```
GitHub → CodePipeline → CodeBuild → ECS Deploy
```

---

## Stage 1：Source
- GitHub pushでトリガー
- OAuth認証（Secrets Manager）

---

## Stage 2：Build
- Docker build
- ECRへpush
- CDK synth実行

※ privileged mode 必須

---

## Stage 3：Approve
- 手動承認ステージ

---

## Stage 4：Deploy
- CDKでインフラ更新
- CloudFormation経由でECSデプロイ

---

## Artifact管理
- S3（バージョニング有効）
- CloudWatch Logs（30日保存）

---

# セキュリティ設計まとめ

- 外部公開はCloudFrontのみ
- ECSはPrivate Subnetに配置
- Security Groupで通信制御
- Secrets Managerで機密情報管理

---

# 本構成のメリット

## 1. フルマネージド
- Fargateによりサーバ管理不要

## 2. 高可用性
- Multi-AZ構成

## 3. セキュリティ
- HTTPS強制
- Public IP未使用

## 4. スケーラビリティ
- Auto Scaling対応

## 5. DevOps対応
- CI/CDパイプライン完備

---

# 改善ポイント

- AWS WAFの導入
- AWS X-Rayによるトレーシング
- Blue/Greenデプロイ（CodeDeploy）
- PrivateLinkの活用

---






