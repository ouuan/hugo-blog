+++
title = "整体二分学习笔记"
date = 2020-02-14T14:50:32+08:00
draft = false
categories = ['知识点']
tags = ['整体二分', '离线算法', '分治']
+++

整体二分是一种离线算法，可以将一个修改同时作用于多个询问，从而减少不必要的开销，将二分答案从单次询问扩展到多次询问。

在一些题目中，相比与其它解法，整体二分可以避免复杂的数据结构，降低代码难度与空间复杂度。

<!--more-->

## 解决的问题

### 概述

给你多组修改与询问，其中每个修改有一个可二分的指标，每个询问有一个目标，你需要对每个询问回答：应用了指标不超过多少的所有修改后，这个询问的目标可以达到，或者应用了所有修改后该目标依然无法达到。并且，你可以在得知所有修改和询问后开始回答，即允许离线。

### 形式化描述

**指标集合** 与 **指标比较** 构成了一个有限 [全序集](https://baike.baidu.com/item/%E5%85%A8%E5%BA%8F%E9%9B%86/9689969) 。其中指标集合用 $\mathbf{INDEX}$ 表示，指标比较用 $\le$ 表示。

**贡献集合** 和 **贡献累加** 构成了一个 [交换幺半群](https://baike.baidu.com/item/%E5%B9%BA%E5%8D%8A%E7%BE%A4#3_1) 。其中贡献集合用 $\mathbf{CONTRIBUTION}$ 表示，贡献累加用 $+$ 表示，也可用 $\Sigma$ 表示多个贡献累加。

一个 **修改** $m$ 具有 **指标** $m.index$ 和 **贡献** $m.contribution$ 两个属性，其中指标属于指标集合，贡献属于贡献集合。

一个 **询问** $q$ 包含一个 **目标函数** $q.f:\mathbf{CONTRIBUTION}\to\\{0, 1\\}$（输入是一个贡献，输出是 $0$ 或 $1$），满足 $\forall x, y\in\mathbf{CONTRIBUTION}, q.f(x)=1\implies q.f(x+y)=1$（这条性质保证了可二分性）。

那么，一个可以被整体二分解决的问题可以表述如下：

{{% question %}}
1. 给定修改集合 $M$ 和询问集合 $Q$，
2. 对于每个 $q\in Q$，求出最小的 $x\in\mathbf{INDEX}$ 使得 $q.f(\sum_{m\in M}[m.index\le x]m.contribution)=1$，或指出这样的 $x$ 不存在。（其中 $[m.index\le x]$ 是 [艾弗森括号](https://baike.baidu.com/item/%E8%89%BE%E4%BD%9B%E6%A3%AE%E6%8B%AC%E5%8F%B7/22361197) 。）
{{% /question %}}

## 对传统二分的分析

如果询问只有一个，使用 [二分答案](https://oi-wiki.org/basic/binary/) 即可求解。

如果对每次询问单独使用二分答案，效率将十分低下，究其原因，在于“计算修改对询问的贡献”这一步有大量重复的操作，冗余的计算没有得到有效的合并。

因此，整体二分的优化重点就在于“将多个操作同时作用于多个询问”。

## 算法框架

### 概述

1. 递归地处理指标/答案在一个区间内的修改和询问。
2. 每次计算出左半的修改对所有询问的贡献。
3. 依次判断每个询问在加上来自左半修改的贡献后是否满足了要求。
4. 按修改的指标，询问是否在加上来自左半修改的贡献后满足了要求，将修改和询问划分成两半，继续递归下去。
5. 递归在指标区间大小为 $1$ 时终止。

### 伪代码 1

这份伪代码中对每个询问维护一个 $q.current$ 属性，表示这个询问已经加上了多少贡献（在实际代码实现中 $q.current$ 往往不必和“贡献”是同一类型的，如“贡献”可能用一个数据结构维护，而“满足了多少”可以简单地用一个数表示）。

$$
\begin{array}{rl}
1&\textbf{function}\text{ PARALLEL_BINARY_SEARCH}(l, r, M, Q)\\\\
2&\qquad\textbf{if }Q=\varnothing\\\\
3&\qquad\qquad\textbf{return}\\\\
4&\qquad \textbf{if }l=r\\\\
5&\qquad\qquad \text{The answer of all queries in }Q\text{ is }l\\\\
6&\qquad\qquad \textbf{return}\\\\
7&\qquad mid\gets \text{middle of }l\text{ and }r\\\\
8&\qquad \mathrm{LM}\gets\varnothing\\\\
9&\qquad \mathrm{RM}\gets\varnothing\\\\
10&\qquad sum\gets(\mathbf{CONTRIBUTION}, +)\text{ 的单位元}\\\\
11&\qquad \textbf{for each }m\in M\\\\
12&\qquad\qquad \textbf{if }m.index\le mid\\\\
13&\qquad\qquad\qquad sum\gets sum+m.contribution\\\\
14&\qquad\qquad\qquad \mathrm{LM}\gets \mathrm{LM}\bigcup\\{m\\}\\\\
15&\qquad\qquad \textbf{else}\\\\
16&\qquad\qquad\qquad \mathrm{RM}\gets \mathrm{RM}\bigcup\\{m\\}\\\\
17&\qquad \mathrm{LQ}\gets\varnothing\\\\
18&\qquad \mathrm{RQ}\gets\varnothing\\\\
19&\qquad \textbf{for each }q\in Q\\\\
20&\qquad\qquad \textbf{if }q.f(q.current+sum)=1\\\\
21&\qquad\qquad\qquad \mathrm{LQ}\gets \mathrm{LQ}\bigcup \\{q\\}\\\\
22&\qquad\qquad \textbf{else}\\\\
23&\qquad\qquad\qquad q.current\gets q.current+sum\\\\
24&\qquad\qquad\qquad \mathrm{RQ}\gets \mathrm{RQ}\bigcup \\{q\\}\\\\
25&\qquad \text{PARALLEL_BINARY_SEARCH}(l, mid, \mathrm{LM}, \mathrm{LQ})\\\\
26&\qquad \text{PARALLEL_BINARY_SEARCH}(mid\text{ 的后继}, r, \mathrm{RM}, \mathrm{RQ})\\\\
27&\\\\
28&\textbf{function}\text{ SOLVE(M, Q)}\\\\
29&\qquad \textbf{for each }q\in Q\\\\
30&\qquad\qquad q.current\gets(\mathbf{CONTRIBUTION}, +)\text{ 的单位元}\\\\
31&\qquad \text{PARALLEL_BINARY_SEARCH}(\min\\{\mathbf{INDEX}\\}, \max\\{\mathbf{INDEX}\\}, M, Q)
\end{array}
$$

### 伪代码 2

这份伪代码维护了一个全局的贡献，每次加上左半修改的贡献后，先递归解决右半部分，然后将这些修改回退，再去解决左半部分。

$$
\begin{array}{rl}
1&current\gets(\mathbf{CONTRIBUTION}, +)\text{ 的单位元}\\\\
2&\\\\
3&\textbf{function}\text{ PARALLEL_BINARY_SEARCH}(l, r, M, Q)\\\\
4&\qquad\textbf{if }Q=\varnothing\\\\
5&\qquad\qquad\textbf{return}\\\\
6&\qquad \textbf{if }l=r\\\\
7&\qquad\qquad \text{The answer of all queries in }Q\text{ is }l\\\\
8&\qquad\qquad \textbf{return}\\\\
9&\qquad mid\gets \text{middle of }l\text{ and }r\\\\
10&\qquad \mathrm{LM}\gets\varnothing\\\\
11&\qquad \mathrm{RM}\gets\varnothing\\\\
12&\qquad old\gets current\\\\
13&\qquad \textbf{for each }m\in M\\\\
14&\qquad\qquad \textbf{if }m.index\le mid\\\\
15&\qquad\qquad\qquad current\gets current+m.contribution\\\\
16&\qquad\qquad\qquad \mathrm{LM}\gets \mathrm{LM}\bigcup\\{m\\}\\\\
17&\qquad\qquad \textbf{else}\\\\
18&\qquad\qquad\qquad \mathrm{RM}\gets \mathrm{RM}\bigcup\\{m\\}\\\\
19&\qquad \mathrm{LQ}\gets\varnothing\\\\
20&\qquad \mathrm{RQ}\gets\varnothing\\\\
21&\qquad \textbf{for each }q\in Q\\\\
22&\qquad\qquad \textbf{if }q.f(current)=1\\\\
23&\qquad\qquad\qquad \mathrm{LQ}\gets \mathrm{LQ}\bigcup \\{q\\}\\\\
24&\qquad\qquad \textbf{else}\\\\
25&\qquad\qquad\qquad \mathrm{RQ}\gets \mathrm{RQ}\bigcup \\{q\\}\\\\
26&\qquad \text{PARALLEL_BINARY_SEARCH}(mid\text{ 的后继}, r, \mathrm{RM}, \mathrm{RQ})\\\\
27&\qquad current\gets old\\\\
28&\qquad \text{PARALLEL_BINARY_SEARCH}(l, mid, \mathrm{LM}, \mathrm{LQ})\\\\
29&\\\\
30&\textbf{function}\text{ SOLVE(M, Q)}\\\\
31&\qquad \text{PARALLEL_BINARY_SEARCH}(\min\\{\mathbf{INDEX}\\}, \max\\{\mathbf{INDEX}\\}, M, Q)
\end{array}
$$

## 算法分析

整体二分算法的核心在于：

1. 整体地计算多个修改对多个询问的贡献。
2. 伪代码 1 的 23 行处 / 伪代码 2 的先处理右半部分再回退左半修改，将左半修改对右半询问的贡献应用到整个处理右半部分的过程中，之后不再重复计算。

令计算两个贡献之和的时间复杂度为 $O(f(\cdots))$，判定一个贡献是否达到一个询问的目标的时间复杂度为 $O(g(\cdots))$，那么总的时间复杂度为:（"$\cdots$" 表示会对复杂度造成影响的各种因素）

$$O(((|M|+|Q|)\cdot f(\cdots)+|Q|\cdot g(\cdots))\cdot\log|\mathbf{INDEX}|)$$

{{% fold "为什么两侧操作个数分布不均而复杂度依然正确？" %}}
虽然操作划分到两侧后可能一侧操作多另一侧操作少，但指标的分布是均匀的，从而保证了分治的层数是 $O(\log|\mathbf{INDEX}|)$，而每一层的操作总数是固定的，总的复杂度就是正确的。
{{% /fold %}}

{{% warning %}}
这里的时间复杂度没有计算预处理的时间复杂度，也就是说，一般情况下，在整体二分的函数中，除继续递归下去的部分，复杂度应该是当前的修改数加上操作数，可能再乘上若干个 log。而复杂度与总操作个数/值域大小多项式相关的步骤（如，初始化数据结构）应该在主函数中完成，在二分过程中，如果需要清空数据结构，应该使用一些快速的清空方法，如将之前的操作回退、使用时间戳清空等。
{{% /warning %}}

## 例题

上面给出的是算法抽象的框架，实际代码中，不需要真的将“贡献”对象化，而应该采用高效的方式（数据结构）去计算修改对询问的贡献。

（注：下面的例题往往可以通过可持久化数据结构、数据结构嵌套来在线地处理，这里只介绍整体二分的解法。）

### 静态区间第 k 小

{{% question %}}
给你一个长为 $n$ 的数列 $a_1, a_2, \ldots, a_n$ 以及 $m$ 组询问，每次询问一个区间 $[l, r]$，求 $\{a_l, a_{l+1}, \ldots, a_r\}$ 中第 $k$ 小的数。
{{% /question %}}

在这个问题中，套用上面的形式化定义，我们可以这样看待：

- **贡献**：一个长为 $n$ 的数组，每个位置上的数为 $1$ 表示这个位置“符合要求”（小于等于某个数），否则表示这个位置“不符合要求”。
- **指标**：一个数的值。
- **修改**：一个修改代表着数组上一个位置的值，如，$a_2=7$ 对应着这样的一个修改：贡献是将下标 $2$ 加一，指标是 $7$。
- **询问的目标**：贡献的 $[l, r]$ 之和不小于 $k$ 。

通过整体二分，我们可以将问题转化成：单点加、区间求和，所有单点加都在区间求和之前，允许离线，要求复杂度与当前操作数相关而非与总操作数 $n$, $m$ 相关。

一个容易想到的做法是使用树状数组来维护，使用时间戳清空。

{{% fold "怎样使用时间戳清空数组？" %}}
维护一个时间戳 `tim` 以及一个数组 `vis`，清空时 `++tim`，访问数组的下标 `i` 前检查 `vis[i]` 是否等于 `tim`，若否就将 `vis[i]` 设为 `tim` 并清空 `vis[i]` 。
{{% /fold %}}

其实，这里还有一个额外的性质：所有修改均在询问前。这样的话，就可以通过计算前缀和来求解。但先计算前缀和再查询复杂度会和 $n$ 相关，先离散化再计算前缀和复杂度会多一个 log。其实还有一种处理方法：如果修改和询问的位置有序，可以在扫一遍的过程中维护前缀和并查询。如果每次都排序的话复杂度会多一个 log，但我们可以只在主函数中进行一遍排序，在整体二分的过程中保持相对顺序不变，这样的话总复杂度就是 $O((n+m)\log n)$（如果值域大可以离散化）。

{{% fold "参考代码" %}}
```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <algorithm>

using namespace std;

struct Operation
{
	int x, id, type;
	Operation(int _x, int _id = -1, int _type = 0) : x(_x), id(_id), type(_type) {}
};

vector<int> ans, cur, tmp;

#define mid ((l + r) >> 1)

void solve(int l, int r, vector<Operation> &ops) // 这里 l 和 r 表示左闭右开区间 [l, r)
{
	if (l == r - 1)
	{
		for (auto op : ops)
		{
			if (op.type) ans[op.id] = l;
		}
		return;
	}
	int sum = 0;
	for (auto op : ops)
	{
		if (op.type) tmp[op.id] += sum * op.type; // 使用 tmp 记录一个询问受到的贡献总和
		else if (op.x < mid) ++sum;
	}
	vector<Operation> lop, rop;
	for (auto op : ops)
	{
		if (op.type)
		{
			if (cur[op.id] + tmp[op.id] >= op.x) lop.push_back(op);
			else
			{
				if (op.type == 1) cur[op.id] += tmp[op.id]; // 在被拆成两个的询问中的后一个处更新 current
				rop.push_back(op);
			}
			if (op.type == 1) tmp[op.id] = 0; // 在被拆成两个的询问中的后一个处清空 tmp
		}
		else // 由于这里操作之间的顺序是重要的，修改和询问要按原序划分到左右两边，而不能先划分修改再划分询问
		{
			if (op.x < mid) lop.push_back(op);
			else rop.push_back(op);
		}
	}
	vector<Operation>().swap(ops); // 释放空间，否则空间复杂度带 log
	solve(l, mid, lop);
	solve(mid, r, rop);
}

int main()
{
	int n, m;
	
	scanf("%d%d", &n, &m);
	
	vector<int> a(n), lsh(n);
	
	for (int i = 0; i < n; ++i)
	{
		scanf("%d", &a[i]);
		lsh[i] = a[i];
	}
	
	sort(lsh.begin(), lsh.end());
	lsh.resize(unique(lsh.begin(), lsh.end()) - lsh.begin()); // 离散化
	
	vector<vector<Operation> > bucket(n); // 桶排序
	
	for (int i = 0; i < n; ++i) bucket[i].emplace_back(lower_bound(lsh.begin(), lsh.end(), a[i]) - lsh.begin());
	
	for (int i = 0; i < m; ++i)
	{
		int l, r, x;
		scanf("%d%d%d", &l, &r, &x);
		if (l > 1) bucket[l - 2].emplace_back(x, i, -1);
		bucket[r - 1].emplace_back(x, i, 1);
	}
	
	vector<Operation> ops;
	
	for (auto i : bucket)
	{
		for (auto j : i)
		{
			ops.push_back(j);
		}
	}
	
	ans.resize(m, 0);
	cur = tmp = ans;
	
	solve(0, lsh.size(), ops);
	
	for (auto x : ans) printf("%d\n", lsh[x]);
	
	return 0;
}
```
{{% /fold %}}

### [「ZJOI2013」K大数查询](https://www.luogu.com.cn/problem/P3332)

{{% question %}}
给你一个由 $n$ 个初始为空的可重集构成的序列，需要支持两种操作：

1. 向一段区间 $[l, r]$ 内的每一个可重集内加入一个数 $c$ 。
2. 查询一段区间 $[l, r]$ 内的所有可重集并起来（不去重）后第 $c$ 大的数。
{{% /question %}}

经过整体二分后问题转化为：区间加，区间求和，要求复杂度与当前操作数相关。

这次没有修改在询问之前的性质，所以没什么可优化的地方，复杂度与当前操作数相关有很多种方式：

1. 动态开点线段树。
2. 离散化+线段树。
3. 树状数组+时间戳清空。

{{% fold "参考代码" %}}
```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <algorithm>

using namespace std;

typedef long long ll;

struct Operation
{
	int l, r, id;
	ll target, current, tmp;
	Operation(int _l, int _r, int _id, ll _target) : l(_l), r(_r), id(_id), target(_target), current(0) {}
};

struct BIT
{
	int tim;
	vector<ll> a;
	vector<int> vis;
	
	BIT() : tim(0) {}
	
	void resize(int size)
	{
		a.resize(size + 1, 0);
		vis.resize(size + 1, 0);
	}
	
	void modify(int p, int x)
	{
		for (; p < (int)a.size(); p += (p & -p))
		{
			if (vis[p] != tim)
			{
				vis[p] = tim;
				a[p] = 0;
			}
			a[p] += x;
		}
	}
	
	ll query(int p)
	{
		ll out = 0;
		for (; p; p -= (p & -p))
		{
			if (vis[p] == tim) out += a[p];
		}
		return out;
	}
	
	void clear() { ++tim; }
} bitk, bitb;

void modify(int p, int x)
{
	bitk.modify(p, x);
	bitb.modify(p, (1 - p) * x);
}

ll query(int p)
{
	return bitk.query(p) * p + bitb.query(p);
}

void clear()
{
	bitk.clear();
	bitb.clear();
}

vector<int> ans;

#define mid ((l + r) >> 1)

void solve(int l, int r, vector<Operation> &ops)
{
	if (l == r - 1)
	{
		for (auto op : ops)
		{
			if (~op.id) ans[op.id] = l;
		}
		return;
	}
	clear();
	for (auto &op : ops)
	{
		if (~op.id) op.tmp = query(op.r) - query(op.l - 1);
		else if (op.target < mid)
		{
			modify(op.l, 1);
			modify(op.r + 1, -1);
		}
	}
	vector<Operation> lop, rop;
	for (auto op : ops)
	{
		if (op.id == -1)
		{
			if (op.target < mid) lop.push_back(op);
			else rop.push_back(op);
		}
		else
		{
			if (op.current + op.tmp >= op.target) lop.push_back(op);
			else
			{
				op.current += op.tmp;
				rop.push_back(op);
			}
		}
	}
	vector<Operation>().swap(ops);
	solve(l, mid, lop);
	solve(mid, r, rop);
}

int main()
{
	int n, m;
	
	scanf("%d%d", &n, &m);
	
	int qid = 0;
	vector<Operation> ops;
	
	for (int i = 0; i < m; ++i)
	{
		int op, l, r;
		ll x;
		scanf("%d%d%d%lld", &op, &l, &r, &x);
		if (op == 1) ops.emplace_back(l, r, -1, -x); // 取负将第 k 大转化为第 k 小
		else ops.emplace_back(l, r, qid++, x);
	}
	
	ans.resize(qid);
	bitk.resize(n);
	bitb.resize(n);
	solve(-n, n + 1, ops);
	
	for (auto x : ans) printf("%d\n", -x);
	
	return 0;
}
```
{{% /fold %}}

## 练习

- [「POI2011」Meteors](https://loj.ac/problem/2169)
- [矩阵乘法](https://www.luogu.com.cn/problem/P1527)
- [「HNOI2016」网络](https://loj.ac/problem/2049)
