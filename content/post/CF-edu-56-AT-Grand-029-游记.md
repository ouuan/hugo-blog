+++
title = "CF edu 56 & AT Grand 029 游记"
date = "2018-12-16T00:29:11+08:00"
categories = ["游记"]
tags = ["AtCoder", "Codeforces"]
description = ""
+++


今天 AT 和 CF 刚好连上了，于是就都打了一下。

<!-- more -->

## 关于模板

今天心血来潮把用于在线比赛的模板换了一下，加了点东西，然后发现 `rep(1,l,r)` 写起来非常不顺手..保留了部分比较好用的。

测试的时候忘记开数组了（~~都是些什么sb错误~~），然后以为 ll 不能作下标，于是没有 `#define long long`，然后 CF 挂惨了..

## AtCoder

上一场打的 Beginner，这场难度正常多了。

A 是个值域为 $2$ 的逆序对..

B 用 multiset 乱搞了一下，对于每个数枚举组成的数，然后从大到小匹配，不知道是不是正解，反正过了，而且是 $log^2$ 的。

C 感觉挺可做的..可能有细节没调出来，赶着回酒店打 CF 就没有继续调了..

D 一开始还在想博弈论完全不会..然后仔细看了一眼，如果 A 不走，B 就会不走，就结束了；所以 A 一定能走则走。然后就随便做了。

只不过 AT 的 rating 真的涨的好快..

## Codeforces

ABC 三道 spj ？？？

都是随便构造就能做的..

然而 C 一开始忘开 ll 了.....

D 黑白染色一下，连通块内两种节点分别有 $a$ 个和 $b$ 个答案就是 $2^a+2^b$ ，把每个连通块的答案加起来就好了。

由于 $O(nq)$ memset 会爆掉，不能 memset 整个数组，于是愉快地在开了 ll 的情况下 memset(...sizeof(int)...)；发现了之后不小心把开 int 的另一个数组也改成 memset(...sizeof(long long)..) 了.. 开场 $40$ 分钟的时候这 $3$ 个关于 ll 的罚时让我排名翻了三倍...

于是，A 了 D 之后我就在板子里加上了 `#define int long long`。

看了会儿 E 不会做，然后一看 standing，惊奇地发现 G 有一堆（$15$ 个，当时 E $7$ F $1$）人 A 了，然后一看，就是POJ 2926+动态RMQ...

感觉自己几年没有写过普通线段树了（最近写的全是平衡树/动态开点线段树），写了半个多小时还写错了..毕竟是 CF，应该去复制个模板才对的...一交，MLE 了，~~woc我好不容易`#define int long long`了就是这个结果？？~~改成 int，跑了 $5.4s$，巨方，于是手动开了 O3，$4.8s$ ，但重交竟然没有罚时。应该去找个 BIT 动态求 RMQ 的模板的...

看到 halyavin 参赛了，感觉自己要 fst ，赶紧把博客写了睡觉去。

## UPD

halyavin 竟然没有 hack...然而 D 有一堆 memset 整个数组的，我也去 hack 了一个（edu hack $\sqrt{}$）。

没有 fst，第一次 A $5$ 题，上 $2k$ 了，感觉海星。

## UUPD

[题解 CF1093D 【Beautiful Graph】](https://www.luogu.org/blog/ouuan/solution-cf1093d)

[题解 CF1093G 【Multidimensional Queries】](https://www.luogu.org/blog/ouuan/solution-cf1093g)