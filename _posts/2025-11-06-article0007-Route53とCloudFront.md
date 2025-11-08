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

´´´text
User　→　Route53　→　CloudFront　→　S3　Bucket（Static　Website）
´´´

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


ケース②　動的コンテンツを含むWebサービス（ロードバランシング型）

背景：EC2のクラスター（cluster）をALBで負荷分散し、静的ファイルはCloudFrontがキャッシュする構成（例：ECサイト）。

構成図

´´´text
User　→　Route53　→　CloudFront　→　ALB　→　EC2（アプリケーションサーバー）
´´´

主な役割：

CloudFront：画像·CSS·JSなどの静的リソースをキャッシュ。

　　　　　　動的コンテンツ（例：/api/*）はALBに転送。

Route53：カスタムドメインを提供。

ALB：ヘルスチェック（heath check）とロードバランシング（load balancing）を実施

実務ポイント：

CloudFrontのキャッシュ動作（Cache　Behavior）を個別設定可能：

　　　/static/*　→　キャッシュ有効

　　　/api/* →　キャッシュ無効（ALBに直通）

Origin　Access　Control（OAC）を使ってセキュリティ強化

注意点：CORS設定漏れ

PS：CORSというのは、ブラウザが異なるドメイン（オリジン）間でデータをやり取りする際の安全ルールです。

例えば、

フロントエンドのURL：http：//www.example.com

APIサーバーのURL：http://api.example.net

この二つのドメインが異なるため、ブラウザは「安全のため」に直接通信をブロックします。

その時に必要なのがCORSの設定です。

APIサーバー側で「このドメインからのアクセスを許可する」と明示する必要があります。


ケース③　：マルチリージョン高可用構成（Route53による遅延ベースルーティング）

背景：SaaSサービスを東京リージョンとシンガポールリージョンに展開し、利用者が最も近い拠点に接続されるようにする。

構成図

´´´text
User　→　Route53（Latenncy　Based　Routing）
　　　　　├──CloudFront　→　ALB（東京）
    　　　├── CloudFront　→　ALB（シンガポール）
´´´

主な役割：

•　Route53は遅延ベースルーティング（Latenncy　Based　Routing）により、最も応答の速いリージョンに誘導。

•　CloudFrontが各地域での配信を最適化。

•　リージョン障害時には、ヘルスチェック　＋　フェイルオーバー　で自動切替。

実務ポイント：

•　Route53とCloudWatchのヘルスチェック連携が重要。

•　多リージョン構成に最適なルーティング方式。

•　手動切替の場合、Route53　API で重み付け変更も可能。（Weighted　Routing）

ケース④　：SaaSプラットフォームのマルチテナントドメイン（multi-tenant-domain）管理

（Route53　＋　CloudFront動的バインドbilding）

背景：自社のSaaSプラットフォームで、顧客が自分の独自ドメイン（例：shop.abc.com）を使えるようにする構成。

構成図

´´´text
Client Domain（CNAME　→　）CloudFront　Distribution　→　ALB
´´´

主な役割：

•　顧客は自社のDNS（またはRoute53）でCNAMEをCloudFrontに設定。

•　CloudFrontはAlternate　Domain　Names（CNAMEs）機能で複数ドメインを扱う。

•　ACMが複数ドメイン証明書を提供。

•　アプリケーションはHost　Headerによりテナント識別を行う。

実務ポイント：

•　ACM　＋　Lambda　＋　APIによる証明書自動化が便利。

•　Lambda＠Edgeで動的リライティングが可能。

•　注意点：CNAME数上限、証明書有効期限、ドメイン認証エラー


ケース⑤　：セキュリティ＆コスト最適化構成（CloudFront　＋　WAF　＋　S3）

背景：















  


  
