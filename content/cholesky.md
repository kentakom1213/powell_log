+++
title = "Rustでコレスキー分解"
date = "2024-04-27"
updated = "2024-05-30"
description = "Rustでコレスキー分解を実装してみる"
[taxonomies]
category = ["プログラミング"]
tags = ["アルゴリズム", "Rust"]
+++

[CMA-ES](https://ja.wikipedia.org/wiki/CMA-ES)を実装したいので，その前段階としてコレスキー分解を実装してみます．

## 式変形

コレスキー分解では，正定値対称行列 $A$ を下三角行列 $L$ を用いて

$$
A = LL^T
$$

の形に分解します．

要素を書き下すと，

$$
\begin{align*}
    \left(
    \begin{array}{ccc}
        A_{11} & \cdots & A_{1n}\\\\
        \vdots & \ddots & \vdots\\\\
        A_{n1} & \cdots & A_{nn}
    \end{array}
    \right)
    &=
    \left(
    \begin{array}{ccc}
        L_{11} & & O\\\\
        \vdots & \ddots & \\\\
        L_{n1} & \cdots & L_{nn}
    \end{array}
    \right)
    \left(
    \begin{array}{ccc}
        L_{11} & \cdots & L_{n1}\\\\
        & \ddots & \vdots\\\\
        O & & L_{nn}
    \end{array}
    \right)\\\\
    &=
    \left(
    \begin{array}{ccc}
        L_{11}^2 & \cdots & L_{11}L_{n1}\\\\
        \vdots & \ddots & \vdots \\\\
        L_{n1}L_{11} & \cdots & {\displaystyle\sum_{i=1}^n} L_{ni}^2
    \end{array}
    \right)
\end{align*}
$$

よって， $A_{ij}$ は

$$
A_{ij} = \sum_{k=1}^{\min(i, j)} L_{ik}L_{jk}
$$

とくに， $i \ge j$ のとき，

$$
A_{ij} = \sum_{k=1}^j L_{ik}L_{jk}
$$

となります．

## 一般項の求め方

$L_{ij} ~ (i \ge j)$ を求める際には，各行ごとに添字が小さいものから順に，

- $1$ 行目：$L_{11} = \sqrt{A_{11}}$
- $2$ 行目：$L_{21} = A_{21} / L_{11}$ ， $L_{22} = \sqrt{A_{22} - L_{21}^2}$
- ︙
- $i$ 行目：\
    $L_{i1} = A_{i1} / L_{11}$\
    ︙\
    $L_{ij} = (A_{ij} - \sum_{k=1}^{j-1} L_{ik}L_{jk})/L_{jj}$\
    ︙\
    $L_{ii} = \sqrt{A_{ii} - \sum_{k=1}^{i-1} L_{ik}}$

と求めていけばよいです．

## 実装

`f64.is_normal()` で値が `0` や `NaN` になった場合を弾いています．

```rust
#![allow(non_snake_case)]

/// 対称行列 `A` をコレスキー分解する．
/// 分解が存在しない場合，`None`を返す．
pub fn cholesky_decomposition(A: &Vec<Vec<f64>>) -> Option<Vec<Vec<f64>>> {
    let N = A.len();
    let mut L: Vec<Vec<f64>> = vec![vec![0.0; N]; N];

    // 添字が小さい方から順に求める
    for i in 0..N {
        for j in 0..i {
            L[i][j] = (
                    A[i][j] - (0..j).map(|k| L[i][k] * L[j][k]).sum::<f64>()
                ) / L[j][j];
            // 0 や Nan になったときは終了
            if !L[i][j].is_normal() {
                return None;
            }
        }
        // 対角成分
        L[i][i] = (
                A[i][i] - (0..i).map(|k| L[i][k].powf(2.0)).sum::<f64>()
            ).sqrt();
        // 0 や Nan になったときは終了
        if !L[i][i].is_normal() {
            return None;
        }
    }
    Some(L)
}
```

### 実験

```rust
/// コレスキー分解の関数
/// ...

/// 行列の転置
pub fn T(A: &Vec<Vec<f64>>) -> Vec<Vec<f64>> {
    let N = A.len();
    let mut T = vec![vec![0.0; N]; N];
    for i in 0..N {
        for j in 0..N {
            T[i][j] = A[j][i];
        }
    }
    T
}

/// 行列の積を返す
pub fn dot(A: &Vec<Vec<f64>>, B: &Vec<Vec<f64>>) -> Vec<Vec<f64>> {
    let N = A.len();
    let mut res = vec![vec![0.0; N]; N];
    for i in 0..N {
        for j in 0..N {
            res[i][j] = (0..N).map(|k| A[i][k] * B[k][j]).sum();
        }
    }
    res
}

fn main() {
    {
        let A = vec![
            vec![4.0, 2.0, 6.0],
            vec![2.0, 5.0, 5.0],
            vec![6.0, 5.0, 14.0],
        ];
        let L = cholesky_decomposition(&A).unwrap();
        let LL = dot(&L, &T(&L));

        println!("A: {:?}", A);
        println!("L: {:?}", L);
        println!("LL^T: {:?}", LL);

        // 再構成できる
        assert_eq!(A, LL);
    }

    {
        let A = vec![vec![0.0, 0.0], vec![0.0, 0.0]];
        let L = cholesky_decomposition(&A);

        // 分解に失敗
        assert_eq!(L, None);
    }
}
```

### 出力

```
A: [[4.0, 2.0, 6.0], [2.0, 5.0, 5.0], [6.0, 5.0, 14.0]]
L: [[2.0, 0.0, 0.0], [1.0, 2.0, 0.0], [3.0, 1.0, 2.0]]
LL^T: [[4.0, 2.0, 6.0], [2.0, 5.0, 5.0], [6.0, 5.0, 14.0]]
A: [[0.0, 0.0], [0.0, 0.0]]
L: None
```

うまくできていそうです．
数学的に正しいかどうか（特に値が`0`や`NaN`になったときの処理あたり）は確認できていないので，間違っていましたらご指摘お願いします．
