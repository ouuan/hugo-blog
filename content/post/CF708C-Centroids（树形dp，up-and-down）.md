+++
title = "CF708C Centroids（树形dp，up and down）"
categories = ["题解"]
tags = ["树形dp", "up and down"]
date = "2019-06-23T21:34:21+08:00"
description = ""
aliases = ["/post/CF708C-Centroids（树形dp，up-and-down）", "/CF708C-Centroids（树形dp，up-and-down）"]
+++


## 题目链接

[CF](https://codeforc.es/contest/708/problem/C)

[洛谷](https://www.luogu.org/problemnew/show/CF708C)

## 题意简述

给你一棵 $n$ 个点的树，对每个点，判断能否删去一条边再加上一条边，使得这个点成为树的重心。（树的重心：将其删去后每个联通块大小不超过 $\frac n 2$）

$2\le n\le 4\cdot10^5​$

<!--more-->

## 简要做法

up and down，即用两遍 dfs，第一遍用孩子更新父亲，第二遍用父亲更新孩子，好像也叫做“换根 dp”。

如何修改一条边使一个点成为重心？要把那个点原来最大的子树删掉一条边分成不超过 $\frac n 2$ 的两半，再把切下来那个子树接到这个点上。切下来的那个子树只要不超过 $\frac n 2$ 即可，所以我们希望切下来一个尽可能大的不超过 $\frac n 2$ 的子树，这样剩下来那一半就可以尽量小。

也就是说，我们需要找到以每个点为根的最大子树，以及每个子树（注意是无根树的每个子树）可以切出来的最大的不超过 $\frac n 2$ 的子树。求这个可以使用 up and down 这个技巧，详见代码。

求出这个之后，就可以 $\mathcal O(1)$ 判断每个点是否合法了。

up and down 的过程中有一个小技巧：第二遍 dfs 中用父亲更新孩子时，父亲的 down（父亲的孩子们对父亲的贡献）中要减去当前节点的贡献再更新当前节点的值，所以需要存最大值和次大值，这个可以利用一个小数据结构来简化代码，详见代码中的 `struct Node`。

## 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <algorithm>

using namespace std;

const int N = 400010;

struct Node
{
    int fi, se;
    void insert(int x) // 向最大值和次大值中插入一个值
    {
        if (x > fi)
        {
            se = fi;
            fi = x;
        }
        else if (x > se) se = x;
    }
    int get(int x) // 得到除了 x 外的最大值
    {
        if (x == fi) return se;
        else return fi;
    }
} dn[N];

int calc(int u);
void dfs1(int u);
void dfs2(int u);
void add(int u, int v);

int head[N], nxt[N << 1], to[N << 1], cnt;
int n, siz[N], fa[N], son[N], up[N];

int main()
{
    int i, u, v, mx, sz;

    scanf("%d", &n);

    for (i = 1; i < n; ++i)
    {
        scanf("%d%d", &u, &v);
        add(u, v);
        add(v, u);
    }

    dfs1(1);
    dfs2(1);

    for (u = 1; u <= n; ++u)
    {
        v = son[u];
        if (v == fa[u])
        {
            sz = n - siz[u];
            mx = up[u]; // 也可以 max(dn[v].get(calc(u)), up[v])
        }
        else
        {
            sz = siz[v];
            mx = dn[v].fi; // 也可以 calc(v)
        }
        printf("%d ", sz - mx <= n / 2);
    }

    return 0;
}

void dfs1(int u)
{
    int i, v;
    siz[u] = 1;
    for (i = head[u]; i; i = nxt[i])
    {
        v = to[i];
        if (v == fa[u]) continue;
        fa[v] = u;
        dfs1(v);
        siz[u] += siz[v];
        dn[u].insert(calc(v));
    }
}

void dfs2(int u)
{
    int i, v;
    if (n - siz[u] <= n / 2)  up[u] = n - siz[u];
    else up[u] = max(up[fa[u]], dn[fa[u]].get(calc(u)));
    for (i = head[u]; i; i = nxt[i])
    {
        v = to[i];
        if (v == fa[u]) continue;
        dfs2(v);
        if (siz[v] > siz[son[u]]) son[u] = v; // son 是一个节点的最大子树
    }
    if (n - siz[u] > siz[son[u]]) son[u] = fa[u];
}

int calc(int u) // 计算孩子对父亲的贡献
{
    return siz[u] <= n / 2 ? siz[u] : dn[u].fi;
}

void add(int u, int v)
{
    nxt[++cnt] = head[u];
    head[u] = cnt;
    to[cnt] = v;
}
```

