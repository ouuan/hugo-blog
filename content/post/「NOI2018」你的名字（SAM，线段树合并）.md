+++
title = "「NOI2018」你的名字（SAM，线段树合并）"
categories = ["题解"]
tags = ["SAM", "线段树合并", "字符串", "数据结构"]
date = "2020-01-07T18:33:42+08:00"
description = ""
aliases = ["/post/「NOI2018」你的名字（SAM，线段树合并）", "/「NOI2018」你的名字（SAM，线段树合并）"]
+++


## 题目链接

[UOJ](https://uoj.ac/problem/395)

[LOJ](https://loj.ac/problem/2720)

[洛谷](https://www.luogu.com.cn/problem/P4770)

## 题意简述

给你一个字符串 $S$，有 $q$ 次询问，每次给你询问串 $T$ 以及左右端点 $l$, $r$，询问 $T$ 有多少个 **本质不同** 的子串 **不是** $S[l..r]$ 的子串。

$|S|\le 5\cdot 10^5$, $\sum|T|\le 10^6$, $q\le 10^5$ 。

<!--more-->

## 简要做法

### 不需要本质不同、l=1、r=|S|

枚举 $T$ 的前缀，找到这个前缀的最长后缀使其是 $S$ 的子串，就可以求出答案了。

换句话说，对于每个 $i$，求出最小的 $j$ ($1\le j\le i+1$) 使得 $T[j..i]$ 是 $S$ 的子串，那么 $\sum j-1$ 就是答案。

可以对 $S$ 建 SAM，维护一个指针指向 $T[j..i]$ 在 SAM 上对应的顶点，如果拓展一个字符后不是 $S$ 的子串就把 $j$ 加一，如果低于当前 SAM 上节点的长度限制就跳 parent 边，也就是 [从首部删去字符](/后缀自动机（SAM）学习笔记/#读入字符串时删除首字符) 。

### 需要本质不同、l=1、r=|S|

由本质不同可以想到用 SAM 去重。具体来说，对 $T$ 建 SAM，对每个 $T[j..i]$ 在 SAM 上打个标记，表示 $T[j..i]$ 及其所有后缀（即对应顶点及其在 parent 树上的祖先）都是 $S$ 的子串，最后 DFS 一遍即可求出答案。

### 原问题

如果询问的不是整个 $S$ 而是其一个子串，关键在于如何判断 $T[j..i]$ 是不是 $S[l..r]$ 的子串。

可以使用线段树合并维护 $S$ 的 SAM 上的每个点的 right 集合，$T[j..i]$ 是 $S[l..r]$ 的子串等价于 $T[j..i]$ 在 $S$ 中的 right 集合含有某个区间内的元素，区间求和即可查询。

还有一个细节，在 $T$ 的 SAM 上打标记时，可能出现 SAM 上的一个节点对应的较短的那些串是 $S[l..r]$ 的子串，但较长的串不是。所以标记还需要记录不是 $S[l..r]$ 子串所需的长度。

## 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <vector>

using namespace std;

typedef long long ll;

struct SAM
{
	struct Node
	{
		int len, pa, tag;
		vector<int> ch;
		Node(int _len = 0, int _pa = -1, vector<int> _ch = vector<int>(26, 0)):
			len(_len), pa(_pa), tag(0), ch(_ch) {}
	};
	
	vector<Node> t;
	int p;
	vector<int> endp;
	vector<vector<int> > g;
	
	void extend(int x)
	{
		int np = t.size();
		t.emplace_back(t[p].len + 1);
		while (~p && !t[p].ch[x])
		{
			t[p].ch[x] = np;
			p = t[p].pa;
		}
		if (p == -1) t[np].pa = 0;
		else
		{
			int q = t[p].ch[x];
			if (t[q].len == t[p].len + 1) t[np].pa = q;
			else
			{
				int nq = t.size();
				t.emplace_back(t[p].len + 1, t[q].pa, t[q].ch);
				t[q].pa = t[np].pa = nq;
				while (~p && t[p].ch[x] == q)
				{
					t[p].ch[x] = nq;
					p = t[p].pa;
				}
			}
		}
		p = np;
	}
	
	SAM(const string& s): t(1), p(0)
	{
		t.reserve(s.size() * 2);
		endp.resize(s.size());
		for (int i = 0; i < s.size(); ++i)
		{
			extend(s[i] - 'a');
			endp[i] = p;
		}
		g.resize(t.size());
		for (int i = 1; i < t.size(); ++i) g[t[i].pa].push_back(i);
	}
	
	ll dfs(int u = 0)
	{
		ll out = 0;
		for (auto v : g[u])
		{
			if (!v) continue;
			out += dfs(v);
			t[u].tag = max(t[u].tag, t[v].tag);
		}
		if (u) out += max(0, t[u].len - max(t[u].tag, t[t[u].pa].len));
		return out;
	}
};

struct SegmentTree
{
#define mid ((l + r) >> 1)

	struct Node
	{
		int sum, ls, rs;
		Node(int _sum = 0, int _ls = 0, int _rs = 0): sum(_sum), ls(_ls), rs(_rs) {}
	};
	
	vector<Node> t;
	
	int add(int x, int p, int l, int r)
	{
		int cur = t.size();
		t.push_back(t[x]);
		++t[cur].sum;
		if (l == r - 1) return cur;
		if (p < mid) t[cur].ls = add(t[x].ls, p, l, mid);
		else t[cur].rs = add(t[x].rs, p, mid, r);
		return cur;
	}
	
	int merge(int x, int y)
	{
		if (!x || !y) return x | y;
		t.emplace_back(t[x].sum + t[y].sum, merge(t[x].ls, t[y].ls), merge(t[x].rs, t[y].rs));
		return t.size() - 1;
	}
	
	int query(int x, int L, int R, int l, int r)
	{
		if (L >= r || R <= l) return 0;
		if (L <= l && r <= R) return t[x].sum;
		return query(t[x].ls, L, R, l, mid) + query(t[x].rs, L, R, mid, r);
	}

#undef mid
} seg;

vector<int> rt;

void dfs(const vector<vector<int> >& g, int u = 0)
{
	for (auto v : g[u])
	{
		dfs(g, v);
		rt[u] = seg.merge(rt[u], rt[v]);
	}
}

int main()
{
	ios_base::sync_with_stdio(false);
	cin.tie(0);
	cout.tie(0);

	string s;
	cin >> s;
	SAM sams(s);
	
	seg.t.resize(1);
	seg.t.reserve(s.size() << 5);
	
	rt.resize(sams.t.size(), 0);
	
	for (int i = 0; i < s.size(); ++i) rt[sams.endp[i]] = seg.add(rt[sams.endp[i]], i, 0, s.size());
	dfs(sams.g);
	
	int q;
	cin >> q;
	
	while (q--)
	{
		int l, r;
		string t;
		cin >> t >> l >> r;
		
		SAM samt(t);
		
		for (int i = 0, u = 0, v = 0, p = 0; i < t.size(); ++i)
		{
			while (p < i && !sams.t[u].ch[t[i] - 'a'])
			{
				++p;
				if (u && i - p <= sams.t[sams.t[u].pa].len) u = sams.t[u].pa;
				if (v && i - p <= samt.t[samt.t[v].pa].len) v = samt.t[v].pa;
			}
			if (!sams.t[u].ch[t[i] - 'a'])
			{
				p = i + 1;
				u = v = 0;
				continue;
			}
			u = sams.t[u].ch[t[i] - 'a'];
			v = samt.t[v].ch[t[i] - 'a'];
			while (p <= i && seg.query(rt[u], l + i - p - 1, r, 0, s.size()) == 0)
			{
				++p;
				if (u && i - p + 1 <= sams.t[sams.t[u].pa].len) u = sams.t[u].pa;
				if (v && i - p + 1 <= samt.t[samt.t[v].pa].len) v = samt.t[v].pa;
			}
			samt.t[v].tag = max(samt.t[v].tag, i - p + 1);
		}
		
		cout << samt.dfs() << '\n';
	}
	
	return 0;
}
```
