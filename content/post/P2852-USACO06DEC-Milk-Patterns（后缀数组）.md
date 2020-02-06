+++
title = "P2852 [USACO06DEC]Milk Patterns（后缀数组）"
date = "2019-02-14T11:21:26+08:00"
categories = ["题解"]
tags = ["字符串", "后缀数组"]
description = ""
+++


# 题目链接

[洛谷](https://www.luogu.org/problemnew/show/P2852)

# 题意简述

给你一个字符串，求最长的出现了至少 $k$ 次的子串的长度。

<!--more-->

# 简要做法

求出 `height` 数组，若一个长为 $x$ 的子串在原串中出现了至少 $k$ 次，则 `height` 数组中一定存在至少 $k-1$ 个 连续的大于等于 $x$ 的值。所以，问题就转化成了：求 `height` 数组中 **每连续 $k-1$ 个数的最小值** 的最大值。即：$a_i=\min\\{height_{i..i+k-2}\\}$，求 $a_i$ 的最大值。可以用RMQ/平衡树/线段树/multiset解决。

# 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <cstring>
#include <set>

using namespace std;

const int N=40010;

int n,k,a[N],sa[N],rk[N],sa2[N],px[N],cnt[1000010],height[N],ans;
multiset<int> t;

int main()
{
    int i,j,w,p,m=1000000;

    scanf("%d%d",&n,&k);
    --k;

    for (i=1;i<=n;++i) scanf("%d",a+i);
    for (i=1;i<=n;++i) ++cnt[rk[i]=a[i]];
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
        swap(rk,sa2);
        for (p=0,i=1;i<=n;++i) rk[sa[i]]=sa2[sa[i]]==sa2[sa[i-1]]&&sa2[sa[i]+w]==sa2[sa[i-1]+w]?p:++p;
    }

    for (i=1,j=0;i<=n;++i)
    {
        if (j) --j;
        while (a[i+j]==a[sa[rk[i]-1]+j]) ++j;
        height[rk[i]]=j;
    }

    for (i=1;i<=n;++i)
    {
        t.insert(height[i]);
        if (i>k) t.erase(t.find(height[i-k]));
        ans=max(ans,*t.begin());
    }

    cout<<ans;

    return 0;
}
```

