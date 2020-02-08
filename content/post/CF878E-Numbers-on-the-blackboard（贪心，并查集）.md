+++
title = "CF878E Numbers on the blackboard（贪心，并查集）"
categories = ["题解"]
tags = ["贪心", "并查集", "离线算法"]
date = "2019-10-08T20:49:18+08:00"
description = ""
aliases = ["/post/CF878E-Numbers-on-the-blackboard（贪心，并查集）", "/CF878E-Numbers-on-the-blackboard（贪心，并查集）"]
+++


## 题目链接

[CF](https://codeforces.com/contest/878/problem/E)

## 题意简述

对于一个数列，每次操作可以将相邻的两个数 $x$ 和 $y$ 合并成一个数 $x+2y$，定义一个数列的权值为进行操作直至只剩一个数能得到的最大值。

多组询问，每次给定一个区间，求这个区间的权值。

数列值域 $-10^9\sim10^9$，数列长度和询问个数不超过 $10^5$。

<!--more-->

## 简要做法

先不考虑多组询问。

经过（并不）简单的推理，可以发现，设数列 $a_{1..n}$ 的权值为 $\sum_{i=1}^na_i2^{k_i}$，那么 $k_1=0$, $1\le k_i\le k_{i-1}+1(i\ge 2)$。

那么，最优方案中，$k_{2..n}$ 一定是一块一块从 $1$ 开始严格递增的。

如果我们已经知道了一个数列 $k_i$ 的构成，这时要在其末端加入一个数，那么可以得到贪心策略：

1. 若加入的数是正数，与前一块合并。若合并后整块构成的等比数列之和仍为正数，继续合并。
2. 否则结束合并过程。

这个合并的过程可以用并查集维护。

问题在于，如何判断一块的正负。可以对每块维护它的大小（块中的第 $i$ 个数与 $2^i$ 的乘积之和），合并时更新。但这样做可能会溢出，但可以发现，一旦一块的大小达到 $10^9$，一定会一直合并到最前面，所以大于 $10^9$ 的都可以视作 $10^9$；一旦前一块的长度超过 $30$ 且当前块大小为正，也一定会一直合并到最前面，也可以视作 $10^9$。

接下来考虑如何回答询问。

把询问离线下来，右端点相同的询问一起处理。处理一个询问之前先计算出 $a_{1..r}$ 的块，若 $l=1$ 答案就是所有块大小的和，否则的话答案会是若干块的答案之和加上一个块的后缀。因为取一个块的后缀，断点所在块一定不会分开（一块的真后缀一定为正），后面的块也不会合并到前面去。处理出模意义下块答案的前缀和，以及 $presum_i=\sum_{j=1}^i2^{i-1}a_i$，就可以快速回答询问了。

还有一个小问题：只有第一块的系数是从 $2^0=1$ 开始的。由于计算时第一块一定是那个后缀，计算后缀答案时从 $1$ 开始，计算整块答案时从 $2$ 开始即可。

## 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <algorithm>

using namespace std;

typedef long long ll;
typedef pair<int, int> pii;

const int N = 100010;
const int mod = 1e9 + 7;

int n, m, a[N], presum[N], inv2[N], f[N], len[N], sum[N], preans[N], out[N];
vector<pii> q[N];

int find(int x)
{
    return x == f[x] ? x : f[x] = find(f[x]);
}

int calc(int l, int r)
{
    return (ll) (presum[r] - presum[l - 1] + mod) * inv2[l - 1] % mod;
}

int main()
{
    scanf("%d%d", &n, &m);

    inv2[0] = 1;
    for (int i = 1, two = 1; i <= n; ++i, two = two * 2 % mod)
    {
        scanf("%d", a + i);
        inv2[i] = (ll) inv2[i - 1] * (mod + 1) / 2 % mod;
        presum[i] = (presum[i - 1] + (ll) two * a[i] % mod + mod) % mod;
    }

    for (int i = 1; i <= m; ++i)
    {
        int l, r;
        scanf("%d%d", &l, &r);
        q[r].push_back(pii(l, i));
    }

    for (int i = 1; i <= n; ++i)
    {
        f[i] = i;
        len[i] = 1;
        sum[i] = a[i];
        while (find(i) > 1 && sum[find(i)] > 0)
        {
            int x = find(i);
            int y = find(x - 1);
            if (len[y] >= 30 || (((ll) sum[x]) << len[y]) + sum[y] >= 1e9) sum[y] = 1e9;
            else sum[y] += sum[x] << len[y];
            len[y] += len[x];
            f[x] = y;
        }
        preans[find(i)] = (preans[find(find(i) - 1)] + 2ll * calc(find(i), i)) % mod;
        for (int j = 0; j < q[i].size(); ++j)
        {
            int l = q[i][j].first;
            int id = q[i][j].second;
            out[id] = (0ll + preans[find(i)] - preans[find(l)] + mod + calc(l, find(l) + len[find(l)] - 1)) % mod;
        }
    }

    for (int i = 1; i <= m; ++i) printf("%d\n", out[i]);

    return 0;
}
```