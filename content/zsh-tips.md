+++
title = "zshでパスから必要な部分を抽出する"
date = "2022-11-15"
[taxonomies]
category = ["プログラミング"]
+++

zsh でファイルパスをいい感じに処理するための tips

<!-- more -->

```sh
local f='./dir/file.txt'

echo $f:a  # /Users/.../dir/file.txt
echo $f:e  # txt
echo $f:u  # ./DIR/FILE.TXT
echo $f:t  # file.txt
echo $f:h  # ./dir
```

- `path:a` ：フルパスを取得する
- `path:e` ：拡張子を取得する
- `path:u` ：大文字で出力する
- `path:t` ：ファイル名のみを取得する
- `path:h` ：ディレクトリを取得する
