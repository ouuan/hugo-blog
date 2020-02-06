+++
title = "BZOJ4516 [SDOI2016]生成魔咒（SAM）"
date = "2019-02-21T14:10:35+08:00"
categories = ["题解"]
tags = ["字符串", "SAM"]
description = ""
+++


# 题目链接

[洛谷](https://www.luogu.org/problemnew/show/P4070)

[dark bzoj](https://darkbzoj.tk/problem/4516)

# 题意简述

给你一个字符串（字符集大小 $10^9$，长度 $10^5$），求每个前缀的本质不同子串数。

<!--more-->

# 简要做法

如果只求整个串的本质不同子串，由于每个本质不同子串可以与 SAM 上一个状态+串的长度一一对应，所以本质不同子串数就是每个状态的 $maxlen-minlen+1$，也就是 $len-parent.len$。

$parent$ 不改变时，逐个加入字符并计算即可。

考虑 $parent$ 改变的情况，四次 $parent$ 改变分别为 $nq-q.parent$，$-(q-q.parent)$，$q-nq$，$np-nq$。总贡献为 $nq-q.parent-q+q.parent+q-nq+np-nq=np-nq$，也是新加入的点的 $len$ 减去 $parent.len$，是一样的。

由于字符集较大，用 `map` 存边。

# 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <map>
#include <cctype>

using namespace std;

typedef long long ll;

int read()
{
    int out=0;
    char c;
    while (!isdigit(c=getchar()));
    for (;isdigit(c);c=getchar()) out=out*10+c-'0';
    return out;
}

const int N=100010;

struct Node
{
    int len,par;
    map<int,int> ch;
} sam[N<<1];

void insert(int x);

int n,p,tot;
ll ans;

int main()
{
    int i;

    n=read();

    sam[0].par=-1;

    for (i=1;i<=n;++i)
    {
        insert(read());
        printf("%lld\n",ans);
    }

    return 0;
}

void insert(int x)
{
    int np=++tot;
    sam[np].len=sam[p].len+1;
    while (~p&&sam[p].ch.find(x)==sam[p].ch.end())
    {
        sam[p].ch[x]=np;
        p=sam[p].par;
    }
    if (p==-1) sam[np].par=0;
    else
    {
        int q=sam[p].ch[x];
        if (sam[q].len==sam[p].len+1) sam[np].par=q;
        else
        {
            int nq=++tot;
            sam[nq].len=sam[p].len+1;
            sam[nq].ch=sam[q].ch;
            sam[nq].par=sam[q].par;
            sam[q].par=sam[np].par=nq;
            while (~p&&sam[p].ch[x]==q)
            {
                sam[p].ch[x]=nq;
                p=sam[p].par;
            }
        }
    }
    ans+=sam[np].len-sam[sam[np].par].len;
    p=np;
}
```





