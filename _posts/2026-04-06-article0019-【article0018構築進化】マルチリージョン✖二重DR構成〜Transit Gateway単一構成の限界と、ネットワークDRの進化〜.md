---
layout: post
title: "article0019-【article0018構築進化】マルチリージョン✖二重DR構成〜Transit Gateway単一構成の限界と、ネットワークDRの進化〜"
date: 2026-04-06
tags: [DR, BGP, Transit Gateway, VPN]
author: Seal
---



## はじめに

前回の記事では、単一のTransit Gateway（TGW）を中心としたハブ＆スポーク型ネットワークを構築しました。

![0018-1]( /assets/images/0018-1.svg )

しかし実際に設計を進める中で、ある重大な問題に気づきました。

> **TokyoリージョンのTGWがダウンした場合、オンプレミスからAWS全体に接続できなくなる**

つまり：

* VPC自体は生きている
* アプリも動いている
* でも「ネットワーク入口」が死んでいる

これはDR（Disaster Recovery）の観点では致命的です。

---

## 本記事の目的

本記事では以下を実現します：

### 🎯 目的

* TGWの単一障害点（SPOF）を排除
* マルチリージョン構成でのネットワーク冗長化
* **アプリ層 + ネットワーク層の二重DR構成**

---

# 🧩 全体構成

本構成は以下の2層のDRで成り立っています：

---

## 🟢 レイヤー①：アプリケーションDR（Route53）

* ユーザアクセスの切り替え
* Tokyo → Osaka へフェイルオーバー

---

## 🔵 レイヤー②：ネットワークDR（TGW + BGP）

* オンプレミス接続の維持
* Tokyo TGW障害時でも Osaka TGW経由で接続可能

---

# 🧠 なぜ単一TGWではダメなのか？

従来構成：

```
On-Prem → VPN → Tokyo TGW → VPC
```

この構成の問題：

```
Tokyo TGW が落ちる
　↓
VPN終端が消える
　↓
オンプレからAWSへ到達不可
```

👉 **＝ネットワークレベルで完全断**

---

# 🚀 改善後今回のアーキテクチャ（Double TGW + TGW Peering）

![0019-1]( /assets/images/0019-1.svg )

---

二層DRの動作をケース別に解説

# 🟢 ケース①：正常時（Primary Path）

## 🌐 ユーザアクセス

```
User → Route53 → Tokyo ALB → Tokyo VPC
```

## 🏢 オンプレ通信

```
On-Prem
   ↓
VPN（BGP）
   ↓
Tokyo TGW（Primary）
   ↓
Tokyo VPC
```

## 💡 ポイント

* BGPの `LOCAL_PREF` によりTokyo経路が優先
* Osakaはスタンバイ(stand by)

---

# 🔴 ケース②：Tokyoアプリ障害（TGWは正常）

## 🌐 ユーザアクセス

```
Route53 ヘルスチェック NG
   ↓
Osaka にfailover
```

```
User → Osaka ALB → Osaka VPC
```

## 🏢 オンプレ通信

```
On-Prem → Tokyo TGW → Tokyo VPC
```

👉 **ネットワーク経路は変わらない**

## 💡 ポイント

* アプリ層のみDR
* ネットワークは正常

---

# 💥 ケース③：Tokyo TGW障害（最重要）

## ❌ 発生すること

```
Tokyo TGW ダウン
   ↓
VPNトンネル切断
   ↓
BGP経路消失
```

## 🧠 BGPの挙動

```
Tokyoへの経路を撤回（withdraw）
```

## 🏢 自動フェイルオーバー

```
On-Prem
   ↓
VPN（生きているトンネル）
   ↓
Osaka TGW
   ↓
Osaka VPC
```

## 💡 ポイント

* 完全自動切替
* Lambda不要
* BGPによる動的ルーティング

---

# 🟣 ケース④：Tokyo完全障害（アプリ + TGW）

## 🌐 ユーザアクセス

```
Route53 → Osaka
```

## 🏢 オンプレ通信

```
On-Prem → Osaka TGW → Osaka VPC
```

## 🎯 結果

```
全トラフィックがOsakaへfailover
```

---

# 🔑 技術ポイントまとめ

## ① BGPによる経路制御

* `LOCAL_PREF` による優先制御
  
* 障害時の自動経路撤回

---

## ② VPN（2トンネル構成）

* 冗長構成（片系障害耐性）
  
* 高可用性の確保

---

## ③ Transit Gateway（リージョン単位）

* VPCの集約
  
* ルーティング制御の中枢

---

## ④ TGW Peering（リージョン間接続）

* Tokyo ↔ Osaka 接続
  
* DR時のバックアップ経路

---

## ⑤ Route53（アプリ層DR）

* ヘルスチェックによる切替
  
* DNSレベルのfailover

---

# ⚠️ よくある誤解

## ❌ Lambdaでルートを書き換える必要がある？

→ **BGP構成なら不要**

---

# 🎯 まとめ

今回の構成の本質：**アプリ層とネットワーク層を分離してDR設計する**

---


