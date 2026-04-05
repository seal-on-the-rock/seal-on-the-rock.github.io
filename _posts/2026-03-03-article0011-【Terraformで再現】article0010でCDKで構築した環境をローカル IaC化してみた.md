---
layout: post
title: "article0011-【Terraformで再現】article0010でCDKで構築した環境をローカル IaC化してみた"
date: 2026-03-03
tags: [Terraform, IaC]
author: Seal
---


こんにちは～ 👋

悲報がありますが、
AWS Organizations を作った瞬間、無料クレジットが消えた… 😇💸

昨日はちょっと真面目に、
「**マルチアカウント構成での統制実験**」をやろうと思っていました 💪

やりたかったことは：

✅ Organizations を作る

✅ メンバーアカウントを追加する

✅ SCP を試す

✅ セキュリティガードレールを設計する

いわゆる
「ちゃんとした企業アーキテクチャごっこ」🏢✨

テンションは完全にアーキテクトモード。

「今日はクラウドを統治する側になるぞ 😎☁️」
そう思っていました。

そして、その瞬間は突然やってきた…

Organizations を作成
　　　⇒　📩 メール受信

![Organizations1]( /assets/images/0011-01.png )

「ん…？なんか嫌な予感…😨」

後で規約を読んでみると、

![Organizations2]( /assets/images/0011-02.png )

と、ちゃんと書いてある。

しかも結構ハッキリ書いてある。
びっくりするくらい書いてある。📚

でも、

ちゃんと読んでいなかった 🤦‍♂️

あるあるですね（自業自得）。

ということで、
Amazon Web Services の勉強はまだまだ止まらないので 🚀

今回は気持ちを切り替えて、

👉 【article0010でCDKで構築した環境をTerraformでローカル IaC 実験】編に突入します 🧱⚙️

クレジットがなくても、
学習は止められない。
インフラエンジニアの魂です 🔥

---

まずはディレクトリは下記の通りです。

![0011-3]( /assets/images/0011-03.png )

📎main.tf

```
############################
# S3 Site Bucket
############################
resource "aws_s3_bucket" "site" {
  force_destroy = true
}

############################
# S3 Log Bucket
############################
resource "aws_s3_bucket" "log" {
  force_destroy = true
}

resource "aws_s3_bucket_versioning" "log" {
  bucket = aws_s3_bucket.log.id
  versioning_configuration {
    status = "Enabled"
  }
}

############################
# IAM for Lambda
############################
resource "aws_iam_role" "lambda_role" {
  name = "cf-monitor-lambda-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = { Service = "lambda.amazonaws.com" }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "basic" {
  role       = aws_iam_role.lambda_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

resource "aws_iam_role_policy" "custom" {
  role = aws_iam_role.lambda_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action   = ["cloudwatch:PutMetricData"]
        Effect   = "Allow"
        Resource = "*"
      },
      {
        Action   = ["cloudfront:CreateInvalidation"]
        Effect   = "Allow"
        Resource = "*"
      },
      {
        Action   = ["s3:GetObject"]
        Effect   = "Allow"
        Resource = "${aws_s3_bucket.log.arn}/*"
      }
    ]
  })
}

############################
# Lambda Parser
############################
data "archive_file" "parser_zip" {
  type        = "zip"
  source_dir  = "${path.module}/lambda/parser"
  output_path = "${path.module}/parser.zip"
}

resource "aws_lambda_function" "parser" {
  filename         = data.archive_file.parser_zip.output_path
  function_name    = "cf-log-parser"
  role             = aws_iam_role.lambda_role.arn
  handler          = "log-parser.lambda_handler"
  runtime          = "python3.11"
  timeout          = 60
}

############################
# Lambda Invalidate
############################
data "archive_file" "invalidate_zip" {
  type        = "zip"
  source_dir  = "${path.module}/lambda/invalidate"
  output_path = "${path.module}/invalidate.zip"
}

resource "aws_lambda_function" "invalidate" {
  filename      = data.archive_file.invalidate_zip.output_path
  function_name = "cf-invalidate"
  role          = aws_iam_role.lambda_role.arn
  handler       = "invalidate.handler"
  runtime       = "python3.11"

  environment {
    variables = {
      DISTRIBUTION_ID = aws_cloudfront_distribution.main.id
    }
  }
}

############################
# S3 → Lambda Trigger
############################
resource "aws_s3_bucket_notification" "log_trigger" {
  bucket = aws_s3_bucket.log.id

  lambda_function {
    lambda_function_arn = aws_lambda_function.parser.arn
    events              = ["s3:ObjectCreated:*"]
  }

  depends_on = [aws_lambda_permission.allow_s3]
}

resource "aws_lambda_permission" "allow_s3" {
  statement_id  = "AllowS3Invoke"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.parser.function_name
  principal     = "s3.amazonaws.com"
  source_arn    = aws_s3_bucket.log.arn
}

############################
# CloudFront
############################
resource "aws_cloudfront_origin_access_identity" "oai" {}

resource "aws_cloudfront_distribution" "main" {
  enabled             = true
  default_root_object = "index.html"

  origin {
    domain_name = aws_s3_bucket.site.bucket_regional_domain_name
    origin_id   = "site"

    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.oai.cloudfront_access_identity_path
    }
  }

  logging_config {
    bucket = aws_s3_bucket.log.bucket_domain_name
    prefix = "access-logs/"
  }

  default_cache_behavior {
    target_origin_id       = "site"
    viewer_protocol_policy = "redirect-to-https"

    allowed_methods = ["GET", "HEAD"]
    cached_methods  = ["GET", "HEAD"]

    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    cloudfront_default_certificate = true
  }
}

############################
# SNS
############################
resource "aws_sns_topic" "alert" {}

resource "aws_sns_topic_subscription" "email" {
  topic_arn = aws_sns_topic.alert.arn
  protocol  = "email"
  endpoint  = var.alert_email
}

############################
# CloudWatch Alarm
############################
resource "aws_cloudwatch_metric_alarm" "error_alarm" {
  alarm_name          = "cf-error-alarm"
  namespace           = "CFLog"
  metric_name         = "ErrorCount"
  statistic           = "Sum"
  period              = 60
  evaluation_periods  = 1
  threshold           = 5
  comparison_operator = "GreaterThanOrEqualToThreshold"

  alarm_actions = [
    aws_sns_topic.alert.arn,
    aws_lambda_function.invalidate.arn
  ]
}

```

📎versions.tf

ローカルでやるので、ここは重要！

![0011-4]( /assets/images/0011-04.png )

## terraform graphとGraphvizでリソース依存関係を可視化して、作成順序と依存構造を事前に検証できます。

実際の流れ
1️⃣ グラフを出力

```
terraform graph > graph.dot
```

2️⃣ 画像に変換

```
dot -Tpng graph.dot -o graph.png
```

✅apply なし。
✅AWS 接続なし。
✅課金リスクゼロ。

graph.png

![0011-5]( /assets/images/0011-05.png )


CDKと比べると、ちょっと気になるのは

**Terraform によるデプロイ自動化**

```
aws_lambda_function.parser
↓
data.archive_file.parser_zip
```

archive_file データソースを利用して、Lambda のソースコードを zip 形式に自動圧縮し、
その成果物を aws_lambda_function に渡してデプロイしている。

これにより、ビルドからデプロイまでを Terraform 内で自動化でき、
手動操作なしで IaC ベースの運用が可能となる。
