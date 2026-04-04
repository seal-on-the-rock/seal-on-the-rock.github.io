---
layout: post
title: "article0015-【AWS Step Functions】で実装するSagaパターンを作ってみた  〜LambdaとDynamoDBを用いた分散トランザクション設計〜"
date: 2026-04-02
author: Seal
---

---

## はじめに

去年、AWS DVA資格を勉強する時は、Step Functionsは凄く重要な知識です。

本記事では、AWSのマネージドサービスを活用し、  
**Step Functions + Lambda + DynamoDB** を用いた注文処理システムを構築しながら、  
分散システムにおける重要な設計パターンである **Sagaパターン** を解説します。

特に以下のポイントにフォーカスします：

- Step Functionsによるワークフロー制御
- Lambda間の疎結合な連携
- エラー処理（Retry / Catch）
- 補償トランザクション（Compensation）

---

## システム構成（概要）

本システムは以下の構成で動作します：

![0015-2]( /assets/images/0015-2.png )

### 特徴

- APIは**同期的（HTTP 202）に応答**  　　　　　　　→　カウンター
- 実際の処理は**非同期でStep Functionsが実行**　　　→　　キッチン
- 各処理は**個別のLambdaとして分離** 　　　　　　　→　　疎結合

---

## 処理フロー

注文処理は以下のステップで構成されます：

```
CreateOrder → ReserveInventory → ProcessPayment
```

### 正常系

```
ProcessPayment（成功）
↓
NotifySuccess
↓
完了
```

### 異常系（Saga）

```
ProcessPayment（失敗）
↓
RestoreInventory（在庫戻し）
↓
NotifyFailure
↓
終了
```

![0015-1]( /assets/images/0015-1.png )

---

## なぜStep Functionsを使うのか？

単一のLambdaでも処理は可能ですが、以下の課題があります：

- 処理の可視性が低い
- エラー制御が複雑
- 再試行ロジックの実装が煩雑

Step Functionsを使うことで：

- ワークフローを**可視化**
- Retry / Catchを**宣言的に定義**
- 状態管理を**自動化**

できます。

---

## Sagaパターンとは？

分散システムでは、単一のトランザクション（ACID）を維持することが困難です。  
その代替として採用されるのが **Sagaパターン** です。

### 基本思想

```
各処理を独立したトランザクションとして実行
→ 失敗時は「逆処理」で元に戻す
```

### 本システムでの例

| ステップ | 内容 |
|----------|------|
| ReserveInventory | 在庫を減らす |
| ProcessPayment | 支払い処理 |
| RestoreInventory | 在庫を戻す（補償） |

---

## エラー処理の設計

本システムでは、2種類の失敗を明確に分けています。

---

### ① ハードエラー（例外）

- Lambdaの例外
- タイムアウト
- 実行失敗

👉 Step Functionsの **Catch** で処理

```
ProcessPayment → Catch → Compensation
```

![0015-4]( /assets/images/0015-4.png )

---

### ② ソフトエラー（業務失敗）

- 支払い拒否
- ビジネスロジック的な失敗

👉 **Choiceステート**で分岐

```
status == FAILED → Compensation
```

![0015-3]( /assets/images/0015-3.png )

---

### なぜ分けるのか？

- 技術的失敗と業務失敗は性質が違う
- リトライ可否が異なる
- ログや監視の観点でも重要

---

## Retryの活用

一時的なエラー（transient failure）に対しては、Retryを設定します。

例：

- ネットワーク一時障害
- DynamoDBのスロットリング

```
Retry: 最大3回、指数バックオフ
```

![0015-5]( /assets/images/0015-5.png )

---

## 非同期処理の設計

![0015-8]( /assets/images/0015-8.png )

APIは以下のように設計されています：

```
POST /orders → 202 Accepted
```

![0015-6]( /assets/images/0015-6.png )

これは：

- リクエストは受け付けた
- 処理はバックグラウンドで継続

という意味です。

👉 フロントとバックエンドの分離が可能になります。

---

## 実装から得られた学び

今回の構築を通して、以下の理解が深まりました：

### 1. ワークフローはコードではなく「設計」である

Step Functionsにより、処理の流れが明確になり、  
「どこで何が起きているか」が可視化されます。

---

### 2. エラー処理は最初から設計すべき

後付けではなく：

- Retry
- Catch
- Compensation

を最初から考慮することが重要です。

---

### 3. 分散システムでは「戻す設計」が重要

成功させるよりも：

> 失敗したときにどう戻すか

が重要になります。

---

## まとめ

本記事では、AWS Step Functionsを用いたSagaパターンの実装を紹介しました。

- Step Functionsによるワークフロー制御
- Lambdaの疎結合設計
- 2種類のエラー処理（Catch / Choice）
- 補償トランザクション（Saga）

これらを組み合わせることで、  
実運用に耐えうる分散システムの設計が可能になります。

---

## 注意！

いつもですが、権限の問題をちゃんと注意しましょう！

![0015-7]( /assets/images/0015-7.png )

このコードは、IAM権限の設定を行っています。

Lambda関数がStep Functionsを起動できるように、
StartExecutionの権限を最小限で付与しています。
