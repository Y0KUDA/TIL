imported from Qitta
https://qiita.com/Y0KUDA/items/18cbc53feea8215b3e92

---
title: Microsoft Flowをコードに組み込む
tags: MicrosoftFlow ChatOps serverless CI faas
author: Y0KUDA
slide: false
---
# TL;DR
Microsoft FlowのトリガをHTTPリクエストに設定すると、AWS LambdaみたいなFaaSっぽい使い方ができる。(戻り値を使うとかはできないけど・・・)
また、デフォルトで登録されていない、任意のサービスに対応することができる。
# Microsoft Flowって？
ノードベースプログラミングで簡単に操作のフローを自動化できる。
Slackやメール、Twitter等のアプリと連携が可能。
フリープランが存在し、月ごと750回まで実行可能 (2019/3/21現在)
https://japan.flow.microsoft.com/ja-jp/pricing

# 簡単な例 ( Hello World )
1. トリガに`[HTTP要求の受信時]`を設定します。
2. トリガに好きなフローを繋げる。ここではSlackにHello Worldを投稿させます。
![スクリーンショット (24).png](https://qiita-image-store.s3.amazonaws.com/0/275289/bf9d7c7f-18ca-cdbc-e783-a2f941a34644.png)
3. 生成されたURLにHTTPリクエストします。メソッドはデフォルトでPOSTです。powershellだと以下のようなコードで発火できます。

```powershell:helloworld.ps1
Invoke-RestMethod -Uri "https://prod-27.japaneast.logic.azure.com:443/XXXXXXXXXXXXXXXXXXXXXX" -Method POST
```

# 引数を渡す例 ( Slackに任意の内容を投稿 )
トリガに`[HTTP要求の受信時]`を設定する際、JSONスキーマを指定すると、受け取ったJSONのデータをフロー内で扱うことができます。(ペイロードをアップロードすると、自動でスキーマ生成してくれます。)

1. 以下の`sample_payload.json`から`[HTTP要求の受信時]`にスキーマを生成します。
2. Slackのチャネル名で`[カスタム値の入力]`を選択し、`channel`を選択します。メッセージテキストも同様にし、`message`を選択します。
![スクリーンショット (25).png](https://qiita-image-store.s3.amazonaws.com/0/275289/b68fb5e0-a638-fbbe-c36b-c2db5267eedb.png)

3. 生成されたURLにHTTPリクエストします。powershellだと`helloworld2.ps1`のようなコードで発火できます。今度はrandomにhello worldを投稿します。


```json:sample_payload.json
{
	"channel":"general",
	"message":"hello world"
}
```

```powershell:helloworld2.ps1
Invoke-RestMethod -Uri "https://prod-27.japaneast.logic.azure.com:443/XXXXXXXXXX" -Method POST -Body ( @{"channel"="random";"message"="hello world"} | ConvertTo-Json) -ContentType 'application/json'
```

# サービス連携 ( GitHub : forced push通知 )
Microsoft Flowはアプリケーション連携が豊富で便利だけど、GitHubにforced pushされたとき用のトリガは存在しないので、自分で作っていくしかありません。

1. github webhookのサンプルペイロード( https://gist.github.com/gjtorikian/5171861 )から`[HTTP要求の受信時]`にスキーマを生成します
2. 条件フローを作成し、`[forced]`が`true`に等しいとき、という設定にします。
3. `[はいの場合]`にSlackを繋げてメッセージの投稿を設定します。
![スクリーンショット (26).png](https://qiita-image-store.s3.amazonaws.com/0/275289/4064d93a-3f21-8657-b950-9d7171e0b7f4.png)
4. 最後に、GitHubのWebhookに`[HTTP要求の受信時]`から生成されたURLを設定します。

これでforced pushされたときに、Slackに通知が飛びます。


