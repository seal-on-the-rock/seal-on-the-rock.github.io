---
layout: post
title: "article0017-【PrivateLink】によるサービス単位アクセス制御～AWS Transit Gateway 構成をさらに進化させる～"
date: 2026-04-03
author: Seal
---

## Hello ~ 

前回の記事では、AWS Transit Gateway を利用して、
**全VPCのインターネット出口を Security VPC に集約する構成**を解説しました。

その構成により：

* すべてのアウトバウンド通信を一元管理
* セキュリティ・監査の強化
* DevVPC、ProdVPCとSecVPCは1つだけのNATを共用して、コストダウン

を実現できました。

---

しかし今回はさらに一歩進めます。

> **「VPC同士はつながっているが、必要以上にはアクセスさせない」**

つまり：

> **ネットワーク単位ではなく「サービス単位」でアクセス制御する**

これが今回のテーマです。

---

# なぜアップグレードが必要なのか？

従来の TGW 構成では  Dev → TGW → Prod（VPC全体にアクセス可能）

![0016-1]( /assets/images/0016-1.svg )

👉 つまり：

* Dev から Prod のすべてのリソースに到達可能
* 不要なアクセス経路が存在
* セキュリティ境界が粗い

---

## 問題点

* 最小権限の原則に反する
* 万が一 Dev が侵害された場合、Prod に横展開される可能性
* 「ネットワーク信頼」に依存した設計

---

# 解決アプローチ

ここで登場するのが：

👉 AWS PrivateLink

---

## コンセプト

> ❌ VPC同士をつなぐ

> ✅ 必要な「サービス」だけ公開する

---

#  改造後の構成イメージ

![0017-1]( /assets/images/0017-1.svg )

---

#  今回の変更って何をしているの？

ここからは「コードがどう変わったか」ではなく、

👉 実際に何を作って、どう振る舞いが変わったのかを整理します。

## ① Prod VPC：サービスを公開する

コード：

```text
import * as cdk from 'aws-cdk-lib';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import * as elbv2 from 'aws-cdk-lib/aws-elasticloadbalancingv2';
import * as elbv2_targets from 'aws-cdk-lib/aws-elasticloadbalancingv2-targets';
import * as iam from 'aws-cdk-lib/aws-iam';
import { Construct } from 'constructs';

export class ProdVpc extends Construct {
  public readonly vpc: ec2.Vpc;
  public readonly privateSubnet: ec2.ISubnet;
  // PrivateLink: Dev VPCがInterface Endpointを作成する際に参照するEndpoint Service名
  public readonly endpointServiceName: string;

  constructor(scope: Construct, id: string) {
    super(scope, id);

    this.vpc = new ec2.Vpc(this, 'ProdVpc', {
      vpcName: 'prod-vpc',
      ipAddresses: ec2.IpAddresses.cidr('10.0.0.0/16'),
      maxAzs: 1,
      // プライベートサブネットのみ構成。NAT GatewayおよびIGWは作成しない
      subnetConfiguration: [
        {
          name: 'prod-private',
          subnetType: ec2.SubnetType.PRIVATE_ISOLATED,
          cidrMask: 24,
        },
      ],
      natGateways: 0,
    });

    this.privateSubnet = this.vpc.isolatedSubnets[0];

    cdk.Tags.of(this.vpc).add('Name', 'prod-vpc');
    cdk.Tags.of(this.privateSubnet).add('Name', 'prod-private-subnet-1a');

    // ── PrivateLink プロバイダー側 ──────────────────────────────────────────

    // 1. バックエンドサービス用EC2（HTTPサービスのサンプル、ポート80）
    const svcSg = new ec2.SecurityGroup(this, 'ProdSvcSg', {
      vpc: this.vpc,
      description: 'Prod backend service SG',
      allowAllOutbound: false,
    });
    // NLBはソースIPを変換しないため、VPC CIDRからのTCP 80を許可する
    svcSg.addIngressRule(ec2.Peer.ipv4('10.0.0.0/16'), ec2.Port.tcp(80), 'Allow HTTP from NLB');

    const svcInstance = new ec2.Instance(this, 'ProdSvcInstance', {
      vpc: this.vpc,
      vpcSubnets: { subnets: [this.privateSubnet] },
      instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
      machineImage: ec2.MachineImage.latestAmazonLinux2(),
      securityGroup: svcSg,
      // 起動時にhttpdをインストールしてサービスを模擬する
      userData: ec2.UserData.custom([
        '#!/bin/bash',
        'yum install -y httpd',
        'echo "Hello from Prod Service" > /var/www/html/index.html',
        'systemctl start httpd',
        'systemctl enable httpd',
      ].join('\n')),
    });
    cdk.Tags.of(svcInstance).add('Name', 'prod-svc-instance');

    // 2. 内部向けNetwork Load Balancer
    const nlb = new elbv2.NetworkLoadBalancer(this, 'ProdNlb', {
      vpc: this.vpc,
      internetFacing: false, // 内部NLB。インターネットへは公開しない
      vpcSubnets: { subnets: [this.privateSubnet] },
      loadBalancerName: 'prod-internal-nlb',
    });

    const listener = nlb.addListener('HttpListener', { port: 80 });
    listener.addTargets('ProdSvcTarget', {
      port: 80,
      targets: [new elbv2_targets.InstanceTarget(svcInstance, 80)],
      healthCheck: { port: '80', protocol: elbv2.Protocol.TCP },
    });

    // 3. VPC Endpoint Service（PrivateLink プロバイダー）
    const endpointService = new ec2.VpcEndpointService(this, 'ProdEndpointService', {
      vpcEndpointServiceLoadBalancers: [nlb],
      acceptanceRequired: false, // 接続要求を自動承認する（手動承認不要）
      allowedPrincipals: [
        // 同一AWSアカウントからの接続のみ許可し、クロスアカウント接続を防止する
        new iam.ArnPrincipal(`arn:aws:iam::${cdk.Stack.of(this).account}:root`),
      ],
    });

    this.endpointServiceName = endpointService.vpcEndpointServiceName;
  }
}
```


Prod 側では、EC2 をそのまま使うのではなく、

👉 他のVPCから利用できるサービスとして公開する

構成にしています。

やっていること：

1. EC2（Webサーバ）を用意

2. NLB を前段に配置

3. NLB を Endpoint Service として公開

つまり Prod 側は：

✅ 「サービス提供側」として構成を整えている

## ② Dev VPC：サービスに接続する

コード：

```text
import * as cdk from 'aws-cdk-lib';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import { Construct } from 'constructs';

export class DevVpc extends Construct {
  public readonly vpc: ec2.Vpc;
  public readonly privateSubnet: ec2.ISubnet;
  // PrivateLink: Dev側サービスがProdサービスへアクセスする際に使用するEndpoint DNS名
  public readonly endpointDnsName: string;

  constructor(scope: Construct, id: string, prodEndpointServiceName: string) {
    super(scope, id);

    this.vpc = new ec2.Vpc(this, 'DevVpc', {
      vpcName: 'dev-vpc',
      ipAddresses: ec2.IpAddresses.cidr('10.1.0.0/16'),
      maxAzs: 1,
      // プライベートサブネットのみ構成。NAT GatewayおよびIGWは作成しない
      subnetConfiguration: [
        {
          name: 'dev-private',
          subnetType: ec2.SubnetType.PRIVATE_ISOLATED,
          cidrMask: 24,
        },
      ],
      natGateways: 0,
    });

    this.privateSubnet = this.vpc.isolatedSubnets[0];

    cdk.Tags.of(this.vpc).add('Name', 'dev-vpc');
    cdk.Tags.of(this.privateSubnet).add('Name', 'dev-private-subnet-1a');

    // ── PrivateLink コンシューマー側 ────────────────────────────────────────

    // EndpointのSecurity Group: Prod Endpoint ServiceへのTCP 80アウトバウンドのみ許可
    const endpointSg = new ec2.SecurityGroup(this, 'DevEndpointSg', {
      vpc: this.vpc,
      description: 'SG for Prod PrivateLink Interface Endpoint',
      allowAllOutbound: false,
    });
    endpointSg.addEgressRule(ec2.Peer.anyIpv4(), ec2.Port.tcp(80), 'Allow HTTP to Prod via PrivateLink');
    // Dev VPC内のインスタンスからこのEndpointへのアクセスを許可
    endpointSg.addIngressRule(ec2.Peer.ipv4('10.1.0.0/16'), ec2.Port.tcp(80), 'Allow HTTP from Dev VPC');

    // Interface VPC Endpoint → ProdのEndpoint Serviceへ接続
    const endpoint = new ec2.InterfaceVpcEndpoint(this, 'ProdServiceEndpoint', {
      vpc: this.vpc,
      service: new ec2.InterfaceVpcEndpointService(prodEndpointServiceName, 80),
      subnets: { subnets: [this.privateSubnet] },
      securityGroups: [endpointSg],
      // カスタムEndpoint ServiceはprivateDns非対応のため、返却されるDNS名を使用する
      privateDnsEnabled: false,
      open: false,
    });
    cdk.Tags.of(endpoint).add('Name', 'dev-to-prod-privatelink-endpoint');

    // 最初のDNSエントリをアクセス用アドレスとして使用
    this.endpointDnsName = endpoint.vpcEndpointDnsEntries[0];
  }
}
```

Dev 側では、

👉 Prod が公開したサービスに接続するための入口を作る

構成にしています。

やっていること：

1. Interface Endpoint を作成
2. Prod の Endpoint Service に接続

つまり Dev 側は：

✅ 「サービス利用側」として接続ポイントを持つ

##  ③ 直接通信は削除

![0017-2]( /assets/images/0017-2.png )

理由：

👉 必要なサービスだけにアクセスさせるため

結果：

❌ VPC内の他リソースにはアクセス不可

✅ 公開されたサービスのみ利用可能

##  ④ 作成順序の変更

先に Prod を作成

その後 Dev を作成

理由：

👉 Dev が接続先（Endpoint Service）を参照する必要があるため

##  ⑤ アクセス方法

Dev 側では：

curl http://xxxx.vpce.amazonaws.com

内部では：

Dev → Interface Endpoint → AWS内部 → ProdのNLB → EC2


## まとめ

今回の変更は：

Prod のサービスを Dev から安全に利用できるようにした

ポイント：

❌ VPC同士を直接接続しない

✅ 必要なサービスのみ公開・利用する

👉 これが PrivateLink の基本的な使い方です。

---

# 通信フロー解説

## ✅ Dev → Prod（PrivateLink経由）

```text
Dev EC2
 → Interface Endpoint（ENI）
 → AWS内部ネットワーク
 → NLB（Prod）
 → EC2（httpd）
```

✔ TGWを通らない

✔ 公開IP不要

✔ 特定サービスのみアクセス可能

---

## ✅ Dev → Internet（従来通り）

```text
Dev → TGW → Security → NAT → IGW → Internet
```

✔ 変更なし

✔ 集中出口維持

---

## ❌ Dev → Prod VPC 全体（ブロック）

```text
Dev → TGW → ルートなし → DROP
```

👉 不要なネットワークアクセスを完全遮断

---

# セキュリティ的な進化

## Before（従来）

* ネットワーク単位の許可
  
* Dev → Prod 全体アクセス可能

---

## After（今回）

* サービス単位の許可
  
* Dev → 特定サービスのみ

---

## 効果

* 最小権限の徹底
  
* 横方向移動の防止
  
* ゼロトラストに近づく構成

---

# 設計思想まとめ

| レイヤー   | 技術          | 役割      |
| ------ | ----------- | ------- |
| ネットワーク | TGW         | 接続制御    |
| 出口     | NAT Gateway | インターネット |
| サービス公開 | PrivateLink | 最小公開    |

---

# 一言でまとめると

> **TGWは「どこに行けるか」を制御し、PrivateLinkは「何にアクセスできるか」を制御する**

---




