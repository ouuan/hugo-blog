+++
title = "P2870 [USACO07DEC]Best Cow Line, Gold（后缀数组）"
date = "2019-02-14T11:20:59+08:00"
categories = ["题解"]
tags = ["字符串", "后缀数组"]
description = ""
aliases = ["/post/P2870-USACO07DEC-Best-Cow-Line-Gold（后缀数组）", "/P2870-USACO07DEC-Best-Cow-Line-Gold（后缀数组）"]
+++


# 题目链接

[洛谷](https://www.luogu.org/problemnew/show/P2870)

# 题意简述

给你一个字符串，每次从首或尾取一个字符组成字符串，问所有能够组成的字符串中字典序最小的一个。

<!--more-->

# 简要做法

暴力做法就是每次最坏 $O(n)$ 地判断当前应该取首还是尾，只需优化这一判断过程即可。

将原串reverse后拼接在原串后，并在中间加上一个没出现过的字符（如 `#` ），求SA，即可 $O(1)$ 完成这一判断。

# 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <cstring>

using namespace std;

const int N=1000010;

char s[N];
int n,sa[N],sa2[N<<1],rk[N<<1],px[N],cnt[N];

int main()
{
    int i,w,m=200,p,l=1,r,tot=0;

    cin>>n;
    r=n;

    for (i=1;i<=n;++i) while (!isalpha(s[i]=getchar()));
    for (i=1;i<=n;++i) rk[i]=rk[2*n+2-i]=s[i];

    n=2*n+1;

    for (i=1;i<=n;++i) ++cnt[rk[i]];
    for (i=1;i<=m;++i) cnt[i]+=cnt[i-1];
    for (i=n;i>=1;--i) sa[cnt[rk[i]]--]=i;

    for (w=1;w<n;w<<=1,m=p)
    {
        memset(cnt,0,sizeof(cnt));
        for (p=0,i=n;i>n-w;--i) sa2[++p]=i;
        for (i=1;i<=n;++i) if (sa[i]>w) sa2[++p]=sa[i]-w;
        for (i=1;i<=n;++i) ++cnt[px[i]=rk[sa2[i]]];
        for (i=1;i<=m;++i) cnt[i]+=cnt[i-1];
        for (i=n;i>=1;--i) sa[cnt[px[i]]--]=sa2[i];
        swap(sa2,rk);
        for (p=0,i=1;i<=n;++i) rk[sa[i]]=sa2[sa[i]]==sa2[sa[i-1]]&&sa2[sa[i]+w]==sa2[sa[i-1]+w]?p:++p;
    }

    while (l<=r)
    {
        printf("%c",rk[l]<rk[n+1-r]?s[l++]:s[r--]);
        if ((++tot)%80==0) puts("");
    }

    return 0;
}
```

