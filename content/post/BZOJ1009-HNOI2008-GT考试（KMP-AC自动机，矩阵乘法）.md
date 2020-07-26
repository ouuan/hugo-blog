+++
title = "BZOJ1009 [HNOI2008]GT考试（KMP/AC自动机，矩阵乘法）"
categories = ["题解"]
tags = ["字符串", "KMP", "AC自动机", "矩阵"]
date = "2019-05-03T19:07:11+08:00"
description = ""
aliases = ["/post/BZOJ1009-HNOI2008-GT考试（KMP-AC自动机，矩阵乘法）", "/BZOJ1009-HNOI2008-GT考试（KMP-AC自动机，矩阵乘法）"]
+++


## 题目链接

[洛谷](https://www.luogu.org/problemnew/show/P3193)

[darkbzoj](https://darkbzoj.tk/problem/1009)

## 题意简述

给一个长为 $m$ 的字符串，字符集 $0$ ~ $9$，求长为 $n$ 的字符串中不含给定字符串作为子串的字符串有多少个，对 $k$ 取模。

$n\le10^9$，$m\le20$，$k\le1000$。

<!--more-->

## 简要做法

首先用 KMP / AC 自动机搞出转移：$f_{i,j}$ 表示从状态 $i$ （或者说位置 $i$，KMP 里就是位置，AC 自动机里虽然是节点但也和位置差不多）起，在后面加 $j$ 个字符，满足要求的字符串个数，则 $f_{i,j} = \sum\limits_{x=0}^9f_{tr[i][x],j-1}$，其中 $tr[i][x]$ 表示状态 $i$ 的字符 $x$ 转移。AC 自动机中不用解释了，KMP 里面就是 $tr[i][x]=\begin{cases}i+1&(s[i+1]=x)\\\\tr[next[i]][x]&(s[i+1]\ne x)\end{cases}$。

发现每层（相同的 $j$）的转移都是类似的，实际上可以用矩阵表示两层间的转移：

$$A\times\begin{bmatrix}f_{0,j}\\\\f_{1,j}\\\\\vdots\\\\f_{m-1,j}\end{bmatrix}=\begin{bmatrix}f_{0,j+1}\\\\f_{1,j+1}\\\\\vdots\\\\f_{m-1,j+1}\end{bmatrix}$$

（只到 $m-1$ 是因为 $f_{m,j}=0$）

其中 $A​$ 是我们要求的一个矩阵，$A_{i,j}​$ 就是 $\sum\limits_{x=0}^9[tr[i][x]=j]​$，求出来之后矩阵快速幂就好了。

## 参考代码

### KMP

```cpp
#include <iostream>
#include <cstdio>
#include <cctype>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 21;

int n, m, mod, nxt[N];
char s[N];

struct Matrix
{
    int a[N][N];
    Matrix() { memset(a, 0, sizeof(a)); }
    int* operator[](int x) { return a[x]; }
    Matrix operator*(Matrix & b)
    {
        Matrix out;
        int i, j, k;
        for (i = 0; i < m; ++i)
        {
            for (j = 0; j < m; ++j)
            {
                for (k = 0; k < m; ++k)
                {
                    out[i][j] = (out[i][j] + a[i][k] * b[k][j]) % mod;
                }
            }
        }
        return out;
    }
};

Matrix qpow(Matrix x, int y);

int main()
{
    int i, j, k, ans = 0;
    Matrix mul;

    scanf("%d%d%d%s", &n, &m, &mod, s + 1);

    for (i = 2, k = 0; i < m; ++i)
    {
        while (k && s[i] != s[k + 1]) k = nxt[k];
        if (s[i] == s[k + 1]) ++k;
        nxt[i] = k;
    }

    for (i = 0; i < m; ++i)
    {
        for (j = 0; j <= 9; ++j)
        {
            for (k = i; k && s[k + 1] - '0' != j; k = nxt[k]);
            if (s[k + 1] - '0' == j) ++k;
            ++mul[i][k];
        }
    }

    mul = qpow(mul, n);

    for (i = 0; i < m; ++i) ans = (ans + mul[0][i]) % mod;

    cout << ans;

    return 0;
}

Matrix qpow(Matrix x, int y)
{
    Matrix out;
    for (int i = 0; i < m; ++i) out[i][i] = 1;
    while (y)
    {
        if (y & 1) out = out * x;
        x = x * x;
        y >>= 1;
    }
    return out;
}
```

### AC 自动机

如果觉得我的 AC 自动机写法比较清奇可以看看[我的 AC 自动机学习笔记](/AC自动机学习笔记)..

```cpp
#include <iostream>
#include <cstdio>
#include <cctype>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 21;

int n, m, mod, tr[N][10], fail[N];
char s[N];

struct Matrix
{
    int a[N][N];
    Matrix() { memset(a, 0, sizeof(a)); }
    int* operator[](int x) { return a[x]; }
    Matrix operator*(Matrix & b)
    {
        Matrix out;
        int i, j, k;
        for (i = 0; i < m; ++i)
        {
            for (j = 0; j < m; ++j)
            {
                for (k = 0; k < m; ++k)
                {
                    out[i][j] = (out[i][j] + a[i][k] * b[k][j]) % mod;
                }
            }
        }
        return out;
    }
};

Matrix qpow(Matrix x, int y);

int main()
{
    int i, j, ans = 0;
    Matrix mul;

    scanf("%d%d%d%s", &n, &m, &mod, s + 2);

    for (i = 0; i <= 9; ++i) tr[0][i] = 1;

    for (i = 1; i <= m; ++i)
    {
        tr[i][s[i + 1] - '0'] = i + 1;
        for (j = 0; j <= 9; ++j)
        {
            if (s[i + 1] - '0' == j) fail[i + 1] = tr[fail[i]][j];
            else tr[i][j] = tr[fail[i]][j];
            ++mul[i - 1][tr[i][j] - 1];
        }
    }

    mul = qpow(mul, n);

    for (i = 0; i < m; ++i) ans = (ans + mul[0][i]) % mod;

    cout << ans;

    return 0;
}

Matrix qpow(Matrix x, int y)
{
    Matrix out;
    for (int i = 0; i < m; ++i) out[i][i] = 1;
    while (y)
    {
        if (y & 1) out = out * x;
        x = x * x;
        y >>= 1;
    }
    return out;
}
```

