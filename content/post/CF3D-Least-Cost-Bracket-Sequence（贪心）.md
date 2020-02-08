+++
title = "CF3D Least Cost Bracket Sequence（贪心）"
categories = ["题解"]
tags = ["贪心"]
date = "2019-10-16T21:50:44+08:00"
description = ""
aliases = ["/post/CF3D-Least-Cost-Bracket-Sequence（贪心）", "/CF3D-Least-Cost-Bracket-Sequence（贪心）"]
+++


## 题目链接

[CF](https://codeforces.com/contest/3/problem/D)

## 题意简述

有一个长为 $n$ 的括号序列，其中一些位置是问号，每个问号替换成左括号或替换成右括号各有给定的代价，判断是否能够构造出一个合法的括号序列，如果可以，求出最小代价。

$n\le 5\cdot 10^4$（实际上可以大很多）。

<!--more-->

## 简要做法

考虑使用带反悔的贪心。

如果用 $cnt[i]$ 表示 $\sum_{j=1}^i(-1)^{[a_i=')']}$（左括号比右括号多的个数），括号序列合法当且仅当 $\forall i,cnt[i]\ge 0$ 且 $cnt[n]=0$。

如果把右括号反悔成左括号，一定可以保证 $cnt[i]\ge 0$ 这个条件依然满足。

但是，如果把左括号反悔成右括号，有可能造成本来 $cnt[i]\ge 0$ 的位置小于 $0$。

并且，如果选择了多余的左括号，还会导致 $cnt[n]>0$。

如何解决这些问题呢？

可以发现，如果初始时优先选择右括号，上述问题就都得到解决了。

即，每次碰到问号都选右括号，并且将其标记为可以反悔为左括号。如果 $cnt[i]<0$，就从可反悔的右括号里选反悔代价最小的改成左括号。这样的话，$cnt[i]\ge 0$ 不会因反悔而被破坏，$cnt[n]>0$ 也不会在有解时发生。

具体实现可以用堆（`priority_queue`）。

## 参考代码

```cpp
#include <iostream>
#include <string>
#include <queue>
#include <cstring>
#include <algorithm>
 
using namespace std;
 
typedef pair<int, int> pii;
 
const int N = 50010;
 
int n;
char s[N];
long long ans;
priority_queue<pii, vector<pii>, greater<pii> > q;
 
int main()
{
	scanf("%s", s + 1);
	n = strlen(s + 1);
	
	int cnt = 0;
	
	for (int i = 1; i <= n; ++i)
	{
		if (s[i] == '(') ++cnt;
		else if (s[i] == ')') --cnt;
		else
		{
			int a, b;
			scanf("%d%d", &a, &b);
			--cnt;
			s[i] = ')';
			ans += b;
			q.push(pii(a - b, i));
		}
		if (cnt < 0)
		{
			if (q.empty()) break;
			cnt += 2;
			ans += q.top().first;
			s[q.top().second] = '(';
			q.pop();
		}
	}
	
	if (cnt == 0) printf("%I64d\n%s", ans, s + 1);
	else puts("-1");
	
	return 0;
}
```

