+++
title = "CF594D REQ（数颜色，莫队，树状数组，数论）"
categories = ["题解"]
tags = ["数颜色", "莫队", "树状数组", "数论", "数据结构"]
date = "2019-12-05T16:41:23+08:00"
description = ""
+++


## 题目链接

[CF](https://codeforces.com/contest/594/problem/D)

## 题意简述

给你一个正整数序列，多次询问给定区间内每个数之积的欧拉函数 ($\varphi$)，对 $10^9+7$ 取模。

数列长度、询问个数不超过 $2\cdot 10^5$，数的大小不超过 $10^6$ 。

<!--more-->

## 简要做法

其实这题和 [「SDOI2009」HH 的项链](https://www.luogu.com.cn/problem/P1972) 本质是一样的。

由计算欧拉函数的公式得：

$$
\varphi\left(\prod\limits_{i=l}^ra_i\right)=\left(\prod\limits_{i=l}^ra_i\right)\cdot\left(\prod\limits_{p\text{ is a prime factor of }a_{l..r}}\frac{p-1}p\right)
$$

即，区间积乘上每个在这个区间中出现了的质因数减一除以本身。

区间积可以通过计算前缀积线性解决，关键在于求出分母。

如果每个质因数的贡献不是 $(p-1)/p$ 而是 $1$，并且贡献不是相乘而是相加，这就是一个区间数颜色问题了。事实上，区间数颜色问题的解法的确可以套用过来。

（下文中复杂度里的 $w$ 均指值域，$\omega(w)$ 指值域内一个数不可重质因子个数的最大值，即 [A111972](http://oeis.org/A111972) ，$m$ 指模数，即 $10^9+7$ 。）

### 莫队做法

感觉 $O(w+w\log m/\log w+n\sqrt q\log w+q\log q)$ 做法没啥好讲的，会莫队就会了吧..但这个复杂度我写的过不了。

数论相关的莫队题经常可以根号分治去掉一个 log 。

具体来说，大小超过 $\sqrt w$ 的质因数在一个数中最多出现一次，所以可以把这些“大质数”用莫队处理，其它“小质数”用前缀和预处理，然后时间复杂度就是 $O(w+w\log m/\log w+n\sqrt q+q\log q)$ 了（$O(w\log m/\log w)$ 是计算 $(p-1)/p$ 以及 $p/(p-1)$ 的复杂度，当然，可以通过线性求逆元去掉这个 $\log m$）（空间复杂度为 $O(w+n\sqrt w + m)$）。

这样优化之后可以通过本题。

还有另外一种方式：不枚举重复的质因数，可以优化到 $O(w+w\log p/\log w+n\sqrt q\omega(w)+q\log q)$，也可以通过本题。

### 树状数组做法

这也是区间数颜色的经典做法。

记 $pre(p, r)$ 表示质因数 $p$ 在 $[1, r]$ 中最后一次出现的位置，即：

$$
pre(p, r)=\max\left(\\{x|x\in\mathbb{N}, x\in[1, r], p|a_x\\}\bigcup\\{0\\}\right)
$$

那么，$ans(l, r)=\left(\prod_{pre(p, r)\ge l}(p-1)/p\right)\cdot\left(\prod_{i=l}^ra_i\right)$ 。

将询问离线下来，按右端点排序，从左往右遍历每个点作为 $r$，使用数据结构（如树状数组）维护 $pre(1..r, r)$ 并查询答案，就好了。

如果不枚举重复的质因数，复杂度为 $O(w+w\log p/\log w+n\omega(w)\log n+q(\log n+\log m))$ 。

## 参考代码

{{% admonition note "瓶颈不带 log 的莫队做法" true %}}

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <cctype>
#include <algorithm>

using namespace std;

typedef long long ll;

int read()
{
    int out = 0;
    char c;
    while (!isdigit(c = getchar()));
    for (; isdigit(c); c = getchar()) out = out * 10 + c - '0';
    return out;
}

const int B = 168;
const int W = 1e6;
const int BLOCK = 500;
const int mod = 1e9 + 7;

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

vector<bool> np;
int n, m, ans = 1;
vector<vector<int> > tot;
vector<int> a, p, k, invk, maxp, mul, cnt, out;

struct Query
{
    int l, r, id;
    Query(int _l, int _r, int _id)
    {
        l = _l;
        r = _r;
        id = _id;
    }
    bool operator<(const Query& y) const
    {
        return l / BLOCK == y.l / BLOCK ? ((l / BLOCK) & 1 ? r > y.r : r < y.r) : l < y.l;
    }
};

vector<Query> q;

void add(int x)
{
    if (maxp[x] >= B)
    {
        if (++cnt[maxp[x]] == 1)
        {
            ans = (ll) ans * k[maxp[x]] % mod;
        }
    }
}

void del(int x)
{
    if (maxp[x] >= B)
    {
        if (--cnt[maxp[x]] == 0)
        {
            ans = (ll) ans * invk[maxp[x]] % mod;
        }
    }
}

int main()
{
    n = read();

    np.resize(W + 1, false);
    maxp.resize(W + 1);

    for (int i = 2; i <= W; ++i)
    {
        if (!np[i])
        {
            maxp[i] = p.size();
            p.push_back(i);
            k.push_back((ll) (i - 1) * qpow(i, mod - 2) % mod);
            invk.push_back(qpow(k.back(), mod - 2));
        }
        for (int j = 0; j < p.size() && i * p[j] <= W; ++j)
        {
            int x = i * p[j];
            np[x] = true;
            maxp[x] = max(j, maxp[i]);
            if (i % p[j] == 0) break;
        }
    }

    a.resize(n + 1);
    mul.resize(n + 1, 1);
    cnt.resize(p.size(), 0);
    tot.resize(B, vector<int>(n + 1, 0));

    for (int i = 1; i <= n; ++i)
    {
        a[i] = read();
        mul[i] = (ll) mul[i - 1] * a[i] % mod;
        for (int j = 0; j < B; ++j) tot[j][i] = tot[j][i - 1];
        for (int x = a[i]; x > 1; x /= p[maxp[x]]) if (maxp[x] < B) ++tot[maxp[x]][i];
    }

    m = read();

    out.resize(m);

    int l, r;

    for (int i = 0; i < m; ++i)
    {
        l = read();
        r = read();
        q.push_back(Query(l, r, i));
    }

    sort(q.begin(), q.end());

    l = 1, r = 0;

    for (int i = 0; i < m; ++i)
    {
        while (l > q[i].l) add(a[--l]);
        while (r < q[i].r) add(a[++r]);
        while (l < q[i].l) del(a[l++]);
        while (r > q[i].r) del(a[r--]);
        int tmp = (ll) ans * mul[q[i].r] % mod * qpow(mul[q[i].l - 1], mod - 2) % mod;
        for (int j = 0; j < B; ++j) if (tot[j][q[i].r] - tot[j][q[i].l - 1]) tmp = (ll) tmp * k[j] % mod;
        out[q[i].id] = tmp;
    }

    for (int i = 0; i < m; ++i) printf("%d\n", out[i]);

    return 0;
}
```

{{% /admonition %}}

{{% admonition note "树状数组做法" true %}}

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <cctype>
#include <algorithm>

using namespace std;

typedef long long ll;
typedef pair<int, int> pii;

int read()
{
    int out = 0;
    char c;
    while (!isdigit(c = getchar()));
    for (; isdigit(c); c = getchar()) out = out * 10 + c - '0';
    return out;
}

const int mod = 1e9 + 7;
const int W = 1e6;

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

struct BIT
{
    vector<int> A;

    void resize(int size)
    {
        A.resize(size + 1, 1);
    }

    void add(int p, int x)
    {
        for (; p < A.size(); p += (p & -p))
        {
            A[p] = (ll) A[p] * x % mod;
        }
    }

    int query(int p)
    {
        int out = 1;
        for (; p; p -= (p & -p)) out = (ll) out * A[p] % mod;
        return out;
    }
} bit;

vector<vector<pii> > q;

int n, m;
vector<bool> np;
vector<int> a, mul, p, minp, nxt, k, invk, pre, out;

int main()
{
    np.resize(W + 1);
    nxt.resize(W + 1);
    minp.resize(W + 1);

    for (int i = 2; i <= W; ++i)
    {
        if (!np[i])
        {
            nxt[i] = 1;
            minp[i] = p.size();
            p.push_back(i);
            k.push_back((ll) (i - 1) * qpow(i, mod - 2) % mod);
            invk.push_back(qpow(k.back(), mod - 2));
        }
        for (int j = 0; j < p.size() && i * p[j] <= W; ++j)
        {
            int x = i * p[j];
            np[x] = true;
            minp[x] = j;
            if (i % p[j]) nxt[x] = i;
            else
            {
                nxt[x] = nxt[i];
                break;
            }
        }
    }

    n = read();

    a.resize(n + 1);
    mul.resize(n + 1, 1);

    for (int i = 1; i <= n; ++i)
    {
        a[i] = read();
        mul[i] = (ll) mul[i - 1] * a[i] % mod;
    }

    m = read();

    q.resize(n + 1);

    for (int i = 0; i < m; ++i)
    {
        int l = read();
        int r = read();
        q[r].push_back(pii(l, i));
    }

    pre.resize(p.size(), 0);
    out.resize(m);
    bit.resize(n);

    for (int i = 1; i <= n; ++i)
    {
        for (int x = a[i]; x > 1; x = nxt[x])
        {
            if (pre[minp[x]]) bit.add(pre[minp[x]], invk[minp[x]]);
            bit.add(pre[minp[x]] = i, k[minp[x]]);
        }
        int tmp = (ll) bit.query(i) * mul[i] % mod;
        for (auto x : q[i]) out[x.second] = (ll) tmp * qpow((ll) bit.query(x.first - 1) * mul[x.first - 1] % mod, mod - 2) % mod;
    }

    for (int i = 0; i < m; ++i) printf("%d\n", out[i]);

    return 0;
}
```

{{% /admonition %}}
