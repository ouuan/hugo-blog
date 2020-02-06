+++
title = "BZOJ2434 [NOI2011]阿狸的打字机（AC自动机，树状数组）"
categories = ["题解"]
tags = ["字符串", "AC自动机", "树状数组"]
date = "2019-04-29T21:36:58+08:00"
description = ""
+++


## 题目链接

[洛谷](https://www.luogu.org/problemnew/show/P2414)

[darkbzoj](https://darkbzoj.tk/problem/2434)

## 题意简述

初始一个空串，三种操作：

1. 添加一个字符。
2. 删除一个字符。
3. 打印当前字符串、

多组询问，每次问第 $x$ 个打印的字符串在第 $y$ 个打印的字符串中出现了几次。

操作数和询问数都不超过 $10^5$。

<!--more-->

## 简要做法

先建个 AC 自动机。

fail 树上的祖先是后缀，Trie 上根到一个点的路径是一个前缀，后缀的前缀是子串，因此只要把 Trie 上路径标出来，在 fail 树里统计子树就好了。也就是说，求出 $y$ 的所有前缀中以 $x$ 为后缀的数量。

具体来说，用 fail 树求 dfs 序，然后对 Trie 树进行 dfs，进入一个节点将其（在树状数组中）加一，退出时减一，把询问按 $y$ 存下来，访问到一个点时对以其作为 $y$ 的所有询问计算 $x​$ 在 fail 树中的子树和即为答案。

## 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <cctype>
#include <queue>
#include <vector>
#include <cstring>
#include <algorithm>

using namespace std;

int read()
{
    int out = 0;
    char c;
    while (!isdigit(c = getchar()));
    for (; isdigit(c); c = getchar()) out = out * 10 + c - '0';
    return out;
}

typedef pair<int, int> pii;

const int N = 100010;

void modify(int p, int x);
void add(int u, int v);
void dfs1(int u);
void dfs2(int u);
int qsum(int p);

int head[N], nxt[N], to[N], cnt;
int n, m, tr[N][26], trie[N][26], tot = 1, fail[N], fa[N], id[N], BIT[N], dfn[N], dfntot, exi[N], ans[N];
vector<pii> query[N];
queue<int> q;
char s[N];

int main()
{
    int i, u, x, y;

    scanf("%s", s);

    for (i = 0; i < 26; ++i) tr[0][i] = 1;

    for (i = 0, u = 1; s[i]; ++i)
    {
        if (s[i] == 'B') u = fa[u];
        else if (s[i] == 'P') id[++n] = u;
        else
        {
            int c = s[i] - 'a';
            if (!tr[u][c]) fa[tr[u][c] = ++tot] = u;
            u = tr[u][c];
        }
    }

    memcpy(trie, tr, sizeof(tr));

    q.push(1);

    while (!q.empty())
    {
        u = q.front();
        q.pop();
        for (i = 0; i < 26; ++i)
        {
            if (tr[u][i])
            {
                fail[tr[u][i]] = tr[fail[u]][i];
                q.push(tr[u][i]);
            }
            else tr[u][i] = tr[fail[u]][i];
        }
    }

    for (i = 2; i <= tot; ++i) add(fail[i], i);

    m = read();

    for (i = 1; i <= m; ++i)
    {
        x = read();
        y = read();
        query[id[y]].push_back(pii(id[x], i));
    }

    dfs1(1);
    dfs2(1);

    for (i = 1; i <= m; ++i) printf("%d\n", ans[i]);

    return 0;
}

void dfs1(int u)
{
    dfn[u] = ++dfntot;
    for (int i = head[u]; i; i = nxt[i]) dfs1(to[i]);
    exi[u] = dfntot;
}

void dfs2(int u)
{
    int i, v;
    modify(dfn[u], 1);
    for (i = 0; i < query[u].size(); ++i) ans[query[u][i].second] = qsum(exi[query[u][i].first]) - qsum(dfn[query[u][i].first] - 1);
    for (i = 0; i < 26; ++i)
    {
        v = trie[u][i];
        if (v) dfs2(v);
    }
    modify(dfn[u], -1);
}

void modify(int p, int x)
{
    for (; p <= tot; p += (p & -p))
    {
        BIT[p] += x;
    }
}

int qsum(int p)
{
    int out = 0;
    for (; p; p -= (p & -p))
    {
        out += BIT[p];
    }
    return out;
}

void add(int u, int v)
{
    nxt[++cnt] = head[u];
    head[u] = cnt;
    to[cnt] = v;
}
```

