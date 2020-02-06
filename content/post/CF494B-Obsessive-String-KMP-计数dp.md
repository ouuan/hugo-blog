+++
title = "CF494B Obsessive String (KMP,计数dp)"
categories = ["题解"]
tags = ["字符串", "KMP", "计数dp"]
date = "2019-05-05T13:16:44+08:00"
description = ""
+++


## 题目链接

[洛谷](https://www.luogu.org/problemnew/show/CF494B)

[CF contest](https://codeforces.com/contest/494/problem/B)

[CF problemset](http://codeforces.com/problemset/problem/494/B)

## 题意简述

给你两个字符串 $S$ 和 $T$，你需要在 $S$ 中取若干个（至少一个）不相交的子串，使得每个子串都包含 $T$，求方案数模 $10^9+7$。

字符串长度小于等于 $10^5$。

<!--more-->

## 简要做法

首先用某种字符串算法求出 $T$ 在 $S$ 中匹配的所有位置（一般人都会选择 KMP）。

记 $|S|=n$, $|T|=m$。

然后，我们记 $p_i$ 表示 $[i+m,n]$ 中最小的匹配位置（也就是 $S[i+1..n]$ 这个子串第一个匹配的位置），记 $q_i$ 表示 $[1,i]$ 中最大的匹配位置（也就是 $S[1..i]$ 这个子串最后一个匹配的位置）。

然后开始 dp，令 $f_i​$ 表示子串 $S[i+1..n]​$ 的答案。转移时，我们考虑选择下一个子串的右端点和左端点。右端点可以在 $[q_i,n]​$ 中选取，而对于一个右端点 $j​$，可以选择的左端点有 $p_j-m-i+1​$ 个。另外，也可以不继续选择下一个子串。所以可以得到转移方程：

$$f_i=1+\sum\limits_{j=q_i}^nf_j(p_j-m-i+1)$$

如果我们把 $g_i=\sum\limits_{j=i}^nf_jp_j$ 和 $h_i=\sum\limits_{j=i}^nf_j$ 分别存下来，就可以得到 $f_i=1+g_{q_i}-(m+i-1)f_{q_i}$，然后就可以 $\mathcal O(n)$ 计算了。

最后，这样计算的话会把一个子串都不选计入总数，所以答案要减一。

## 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <cstring>

using namespace std;

typedef long long ll;

const int N = 100010;
const int mod = 1e9 + 7;

char s[N], t[N];
int n, m, nxt[N], q[N], p[N], f[N], g[N];

int main()
{
    int i, k;

    scanf("%s%s", s + 1, t + 1);
    n = strlen(s + 1);
    m = strlen(t + 1);

    for (i = 2, k = 0; i <= m; ++i)
    {
        while (k && t[i] != t[k + 1]) k = nxt[k];
        if (t[i] == t[k + 1]) ++k;
        nxt[i] = k;
    }

    for (i = 1, k = 0; i <= n; ++i)
    {
        while (k && s[i] != t[k + 1]) k = nxt[k];
        if (s[i] == t[k + 1]) ++k;
        if (k == m) q[i - m] = p[i] = i;
    }

    for (i = 1; i <= n; ++i) if (!p[i]) p[i] = p[i - 1];

    for (i = n; i >= 0; --i)
    {
        if (!q[i]) q[i] = q[i + 1];
        if (q[i]) f[i] = (1 + g[q[i]] - (ll) f[q[i]] * (m + i - 1) % mod + mod) % mod;
        else f[i] = 1;
        g[i] = ((ll) f[i] * p[i] + g[i + 1]) % mod;
        f[i] = (f[i] + f[i + 1]) % mod;
    }

    cout << (f[0] - f[1] + mod - 1) % mod;

    return 0;
}
```

