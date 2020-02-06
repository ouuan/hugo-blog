+++
title = "CF1208G Polygons（数论）"
categories = ["题解"]
tags = ["数论"]
date = "2019-08-27T20:08:56+08:00"
description = ""
+++


## 题目链接

[CF](http://codeforces.com/contest/1208/problem/G)

[洛谷](https://www.luogu.org/problem/CF1208G)

## 题意简述

给定 $n$ 和 $k$，你需要在圆上画 $k$ 个不超过 $n$ 条边的正多边形，求顶点去重后至少有多少个。

$3\le n\le10^6$，$1\le k\le n-2$。

<!--more-->

## 简要做法

1. 所有正多边形至少有一个公共顶点。可以感性理解，也可以看 [imp 的评论](https://codeforces.com/blog/entry/69357?#comment-538545)。
	
2. 选了 $x$ 边形就选了 $x$ 的所有约数（除了 $1$ 和 $2$）边形一定最优，因为选约数相当于是免费的。

那么，我们可以把 $x$ 边形的第 $y$ 个顶点看成分数 $\dfrac y x$，这样的话，在已经选了 $x$ 的所有约数的前提下，选 $x$ 边形的代价就是 $\varphi(x)$，问题就变成了求最小的 $k$ 个 $\varphi$ 之和。

但是，一边形和二边形是不存在的，需要特殊考虑。

“一边形”其实就是那个所有正多边形的公共顶点，只需要在计算答案时加一即可。

“二边形”会且仅会影响偶数边形，相当于“一旦选了某个偶数边形，答案加一”。因为 $\varphi(x)=1$ 的 $x$ 只有 $1$ 和 $2$， 而 $\varphi(x)=2$ 的 $x$ 只有 $3$, $4$, $6$，所以只有仅选择正三角形这种情况会受到影响。特判 $k=1$ 输出 $3$ 即可。

用线性筛 + nth_element（值域不大，其实也可以线性排序）即可做到 $O(n)$。

## 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <algorithm>
 
using namespace std;

const int N = 1000010;
 
int n, k, p[N], tot, phi[N];
bool np[N];
 
int main()
{
	cin >> n >> k;
	
	if (k == 1)
	{
		puts("3");
		return 0;
	}
	
	phi[1] = 1;
	
	for (int i = 2; i <= n; ++i)
	{
		if (!np[i])
		{
			p[++tot] = i;
			phi[i] = i - 1;
		}
		for (int j = 1; j <= tot && i * p[j] <= n; ++j)
		{
			int x = i * p[j];
			np[x] = true;
			if (i % p[j]) phi[x] = phi[i] * (p[j] - 1);
			else
			{
				phi[x] = phi[i] * p[j];
				break;
			}
		}
	}
	
	nth_element(phi + 1, phi + k + 3, phi + n + 1); // 选了最小的 k+2 个，其中前两个是“一边形”和“二边形”的代价
	
	long long ans = 0;
	
	for (int i = 1; i <= k + 2; ++i) ans += phi[i];
	
	cout << ans;
	
	return 0;
}
```