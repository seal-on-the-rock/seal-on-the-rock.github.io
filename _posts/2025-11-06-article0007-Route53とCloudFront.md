---
layout: post
title: "article0007-Route53とCloudFront"
date: 2025-11-06
author: Seal
---


# ☁️ article0007　Route53とCloudFront

今日は、CloudWatchと同じように五つのケースを通じてRoute53とCloudFrontを併せて解説します。

ケース①：企業公式サイト·静的ウェブサイトの公開

背景：企業が公式サイトや製品紹介ページを公開する。

コンテンツは静的なHTML·CSS·JavaScriptで構成され、S3の静的ウェブホスティングに配置される。

構成図（トポロジー）

User　→　Route53　→　CloudFront　→　S3　Bucket（Static　Website）

役割

•　Route53：ユーザーが入力したドメイン名（例：example.com）をCloudFrontのCDNドメインへ解析。

•　CloudFront：世界中のエッジロケーションで静的コンテンツをキャッシュし、遅延を削減。

•　S3：ウェブページファイルの実体を保存。

実務ポイント：

①　S3バケットは「Block all public access = ON」にする。

  　つまり、私有にする。

   CloudFrontはOAC（Origin Access Control）で私有S3へアクセスする。

   まとめると、

　　配信経由のみ許可、オリジンは非公開。

②　CloudFront　の　Origin　は　S3源　を指定。

　　PS：
  
  　　　CloudFron　Distribution　の　Distribution　はどういう意味ですか？

     　Distribution　＝　配信/分配　ですが、下記の2点の意味があります。

      ·　リクエストをどのオリジンに送るか（ルーティング）　S3/ALB/EC2/API Gatewayなど

      ·　コンテンツをどのように世界中にキャッシュ·配信するか（CDN）　→　エッジロケーション

③　Route53　に　Aレコード（Alias）を作成し、CloudFront Distributionを指定。

　「example.com　→　xxxxxx.cloudfront.net」というのは　CNAME　です。

 　CloudFrontの中で、SSL証明書を管理するのは　ACM　です。


  
