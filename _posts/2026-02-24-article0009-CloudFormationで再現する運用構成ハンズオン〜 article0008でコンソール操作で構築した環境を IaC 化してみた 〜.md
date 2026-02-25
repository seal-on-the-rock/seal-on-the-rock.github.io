---
layout: post
title: "article0009-CloudFormationで再現する運用構成ハンズオン〜 article0008でコンソール操作で構築した環境を IaC 化してみた 〜"
date: 2026-02-24
author: Seal
---



こんにちは。

今回は、AWS の主要な運用サービスを一通り含んだ Web インフラ構成を、
CloudFormation を使ってコード化（IaC 化）してみました。

実はこの構成、もともとは AWS コンソール上で一つ一つ手動構築した検証環境です。

- VPC / Subnet
- ALB + Auto Scaling
- Session Manager（SSHレス運用）
- CloudWatch Logs / Alarm
- CloudTrail
- AWS Backup

といった、実務でもよく使われる運用系サービスを組み合わせた小さなハンズオン環境になります。

最初は「仕組み理解」を目的にコンソール操作で作成し、
その後「再現性・自動化・運用効率」を意識して CloudFormation で完全再構築しました。

この記事では、

① コンソールで理解した構成  
② それを CloudFormation でどう表現するか  
③ 各リソースの役割と設計意図  

を、初心者目線で一つずつ解説していきます。

「CloudFormation が難しくて読めない…」
「IaC に挑戦したいけど何から始めればいいかわからない」

そんな方の参考になれば嬉しいです。
  
  ![1]( /assets/images/0009-1.png )
  ![2]( /assets/images/0009-2.png )
  ![3]( /assets/images/0009-3.png )
  ![4]( /assets/images/0009-4.png )
  ![5]( /assets/images/0009-5.png )
  ![6]( /assets/images/0009-6.png )
  ![7]( /assets/images/0009-7.png )
  ![8]( /assets/images/0009-8.png )
  ![9]( /assets/images/0009-9.png )
  ![10]( /assets/images/0009-10.png )
  ![11]( /assets/images/0009-11.png )
  ![12]( /assets/images/0009-12.png )
  ![13]( /assets/images/0009-13.png )
  ![14]( /assets/images/0009-14.png )
  ![15]( /assets/images/0009-15.png )
  ![16]( /assets/images/0009-16.png )
  
