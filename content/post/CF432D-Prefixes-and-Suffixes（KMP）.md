+++
title = "CF432D Prefixes and Suffixes（KMP）"
categories = ["题解"]
tags = ["字符串", "KMP"]
date = "2019-05-10T21:56:54+08:00"
description = ""
+++


## 题目链接

[洛谷](https://www.luogu.org/problemnew/show/CF432D)

[CF](https://codeforces.com/contest/432/problem/D)

## 题意简述

给你一个字符串，分别求出每个长度的既是其前缀又是其后缀的串出现的次数。

字符串长度小于等于 $10^5$。

<!--more-->

## 简要做法

第一眼，SAM 裸题。

然后意识到只用管前缀，KMP 可以达到同样的效果。

做法和 SAM 完全一样：从末尾跳 next 来找到所有前后缀相同的位置（也就是 SAM 的接受状态），然后倒序枚举 $f_{next[i]}+=f_i$（也就是 SAM 的统计子树和）。

多说两句：fail、parent 和 next 其实是一样的东西，但 KMP / AC 自动机的状态是所有前缀，SAM 的状态是所有子串（并被压缩为了 right 集合等价类）。

## 参考代码

```cpp
#include <iostream>
#include <cstring>
#include <cstdio>

using namespace std;

const int N = 100010;

char s[N];
int n, nxt[N], stk[N], top, f[N];

int main()
{
	int i, k;
	
	scanf("%s", s + 1);
	n = strlen(s + 1);
	
	for (i = 2, k = 0; i <= n; ++i)
	{
		while (k && s[i] != s[k + 1]) k = nxt[k];
		if (s[i] == s[k + 1]) ++k;
		nxt[i] = k;
	}
	
	for (i = n; i >= 1; --i)
	{
		++f[i];
		f[nxt[i]] += f[i];
	}
	
	for (i = n; i; i = nxt[i]) stk[++top] = i;
	
	printf("%d\n", top);
	while (top)
	{
		printf("%d %d\n", stk[top], f[stk[top]]);
		--top;
	}
	
	return 0;
}
```

