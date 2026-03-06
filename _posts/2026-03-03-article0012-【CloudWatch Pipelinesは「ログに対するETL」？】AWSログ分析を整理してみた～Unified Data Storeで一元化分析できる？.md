---
layout: post
title: "article0012-【CloudWatch Pipelinesは「ログに対するETL」？】AWSログ分析を整理してみた～Unified Data Storeで一元化分析できる？"
date: 2026-03-06
author: Seal
---
# ☁️ article0012　【CloudWatch Pipelinesは「ログに対するETL」？】AWSログ分析を整理してみた～Unified Data Storeで一元化分析できる？

運用において避けて通れないテーマの一つが **ログ分析** です。

AWSのシステム運用では、ログはさまざまな場所に存在します。
　·アプリケーションログ
　·ALBアクセスログ
　·Lambdaログ
　·セキュリティログ
　·**外部SaaSログ（Oktaなど）**　
 
障害調査では、多くの場合こうなります。

CloudWatch → OpenSearch → Athena → Okta Console　→　ほか外部SaaS Console

複数の画面を行き来する必要があり、調査効率が悪い のが現状です。

![0012-6]( /assets/images/0012-6.png )
    
third party logが分散 　　　　→ 　個別ログイン操作が面倒

ログの形式が統一されていない 　→ 　総合分析為の解析が手間

障害調査　　　　　　　　　　 　→ 　結局行ったり来たり

最近のAWSの方向性：**Unified Data Store + Pipelines + Facet**

AWSは最近、CloudWatchを中心にログ分析を統合する方向に進んでいます。

キーワードは

**CloudWatch Pipelines**

**Unified Data Store**

**Facet / Unified Query**

## 1. CloudWatch Pipelines：ログに対するETL
Pipelinesは、ログを保存前に 整形・変換・補強 する仕組みです。
機械学習のETLと比較しましょう～

![0012-1]( /assets/images/0012-1.png )

この構造化ログがあることで、ログ形式が統一され、後続の分析が容易になります。

## 2. Unified Data Store：ログの一元化

Structured Logs は Unified Data Store に集約されます。

![0012-2]( /assets/images/0012-2.png )

各種ログを一元的に管理

検索・集計・分析を一箇所で可能になります。

外部SaaSログも Pipelines 経由で同じ形式に変換可能

個人的な理解：まさに **ログ版のデータレイク**。

すべてのログが同じ土台に載ることで、分析の障壁が大幅に下がります。

## 3. Facetによる一元化分析
Unified Data Store 上のログは Facet分析 によって、多次元で瞬時に集計できます。

![0012-3]( /assets/images/0012-3.png )

複数ログソースを意識せずに集計可能

エンジニアは Unified Data Store + Facet の画面だけで解析完結

従来必要だった OpenSearch / Athena / Logs Insights / SaaS コンソールを行き来するストレスが解消

## 4. Unified Queryでさらに統一
Unified Query によって、ログ・メトリクス・トレースを一つのインターフェースで検索できます。すごくない?!!

![0012-4]( /assets/images/0012-4.png )

将来的には 観測データを一元検索できるプラットフォーム に
障害解析、性能分析、監査ログ調査などを 同じワークフローで完結可能

## 5. まとめ
全体を文字で整理すると次の通りです。
![0012-5]( /assets/images/0012-5.png )

ポイント：
- ✅Pipelines = ログ版ETL
- ✅Unified Data Store = すべてのログを同じ土台に集約
- ✅Facet / Unified Query = 一箇所で多次元分析・可視化
