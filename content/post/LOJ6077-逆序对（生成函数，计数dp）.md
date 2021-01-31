+++
title = "LOJ6077 逆序对（生成函数，计数dp）"
date = "2019-09-04T09:22:32+08:00"
categories = ["题解"]
tags = ["组合数学", "生成函数", "计数dp"]
description = ""
aliases = ["/post/LOJ6077-逆序对（生成函数，计数dp）", "/LOJ6077-逆序对（生成函数，计数dp）"]
+++


## 题目链接

[LOJ](https://loj.ac/problem/6089)

## 题意简述

求长度为 $n$ 逆序对数为 $k$ 的排列个数。

$1\le n, k\le 10^5$，$k\le \binom n 2$

<!--more-->

## 简要作法

从小到大依次考虑将每个数插入排列，那么每个数 $i$ 都可以贡献 $0\dots i-1$ 个逆序对，所以答案的生成函数为 $(1 + x)(1 + x + x^2)\cdots(1+x+\cdots+x^{n-1})$。

上下同时乘上 $(1-x)^n$，即求：
$$
\frac{(1-x)(1-x^2)\cdots(1-x^n)}{(1-x)^n}
$$
（不约分是为了方便求。）

分母 $\frac{1}{(1-x)^n}=\sum\limits_{i\ge 0}\binom{n-1+i}{n-1}x^i$，<del>是一个大家熟知的结论</del>，可以利用 $(1+x+x^2+\cdots)^n$ 的组合意义说明。

分子的 $x^i$ 项系数的组合意义为：考虑从 $1,2,\ldots,n$ 中选若干个和为 $i$ 的数（每个数只能选一遍）的所有方案，若选了奇数个数贡献为 $-1$，若选了偶数个数贡献为 $1$。

这个东西可以用类似 [LOJ6089](/post/loj6089-小y的背包计数问题根号分治计数dp) 的方法求：

令 $f_{i,j}$ 表示选 $i$ 个数和为 $j$ 的方案数。

由于选择的数两两不同，第一维的大小是 $O(\sqrt k)$ 的。

转移有两种方式：

1. 背包里的所有数加一。
2. 背包里的所有数加一，并向背包中放入一个体积为 $1$ 的物品。

$$
f_{i,j}=f_{i-1,j-i}+f_{i,j-i}
$$

但这样算可能会出现体积大于 $n$ 的物品。

具体来说，当 $j\ge n+1$ 时，会有 $f_{i-1,j-n-1}$ 种不合法的方案，需要减去。

计算完 dp 之后，分子的 $x^i$ 项系数即为 $\sum\limits_{j\ge0}(-1)^jf_{j,i}$

最后把分子和分母卷积起来即可，总时间复杂度为 $O(n+k\sqrt k)$ 或 $O(n\log p+k\sqrt k)$（取决于计算组合数与逆元的方式）。

## 参考代码

```cpp
#include <cstdio>
#include <cstring>

using namespace std;

typedef long long ll;

const int N = 100005;
const int mod = 1e9 + 7;

int n, k, f[2][N], cur, ans, fact[N << 1], invf[N << 1];

int qpow(int x, int y)
{
    int out = 1;
    while (y)
    {
        if (y & 1) out = (ll) out * x % mod;
        x = (ll) x * x % mod;
        y >>= 1;
    }
    return out;
}

int c(int x, int y)
{
    return (ll) fact[x] * invf[y] % mod * invf[x - y] % mod;
}

int main()
{
    scanf("%d%d", &n, &k);

    fact[0] = invf[0] = 1;
    for (int i = 1; i <= n + k; ++i)
    {
        fact[i] = (ll) fact[i - 1] * i % mod;
        invf[i] = qpow(fact[i], mod - 2);
    }

    ans = c(n - 1 + k, n - 1);

    f[cur][0] = 1;

    for (int i = 1, sum = 1; sum <= k; sum += (++i))
    {
        memset(f[cur ^= 1], 0, sizeof(int) * i);
        for (int j = i; j <= k; ++j)
        {
            f[cur][j] = (f[cur ^ 1][j - i] + f[cur][j - i]) % mod;
            if (j >= n + 1) f[cur][j] = (f[cur][j] - f[cur ^ 1][j - n - 1] + mod) % mod;
            ans = (ans + (i & 1 ? -1ll : 1ll) * f[cur][j] * c(n - 1 + k - j, n - 1) % mod + mod) % mod;
        }
    }

    printf("%d", ans);

    return 0;
}
```