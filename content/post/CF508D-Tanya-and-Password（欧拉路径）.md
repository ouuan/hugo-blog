+++
title = "CF508D Tanya and Password（欧拉路径）"
categories = ["题解"]
tags = ["图论", "建图", "欧拉路径"]
date = "2019-10-15T15:01:00+08:00"
description = ""
aliases = ["/post/CF508D-Tanya-and-Password（欧拉路径）", "/CF508D-Tanya-and-Password（欧拉路径）"]
+++


## 题目链接

[CF](https://codeforces.com/contest/508/problem/D)

## 题意简述

有一个字符串 $S[1..n+2]$，告诉你 $\forall 1\le i\le n, S[i..i+2]$（所有长为 $3$ 的子串），求任意一个满足条件的 $S$。

$1\le n\le 2\cdot 10^5$，字符集为大小写字母 + 数字。

<!--more-->

## 简要做法

容易想到需要建图。

但是，如果把每个长为 $3$ 的子串看成点，前后缀匹配看成边，就做不下去了。

正确做法是把每个长为 $2$ 的子串看成点，长为 $3$ 的子串看成边。这样原问题就转化成了求有向图的欧拉路径。（不会这个的话建议自行搜索一下。）

如果每次 dfs 同一个点时都遍历所有出边，度数比较大就会挂。使用 vector 存边的话可以 `pop_back()` 或者记录一下已经遍历到了哪一条边，使用前向星的话可以像当前弧优化那样修改 `head`。

## 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <cctype>
#include <cstdlib>

using namespace std;

const int N = 200010;
const int W = 62 * 62;

int head[N], nxt[N], to[N], cnt;
int n, ind[N], outd[N], tot;
char s[N], ans[N];

int charToInt(char x)
{
    if (isdigit(x)) return x - '0';
    if (islower(x)) return x - 'a' + 10;
    return x - 'A' + 36;
}

char intToChar(int x)
{
    if (x < 10) return x + '0';
    if (x < 36) return x - 10 + 'a';
    return x - 36 + 'A';
}

int wordToInt(char x, char y)
{
    return charToInt(x) * 62 + charToInt(y);
}

void add(int u, int v)
{
    nxt[++cnt] = head[u];
    head[u] = cnt;
    to[cnt] = v;
}

void fail()
{
    puts("NO");
    exit(0);
}

void dfs(int u)
{
    for (int& i = head[u]; i; )
    {
        int v = to[i];
        i = nxt[i];
        dfs(v);
    }
    ans[--tot] = intToChar(u % 62);
}

int main()
{
    scanf("%d", &n);

    for (int i = 1; i <= n; ++i)
    {
        scanf("%s", s);
        int u = wordToInt(s[0], s[1]);
        int v = wordToInt(s[1], s[2]);
        ++outd[u];
        ++ind[v];
        add(u, v);
    }

    int oneMoreIn = 0, oneMoreOut = 0;

    for (int i = 0; i < W; ++i)
    {
        if (ind[i] == outd[i]) continue;
        if (ind[i] == outd[i] + 1)
        {
            if (oneMoreIn) fail();
            oneMoreIn = i;
        }
        else if (ind[i] + 1 == outd[i])
        {
            if (oneMoreOut) fail();
            oneMoreOut = i;
        }
        else fail();
    }

    if (!oneMoreOut)
    {
        for (int i = 0; i < W; ++i)
        {
            if (outd[i])
            {
                oneMoreOut = i;
                break;
            }
        }
    }

    tot = n + 2;
    dfs(oneMoreOut);
    ans[--tot] = intToChar(oneMoreOut / 62);

    if (!tot) printf("YES\n%s", ans);
    else puts("NO");

    return 0;
}
```

