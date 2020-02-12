+++
title = "LOJ6519 魔力环（Burnside引理，容斥原理）"
categories = ["题解"]
tags = ["群论", "Burnside引理", "组合数学", "容斥原理"]
date = "2019-09-05T10:45:59+08:00"
description = ""
aliases = ["/post/LOJ6519-魔力环（Burnside引理，容斥原理）", "/LOJ6519-魔力环（Burnside引理，容斥原理）"]
+++


## 题目链接

[LOJ](https://loj.ac/problem/6519)

[洛谷](https://www.luogu.org/problem/P4916)

## 题意简述

你需要给 $n$ 颗珠子的项链染 $m$ 颗黑色，$n-m$ 颗白色，不能有连续的一串黑色珠子长度超过 $k$，求旋转同构下本质不同的染色方案数。

$1\le m,k\le n\le10^5$

<!--more-->

## 简要做法

首先套用 Burnside 引理，以及位移为 $r$ 的旋转周期为 $\gcd(r, n)$ 的结论，得到答案的式子：
$$
\begin{aligned}
answer&=\frac 1 n\sum\limits_{i=1}^nf\left(\frac n{\gcd(i,n)}\right)\\\\
&=\frac 1 n\sum\limits_{d|n}\varphi(d)f(d)
\end{aligned}
$$
其中 $f(x)$ 表示在一个长为 $\frac n x$ 的项链上，染 $\frac{m}{x}$ 个黑珠子，$\frac{n-m}x$ 个白珠子，不能有连续的一串黑色珠子长度超过 $k$ 的方案数（在不旋转的意义下计数）。

可以看出只有 $d|m$ 时 $f(d)$ 可能不为零，如果用 $f(x, y)$ 表示在一个长为 $x+y$ 的项链上，染 $x$ 个黑珠子，$y$ 个白珠子，不能有连续的一串黑色珠子长度超过 $k$ 的方案数（在不旋转的意义下计数），答案的式子可以写成：

$$
answer=\frac 1 n\sum\limits_{d|\gcd(n, m)}\varphi(d)f\left(\frac m d, \frac{n-m}d\right)
$$

现在的问题转化成了快速求 $f(x, y)$。

首先，特判掉两种情况：

1. $k=n$
2. $y\ne 0$ 且 $x\le k$

这两种情况下 $f(x, y)=\binom{x+y}x$

由于是在环上不好处理，枚举两侧的黑珠子个数，就可以转化为序列上的问题。

而序列上的问题，就相当于求方程 $x_1+x_2+\cdots+x_{y+1}=x\\ (0\le x_i\le k)$ 的解的个数。

考虑容斥，枚举至少有 $i$ 个变量的值大于 $k$（实际上是枚举大小为 $i$ 的子集都大于 $k$），解的个数为 $\binom{x+y-i(k+1)}y$。

这样的话，枚举两侧黑珠子个数最多枚举到 $k$，容斥复杂度为 $O(\frac{x+y}k)$，计算 $f(x,y)$ 的复杂度为 $O(x+y)$，整道题的复杂度就是 $O(\text{预处理组合数}+\sigma(n))$，其中 $\sigma(n)$ 表示 $n$ 的所有约数之和，在数据范围内最大为 $403200$。

## 参考代码

```cpp
#include <iostream>
#include <cstdio>

using namespace std;

typedef long long ll;

const int N = 100005;
const int mod = 998244353;

int n, m, k, p[N], ptot, phi[N], fact[N], invf[N];
bool np[N];

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

int c(int x, int y)
{
	if (y > x || y < 0) return 0;
	return (ll) fact[x] * invf[y] % mod * invf[x - y] % mod;
}

int calc(int x, int y)
{
	int out = 0;
	for (int i = 0; i * (k + 1) <= x + y; ++i)
	{
		out = (out + (i & 1 ? -1ll : 1ll) * c(x + y - (k + 1) * i, y) * c(y + 1, i) % mod + mod) % mod;
	}
	return out;
}

int f(int x, int y)
{
	if (k == n || y != 0 && x <= k) return c(x + y, x);
	int out = 0;
	for (int i = 0; i <= x && i <= k; ++i)
	{
		out = (out + (ll) (i + 1) * calc(x - i, y - 2)) % mod;
	}
	return out;
}

int gcd(int x, int y)
{
	return y ? gcd(y, x % y) : x;
}

int main()
{
	cin >> n >> m >> k;
	
	fact[0] = invf[0] = 1;
	for (int i = 1; i <= n; ++i) fact[i] = (ll) fact[i - 1] * i % mod;
	invf[n] = qpow(fact[n], mod - 2);
	for (int i = n - 1; i >= 1; --i) invf[i] = (ll) invf[i + 1] * (i + 1) % mod;
	
	phi[1] = 1;
	for (int i = 2; i <= n; ++i)
	{
		if (!np[i])
		{
			p[++ptot] = i;
			phi[i] = i - 1;
		}
		for (int j = 1; j <= ptot && i * p[j] <= n; ++j)
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
	
	int ans = 0;
	int g = gcd(m, n);
	
	for (int i = 1; i * i <= g; ++i)
	{
		if (g % i == 0)
		{
			if (i * i == g) ans = (ans + (ll) f(m / i, (n - m) / i) * phi[i]) % mod;
			else ans = (ans + (ll) f(m / i, (n - m) / i) * phi[i] + (ll) f(m / (g / i), (n - m) / (g / i)) * phi[g / i]) % mod;
		}
	}
	
	cout << (ll) ans * qpow(n, mod - 2) % mod;
	
	return 0;
}
```
