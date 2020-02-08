+++
title = "CF901C Bipartite Segments（二分图）"
categories = ["题解"]
tags = ["二分图"]
date = "2019-11-29T20:54:11+08:00"
description = ""
aliases = ["/post/CF901C-Bipartite-Segments（二分图）", "/CF901C-Bipartite-Segments（二分图）"]
+++


## 题目链接

[CF](https://codeforces.com/contest/901/problem/D)

## 题意简述

定义一个“偶环”为边数为偶数的回路（回路又被称作“边简单环”，即不经过重复边且首尾点相同的途径）。

给你一张不含“偶环”的无向图。称一个区间是“好的”，当且仅当编号在这个区间中的点的导出子图是一张二分图。

多组询问，每次询问一个给定区间有多少个子区间是“好的”。

点数、边数、询问数均不超过 $3\cdot 10^5$ 。

<!--more-->

## 简要做法

由于图中没有偶环，容易想到一个子图是二分图等价于没有环（因为二分图等价于没有奇环）。

但“没有偶环”的用处远不止此：没有偶环还意味着每个点最多处于一个环内（否则取两个包含同一个点的奇环，把重复的边去掉，就可以得到一个偶环）。

那么，找出图中的每个环的最小节点编号和最大节点编号，以最小值和最大值作为左右端点，就可以得到若干条线段。而一个区间合法就等价于其不包含这些线段中的任意一条。

现在问题已经转化成了：给你 $k$ 条线段 $\\{[c_i, d_i]\\}_{i=1}^k$，$q$ 次询问，第 $i$ 次询问给你一个区间 $[l_i, r_i]$，求 $[l_i, r_i]$ 有多少个子区间不包含这 $k$ 条线段中的任意一条。

首先，可以观察到，若有一条线段被另一条线段完全包含，包含它的这条线段就可以被无视了。

去除掉上述无用线段后，剩下的线段两两互不包含，那么将它们按左端点排序，会满足：$\forall 1\le i< k, c_i< c_{i+1}, d_i< d_{i+1}$ （不取等号是因为 $1\sim n$ 中的每个数在 $c_{1..n}$ 和 $d_{1..n}$ 中最多只出现一次，而这是因为在原题意中每个点最多处于一个环内）。 

如果不管 $r_i$ 的限制，对于每个左端点求答案，画个图手玩一下可以发现，以 $p$ 为左端点时，合法区间的右端点最多到 $\min\left(\\{d_j-1|c_j\ge p\\}\bigcup\\{n\\}\right)$ ，即 $p$ “右边”的第一条线段的右端点减一（不存在则为 $n$）。每个左端点无视右端点限制的合法区间数可以求个前缀和，而 $p$ “右边”的第一条线段可以二分求得，也可以线性预处理后 $O(1)$ 求得。那么，我们就会做 $r_i=n$ 的情况了。

然后考虑如何处理 $r_i$ 的限制。对称地（指这个式子和上面那个式子几乎是对称的），令 $t=\max\left(\\{c_j|d_j\le r_i\\}\bigcup\\\{0\\\}\right)$，即 $r_i$ “左边”的第一条线段的左端点（不存在则为 $0$），那么对于不小于 $t$ 的左端点，$r_i$ 这个限制是无用的，而对于大于 $t$ 的左端点，右端点最多可以取到 $r_i$ 。 $t$ 同样可以二分求得或者线性预处理后 $O(1)$ 求得，将左端点分成 $[l_i, t]$ 和 $[t+1, r_i]$ 两部分，前半部分用前缀和计算，后半部分即为 $[t+1, r_i]$ 的子区间数，然后就做完了。实现时注意特判 $t< l_i$ 的情况。

## 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <algorithm>
 
using namespace std;
 
typedef long long ll;
typedef pair<int, int> pii;
 
vector<int> pa, dfn, rt, uncut;
vector<vector<int> > g;
int n, m, q, dfntot;
vector<ll> pre;
vector<pii> s;
 
void dfs(int u)
{
	dfn[u] = ++dfntot;
	for (auto v : g[u])
	{
		if (v == pa[u]) continue;
		if (!dfn[v])
		{
			pa[v] = u;
			dfs(v);
		}
		else if (dfn[v] < dfn[u])
		{
			int mn = u, mx = u, x = u;
			while (x != v)
			{
				x = pa[x];
				mn = min(mn, x);
				mx = max(mx, x);
			}
			rt[mn] = mx;
		}
	}
}
 
int main()
{
	scanf("%d%d", &n, &m);
	
	g.resize(n + 1);
	pa.resize(n + 1, 0);
	pre.resize(n + 1, 0);
	dfn.resize(n + 1, 0);
	uncut.resize(n + 1, 0);
	rt.resize(n + 1, n + 1);
	
	for (int i = 1; i <= m; ++i)
	{
		int u, v;
		scanf("%d%d", &u, &v);
		g[u].push_back(v);
		g[v].push_back(u);
	}
	
	for (int i = 1; i <= n; ++i) if (!dfn[i]) dfs(i);
	
	int mn = n;
	for (int i = n; i >= 1; --i)
	{
		mn = min(mn, rt[i]);
		if (rt[i] <= mn) s.push_back(pii(i, rt[i]));
	}
	
	reverse(s.begin(), s.end());
	s.push_back(pii(n + 1, n + 1));
	
	int p = 0, ban = s[0].second;
	for (int i = 1; i <= n; ++i)
	{
		if (i > s[p].first) ban = s[++p].second;
		pre[i] = pre[i - 1] + ban - i;
		uncut[i] = uncut[i - 1] + (i == s[uncut[i - 1]].second);
	}
	
	scanf("%d", &q);
	
	while (q--)
	{
		int l, r;
		scanf("%d%d", &l, &r);
		int k = uncut[r];
		if (k == 0 || s[k - 1].first < l) printf("%I64d\n", (ll) (r - l + 1) * (r - l + 2) / 2);
		else printf("%I64d\n", pre[s[k - 1].first] - pre[l - 1] + (ll) (r - s[k - 1].first) * (r - s[k - 1].first + 1) / 2);
	}
	
	return 0;
}
```
