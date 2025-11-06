---
layout: post
title: "article0006-CloudWatch"
date: 2025-11-02
author: Seal
---


# ☁️ article0006　CloudWatch

<img src="/assets/images/0006-10.png" alt="CloudWatch" class="responsive-img">

久しぶりです。

今日はSREをしている人は良く使っているAWSのサービスであるCloudWatchを紹介します。

五つのケースを通じてCloudWatchの使い方を解説します。

CASE１　システムリソースの監視（Metrics　→　Alram　→　SNS）

<img src="/assets/images/0006-1.png" alt="case1">

説明：

CloudWatch Agent：先ずは、CloudWatch　AgentはEC2で動作するアプリケーションです。	

三つの役割があります。			

①　EC2のメトリクスを収集して、CloudWatchに発送する				

②　EC2のlogsを収集して、CloudWatchに発送する

　　　（CloudWatch　Agentがなければ、logsがEC2にしか保存されず、SSHでEC2にログインしないと取得できません。）										

③　カスタマイズなイベントを作成できます。			

---

CASE２　ログ駆動型モニタリング（Agent　→　Logs　→　Metric Filter →　Alarm　→　SNS）

<img src="/assets/images/0006-2.png" alt="case2">

説明：Logsの収集するには、CloudWatch Agentが必要です。

---

CASE３　自動修復システム（Alarm　→　EventBridge　→　Lambda）

<img src="/assets/images/0006-3.png" alt="case3" class="responsive-img">

説明：前のケースに基づいて、自動的に対応する仕組みも導入しています。

---

CASE４　分散トレーシング監視（X-Ray + CloudWatch Logs + ServiceLens）

<img src="/assets/images/0006-4.png" alt="case4" class="responsive-img">

X-Rayの仕組み：

<img src="/assets/images/0006-5.png" alt="X-Rayの仕組み" class="responsive-img">

ServiceLensの仕組み：

<img src="/assets/images/0006-6.png" alt="ServiceLensの仕組み" class="responsive-img">


CASE５　総合可観測性プラットフォーム（Dashboard　＋　Logs　＋　Metrics Explorer）

<img src="/assets/images/0006-7.png" alt="case5" class="responsive-img">

一言でまとめると

CloudWatchの「可観測性（Observability)」モジュールとは：

各AWSサービスから自動的にデータ（Metrics＋Logs＋Traces）を収集し、

取得したデータをExplorer・Insights・X-Rayなどのツールで分析し、

最終的にDashboardで統合的に可視化・アラート・レポートを行う仕組みです。

---

SRE（Site　Reliability　Engineer）にとっての意味

このモジュールは、SREにとって次のような役割を果たします：

・システム健康状態の計器盤　　（Dashboard）

・データ分析ラボ　　　　　　　（Logs　Insights/Metrics　Explorer）

・問題の根本原因追跡システム　（X-Ray）

・アラート自動通知のトリガー　　（Alarms　＋　SNS）

つまり、

「SSHでサーバーにログインしなくても、システム全体の健康状態·異常傾向·エラー分布·性能ボトルネックを把握できる」ということです。

---

まとめ：

すべてを収集し　→　相関分析し　→　可視化し　→　自動的に対応する仕組み

---

ちなみに、他の二つのAWSサービスで、CloudTrailとAWS Configを紹介します。

•　CloudTrail

<img src="/assets/images/0006-8.png" alt="CloudTrail" class="responsive-img">

AWSのすべての操作はAPIコール（呼び出し）である

↓

CloudTrailは、すべてのAPIコールのログを記録する

↓

記録されたログはS3に保存することも、CloudWatch　Logsに転送することもできる

↓

CloudWatchでは、これらのログを検索·フィルタリング·アラート・可視化できる

---

•　AWS　Config

<img src="/assets/images/0006-9.png" alt="AWS Config" class="responsive-img">

configファイルのバージョン管理。

---

CloudTrail と AWS Config の違い

CloudTrail：操作イベントを記録する（＝何が起こったか/What happened） PL

AWS　Config：リソースの状態変化を記録する（＝どう変わったか/What it became） BS


