---
layout: post
title: "article0007-Route53とCloudFront"
date: 2025-11-06
author: Seal
---


# ☁️ article0007　Route53とCloudFront

<img src="/assets/images/0007-1.png" alt="Route53とCloudFront" class="responsive-img">

今日は、CloudWatchと同じように五つのケースを通じてRoute53とCloudFrontを併せて解説します。

ケース①：企業公式サイト·静的ウェブサイトの公開

背景：企業が公式サイトや製品紹介ページを公開する。

コンテンツは静的なHTML·CSS·JavaScriptで構成され、S3の静的ウェブホスティングに配置される。

構成図（トポロジー）

```text
User　→　Route53　→　CloudFront　→　S3　Bucket（Static　Website）
```

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

```text
User　→　Route53　→　CloudFront　→　ALB　→　EC2（アプリケーションサーバー）
```

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

```text
User　→　Route53（Latenncy　Based　Routing）
　　　　　├──CloudFront　→　ALB（東京）
    　　　├── CloudFront　→　ALB（シンガポール）
```

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

```text
Client Domain（CNAME　→　）CloudFront　Distribution　→　ALB
```

<img src="/assets/images/0007-2.png" alt="Route53とCloudFront" class="responsive-img">

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

背景：S3に大量の静的コンテンツをホスティングしている企業が、S3直アクセスの防止と攻撃対策を目的とする構成。

構成図：

```text
User　→　Route53　→　CloudFront（＋WAF）→　S3（Origin　Access　Control）
```

主な役割：

•　CloudFrontにWAFを組み込み、不正アクセスを遮断（しゃだん）。

•　S3はCloudFront経由アクセスのみ許可。

•　Route53はわかりやすい独自ドメインを提供。

•　CloudFrontログをCloudWatch　＋　Athena　で分析。

実務ポイント：

•　WAFと組み合わせてDDoS防御を実現。

•　ログ活用でセキュリティとコストの最適化。

補充説明：

コスト最適化については？

A. CloudFrontレイヤーでの最適化

CloudFrontはCDNとして、ユーザーのリクエストを受けて必要に応じてS3（オリジン）からコンテンツを取得します。

この「S3への取得（Origin　Fetch）」が多いと、S3リクエスト課金やデータ転送料金が増加します。

特に、

•　ボット（bot）やクローラーによる不要アクセス

•　同じファイルを何度も取得する無駄なリクエストがあると、無駄なコストが発生します。

効果（ログ活用のポイント）：

１．CloudFrontアクセスログをAthenaで分析

  •　アクセス頻度が異常に高いIP

  •　同一URLを短時間で繰り返すアクセス

  •　国や地域ごとのアクセス傾向を特定します。

２．WAFで不正·不要アクセスを遮断

　•　頻繫なbotアクセスや国外からの異常トラフィックをブロック

３．キャッシュTTLの調整

　•　CloudFrontでキャッシュ時間を延ばし、S3へのリクエスト数を削減。

 →　結果：S3リクエスト回数とデータ転送量が減り、配信コストが低下します。

<br><br>

B. S3レイヤーでの最適化

S3に保存している静的コンテンツの中には、「ほとんどアクセスされない古いファイル」も存在します。

効果：

CloudFrontログやS3アクセスログをAthenaで集計し、過去何日間アクセスがなかったオブジェクトを特定。

それらをGlacierやIntelligent‐Tiering（自動階層化）に移行します。

→　保存コストの最適化（長期保管コストを大幅削減）

<br><br>

C.　WAFレイヤーでの最適化（セキュリティ＋コスト）

DDoSやクローラーなどの攻撃的トラフィックも、実際にはCloudFront　→　S3への通信を発生させるため、「セキュリティ問題＋無駄な通信コスト」を引き起こします。

効果：

•　WAFログをAthenaで分析し、攻撃元IPや異常パターンを抽出。

•　CloudFrontで同様のパターンを早期遮断（WAFルール更新）。

→　攻撃対策（セキュリティ向上）と同時に、無駄な課金トラフィックを削減できます。

---

Route53とCloudFrontの典型的な役割まとめ

| サービス | 主な役割 |設定の重点ポイント |
|:-------------|:----------------------|:----------------------------------|
| Route53| DNS解決、トラフィック制御　| レコード設定、ルーティングポリシー、チェック　| 
| CloudFront | コンテンツ配信、キャッシュ、セキュリティ強化 | キャッシュ動作、Origin　Access　Control、WAF、HTTPS|

---

CloudFrontの本質的な理解

「Distribution」とは何か？

AWSにおける“Distribution”は「ファイルの配布」ではなく、

“CloudFrontが外部へ公開する統一的な入口設定”を指します。

つまり、Distributionとは：

•　グローバルなアクセス入口（エッジ層）

•　どのオリジン（S3、ALB、API　Gateway）へ転送するかを制御

•　複数ドメイン（CNAME）を統合

•　HTTPS証明書（ACM）やセキュリティルールを集中管理

•　Lambda＠Edge/CloudFront　Functionsで動的処理

---

CloudFrontの多層構造（イメージ）

<img src="/assets/images/0007-3.png" alt="Route53とCloudFront" class="responsive-img">

補充説明：

①　「APIレスポンスをキャッシュ」の場合がある？

   多分誰が「APIレスポンスをキャッシュ」を見ると、戸惑うようになっちゃうかもしれません。そういうケースが本当にありますか？

   実はありますよ。

   まずは、結論というと、すべてのAPIをキャッシュすべきという意味ではありません。
　　「キャッシュに向いているAPIレスポンス」をCloudFrontにキャッシュされることで、パフォーマンスとコストの両面で大きなメリットが得られます。

  　S3の静的ファイルは100%キャッシュ向きですが、

    多くのAPIも「読み取りが多く、更新が少ない（読取多、書込少）」ため、キャッシュ適用が非常に効果的です。

　　Ａ．CloudFrontにとって、S3もAPIも「HTTPレスポンス」

  　　　CloudFrontは本質的に「HTTPリクエストとレスポンス」を中継·最適化する仕組みです。

     　　S3でもAPIでも、CloudFrontから見れば単なるHTTPレスポンスに過ぎません。

       S3：HTML/CSS/JS/画像　→　静的コンテンツ
       API：JSON/XML/テキスト　→　動的データ

       従って「GET/index.html」をキャッシュできるなら、

       「GET/api/products」も（内容が安定していれば）同じようにキャッシュできます。

    Ｂ．なぜAPIレスポンスをキャッシュするのか？

    理由1：APIも「読取多、書込少」が多い

    例えば、

　　　　·　/api/products（商品一覧）

　　　　·　/api/posts（ブログ記事一覧）

　　　　·　/api/ranking（ランキング）

　　　　·　/api/config（設定情報）

    これらのデータは多くのユーザーに共通で、頻繫には変わりません。
    数秒～数分の遅延が許容できるなら、キャッシュで大きな効果を得られます。

    例：

```text
    　　1秒間に1000件のリクエスト
      　↓
　　　　TTL　＝　１秒のキャッシュを設定
　　　　→　最初の1件だけオリジンへ、残り999件はキャッシュヒット
```

　　　結果：オリジン負荷99.9％削減。

　　　理由2：オリジン（ALB/EC2/DB）の負荷軽減

　　　APIは通常こう流れます：

　　　Client　→　CloudFront　→　ALB　→　EC2　→　DB

　　　キャッシュなしなど、すべてのリクエストがALBとEC2に到達。

　　　しかしキャッシュを入れると：

　　　•　CloudFrontが同一リクエストをエッジで処理

　　　•　オリジン到達はごく一部だけ

　　　→　突発的なアクセス（セールやキャンペーン）にも耐えやすくなります。

　　　理由3：レスポンス高速化

　　　キャッシュが有効なら、ユーザーに最も近いエッジロケーションから即時レスポンス。

　　　特にスマホ回線や海外アクセスでは体感速度が大幅アップします。

　　　C．すべてのAPIをキャッシュしていいわけではない

   　　キャッシュしてはいけない体表的なAPI：

     　/api/login : 認証データ

      /api/order：注文情報（毎回異なる）

      /api/pay：支払処理（即時反映が必要）

      /api/user/profile：個人データ

      /api/user/123/balance：機密情報

      これらはユーザーごとに異なるか、または即時性が求められるため、

      Cache‐Control：no-store/privateを設定してキャッシュを禁止します。

      D:現場での典型的な運用パターン

      |  パターン　|  内容　|  TTL設定  |
      |:-----------|:------------|:------------|
      |公開データ（全員同じ）| /api/posts,/api/ranking | 長め（60～300秒）|
      |準動的データ（多少変動あり）| /api/home,/api/recommendations | 短め（10～30秒）|
      |ユーザー別データ　| /api/user/dashboard　| Cookie/ヘッダー単位で分離（5～10秒）|

      E：S3とAPIの違い

          |  比較項目　|  内容　|  TTL設定  |
      |:-----------|:------------|:------------|
      |公開データ（全員同じ）| /api/posts,/api/ranking | 長め（60～300秒）|
      |準動的データ（多少変動あり）| /api/home,/api/recommendations | 短め（10～30秒）|
      |ユーザー別データ　| /api/user/dashboard　| Cookie/ヘッダー単位で分離（5～10秒）|  

      
      

    

---

②　ヘッダー書き換えとは？

　　ヘッダー書き換え　＝　Header　Rewrite

  　CloudFrontは、ユーザーからのリクエストやオリジンのレスポンスに含まれるヘッダー情報（Header）を途中で書き換えることができる機能を持っています。

   つまり、リクエストがS3やALBなどのオリジンに届く前、またはレスポンスがユーザーに返る前に、CloudFrontがヘッダーを追加·削除·変更できます。

   <img src="/assets/images/0007-4.png" alt="Route53とCloudFront" class="responsive-img">

   使われるケース

```text
１．カスタムヘッダーの追加
　例：「X‐From‐CloudFront：yes」を追加し、リクエストがCloudFront経由で来たことを示す。　→　セキュリティ制御。

 　　　「X-Experiment-Group：A」または「X-Experiment-Group：B」を追加して、返ったページのバージョンが異なります。

２．User‐Agent判定

　CloudFrontがUser-Agentを解析し、モバイルか㍶かを判定。

 例：「X‐Device‐Type:mobile」のようなヘッダーを追加し、オリジン側で簡単に処理可能になります。
 
３．不要なヘッダー（cookieなど）の削除

　　　キャッシュ判定に不要なCookieを削除し、キャッシュヒット率を向上。

　　　「同じ内容だ」に認定されるようにします。

４．URLパスのリライト

　　古いパスを新しい構成に自動変換。

  　例：　·　システムバージョンアップしたら、古いパスでもアクセスできるようにしたい

   　　　　·エンド側のurlは見せたくない
```

---

③　認証とは？

　　意味：CloudFrontのエッジ層で「ユーザーのリクエストを通すかどうか判断する」機能。

  　　　　アプリ側に到達する前に、認証·認可を行うことができる。

      例：

      ·　JWTトークン（JSON　Web　Token）の検証

      ·　CookieやAuthorizationヘッダーのチェック

      ·　CloudFront　Functionsで簡易的なBasic認証を実装

      ·　未ログインユーザーをログインページにリダイレクト

      イメージ：

```text
     　　　 User → CloudFront（認証）→ ALB/S3
　　　　　　　└→ 認証失敗なら 403 / Login ページへ
```

これにより、オリジン（S3やEC2）に余計な不正アクセスが届かない　＝　セキュリティとコストの両方が改善されます。

---

④　ABテストとは？

　　A/B　Testing意味：同じURLへのアクセスを、ランダムまたは条件付きで異なるバックエンドへ分配する。

  　新しい機能やデザインを小規模ユーザーで試す時に使う。

   例：·　50％のユーザーを「新バージョンAPI」へ、残り50％を「旧バージョン」へ

   　　·　特定のCookieを持つユーザーだけ新デザインへ

     　·　地域別、端末別で異なるページを返す

```text
　　　User
　　　 ├─→ CloudFront → Origin A（旧版）
 　　　└─→ CloudFront → Origin B（新版）
```

　　　この分岐ロジックをLambda＠Edgeに書くことで、オリジンに負担をかけずに分流テストができます。

   


















 

















  


  
