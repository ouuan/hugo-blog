+++
title = "CF17E Palisection（manacher）"
categories = ["题解"]
tags = ["字符串", "manacher"]
date = "2019-05-10T22:06:18+08:00"
description = ""
aliases = ["/post/CF17E-Palisection（manacher）", "/CF17E-Palisection（manacher）"]
+++


## 题目链接

[洛谷](https://www.luogu.org/problemnew/show/CF17E)

[CF](https://codeforces.com/contest/17/problem/E)

## 题意简述

给你一个字符串，求有多少对相交的回文子串。（包含算作相交，~~自交不算~~）

字符串长度小于等于 $2\times 10^6$。

<!--more-->

## 简要做法

首先 manacher 求出每个中心的最长回文串半径。

然后，通过差分可以求出每个位置作为左端点 / 右端点各有多少个回文串（知道每个中心的半径之后就相当于区间加），记为 $l_i$, $r_i$。

最后，我们把问题转化为求不相交的回文子串对数，这样的话就是 $\sum\limits_{i = 2}^n\sum\limits_{j=1}^{i-1}l_ir_j$，预处理一下 $r​$ 的前缀和就可以算了。

## 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <algorithm>

using namespace std;

typedef long long ll;

const int N = 2000010;
const int mod = 51123987;

char s[N << 1];
int n, p[N << 1], mid, rt, l[N], r[N], pre[N], ans;

int main()
{
	int i;
	
	scanf("%d%s", &n, s + 1);
	
	s[0] = '$';
	for (i = n * 2 + 1; i >= 1; --i)
	{
		if (i & 1) s[i] = '#';
		else s[i] = s[i / 2];
	}
	
	mid = rt = 1;
	for (i = 2; i <= n * 2 + 1; ++i)
	{
		p[i] = min(rt - i, p[mid * 2 - i]);
		while (s[i + p[i] + 1] == s[i - p[i] - 1]) ++p[i];
		if (i + p[i] > rt)
		{
			mid = i;
			rt = i + p[i];
		}
		++l[(i - p[i] + 1) >> 1];
		--l[(i >> 1) + 1];
		++r[(i + 1) >> 1];
		--r[(i + p[i] + 1) >> 1];
		ans = (ans + (p[i] + 1) / 2) % mod;
	}
	
	ans = (ll) ans * (ans - 1) % mod * (mod + 1) / 2 % mod;
	
	for (i = 1; i <= n; ++i)
	{
		l[i] += l[i - 1];
		r[i] += r[i - 1];
		pre[i] = (pre[i - 1] + r[i]) % mod;
		ans = (ans - (ll) l[i] * pre[i - 1] % mod + mod) % mod;
	}
	
	cout << ans;
	
	return 0;
}
```

