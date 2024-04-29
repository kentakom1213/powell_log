+++
title = "解説：ABC048 D - An Ordinary Game"
date = "2024-04-29 15:00:00"
draft = true
[taxonomies]
category = ["競プロ"]
tags = ["アルゴリズム"]
[extra]
+++

{% mermaid() %}
graph TD
    abcab --"2文字目を取る"--> acab
    abcab --"3文字目を取る"--> abab
    abcab --"4文字目を取る"--> abcb
    acab --"2文字目を取る"--> aab1[aab]
    acab --"3文字目を取る"--> acb1[acb]
    acb1 --"2文字目を取る"--> ab1[ab]
    abab --"2文字目を取る"--> aab2[aab]
    abab --"3文字目を取る"--> abb1[abb]
    abcb --"2文字目を取る"--> acb2[acb]
    abcb --"3文字目を取る"--> abb2[abb]
    acb2 --"2文字目を取る"--> ab2[ab]
    subgraph W1[先攻の勝ち]
        abcab
    end
    subgraph L1[先攻の負け]
        acab
        abab
        abcb
    end
    subgraph W2[先攻の勝ち]
        aab1
        acb1
        aab2
        abb1
        abb2
        acb2
    end 
    subgraph L2[先攻の負け]
        ab1
        ab2
    end
{% end %}
