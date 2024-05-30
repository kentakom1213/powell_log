+++
title = "シェルから簡単にLINE通知しよう！"
date = "2022-06-26"
[taxonomies]
category = ["プログラミング"]
tags = []
[extra]
cover_image = "/images/line_notify/notify_mobile.jpeg"
+++

LINE-Notify を使えばコマンドラインから簡単にメッセージを送信できることがわかったので共有します。

<!-- more -->

## やること

### 1. LINE Notify を友達追加する

[LINE Notify](https://notify-bot.line.me/ja/) にアクセスします。
QR コードがあるので、そこから LINE Notify を友達登録しておきましょう。

### 2. LINE Notify のアクセストークンを取得する

[マイページ](https://notify-bot.line.me/my/) のページ下部にある、 **トークンを発行する** ボタンからトークンを発行します。

通知を送りたいグループを選択します。（テストだったら **1:1 で LINE Notify から通知を受け取る** にしておくのが無難）

**次の画面で出てくる文字列をコピーして、どこかに保存しておきます。**

{% admonition(type="warning") %}
悪用の恐れがあるのでトークンは誰にも見せないように！！
{% end %}

### 3. curl コマンドで通知を送信

下のコマンドをターミナルで実行します。

```shell
curl -X POST \
    -H 'Authorization: Bearer [先ほどのトークン]' \
    -F 'message=[送信したいメッセージ]' \
    https://notify-api.line.me/api/notify
```

うまく行くと、シェルに ↓ のように出力されて、

```json
{ "status": 200, "message": "ok" }
```

↓ こんな感じの通知が来るはずです。

{{ resize_image(path="/images/line_notify/notify_mobile.jpeg", width=700, height=300, op="fill") }}

クエリに url を指定すると、画像も送れます。（サーバーにアップロードするとかもできるみたい。）

```shell
curl -X POST \
    -H 'Authorization: Bearer [トークン]' \
    -F 'message=画像のテスト' \
    -F 'imageThumbnail=https://blog.pwll.dev/images/sudoku/sudoku.png' \
    -F 'imageFullsize=https://blog.pwll.dev/images/sudoku/sudoku.png' \
    https://notify-api.line.me/api/notify
```

{{ resize_image(path="/images/line_notify/send_image.jpeg", width=400, height=300, op="fill") }}

### 4. シェルスクリプトの作成

このままでも十分なのですが、こんなスクリプトを書いてみました。

```sh
#!/bin/zsh

local ACCESS_TOKEN='<トークン>'
local __url='https://notify-api.line.me/api/notify'

# パラメータの受け取り
if [[ "`echo $@`" == "" ]]; then
    __str=`cat -`  # パイプの受け取り
else
    __str=$@  # 引数の受け取り
fi

# 処理
echo "${__str}"

# LINE Notifyで通知
curl -s -X POST \
    -H "Authorization: Bearer $ACCESS_TOKEN" \
    -F "message=${__str}" \
    $__url > /dev/null
```

これで、

```shell
./notify.sh ほげほげ〜
```

```shell
cat hoge.txt | ./notify.sh
```

というようなコマンドで通知を送信できます！

### set alias

```sh
alias notify='<notify.shのパス>'
```

などとエイリアスを設定しておくと便利かもしれませんね。

## 最後に

活用方法がいろいろあって面白そうですね。また何か作ってみようと思います。

## 参考

- [コマンドラインから LINE にメッセージを送れる LINE Notify - LINE Engineering](https://engineering.linecorp.com/ja/blog/using-line-notify-to-send-messages-to-line-from-the-command-line/)
- [LINE Notify API Document](https://notify-bot.line.me/doc/ja/)
