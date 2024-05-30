+++
title = "ワンライナーでマージソートを実装したい（Python3）"
date = "2022-04-14"
[taxonomies]
category = ["プログラミング"]
tags = ["アルゴリズム"]
[extra]
cover_image = "/images/oneliner-merge-sort/oneliner_merge_sort.jpg"
+++

Python はインデントで構造化されている設計上、ワンライナー(一行だけで動作するコード)を書くことが難しい言語ですが、その分かなり面白いです。

<!-- more -->

## コード

```python
def merge_sort(arr): return arr if len(arr) <= 1 else None if (l := merge_sort(arr[:len(arr)//2])) and (r := merge_sort(arr[len(arr)//2:])) and False else [l.pop(0) if l and r and l[0]<=r[0] else (r.pop(0) if r else l.pop(0)) for _ in range(len(arr))]
```

みやすく

```python
def merge_sort(arr):
    return arr if len(arr) <= 1 \
        else None if (l := merge_sort(arr[:len(arr)//2])) \
                    and (r := merge_sort(arr[len(arr)//2:])) \
                    and False \
        else [
            l.pop(0) if l and r and l[0]<=r[0] else (r.pop(0) if r else l.pop(0))
              for _ in range(len(arr))
            ]
```

### 解説

展開すると以下のようになります。

```python
def merge_sort(arr):

    # 再帰の終了条件
    if len(arr) <= 1:
        return arr

    # left, rightに分割
    l = merge_sort(arr[:len(arr)//2])
    r = merge_sort(arr[len(arr)//2:])

    # mergeする
    return [l.pop(0) if l and r and l[0]<=r[0] else (r.pop(0) if r else l.pop(0)) for _ in range(len(arr))]
```

一番苦労した merge 操作の部分ですが、下のように実装してみました。

```python
# mergeの原型
res = []
while l and r:
    if l[0] <= r[0]:
        res.append( l.pop(0) )
    else:
        res.append( r.pop(0) )
res += l + r

# for文で書き換え
res = []
for _ in range(len(l) + len(r)):
    if l and r and l[0] <= r[0]:
        res.append( l.pop(0) )
    elif r:
        res.append( r.pop(0) )
    else:
        res.append( l.pop(0) )  # lだけに残っている場合

# 内包表記にまとめる
[l.pop(0) if l and r and l[0]<=r[0] else (r.pop(0) if r else l.pop(0)) for _ in range(len(arr))]
```

また、下のような謎の式が入っているのは

```python
... if (l := merge_sort(arr[:len(arr)//2])) \
        and (r := merge_sort(arr[len(arr)//2:])) \
        and False \
    else ...
```

入力`arr`を`left`, `right`の二つのリストに分割するためです。
代入文を書くためにわざわざ 3 項演算子の中でセイウチ演算子`:=`を使うという回りくどい構造になってます。

## おまけ

世界中のワンライナー好きが集う**one-line-wonders**という github リポジトリがあったりします。

[one-line-wonders](https://github.com/wzhouwzhou/one-line-wonders)
