+++
title = '带修改的树上 DP 问题（动态 DP）'
date = 2020-05-07T20:58:54+08:00
draft = false
categories = ['知识点']
tags = ['动态DP', '树形DP', '线段树', 'LCT', '数据结构']
+++

“动态 DP”通过树链剖分将带修改的树上 DP 问题拆分为规模更小的序列上的半群合并问题和树上 DP 问题，从而可以快速地支持修改。

<!--more-->

（这一篇中我会尝试将形式化的问题通式和具体的例题一并进行讲解。）

## 需解决的问题

### 概述

一个树上 DP 问题，需要支持单点修改，单点询问 DP 值。

### 形式化描述

给你一棵有根树，树上的每个点有 **权值** 与 **DP 值** 两个属性，其中 **权值** 是题目给定的，**DP 值** 是你需要计算的。

令点 $u$ 的权值为 $a_u$，DP 值为 $d_u$。题目给定了树的形态，每个点的初始权值，以及 DP 值的计算方法：$d_u=f(a_u, d_{v_1}, d_{v_2}, \ldots)$，其中 $v_i$ 是 $u$ 的儿子节点，$f$ 是一个接受一个权值和多个 DP 值，返回一个 DP 值的函数，每个 $d_{v_i}$ 对 $f$ 的贡献都是独立的，且可以快速地计算加入/删除一个 $d_{v_i}$ 后的 $f$.

现在，你需要执行若干次操作，每次操作为以下之一：

1. 给定一个点和一个权值，将这个点的权值修改为给定的权值。
2. 给定一个点，求此时这个点的 DP 值。

额外地，如果将计算 $f(a_1, f(a_2, f(a_3, \ldots)))$ 视作一连串二元运算的话，应当可以将权值和 DP 值映射到一个半群来进行这样的运算，也就是说这样的运算满足结合律。并且，这样的运算应当能够快速进行。

### 例题

{{% question "洛谷 P4719" "https://www.luogu.com.cn/problem/P4719" %}}
给你一棵由 $n$ 个点构成的，点带权的树。$m$ 次修改单点权值，每次修改后你需要求出整棵树的最大权独立集的权值和。

“独立集”是给定的树点集的一个子集，其中任意两点在树上不相邻。

$1\le n, m\le 10^5$。
{{% /question %}}

## 在链上解决问题

如果树是一条链，那么我们要做的其实就是求一个后缀的权值合并起来的结果，支持单点修改。

在序列上，这样的问题往往可以通过线段树来解决。使用线段树来动态维护后缀和，我们需要能够将两段相邻区间的信息进行合并，这一点是由题目性质所保证的。

以例题为例：我们可以维护一段区间开头选/不选，结尾选/不选共四种情况各自的最大权独立集权值和，那么，在合并后，两端是否选择保持不变，中间只有两段都不选或其中一段选才能合并。也就是说：

用 $l_{i, j}$ ($i, j\in\\{0, 1\\}$) 表示左半段开头是否选择为 $i$，结尾是否选择为 $j$ 时的最大权独立集权值和，右半段用 $r_{i, j}$，合并后用 $c_{i, j}$ 类似地表示，那么：

$$
c_{i, j} = \max(l_{i, 0} + r_{0, j}, l_{i, 0} + r_{1, j}, l_{i, 1} + r_{0, j})
$$

这样的话，使用线段树来维护信息并，用这个公式来合并相邻两段，我们就在树是一条链时解决了带修改的树上 DP 问题。实际上，我们可以快速地求出并更新链上任何一个区间的信息并。

## 解决原问题

当链上问题可以解决时，我们往往可以使用树链剖分来将问题推广到树上。

注意，这里的“树链剖分”指任意的一种剖分，不一定是“重链剖分”。但为了叙述方便，依然使用“重链”“轻链”“重儿子”“轻儿子”等术语。

当我们将树剖分成若干条链后，我们可以快速获得任意一条重链的一个区间的信息并。我们需要做的，其实就是将轻儿子所在重链的信息合并上来。

我们在进行链与链的信息合并时，要求两条链是首尾相接的，所以直接使用链与链的信息合并来把轻儿子所在重链的信息合并上来是行不通的。但不要忘了原本的题目：树形 DP。我们除了可以将首尾相接的两条链合并，还能将一个点与它的若干子树合并。于是，我们可以将一个点的轻儿子所在重链的信息合并到这个点上，计算出这个点与其所有轻子树的并，等效为一个点。

还是以例题为例：用 $a_u$ 代指 $u$ 的权值，$c_{v_i, 0/1, 0/1}$ 代指 $v_i$（$u$ 的一个轻儿子）所属的整条重链的信息并，那么：

在计算 $u$ 所属的重链的信息并时，令 $[u, u]$ 这一段的信息并为 $c_{u, 0/1, 0/1}$，那么：

- $c_{u, 0, 1} = c_{u, 1, 0} = -\infty$（一个足够小的数）
- $c_{u, 1, 1} = a_u + \sum \max(c_{v_i, 0, 0}, c_{v_i, 0, 1})$
- $c_{u, 0, 0} = \sum \max(c_{v_i, 0, 0}, c_{v_i, 0, 1}, c_{v_i, 1, 0}, c_{v_i, 1, 1})$

这样的话，我们就可以将轻链的信息合并到重链上。并且，轻子树的贡献是独立的，所以可以快速地更新一个轻子树的信息。此时，一个点到其所在重链的底端的信息并就是这个点的 DP 值，也就是说，这样的一段链实际上是整个子树的信息并。

为了快速地进行这样的计算，可以：（时间复杂度是在信息合并复杂度为 $O(1)$ 的情况下）

1. 使用重链剖分，每次单点修改时不断向上将当前所在重链合并到当前所在重链顶端的父亲。（$O(n\log n+m\log^2n)$）
2. 使用 LCT 来维护实链剖分，用类似维护子树信息的方式来维护轻儿子的信息。（$O((n + m)\log n))$，常数略大）
3. 使用全局平衡二叉树。（$O((n + m)\log n))$，常数较小）

## 例题参考代码

{{% fold 树链剖分 %}}
（使用了 [CPTH](https://github.com/ouuan/CPTH)）

（常数有点大，被迫在 `Val` 里使用数组而非 `vector`）

```cpp
#ifndef CPTH_SEGMENTTREE
#define CPTH_SEGMENTTREE
#include<cassert>
#include<climits>
#include<functional>
#include<vector>
namespace CPTH{template<typename valueType,typename modType>struct SegmentTreeNode{std::size_t id;long long left,right;valueType val;modType mod;};template<typename valueType,typename modType,bool elementModificationOnly=false>class SegmentTree{public:SegmentTree()=default;SegmentTree(const std::vector<valueType>&_initValue,std::function<valueType(const valueType&,const valueType&)>_merge,std::function<void(SegmentTreeNode<valueType,modType>&,const modType&)>_update,long long _startPoint=1,const valueType&_valueZero=valueType(),const modType&_modZero=modType());void init(const std::vector<valueType>&_initValue,std::function<valueType(const valueType&,const valueType&)>_merge,std::function<void(SegmentTreeNode<valueType,modType>&,const modType&)>_update,long long _startPoint=1,const valueType&_valueZero=valueType(),const modType&_modZero=modType());void modify(long long l,long long r,const modType&mod);void modify(long long p,const modType&mod);valueType query(long long l,long long r);valueType query(long long p);private:void pushup(std::size_t cur);void pushdown(std::size_t cur);void build(std::size_t cur,long long l,long long r,const std::vector<valueType>&initValue);void m_init(const std::vector<valueType>&_initValue,std::function<valueType(const valueType&,const valueType&)>_merge,std::function<void(SegmentTreeNode<valueType,modType>&,const modType&)>_update,const valueType&_valueZero,const modType&_modZero);void modify(std::size_t cur,long long l,long long r,long long L,long long R,const modType&mod);valueType query(std::size_t cur,long long l,long long r,long long L,long long R);std::function<valueType(const valueType&,const valueType&)>merge;std::function<void(SegmentTreeNode<valueType,modType>&,const modType&)>update;std::vector<SegmentTreeNode<valueType,modType>>nodes;long long leftRange=0,rightRange=0;valueType valueZero;modType modZero;};template<typename valueType,typename modType,bool elementModificationOnly>SegmentTree<valueType,modType,elementModificationOnly>::SegmentTree(const std::vector<valueType>&_initValue,std::function<valueType(const valueType&,const valueType&)>_merge,std::function<void(SegmentTreeNode<valueType,modType>&,const modType&)>_update,long long _startPoint,const valueType&_valueZero,const modType&_modZero){init(_initValue,_merge,_update,_startPoint,_valueZero,_modZero);}template<typename valueType,typename modType,bool elementModificationOnly>void SegmentTree<valueType,modType,elementModificationOnly>::init(const std::vector<valueType>&_initValue,std::function<valueType(const valueType&,const valueType&)>_merge,std::function<void(SegmentTreeNode<valueType,modType>&,const modType&)>_update,long long _startPoint,const valueType&_valueZero,const modType&_modZero){assert(_startPoint>=LLONG_MIN/2);assert(_startPoint<=LLONG_MAX/2-(long long)_initValue.size());leftRange=_startPoint;rightRange=_startPoint+_initValue.size();m_init(_initValue,_merge,_update,_valueZero,_modZero);}template<typename valueType,typename modType,bool elementModificationOnly>void SegmentTree<valueType,modType,elementModificationOnly>::modify(long long l,long long r,const modType&mod){assert(!elementModificationOnly);modify(1,leftRange,rightRange,l,r,mod);}template<typename valueType,typename modType,bool elementModificationOnly>void SegmentTree<valueType,modType,elementModificationOnly>::modify(long long p,const modType&mod){assert(p<LLONG_MAX);modify(1,leftRange,rightRange,p,p+1,mod);}template<typename valueType,typename modType,bool elementModificationOnly>valueType SegmentTree<valueType,modType,elementModificationOnly>::query(long long l,long long r){return query(1,leftRange,rightRange,l,r);}template<typename valueType,typename modType,bool elementModificationOnly>valueType SegmentTree<valueType,modType,elementModificationOnly>::query(long long p){return query(p,p+1);}template<typename valueType,typename modType,bool elementModificationOnly>void SegmentTree<valueType,modType,elementModificationOnly>::pushup(std::size_t cur){nodes[cur].val=merge(nodes[cur<<1].val,nodes[cur<<1|1].val);}template<typename valueType,typename modType,bool elementModificationOnly>void SegmentTree<valueType,modType,elementModificationOnly>::pushdown(std::size_t cur){update(nodes[cur<<1],nodes[cur].mod);update(nodes[cur<<1|1],nodes[cur].mod);nodes[cur].mod=modZero;}template<typename valueType,typename modType,bool elementModificationOnly>void SegmentTree<valueType,modType,elementModificationOnly>::build(std::size_t cur,long long l,long long r,const std::vector<valueType>&initValue){nodes[cur].id=cur;nodes[cur].left=l;nodes[cur].right=r;nodes[cur].mod=modZero;if(l==r-1)nodes[cur].val=initValue[l-leftRange];else{build(cur<<1,l,(l+r)>>1,initValue);build(cur<<1|1,(l+r)>>1,r,initValue);pushup(cur);}}template<typename valueType,typename modType,bool elementModificationOnly>void SegmentTree<valueType,modType,elementModificationOnly>::m_init(const std::vector<valueType>&_initValue,std::function<valueType(const valueType&,const valueType&)>_merge,std::function<void(SegmentTreeNode<valueType,modType>&,const modType&)>_update,const valueType&_valueZero,const modType&_modZero){merge=_merge;update=_update;valueZero=_valueZero;modZero=_modZero;nodes.resize((rightRange-leftRange)<<2);build(1,leftRange,rightRange,_initValue);}template<typename valueType,typename modType,bool elementModificationOnly>void SegmentTree<valueType,modType,elementModificationOnly>::modify(std::size_t cur,long long l,long long r,long long L,long long R,const modType&mod){if(l>=R||r<=L)return;if(L<=l&&r<=R)update(nodes[cur],mod);else{if(!elementModificationOnly)pushdown(cur);modify(cur<<1,l,(l+r)>>1,L,R,mod);modify(cur<<1|1,(l+r)>>1,r,L,R,mod);pushup(cur);}}template<typename valueType,typename modType,bool elementModificationOnly>valueType SegmentTree<valueType,modType,elementModificationOnly>::query(std::size_t cur,long long l,long long r,long long L,long long R){if(l>=R||r<=L)return valueZero;if(L<=l&&r<=R)return nodes[cur].val;if(!elementModificationOnly)pushdown(cur);return merge(query(cur<<1,l,(l+r)>>1,L,R),query(cur<<1|1,(l+r)>>1,r,L,R));}}
#endif

#ifndef CPTH_HLD
#define CPTH_HLD
#include<algorithm>
#include<cassert>
#include<vector>
namespace CPTH{class HLD{public:HLD(std::size_t size=0);void reset(std::size_t size);void addEdge(std::size_t u,std::size_t v);void build(std::size_t root=1);std::size_t lca(std::size_t u,std::size_t v)const;std::size_t id(std::size_t u)const;std::size_t atId(std::size_t x)const;std::size_t top(std::size_t u)const;std::size_t bottom(std::size_t u)const;std::size_t parent(std::size_t u)const;std::vector<std::size_t>children(std::size_t u)const;std::pair<std::size_t,std::size_t>subtree(std::size_t u)const;std::vector<std::pair<std::size_t,std::size_t>>path(std::size_t u,std::size_t v)const;private:void dfs1(std::size_t u);void dfs2(std::size_t u);void validate(std::size_t u)const;private:bool built;std::size_t n,dfntot;std::vector<std::vector<std::size_t>>g;std::vector<std::size_t>pa,heavy,dep,siz,tp,bt,dfn,rdfn,exi;};HLD::HLD(std::size_t size){reset(size);}void HLD::reset(std::size_t size){assert(size<g.max_size());assert(size<pa.max_size());built=false;n=size;dfntot=0;g.clear();g.resize(n+1);pa.assign(n+1,0);heavy=dep=siz=tp=bt=dfn=rdfn=exi=pa;}void HLD::addEdge(std::size_t u,std::size_t v){validate(u);validate(v);g[u].push_back(v);g[v].push_back(u);}void HLD::build(std::size_t root){if(built)return;assert(n>=1);validate(root);dfs1(root);tp[root]=root;dfs2(root);assert(dfntot==n);built=true;}std::size_t HLD::lca(std::size_t u,std::size_t v)const{assert(built);validate(u);validate(v);while(tp[u]!=tp[v]){if(dep[tp[u]]>dep[tp[v]])u=pa[tp[u]];else v=pa[tp[v]];}return dep[u]<dep[v]?u:v;}std::size_t HLD::id(std::size_t u)const{assert(built);validate(u);return dfn[u];}std::size_t HLD::atId(std::size_t x)const{assert(built);validate(x);return rdfn[x];}std::size_t HLD::top(std::size_t u)const{assert(built);validate(u);return tp[u];}std::size_t HLD::bottom(std::size_t u)const{assert(built);validate(u);return bt[tp[u]];}std::size_t HLD::parent(std::size_t u)const{assert(built);validate(u);return pa[u];}std::vector<std::size_t>HLD::children(std::size_t u)const{auto ret=g[u];if(pa[u])ret.erase(std::find(ret.begin(),ret.end(),pa[u]));return ret;}std::pair<std::size_t,std::size_t>HLD::subtree(std::size_t u)const{assert(built);validate(u);return{dfn[u],exi[u]};}std::vector<std::pair<std::size_t,std::size_t>>HLD::path(std::size_t u,std::size_t v)const{assert(built);validate(u);validate(v);std::vector<std::pair<std::size_t,std::size_t>>res;while(tp[u]!=tp[v]){if(dep[tp[u]]>dep[tp[v]]){res.emplace_back(dfn[tp[u]],dfn[u]);u=pa[tp[u]];}else{res.emplace_back(dfn[tp[v]],dfn[v]);v=pa[tp[v]];}}if(dep[u]>dep[v])std::swap(u,v);res.emplace_back(dfn[u],dfn[v]);return res;}void HLD::dfs1(std::size_t u){assert(siz[u]==0);siz[u]=1;for(auto v:g[u]){if(v==pa[u])continue;dep[v]=dep[u]+1;pa[v]=u;dfs1(v);siz[u]+=siz[v];if(siz[v]>siz[heavy[u]])heavy[u]=v;}}void HLD::dfs2(std::size_t u){bt[tp[u]]=u;dfn[u]=++dfntot;rdfn[dfn[u]]=u;if(heavy[u]){tp[heavy[u]]=tp[u];dfs2(heavy[u]);for(auto v:g[u]){if(v==pa[u]||v==heavy[u])continue;tp[v]=v;dfs2(v);}}exi[u]=dfntot;}void HLD::validate(std::size_t u)const{assert(u>=1);assert(u<=n);}}
#endif

#include <iostream>
#include <cstdio>
#include <cctype>
#include <set>
#include <map>

int read()
{
	int out = 0, f = 1;
	char c = getchar();
	while (!isdigit(c) && c != '-') c = getchar();
	if (c == '-')
	{
		f = -1;
		c = getchar();
	}
	for (; isdigit(c); c = getchar()) out = out * 10 + c - '0';
	return out * f;
}

using namespace CPTH;
using namespace std;

typedef vector<int> vi;
typedef vector<vi> vvi;

const int INF = 1e9;

struct Val
{
	int a[2][2];
	
	Val()
	{
		a[0][0] = a[1][1] = -INF;
		a[0][1] = a[1][0] = 0;
	}
	
	int *operator[](int x) { return a[x]; }
	const int *operator[](int x) const { return a[x]; }
	
	Val operator+(const Val &b) const
	{
		Val ret;
		for (int i = 0; i < 2; ++i)
		{
			for (int j = 0; j < 2; ++j)
			{
				ret[i][j] = max({a[i][0] + b[0][j], a[i][0] + b[1][j], a[i][1] + b[0][j]});
			}
		}
		return ret;
	}
	
	int ans()
	{
		return max({a[0][0], a[0][1], a[1][0], a[1][1]});
	}
};

struct SingleVal : public Val // 自行搜索：C++ 类继承
{
	int self, ch0, ch1;
	
	explicit SingleVal(int x = 0) : Val(), self(x), ch0(0), ch1(0) { update(); }
	
	void update()
	{
		a[0][0] = ch1;
		a[1][1] = self + ch0;
		a[0][1] = a[1][0] = -INF;
	}
	
	void modify(int x)
	{
		self = x;
		update();
	}
	
	void add(const Val &x)
	{
		ch0 += max(x[0][0], x[0][1]);
		ch1 += max({x[0][0], x[0][1], x[1][0], x[1][1]});
		update();
	}
	
	void del(const Val &x)
	{
		ch0 -= max(x[0][0], x[0][1]);
		ch1 -= max({x[0][0], x[0][1], x[1][0], x[1][1]});
		update();
	}
};

int main()
{
	int n = read();
	int m = read();
	
	vector<SingleVal> a(n + 1);
	vector<Val> init(n), b(n + 1);
	
	for (int i = 1; i <= n; ++i) a[i] = SingleVal(read());
	
	HLD hld(n);
	
	for (int i = 1; i < n; ++i) hld.addEdge(read(), read());
	
	hld.build();
	
	for (int i = 1; i <= n; ++i) init[i - 1] = a[hld.atId(i)];
	
	SegmentTree<Val, bool, true> seg(
		init,
		plus<Val>(),
		[&](SegmentTreeNode<Val, bool> &u, bool x) { if (x) u.val = a[hld.atId(u.left)]; }
	);
	
	for (int i = n; i >= 2; --i)
	{
		int u = hld.atId(i);
		if (u == (int)hld.top(u))
		{
			b[u] = seg.query(i, hld.id(hld.bottom(u)) + 1);
			a[hld.parent(u)].add(b[u]);
			seg.modify(hld.id(hld.parent(u)), true);
		}
	}
	
	while (m--)
	{
		int x = read();
		int y = read();
		a[x].modify(y);
		seg.modify(hld.id(x), true);
		while (1)
		{
			x = hld.top(x);
			y = hld.parent(x);
			if (!y) break;
			a[y].del(b[x]);
			b[x] = seg.query(hld.id(x), hld.id(hld.bottom(x)) + 1);
			a[y].add(b[x]);
			seg.modify(hld.id(y), true);
			x = y;
		}
		printf("%d\n", seg.query(1, hld.id(hld.bottom(1)) + 1).ans());
	}
	
	return 0;
}
```
{{% /fold %}}

{{% fold LCT %}}
```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <cctype>
#include <algorithm>
#include <functional>

using namespace std;

int read()
{
	int out = 0, f = 1;
	char c = getchar();
	while (!isdigit(c) && c != '-') c = getchar();
	if (c == '-')
	{
		f = -1;
		c = getchar();
	}
	for (; isdigit(c); c = getchar()) out = out * 10 + c - '0';
	return out * f;
}

const int INF = 1e9;

typedef vector<int> vi;

struct Val
{
	int a[2][2];
	
	Val()
	{
		a[0][0] = a[1][1] = -INF;
		a[0][1] = a[1][0] = 0;
	}
	
	int *operator[](int x) { return a[x]; }
	const int *operator[](int x) const { return a[x]; }
	
	Val operator+(const Val &b) const
	{
		Val ret;
		for (int i = 0; i < 2; ++i)
		{
			for (int j = 0; j < 2; ++j)
			{
				ret[i][j] = max({a[i][0] + b[0][j], a[i][0] + b[1][j], a[i][1] + b[0][j]});
			}
		}
		return ret;
	}
	
	int ans()
	{
		return max({a[0][0], a[0][1], a[1][0], a[1][1]});
	}
};

struct SingleVal : public Val // 自行搜索：C++ 类继承
{
	int self, ch0, ch1;
	
	explicit SingleVal(int x = 0) : Val(), self(x), ch0(0), ch1(0) { update(); }
	
	void update()
	{
		a[0][0] = ch1;
		a[1][1] = self + ch0;
		a[0][1] = a[1][0] = -INF;
	}
	
	void modify(int x)
	{
		self = x;
		update();
	}
	
	void add(const Val &x)
	{
		ch0 += max(x[0][0], x[0][1]);
		ch1 += max({x[0][0], x[0][1], x[1][0], x[1][1]});
		update();
	}
	
	void del(const Val &x)
	{
		ch0 -= max(x[0][0], x[0][1]);
		ch1 -= max({x[0][0], x[0][1], x[1][0], x[1][1]});
		update();
	}
};

struct LCT
{
	struct Node
	{
		vi ch;
		int pa;
		Val sum;
		SingleVal self;
		Node() : ch(2), pa(0), sum(), self() {}
	};
	
	vector<Node> t;
	
	LCT(int n) : t(n + 1) {}
	
	bool nroot(int x)
	{
		return x == t[t[x].pa].ch[0] || x == t[t[x].pa].ch[1];
	}
	
	void pushup(int x)
	{
		t[x].sum = t[t[x].ch[0]].sum + t[x].self + t[t[x].ch[1]].sum;
	}
	
	void rotate(int x)
	{
		int y = t[x].pa;
		int z = t[y].pa;
		int k = x == t[y].ch[1];
		if (nroot(y)) t[z].ch[y == t[z].ch[1]] = x;
		t[x].pa = z;
		t[y].ch[k] = t[x].ch[k ^ 1];
		t[t[x].ch[k ^ 1]].pa = y;
		t[x].ch[k ^ 1] = y;
		t[y].pa = x;
		pushup(y);
		pushup(x);
	}
	
	void Splay(int x)
	{
		while (nroot(x))
		{
			int y = t[x].pa;
			int z = t[y].pa;
			if (nroot(y)) rotate((x == t[y].ch[1]) ^ (y == t[z].ch[1]) ? x : y);
			rotate(x);
		}
	}
	
	void access(int x)
	{
		for (int y = 0; x; x = t[y = x].pa)
		{
			Splay(x);
			if (y) t[x].self.del(t[y].sum);
			if (t[x].ch[1]) t[x].self.add(t[t[x].ch[1]].sum);
			t[x].ch[1] = y;
			pushup(x);
		}
	}
	
	void link(int u, int pa)
	{
		// 为了只用从轻儿子向上更新一次，link 时两段都要 access, Splay
		access(u);
		Splay(u);
		access(pa);
		Splay(pa);
		t[u].pa = pa;
		t[pa].self.add(t[u].sum);
		pushup(pa);
	}
	
	void modify(int u, int x)
	{
		access(u);
		Splay(u);
		t[u].self.modify(x);
		pushup(u);
	}
	
	int query()
	{
		Splay(1);
		return max({t[1].sum[0][0], t[1].sum[0][1], t[1].sum[1][0], t[1].sum[1][1]});
	}
};

int main()
{
	int n = read();
	int m = read();
	
	LCT lct(n);
	
	for (int i = 1; i <= n; ++i) lct.modify(i, read());
	
	vector<vi> g(n + 1);
	
	for (int i = 1; i < n; ++i)
	{
		int u = read();
		int v = read();
		g[u].push_back(v);
		g[v].push_back(u);
	}
	
	function<void(int, int)> dfs = [&](int u, int pa)
	{
		for (auto v : g[u])
		{
			if (v == pa) continue;
			lct.link(v, u);
			dfs(v, u);
		}
	};
	
	dfs(1, 0);
	
	while (m--)
	{
		int u = read();
		int x = read();
		lct.modify(u, x);
		printf("%d\n", lct.query());
	}
	
	return 0;
}
```
{{% /fold %}}

## 其它题目

[BZOJ5210 最大连通子块和](http://www.lydsy.com/JudgeOnline/problem.php?id=5210)<font color="white">，感觉能用线段树维护最大子段和那一套（根本不需要什么“广义矩阵”，会序列问题就能做的啊..），写了下过了样例，但只在 BZOJ 找到了这道题，~~一不小心~~ 用了 C++11，懒得改成 C++98 了，就没交。感觉挺对的，就算挂了也应该是写挂了..</font>

{{% fold 说句闲话 %}}
为啥其他人的博客讲动态 DP 上来就一个矩阵啊..凭啥想到可以用矩阵转移啊..根本就不需要矩阵的..随便来个半群就好了..精髓根本不在矩阵吧..能转化成矩阵的形式而已..
{{% /fold %}}
