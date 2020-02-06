+++
title = "BZOJ2115 [WC2011]最大XOR和路径（线性基，图论）"
categories = ["题解"]
tags = ["线性基", "图论"]
date = "2019-06-16T20:46:02+08:00"
description = ""
+++


## 题目链接

[洛谷](https://www.luogu.org/problemnew/show/P4151)

[darkbzoj](https://darkbzoj.tk/problem/2115)

## 题意简述

给你一张带边权的无向图，求 $1$ 到 $n$ 的边权异或和最大的路径。

点数 $5\times 10^4$，边数 $10^5$。

<!--more-->

## 简要做法

我们先随便找一条从 $1$ 到 $n$ 的链，然后看能如何修改它。

如果我们不走这条链，从某个位置分叉出去，为了回到这条链上，我们一定是从某条岔路走出去，走一个环，再沿着这条岔路走回来。由于边权异或，这条岔路就不会产生贡献。因此，最终答案可以表示为一条链 + 若干个环的异或和。这条链是可以随便选的，因为一条链可以异或若干个环得到另一条链。

然而，环可能有很多，事实上我们可以得到 dfs 树，只需考虑那些仅包含一条返祖边的环，其它环都可以由若干个这样的环异或得到。代码实现非常简单，具体可以看参考代码。

找到这些环之后用线性基就可以求出答案了。

## 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <cctype>

using namespace std;

typedef long long ll;

ll read()
{
    ll out = 0;
    char c;
    while (!isdigit(c = getchar()));
    for (; isdigit(c); c = getchar()) out = out * 10 + c - '0';
    return out;
}

const int N = 50010;
const int M = 100010;

void add(int u, int v, ll w);
void dfs(int u, int fa);
void insert(ll x);

int n, m, head[N], nxt[M << 1], to[M << 1], cnt;
ll edge[M << 1], dis[N], p[70], ans;
bool vis[N];

int main()
{
    int i, u, v;
    ll w;

    n = read();
    m = read();

    for (i = 0; i < m; ++i)
    {
        u = read();
        v = read();
        w = read();
        add(u, v, w);
        add(v, u, w);
    }

    dfs(1, 0);

    ans = dis[n];

    for (i = 59; ~i; --i)
    {
        if (!((ans >> i) & 1))
        {
            ans ^= p[i];
        }
    }

    cout << ans;

    return 0;
}

void insert(ll x)
{
    for (int i = 59; ~i; --i)
    {
        if ((x >> i) & 1)
        {
            if (p[i]) x ^= p[i];
            else
            {
                p[i] = x;
                break;
            }
        }
    }
}

void dfs(int u, int fa)
{
    ll w;
    int i, v;
    vis[u] = true;
    for (i = head[u]; i; i = nxt[i])
    {
        v = to[i];
        w = edge[i];
        if (v == fa) continue;
        if (vis[v]) insert(dis[u] ^ dis[v] ^ w);
        else
        {
            dis[v] = dis[u] ^ w;
            dfs(v, u);
        }
    }
}

void add(int u, int v, ll w)
{
    nxt[++cnt] = head[u];
    head[u] = cnt;
    to[cnt] = v;
    edge[cnt] = w;
}
```

