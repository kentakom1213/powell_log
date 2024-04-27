+++
title = "Pythonで数独を解く"
date = "2022-04-19 15:43:41"
[taxonomies]
tags = ["アルゴリズム", "Python"]
+++

Python で数独ソルバーを作ってみました。

<!-- more-->

## 数独とは

{{ resize_image(path="/images/sudoku/sudoku.png", width=300, height=300, op="fill") }}

$9\times 9$ マスの枠に**以下の 3 つのルール**を守って $1〜9$ までの数字を当てはめていくゲームです。ナンプレとかナンバープレースみたいな名前で呼ばれていたりもします。

### ルール

1. 同じ行に同じ数字が入ってはいけない
1. 同じ列に同じ数字が入ってはいけない
1. 同じブロックに同じ数字が入ってはいけない

## アルゴリズム

比較的単純な**深さ優先探索**(_Depth First Search_)というアルゴリズムを用いました。

## 実際のコード

```python
def solve(arr, cell):
    """単純なdfsで解く"""

    # 求まったとき
    if cell == 81:
        print(*arr, sep="\n")
        exit()

    i, j = cell//9, cell%9
    if arr[i][j] == 0:

        # 順に代入する
        for n in range(1, 10):

            # --- 行 ---
            if n in arr[i]:
                continue

            # --- 列 ---
            is_in_col = False
            for r in range(9):
                is_in_col |= arr[r][j] == n
            if is_in_col:
                continue

            # --- ブロック ---
            is_in_block = False
            for r in range(i//3*3, i//3*3+3):
                for c in range(j//3*3, j//3*3+3):
                    is_in_block |= arr[r][c] == n
            if is_in_block:
                continue

            # 条件をみたしていたとき
            arr[i][j] = n
            solve(arr, cell+1)
            arr[i][j] = 0
    else:
        solve(arr, cell+1)


def main():
    arr = []
    solve(arr, 0)

if __name__ == "__main__":
    main()
```

## おまけ

AtCoder の問題に似たようなものがありました。

- [D - Hanjo (AtCoder)](https://atcoder.jp/contests/abc196/tasks/abc196_d)

この問題が水 diff だったので、数独を解くアルゴリズムも水 diff くらいかなと思います。
