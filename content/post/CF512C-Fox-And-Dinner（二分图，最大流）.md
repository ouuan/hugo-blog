+++
title = "CF512C Fox And Dinner（二分图，最大流）"
categories = ["题解"]
tags = ["图论", "二分图", "网络流", "最大流"]
date = "2019-10-15T19:50:54+08:00"
description = ""
+++


## 题目链接

[CF](https://codeforces.com/contest/512/problem/C)

## 题意简述

给你 $n$ 个整数，需要将它们分成任意个至少包含 $3$ 个数的环，使得每相邻两个数加起来是一个质数。

判断是否有解，若有解输出任意一组解。

$3\le n\le 200$, 数的范围是 $[2,10^4]$。

<!--more-->

## 简要做法

首先，由于每个数都大于等于 $2$，加起来是质数的必要条件是一奇一偶。

所以，如果把数看成点，相加得到质数看成边，就得到了一张二分图。

而题目的要求可以看作是每个点都匹配两个点。因为所有点度数都为 $2$ 的简单无向图一定是一个至少包含 $3$ 个点的环。

所以可以这样建图：源点到奇数，容量为 $2$；奇数到与其之和为质数的偶数，容量为 $1$；偶数到汇点，容量为 $2$。

如果最大流为 $n$ 就有解。输出方案就和普通的网络流输出方案差不多（可以参考代码）。

## 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <queue>
#include <vector>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 210;
const int W = 20010;

int n, a[N], p[W], tot;
bool np[W];

struct Flow
{
    const int s = N - 2;
    const int t = N - 1;

    int head[N], nxt[N * N], to[N * N], edge[N * N], cnt;
    queue<int> q;
    bool vis[N];
    int dep[N];

    void add(int u, int v, int w)
    {
        nxt[++cnt] = head[u];
        head[u] = cnt;
        to[cnt] = v;
        edge[cnt] = w;
    }

    void Add(int u, int v, int w)
    {
        add(u, v, w);
        add(v, u, 0);
    }

    bool bfs()
    {
        memset(dep, -1, sizeof(dep));
        dep[s] = 0;
        q.push(s);

        while (!q.empty())
        {
            int u = q.front();
            q.pop();

            for (int i = head[u]; i; i = nxt[i])
            {
                int v = to[i];
                int w = edge[i];
                if (w > 0 && dep[v] == -1)
                {
                    dep[v] = dep[u] + 1;
                    q.push(v);
                }
            }
        }

        return ~dep[t];
    }

    int dfs(int u, int flow)
    {
        if (dep[u] == dep[t]) return u == t ? flow : 0;

        int out = 0;

        for (int i = head[u]; i && flow - out; i = nxt[i])
        {
            int v = to[i];
            int w = edge[i];
            if (dep[v] == dep[u] + 1)
            {
                int f = dfs(v, min(flow - out, w));
                edge[i] -= f;
                edge[i ^ 1] += f;
                out += f;
            }
        }

        return out;
    }

    int maxFlow()
    {
        int out = 0;
        while (bfs()) out += dfs(s, N);
        return out;
    }

    void init()
    {
        cnt = 1;

        for (int i = 1; i <= n; ++i)
        {
            if (a[i] & 1) Add(s, i, 2);
            else Add(i, t, 2);
        }

        for (int i = 1; i <= n; ++i)
        {
            if (!(a[i] & 1)) continue;
            for (int j = 1; j <= n; ++j)
            {
                if (np[a[i] + a[j]]) continue;
                Add(i, j, 1);
            }
        }
    }

    vector<int> cycle(int u)
    {
        vector<int> out(1, u);
        vis[u] = true;
        while (1)
        {
            bool flag = false;
            for (int i = head[u]; i; i = nxt[i])
            {
                int v = to[i];
                int w = edge[i];
                if (((a[u] & 1) ^ (w > 0)) && v <= n && !vis[v])
                {
                    vis[v] = flag = true;
                    out.push_back(v);
                    u = v;
                    break;
                }
            }
            if (!flag) break;
        }
        return out; 
    }

    void output()
    {
        vector<vector<int> > ans;
        for (int i = 1; i <= n; ++i) if (!vis[i]) ans.push_back(cycle(i));
        printf("%d\n", ans.size());
        for (int i = 0; i < ans.size(); ++i)
        {
            printf("%d", ans[i].size());
            for (int j = 0; j < ans[i].size(); ++j) printf(" %d", ans[i][j]);
            puts("");
        }
    }
} flow;

int main()
{
    for (int i = 2; i < W; ++i)
    {
        if (!np[i]) p[++tot] = i;
        for (int j = 1; j <= tot && i * p[j] < W; ++j)
        {
            np[i * p[j]] = true;
            if (i % p[j] == 0) break;
        }
    }

    scanf("%d", &n);

    for (int i = 1; i <= n; ++i) scanf("%d", a + i);

    flow.init();

    if (flow.maxFlow() != n)
    {
        puts("Impossible");
        return 0;
    }

    flow.output();

    return 0;
}
```

