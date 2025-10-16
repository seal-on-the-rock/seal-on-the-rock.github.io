---
layout: post
title: "article0004-AWS SDK"
date: 2025-10-16
author: Seal
---


# ☁️ article0003　AWS SDK


今日はAWS SDKを紹介します。

AWS SDKを説明するために、先にAPI、WEB API、SDKという三つのものを紹介します。

先ずは、APIは何ですか？

API　＝　Application Programming Intarfaceということによって、実はこれがインタフェースなのです。

そうしたら、下記の例を見ましょう。

<br><br>
ケース：日記アプリで天気情報を自動取得

⑴　背景

```text
　　・あなたは日記編集アプリを作る
　　・機能：毎日自動的に天気情報を取得して日記テンプレートに反映
　　・天気データの提供元：サードパーティの天気会社が提供するWEB　API
　　・簡単に呼び出せるように、天気会社がSDKを提供
```

⑵　概念の対応

|概念  |日記アプリでの対応  |役割 | 
|:----------------|:------------------------|:---------------------------------------------------------|
| API | 天気会社が提供する天気情報を取得関数 |天気情報を提供できる関数 例：getWeather(city)|
| WEB　API | インターネット上に公開されたHTTPインタフェース | 遠隔のサーバーにアクセスして天気情報を取得　例：https://api.weather.com/weather?city=Tokyo|
| SDK  | 天気会社が提供する開発者に向けてのツールキット（tool kit） | WEB　APIをラップしてリクエスト、認証、データ解析を簡単に呼び出す|
| アプリのコード | あなたの日記編集アプリ | SDKを使って天気情報を取得してテンプレートに反映 |

アプリのコードの参照例：

```text
import weather_sdk
from datetime import date

# 日記テンプレート関数
def create_diary_entry(city):
    today = date.today().strftime("%Y-%m-%d")
    
    # SDK を使って天気を取得（内部で Web API を呼び出す）
    weather = weather_sdk.get_today_weather(city=city)
    
    # 日記内容を作成
    diary = f"{today} の天気：{city} 天気 {weather['condition']}, 気温 {weather['temp']}°C"
    return diary

entry = create_diary_entry("Tokyo")
print(entry)
```

⑶　フロー図

![SDK]( /assets/images/0004-1.png )


<br><br>

これから、AWS SDKを見ましょう！

AWS SDKは、簡単に言うと、

　　コードでAWSのサービスを操作すること。


|  操作方法　　　　|　　　　　　操作形式　　　 |　　　　　例　　　　　　　　　　　　　　　　　　　　　　　　 | 
|:----------------|:------------------------|:---------------------------------------------------------|
| AWS コンソール 　| GUI図形界面　　　　| click　click　click　　　　　　　　　　　　　　　　|
| AWS CLI 　　　　| コマンド　　　　　 | aws s3 ls　　　　　　　　　　　　　　　　　　　|
| AWS  SDK　　　　| コード　　　　　　 | import boto3<br>s3 = boto3.client('s3')<br>response = s3.list_buckets()<br>for bucket in response['Buckets']:print(bucket['Name'])|

上記の三つの操作方法は、どちらでも同じHTTPリクエストを発送することです。

```text
GET / HTTP/1.1
Host: s3.ap-northeast-1.amazonaws.com
Authorization: AWS4-HMAC-SHA256 Credential=AKIAxxxx/20251016/ap-northeast-1/s3/aws4_request, SignedHeaders=host;x-amz-date, Signature=0a1b2c3d4e5f...
```

<br>

