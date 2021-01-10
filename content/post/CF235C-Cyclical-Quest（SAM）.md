+++
title = "CF235C Cyclical Quest（SAM）"
date = "2019-02-27T13:23:19+08:00"
categories = ["题解"]
tags = ["字符串", "SAM"]
description = ""
aliases = ["/post/CF235C-Cyclical-Quest（SAM）", "/CF235C-Cyclical-Quest（SAM）"]
+++


# 题目链接

[洛谷](https://www.luogu.org/problemnew/show/CF235C)

[CF problemset](https://codeforces.com/problemset/problem/235/C)

[CF contest](https://codeforces.com/contest/235/problem/C)

# 题意简述

给你一个字符串 $s$ 和 $n$ 个字符串 $x_{1..n}$，对每个 $x_i$，求有多少个 $s$ 的子串可以由 $x_i$ 旋转得到。

旋转一个字符串就是把它的一个前缀移到后面，如 `abcd` 可以旋转得到的字符串有 `abcd`，`bcda`，`cdab`，`dabc`。

<!--more-->

# 简要做法

对 $s$ 建 SAM，把 $x_i$ 旋转得到的每个字符串用 SAM 读入，就可以求答案了。（SAM 求子串出现次数是经典问题，可以参考[我的博客](/post/%E5%90%8E%E7%BC%80%E8%87%AA%E5%8A%A8%E6%9C%BA%EF%BC%88SAM%EF%BC%89%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/#%E5%AD%90%E4%B8%B2%E5%87%BA%E7%8E%B0%E6%AC%A1%E6%95%B0)）

分开读入每个 $x_i$ 旋转得到的字符串显然会超时，然而，SAM 读入字符串是支持删除首字符的：记录当前读入的长度 $l$ 以及所处状态 $u$，删除字符就把 $l$ 减一，若减一后 $l=len(parent(u))$，则转移到 $parent(u)$（把 $u$ 赋值为 $parent(t)$）。需要注意的是，如果读入一个字符的时候当前状态没有这个字符的出边，就需要在 $parent$ 树上向上跳，直到有这个字符的出边，同时更新 $l$ 。这样的话，删除字符前就要先判断 $l$ 与需要保留的字符串的长度的关系。具体细节可以参考代码及注释。

所以，先读入 $x_i$ 统计答案，再删去首字符读入 $x_i[0]$ 统计答案，删去首字符读入 $x_i[1]$ 统计答案……就只用读入 $O(len(x_i))​$ 个字符。

还有一个问题，就是去重。$s$ 的一个子串可能可以和 $x_i$ 不同的旋转匹配。解决这个问题有两个方法，第一个是求出 $x_i​$ 的周期（可以用 kmp 求），第二个方法是在 SAM 上打标记。我用的是打标记的方法，具体细节还是可以参考代码及注释。

# 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <cstdio>

using namespace std;

const int N=1000010;

struct Node
{
    int len,par,ch[26],vis,cnt;
} sam[N<<1];

void insert(int x);
void read(int x);
void del();
void calc();
void add(int u,int v);
void dfs(int u);

char s[N];
int head[N<<1],nxt[N<<1],to[N<<1],cnt;
int p,tot,n,m,l,u,tim,ans;

int main()
{
    int i;

    scanf("%s%d",s,&n);

    sam[0].par=-1;
    for (i=0;s[i];++i) insert(s[i]-'a');
    for (i=1;i<=tot;++i) add(sam[i].par,i);
    dfs(0);

    for (tim=1;tim<=n;++tim)
    {
        scanf("%s",s);
        m=strlen(s);
        ans=u=l=0;
        for (i=0;i<m;++i) read(s[i]-'a');
        calc();
        for (i=0;i<m-1;++i)
        {
            read(s[i]-'a');
            del();
            calc();
        }
        printf("%d\n",ans);
    }

    return 0;
}

void read(int x) //读入一个字符
{
    while (u&&!sam[u].ch[x]) l=sam[u=sam[u].par].len; //向上跳直至有这个字符的出边
    if (sam[u].ch[x])
    {
        ++l;
        u=sam[u].ch[x];
    }
}

void del() //删除首字符
{
    if (l>m&&--l==sam[sam[u].par].len) u=sam[u].par; //m表示当前xi的长度，只有l>m的时候才删除
}

void calc() //计算当前的答案
{
    if (l==m&&sam[u].vis<tim) //只有当前读入的串长度恰好为m且当前状态没有打上标记时才统计答案
    {
        ans+=sam[u].cnt;
        sam[u].vis=tim; //打标记
    }
}

void insert(int x) //向SAM中插入字符，有人把这个函数叫做extend
{
    int np=++tot;
    sam[np].len=sam[p].len+1;
    sam[np].cnt=1;
    while (~p&&!sam[p].ch[x])
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
            memcpy(sam[nq].ch,sam[q].ch,sizeof(sam[q].ch));
            sam[nq].par=sam[q].par;
            sam[q].par=sam[np].par=nq;
            while (~p&&sam[p].ch[x]==q)
            {
                sam[p].ch[x]=nq;
                p=sam[p].par;
            }
        }
    }
    p=np;
}

void add(int u,int v) //加边，用于遍历parent树
{
    nxt[++cnt]=head[u];
    head[u]=cnt;
    to[cnt]=v;
}

void dfs(int u) //遍历parent树，计算每个状态的出现次数
{
    int i,v;
    for (i=head[u];i;i=nxt[i])
    {
        v=to[i];
        dfs(v);
        sam[u].cnt+=sam[v].cnt;
    }
}
```

