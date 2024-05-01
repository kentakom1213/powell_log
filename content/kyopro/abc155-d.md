+++
title = "解説：ABC155 D - Pairs"
date = "2024-05-01 22:30:00"
[taxonomies]
category = ["競プロ"]
tags = ["Rust"]
[extra]
comment = true
+++

## 問題

### 概要

$N$ 個の整数 $A_1, A_2, \ldots, A_N$ が与えられる．このうち $2$ つを選んで積をとったものは $\frac{N(N-1)}{2}$ 通り存在するが，このうち小さい方から $K$ 番目の数を求めよ．

- $2 \le N \le 2\times 10^5$
- $1 \le K \le \frac{N(N-1)}{2}$
- $-10^9 \le A_i \le 10^9$

### リンク

- [ABC155 D - Pairs (AtCoder)](https://atcoder.jp/contests/abc155/tasks/abc155_d)

## 考察

愚直な実装（ペアを列挙してソートする）はすぐに思いつきますが，
$N \le 2\times 10^5$ であるので，すべてを列挙するのはもちろん不可能です．

このように，解の候補が取りうる値が非常に多いタイプの問題でよく使うテクニックとして，**「解の存在性で二分探索をする」** というものがあります．

---

ここで，この問題より少し簡単な

> $A_1,\ldots,A_N$ のペアの積のうち，$x$ 未満であるものはいくつあるか

という問題<sup>*</sup> を考えてみます．このとき，以下の性質が成り立ちます．

{% admonition(type="note", title="性質1") %}
**$A_1,\ldots,A_N$ のペアの積のうち，$x$ 未満であるもの** がちょうど $K$ 個であるとき，$x$ が $K$ 番目に大きい数である．
{% end %}

よって，問題<sup>*</sup> を速く解くことができれば，性質1を使うことで元の問題の答えを求められることがわかります．

## 解答

`Rust`での解説は以下のとおりです．

### コード

```rust
use proconio::{
    input,
    marker::{Bytes, Chars, Usize1},
};

const NEG1: usize = 1_usize.wrapping_neg();
const INF: isize = 1001001001001001001;

fn main() {
    input! {
        N: usize,
        K: Usize1,
        mut A: [isize; N]
    }

    // Aをソート
    A.sort();

    // A[i] * A[j] がx未満である(i,j)の個数を数える
    let count = |x: isize| -> usize {
        let mut ans = 0;
        for i in 0..N {
            if A[i] >= 0 {
                let mut ok = NEG1;
                let mut ng = N;
                while ng.wrapping_sub(ok) > 1 {
                    let mid = ok.wrapping_add(ng) / 2;
                    if A[i] * A[mid] < x {
                        ok = mid;
                    } else {
                        ng = mid;
                    }
                }
                if ok != NEG1 && i <= ok {
                    ans += ok;
                } else {
                    ans += ok.wrapping_add(1);
                }
            } else {
                let mut ok = NEG1;
                let mut ng = N;
                while ng.wrapping_sub(ok) > 1 {
                    let mid = ok.wrapping_add(ng) / 2;
                    if A[i] * A[N - mid - 1] < x {
                        ok = mid;
                    } else {
                        ng = mid;
                    }
                }
                if ok != NEG1 && N.wrapping_sub(ok) - 1 <= i {
                    ans += ok;
                } else {
                    ans += ok.wrapping_add(1);
                }
            }
        }
        ans / 2
    };

    let mut ok = -INF;
    let mut ng = INF;

    while ng - ok > 1 {
        let mid = (ok + ng) / 2;

        if count(mid) <= K {
            ok = mid;
        } else {
            ng = mid;
        }
    }

    println!("{ok}");
}
```


### 提出
