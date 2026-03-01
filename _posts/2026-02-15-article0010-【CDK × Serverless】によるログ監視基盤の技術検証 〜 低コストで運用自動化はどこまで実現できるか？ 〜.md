---
layout: post
title: "article0010-【CDK × Serverless】によるログ監視基盤の技術検証 〜 マネージドサービスのみで運用自動化はどこまで実現できるか 〜"
date: 2026-02-15
author: Seal
---
# ☁️ article0010　【CDK × Serverless】によるログ監視基盤の技術検証 〜 マネージドサービスのみで運用自動化はどこまで実現できるか 〜

こんにちは。

今回は AWS CDK を用いて、CloudFront のアクセスログを自動解析し、異常検知まで行う監視基盤を構築しました。

単なるハンズオンではなく、

「サーバーを一切持たずに、実運用レベルの監視フローを構築できるか」

をテーマに技術検証を行っています。

# ゴールは以下です：

- ✅ すべて IaC（CDK）で構築

- ✅ サーバーレス構成

- ✅ ログ解析の自動化

- ✅ エラー検知 → 通知まで自動化

- ✅ 運用作業ゼロ

# アーキテクチャ：

![アーキテクチャ]( /assets/images/0010-01.png )


# Task 1 — CDK だけで全リソースを構築できるか
## 目的

手動操作なしで再現可能なインフラを作れるか確認

## 対象

S3

CloudFront

Lambda

CloudWatch

SNS

## コード

📎cdk-log-monitor.ts

```
#!/usr/bin/env node
import * as cdk from 'aws-cdk-lib';
import { MonitoringStack } from '../lib/monitoring-stack';

const app = new cdk.App();

new MonitoringStack(app, 'CfLogMonitoringStack', {
  alertEmail: 'oukaihyou@gmail.com',
});
```

---

📎monitoring-stack.ts

```
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as cloudfront from 'aws-cdk-lib/aws-cloudfront';
import * as origins from 'aws-cdk-lib/aws-cloudfront-origins';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as s3n from 'aws-cdk-lib/aws-s3-notifications';
import * as sns from 'aws-cdk-lib/aws-sns';
import * as subs from 'aws-cdk-lib/aws-sns-subscriptions';
import * as cloudwatch from 'aws-cdk-lib/aws-cloudwatch';
import * as cloudwatch_actions from 'aws-cdk-lib/aws-cloudwatch-actions';

interface Props extends cdk.StackProps {
  alertEmail: string;
}

export class MonitoringStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props: Props) {
    super(scope, id, props);

    const siteBucket = new s3.Bucket(this, 'SiteBucket', {
      removalPolicy: cdk.RemovalPolicy.DESTROY,
      autoDeleteObjects: true,
    });

    const logBucket = new s3.Bucket(this, 'LogBucket', {
      versioned: true,
      removalPolicy: cdk.RemovalPolicy.DESTROY,
      autoDeleteObjects: true,
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ACLS, 
      publicReadAccess: false,
      accessControl: s3.BucketAccessControl.LOG_DELIVERY_WRITE,
    });

    const parserFn = new lambda.Function(this, 'ParserFn', {
      runtime: lambda.Runtime.PYTHON_3_11,
      handler: 'log-parser.lambda_handler',
      code: lambda.Code.fromAsset('lambda'),
      timeout: cdk.Duration.seconds(60),
    });

    logBucket.addEventNotification(
      s3.EventType.OBJECT_CREATED,
      new s3n.LambdaDestination(parserFn)
    );

    logBucket.grantRead(parserFn);

    const distribution = new cloudfront.Distribution(this, 'Distribution', {
      defaultBehavior: {
      origin: origins.S3BucketOrigin.withOriginAccessControl(siteBucket),
      viewerProtocolPolicy: cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS,
    },
      defaultRootObject: 'index.html',
      enableLogging: true,
      logBucket: logBucket,
      logFilePrefix: 'access-logs/',
    });

    const topic = new sns.Topic(this, 'AlertTopic');
    topic.addSubscription(new subs.EmailSubscription(props.alertEmail));

    const metric = new cloudwatch.Metric({
      namespace: 'CFLog',
      metricName: 'ErrorCount',
      statistic: 'sum',
      period: cdk.Duration.minutes(1),
    });

    const alarm = new cloudwatch.Alarm(this, 'ErrorAlarm', {
      metric,
      threshold: 5,
      evaluationPeriods: 1,
    });

    alarm.addAlarmAction(new cloudwatch_actions.SnsAction(topic));

    new cdk.CfnOutput(this, 'URL', {
      value: 'https://' + distribution.domainName,
    });
  }
}
```

---



## デプロイ結果

![デプロイ結果1]( /assets/images/0010-02.png )
![デプロイ結果2]( /assets/images/0010-03.png )

✅ 結果

すべてコード管理可能
環境の再構築も cdk deploy のみで完結

# Task 2 — S3 イベントで Lambda は自動起動するか

## 目的

ログ生成をトリガーに自動処理できるか確認

## 手順

1.CloudFront 経由でアクセス

![CloudFront 経由でアクセス]( /assets/images/0010-04.png )

2.S3 にログ生成

![CloudFront 経由でアクセス]( /assets/images/0010-05.png )

3.Lambda 実行ログ確認









✅ 結果

PutObject イベントで即時実行
リアルタイム処理が可能

# Task 3 — ログを解析しメトリクス化できるか
## 目的

ログを数値データとして扱える形に変換

## 実装

5xx エラー数カウント
カスタムメトリクス送信

📎log-parser.py

```
import boto3

cloudwatch = boto3.client('cloudwatch')

def lambda_handler(event, context):
    error_count = 0

    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']

        s3 = boto3.client('s3')
        obj = s3.get_object(Bucket=bucket, Key=key)
        body = obj['Body'].read().decode('utf-8')

        for line in body.splitlines():
            parts = line.split()
            if len(parts) > 8:
                try:
                    status = int(parts[8])
                    if status >= 500:
                        error_count += 1
                except:
                    continue

    cloudwatch.put_metric_data(
        Namespace='CFLog',
        MetricData=[{
            'MetricName': 'ErrorCount',
            'Value': error_count,
            'Unit': 'Count'
        }]
    )
```

📸 Metrics 画面


✅ 結果

CloudWatch 上でグラフ化を確認
数値ベースの監視が可能になった

# Task 4 — 異常時にアラート通知できるか
## 目的

エラー増加時に自動検知 → 通知

## 条件

5xx > 閾値

📸

Alarm 状態

SNS 通知メール

✅ 結果

ALARM → 通知まで自動化成功
人的監視不要

💡 技術的な学び・考察
# 良かった点

```
1.フルマネージド構成のため運用負荷がほぼゼロ　→　全部AWSサービスで構成
2.サーバー保守不要 →　EC2がない
3.CDK による再現性　→　もう一度deploy
4.イベント駆動でスケール自動化　→　Lambdaが自動スケールできる
```

# 改善・拡張アイデア

## 1.Athena でログ分析　→　S3 のログを Athena で SQL 分析可能



## 2.ダッシュボード可視化　→　Observability



## 3.Step Functions 連携　→　異常時は Step Functions で自動復旧フローを実行　→　Auto Remediation/Self-Healing　超重要!!



## 4.Security Hub 連携　→　Security Hub にセキュリティイベントを集約




