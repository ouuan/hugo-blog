+++
title = "CF1209F Koala and Notebook（BFS，最短路）"
categories = ["题解"]
tags = ["图论", "最短路", "BFS"]
date = "2019-09-16T08:58:32+08:00"
description = ""
aliases = ["/post/CF1209F-Koala-and-Notebook（BFS，最短路）", "/CF1209F-Koala-and-Notebook（BFS，最短路）"]
+++


## 题目链接

[CF](http://codeforces.com/contest/1209/problem/F )

## 题意简述

给你一张 $n$ 个点 $m$ 条边的无向连通图，一条路径的权值是路径上的边的编号（十进制）顺次连接而成的数字。求 $1$ 到每个点的最短路，**输出** 对 $10^9+7$ 取模。

$2\le n\le10^5$, $n-1\le m\le10^5$。

<!--more-->

## 简要做法

数字越长就越大，所以转化成优先长度短，其次字典序小。

把每条边拆成位数条边（如 $(u, v, 718)$ 拆成 $(u, x, 7)$, $(x, y, 1)$, $(y, v, 8)$, $(v, x, 7)$, $(y, u, 8)$），这样的话长度的边权就全是一，可以用 BFS 解决。

如何使字典序最小呢？容易想到优先遍历边权（拆边后全是一位数）小的边，但是，如果两个点的最短路相同，这样做就会导致错误。（如：$dis[u]=dis[v]=233$，$u$ 在队列里在 $v$ 的前面，$(u, x, 3)$ 和 $(v, x, 2)$ 这两条边都存在，$dis[x]$ 就会被错误地设为 $2333$，而它应当是 $2332$。）

正确的做法是将最短路相同的点绑在一起放入队列，实现可以使用 `vector`。

## 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <queue>
#include <cstring>

using namespace std;

const int N = 100010;
const int mod = 1e9 + 7;

int n, m, tot, dis[N * 5], digit[10];
vector<int> g[N * 5][10];
queue<vector<int> > q;

int main()
{
    scanf("%d%d", &n, &m);

    tot = n;

    for (int ww = 1; ww <= m; ++ww)
    {
        int u, v, w = ww;
        scanf("%d%d", &u, &v);
        if (w < 10)
        {
            g[u][w].push_back(v);
            g[v][w].push_back(u);
        }
        else // 拆边
        {
            int l = ++tot;
            int r = l;
            int d = 0;
            while (w)
            {
                digit[++d] = w % 10;
                w /= 10;
            }
            for (int i = d - 1; i > 1; --i)
            {
                g[tot][digit[i]].push_back(tot + 1);
                r = ++tot;
            }
            g[u][digit[d]].push_back(l);
            g[r][digit[1]].push_back(u);
            g[v][digit[d]].push_back(l);
            g[r][digit[1]].push_back(v);
        }
    }

    memset(dis, -1, sizeof(dis));
    q.push(vector<int>(1, 1));
    dis[1] = 0;

    while (!q.empty())
    {
        vector<int> vec = q.front();
        q.pop();
        for (int i = 0; i <= 9; ++i)
        {
            vector<int> nxt; // nxt 里存的是最短路相同的点
            for (auto u : vec)
            {
                for (auto v : g[u][i])
                {
                    if (dis[v] == -1)
                    {
                        dis[v] = (dis[u] * 10ll + i) % mod;
                        nxt.push_back(v);
                    }
                }
            }
            if (!nxt.empty()) q.push(nxt);
        }
    }

    for (int i = 2; i <= n; ++i) printf("%d\n", dis[i]);

    return 0;
}
```
