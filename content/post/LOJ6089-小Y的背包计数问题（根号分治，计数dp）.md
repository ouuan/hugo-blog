+++
title = "LOJ6089 小Y的背包计数问题（根号分治，计数dp）"
date = "2019-09-04T09:21:35+08:00"
categories = ["题解"]
tags = ["计数dp", "根号分治"]
description = ""
aliases = ["/post/LOJ6089-小Y的背包计数问题（根号分治，计数dp）", "/LOJ6089-小Y的背包计数问题（根号分治，计数dp）"]
+++


## 题目链接

[LOJ](https://loj.ac/problem/6089)

## 题意简述

你有体积为 $i$ ($1\le i\le n$) 的物品 $i$ 个，同体积物品在计数时没有区别，求装满大小为 $n$ 的背包的方案数。

$1\le n\le 10^5$。

<!--more-->

## 简要作法

体积大于等于 $\sqrt n$ 的物品可以无限选，所以考虑分开处理小于根号的和大于等于根号的。

### 小于根号的

令  $f_{i, j}$ 表示从前 $i$ 种物品中选体积为 $j$ 的方案数。
$$
f_{i, j} = \sum\limits_{k = 0}^{\min(i, \left\lfloor\frac j i\right\rfloor)}f_{i-1, j - ik}
$$
可以使用模 $i$ 同余的前缀和优化。

这部分的时间复杂度为 $O(n\sqrt n)$，空间复杂度可以优化至 $O(n)$。

### 大于等于根号的

令 $g_{i, j}$ 表示选择 $i$ 个物品体积为 $j$ 的方案数。

转移有两种方式：

1. 向背包中放入一个体积为 $\left\lceil\sqrt n\right\rceil$ 的物品。
2. 将背包中所有物品体积加一。

$$
g_{i, j} = g_{i - 1, j - \left\lceil\sqrt n\right\rceil} + g_{i, j - i}
$$

由于最多选 $\left\lfloor\sqrt n\right\rfloor$ 个物品，第一维大小为 $O(\sqrt n)$，这部分复杂度也是 $O(n\sqrt n)$。

### 合并

~~相当于求卷积的一位。~~

两部分加起来体积为 $n$ 就计入答案。

需要注意的是，第二部分中体积为 $k$ 的方案数是 $\sum\limits_{i=0}^{\left\lfloor\sqrt n\right\rfloor}g_{i, k}$，而不是某个单独的 dp 值。

## 参考代码

```cpp
#include <cmath>
#include <cstdio>
#include <cstring>

const int mod = 23333333;
const int N = 100010;

int n, b, f[N], pre[N], g[2][N], cur, ans;

int main()
{
    scanf("%d", &n);

    b = int(sqrt(n) + 1);

    f[0] = 1;

    for (int i = 1; i < b; ++i)
    {
        for (int j = 0; j <= n; ++j) pre[j] = (f[j] + (j >= i ? pre[j - i] : 0)) % mod;
        for (int j = 0; j <= n; ++j)
        {
            if (j < i * (i + 1)) f[j] = pre[j];
            else f[j] = (pre[j] - pre[j - i * (i + 1)] + mod) % mod;
        }
    }

    ans = f[n];

    g[cur][0] = 1;

    for (int i = 1; i < b; ++i)
    {
        memset(g[cur ^= 1], 0, sizeof(int) * (i * b));
        for (int j = i * b; j <= n; ++j)
        {
            g[cur][j] = (g[cur ^ 1][j - b] + g[cur][j - i]) % mod;
            ans = (ans + 1ll * f[n - j] * g[cur][j]) % mod;
        }
    }

    printf("%d", ans);

    return 0;
}
```