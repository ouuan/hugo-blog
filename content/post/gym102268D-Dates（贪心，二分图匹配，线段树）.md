+++
title = "gym102268D Dates（贪心，二分图匹配，线段树）"
categories = ["题解"]
tags = ["贪心", "图论", "二分图", "二分图匹配", "数据结构", "线段树"]
date = "2019-10-11T11:30:32+08:00"
description = ""
aliases = ["/post/gym102268D-Dates（贪心，二分图匹配，线段树）", "/gym102268D-Dates（贪心，二分图匹配，线段树）"]
+++


## 题目链接

[CF](https://codeforces.com/gym/102268/problem/D)

## 题意简述

给你一张二分图：左边 $t$ 个位置，第 $i$ 个位置上有 $a_i$ 个点；右边 $n$ 个带权的点，第 $i$ 个点与位置在 $[l_i, r_i]$ 之间的所有左边的点有连边；匹配权值为匹配中右边点的权值之和；求最大权匹配。

$1\le n,t\le 3\cdot 10^5$，保证 $l_i\le l_{i+1}$, $r_i\le r_{i+1}$。

<!--more-->

## 简要做法

首先，将右边的点按权值从大到小排序，依次加入，看有没有完全匹配，有就选这个点。这样一定是最优的，好像可以用拟阵相关的理论证明，但我不太会..

于是，问题转化成了如何判定是否存在完全匹配，而霍尔定理恰恰是用来做这件事的——考虑右边的点中被选择的那些，选择其一个子集，判断是否所有子集的邻域（即与其相邻的点构成的集合）大小都比子集本身大。

如果选择的子集中元素对应的区间的并集不是连续的，霍尔定理的条件成立等价于对于断点两边分别成立，所以只需要考虑子集对应的区间连续的情况。

又由于 $l_i\le l_{i+1}$, $r_i\le r_{i+1}$，只用考虑子集中的元素本身编号连续的情况。那么，霍尔定理的条件就可以表示为：

$$
\forall 1\le i< j\le n,[i,j]\text{中被选择的右侧点个数}\le [l_i,r_j]\text{中左侧点数量}
$$

如果处理出 $a_{1..t}$ 的前缀和 $pre[i]=\sum_{j=1}^ia_j$，用 $p[i]$ 表示 $[1,i]$ 中被选择的右侧点个数，那么式子就变成了：

$$
\forall1\le i< j\le n, p[j]-p[i-1]\le pre[r_j]-pre[l_i-1]
$$

也就是：

$$
\forall1\le i< j\le n, pre[l_i-1]-p[i-1]\le pre[r_j]-p[j]
$$

所以，可以对每个元素 $i$ 维护 $pre[l_{i+1}-1]-p[i]$ 以及 $pre[r_j]-p[j]$。

$pre$ 是定值，考虑如何更新 $p$。事实上，往已选择的点中加入一个点，就是把一段后缀的 $p$ 加一。所以可以考虑用线段树维护。

并且，一段后缀加一（令这段后缀为 $[x..n]$）后，只有 $i< x,j\ge x$ 的数对 $(i,j)$ 对应的大小关系发生改变，事实上只用判断 $i<x$ 的 $pre[l_i-1]-p[i-1]$ 的最大值与 $j\ge x$ 的 $pre[r_j]-p[j]$ 的最小值的大小关系即可，同样可以使用线段树维护。

另一种判断方法，是在线段树上的每个节点处判断左儿子与右儿子有没有出现不满足霍尔定理条件的情况。

## 参考代码

代码中使用了 [segmenttree.h](https://github.com/ouuan/segmentTree)。

### 每次判断前缀与后缀的大小关系

```cpp
#include <iostream>
#include <cstdio>
#include <cctype>
#include <algorithm>
#include "segmenttree.h"

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

const int N = 300010;
const ll INF = 1e18;

int n, m;
ll pre[N], ans;

struct Girl
{
    int l, r, p, id;
    bool operator<(const Girl& y) const
    {
        return p > y.p;
    }
} g[N];

struct Value
{
    ll mn, mx;
    Value(ll _mn = INF, ll _mx = -INF)
    {
        mn = _mn;
        mx = _mx;
    }
};
vector<Value> a;

Value merge(Value x, Value y)
{
    return Value(min(x.mn, y.mn), max(x.mx, y.mx));
}

void update(segmentTreeNode<Value, int>& u, int x)
{
    u.val.mx += x;
    u.val.mn += x;
    u.tag += x;
}

int main()
{
    n = read();
    m = read();

    for (int i = 1; i <= m; ++i) pre[i] = pre[i - 1] + read();

    a.resize(n + 1);

    for (int i = 1; i <= n; ++i)
    {
        g[i].l = read();
        g[i].r = read();
        g[i].p = read();
        g[i].id = i;
        a[i].mn = pre[g[i].r];
        a[i - 1].mx = pre[g[i].l - 1];
    }

    sort(g + 1, g + n + 1);

    segmentTree<Value, int, merge, update> t(0, n + 1, a, Value());

    for (int i = 1; i <= n; ++i)
    {
        if (t.query(0, g[i].id).mx >= t.query(g[i].id, n + 1).mn) continue;
        t.modify(g[i].id, n + 1, -1);
        ans += g[i].p;
    }

    cout << ans;

    return 0;
}
```

### 在线段树的每个节点处判断

```cpp
#include <iostream>
#include <cstdio>
#include <cctype>
#include <algorithm>
#include "segmenttree.h"

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

const int N = 300010;
const ll INF = 1e18;

int n, m;
ll pre[N], ans;

struct Girl
{
    int l, r, p, id;
    bool operator<(const Girl& y) const
    {
        return p > y.p;
    }
} g[N];

struct Value
{
    ll mn, mx;
    bool inv;
    Value(ll _mn = INF, ll _mx = -INF, int _inv = false)
    {
        mn = _mn;
        mx = _mx;
        inv = _inv;
    }
};
vector<Value> a;

Value merge(Value x, Value y)
{
    return Value(min(x.mn, y.mn), max(x.mx, y.mx), x.inv || y.inv || x.mx > y.mn);
}

void update(segmentTreeNode<Value, int>& u, int x)
{
    u.val.mx += x;
    u.val.mn += x;
    u.tag += x;
}

int main()
{
    n = read();
    m = read();

    for (int i = 1; i <= m; ++i) pre[i] = pre[i - 1] + read();

    a.resize(n + 1);

    for (int i = 1; i <= n; ++i)
    {
        g[i].l = read();
        g[i].r = read();
        g[i].p = read();
        g[i].id = i;
        a[i].mn = pre[g[i].r];
        a[i - 1].mx = pre[g[i].l - 1];
    }

    sort(g + 1, g + n + 1);

    segmentTree<Value, int, merge, update> t(0, n + 1, a, Value());

    for (int i = 1; i <= n; ++i)
    {
        t.modify(g[i].id, n + 1, -1);
        if (t.query(0, n + 1).inv) t.modify(g[i].id, n + 1, 1);
        else ans += g[i].p;
    }

    cout << ans;

    return 0;
}
```

