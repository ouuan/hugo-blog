+++
title = "回文自动机（PAM）学习笔记"
date = "2019-05-10T22:31:05+08:00"
categories = ["算法"]
tags = ["字符串", "PAM"]
description = ""
+++


PAM 是一种处理回文串相关问题的有力工具。

~~又是一句废话。~~

<!--more-->

## PAM 是什么？

### 首先它是个自动机...

PAM 是一个接受且仅接受某个字符串的所有回文子串的**中心及右半部分**的 [DFA](/后缀自动机（SAM）学习笔记/#确定有限状态自动机（DFA）)。

“中心及右边部分”在奇回文串中就是字面意思，在偶回文串中定义为一个特殊字符加上右边部分。这个定义看起来很奇怪，但它能让 PAM 真正成为一个自动机，而不仅是两棵树。

### PAM 的状态及转移

PAM 的每个状态都表示一个回文子串，其中包含两个特殊状态，$len​$ 分别为 $0​$ 和 $-1​$，它们分别作为偶回文子串和奇回文子串两棵树的根。

PAM 的转移表示在串的两侧各加上同一个字符，因此 $len​$ 也会加二。PAM 显然是分别以 $0​$ 和 $-1​$ 为根的两棵树，因为每个状态由唯一的状态转移而来（删掉两端的字符）。

和 SAM / AC 自动机一样，PAM 也有 $fail​$ 边，同样表示真后缀中在自动机里的最长状态（也就是最长回文真后缀）。

为了让 PAM 符合自动机的定义，可以在概念上从 $-1​$ 到 $0​$ 连一条特殊字符边，然后以 $-1​$ 作为起始状态。然而在代码实现里没有人会这么做。

## PAM 的构建

### 一个性质

在一个字符串后添加一个字符，至多增加一个之前没有出现过的回文子串，且该回文子串必定是原串的一个回文真后缀两侧加上新添加的这个字符。

简单证明：如果新添加的字符处在多个回文子串内，找到最长的一个，剩下的都可以沿其中心翻折过去，所以一定出现过。

这个性质既说明了 PAM 的状态数是 $\mathcal O(n)​$ 的，也为后文的构建方法提供了依据。

### 基础构建法

这是一个增量算法，即每次以均摊 $\mathcal O(1)​$ 的复杂度向 PAM 基于的字符串的末尾添加一个字符。

记上次达到的状态为 $p​$，字符串为 $s​$，当前添加的字符是字符串中第 $pos​$ 位，我们在 $p​$ 的 $fail​$ 链上找到最长的一个状态满足 $s[pos-len(u)-1]=s[pos]​$，那么当前到达的状态就是 $\delta(u,s[pos])​$，如果这个转移不存在则新建节点并连 $fail​$：在 $fail(p)​$ 的 $fail​$ 链上找到最长的满足上述条件的状态，其 $s[pos]​$ 转移即为新建节点的 $fail​$。特别地，如果 $p​$ 是特殊状态 $-1​$，新建节点的 $fail​$ 要设为 $0​$。

因为 $p​$ 和 $fail(p)​$ 都是在 $fail​$ 树上爬上爬下，其中每添加一个字符最多向下爬一次，所以复杂度是均摊 $\mathcal O(1)​$ 的。

当然如果用 map 存边复杂度就会带 log。

{{% admonition note "参考代码" true %}}

```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 300010;

struct PAM
{
    int len, fail, ch[26];
} pam[N];

void extend();

char s[N];
int n, p = 2, tot = 2, pos;

int main()
{
    scanf("%s", s + 1);
    n = strlen(s + 1);

    pam[1].len = -1;
    pam[2].fail = 1;
    for (pos = 1; pos <= n; ++pos) extend();

    return 0;
}

void extend()
{
    int x = s[pos] - 'a';
    while (s[pos - pam[p].len - 1] != s[pos]) p = pam[p].fail;
    if (pam[p].ch[x]) p = pam[p].ch[x];
    else
    {
        int np = ++tot;
        pam[p].ch[x] = np;
        pam[np].len = pam[p].len + 2;
        if (p == 1) pam[np].fail = 2;
        else
        {
            for (p = pam[p].fail; s[pos - pam[p].len - 1] != s[pos]; p = pam[p].fail);
            pam[np].fail = pam[p].ch[x];
        }
        p = np;
    }
}
```

{{% /admonition %}}

### 其它构建法

PAM 还有支持前后端插入删除、复杂度不是均摊的构建方法，~~但我先咕着~~..感兴趣的话可以看 2017 国家候选队论文《回文树及其应用 翁文涛》。

## 一些例题

[[APIO2014]回文串](https://www.luogu.org/problemnew/show/P3649)<font color = "white">，裸题。和其它自动机一样通过 fail 树子树和来统计出现次数。</font>

[CF835D Palindromic characteristics](https://www.luogu.org/problemnew/show/CF835D)（注意原题数据范围较小，这题可以线性做）<font color="white">，从 fail 链上 len / 2 处转移即可，我比较菜只会倍增所以多个 log。</font>