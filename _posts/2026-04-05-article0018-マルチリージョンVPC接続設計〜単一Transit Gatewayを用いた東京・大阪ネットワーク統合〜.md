---
layout: post
title: "article0018-マルチリージョンVPC接続設計〜単一Transit Gatewayを用いた東京・大阪ネットワーク統合〜"
date: 2026-04-05
tags: [Transit Gateway, MultiVPC]
author: Seal
---

---

## 🧩 hello~

システムの可用性や災害対策（DR：Disaster Recovery）の観点から、
今日のテーマは**マルチリージョン構成**になりました。

👉 **2つのVPCを安全かつスケーラブルに接続する方法**を、
設計の背景から順を追って解説します。

---

## 解決したい課題

東京（ap-northeast-1）と大阪（ap-northeast-3）にそれぞれVPCが存在しますが、**相互通信（プライベート通信）をしたい**

しかも、**将来的にVPCや拠点が増える可能性あり**

---

## ❌ VPC Peering → ダメ


1.Full Mesh構成になる（管理地獄）

2.Transit Routing不可

3.スケールしない

4.今後Driect Connectを使いたければ、対応するには困難

👉 ❌ 大規模には不向き

---

## 💡 解決策：Transit Gateway

### TGWの役割

```text
VPC同士をつなぐ「ハブ」
```

---

### イメージ

```text
[VPC Tokyo] ─┐
              ├── [Transit Gateway] ─── 他リージョン
[VPC Osaka] ─┘
```

---

## 🏗️ アーキテクチャ概要

今回構築する構成は以下：

![0018-1]( /assets/images/0018-1.png )

---


## TGWはどこに置く？

👉 今回は：東京リージョンにTGWを配置


理由：

1.メインリージョン想定

2.他リージョンをぶら下げる構造にする

---

## VPCをTGWに接続（Attachment）

### 🎯 最大の疑問

東京サブネットと大阪サブネットは同じようにTGWに接続しているのか？

---

### ❗答え:接続方式は同じ（どちらもVPC Attachment）、でも経路（ルート設計）は違う


Tokyo側

```text
Tokyo VPC ── VPC Attachment ── Tokyo TGW
```

---

 Osaka側（重要）

```text
Osaka VPC ── VPC Attachment（クロスリージョン）── Tokyo TGW
```


### 誤解

私が最初TGWを勉強した時に、いつも「❌ サブネットがTGWに繋がっている❌ 」のように誤解しました。

実は、「TGWでVPCを繋がる」と「Attachmentのサブネットの選択」とは全然違いものです。



###  通信成立条件まとめ：

```text
1. 両VPCのルートテーブル

Tokyo: 10.1.0.0/16 →　TGW
Osaka:10.0.0.0/16　→　TGW

2. TGWルートテーブル(next stepはAttachment)

10.0.0.0/16 →　Tokyo Attachment
10.1.0.0/16 →　Osaka Attachment

3. Attachment
Tokyo Private Subnet :VPC Attachment
Osaka Private Subnet :VPC Attachment

```

### 最後に、面白い点

Osaka の VPC Attachment は「東京の TGW に属するリソース」です。

Attachmentの本質はTGW の「ポート」です。

東京の TGW が持っている Attachment の一つが大阪の VPC に接続しています。

Attachment は「どこにあるか」ではなく、「誰に属しているか」で考えます。

**東京の TGW = 本体（タコの体）**

**Attachment = 触手**

大阪の Attachment は、東京の TGW からクロスリージョンで伸びた触手です。

触手の根元は東京（TGW）で、先っぽが大阪の VPC にくっついています。

**面白いでしょうか？！**



