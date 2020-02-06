+++
title = "BZOJ4665 小w的喜糖（容斥原理，组合数学）"
categories = ["题解"]
tags = ["容斥原理", "组合数学", "计数dp"]
date = "2019-03-13T21:10:56+08:00"
description = ""
+++


# 题目链接

[darkbzoj](https://darkbzoj.tk/problem/4665)

# 题意简述

给你一个数列，求重排后每个位置都变了的方案数。

数列长度和值域都是 $2000$。

<!--more-->

# 简要做法

考虑容斥，计 $g(S)​$ 为 $S​$ 这个集合内的所有位置都不变的方案数，$U​$ 为位置的全集（即 $1​$ ~ $n​$），$i​$ 这个数出现了 $cnt[i]​$ 次，答案即为 $\frac{n!}{\prod\limits_{i=1}^ncnt[i]!}+\sum\limits_{S\subseteq U,S\ne\varnothing}(-1)^{|S|}g(S)​$。

计 $P_i$ 为 $i$ 这个数出现的位置集合，那么 $g(S)=\frac{(n-|S|)!\prod\limits_{i=1}^n\frac{cnt[i]!}{(cnt[i]-|P_i\bigcap S|)!}}{\prod\limits_{i=1}^ncnt[i]!}​$。

暴力枚举子集肯定是过不了的，考虑如何优化。

容斥原理的优化一般是将答案相同/结构相似的部分合在一起算，这题中，可以尝试将 $|S|$ 相同的集合合在一起算，问题就在于如何快速计算 $|S|$ 相同的所有 $\prod\limits_{i=1}^n\frac{cnt[i]!}{(cnt[i]-|P_i\bigcap S|)!}$ 之和。

这个东西可以用 $dp$ 解决：设 $f(i,j)$ 为从值为 $1$ ~ $i$ 的所有数中选 $j$ 个数的所有不同方案的贡献之和，其中，一种选法的贡献就是将选的数作为 $S$ 的 $\prod\limits_{i=1}^n\frac{cnt[i]!}{(cnt[i]-|P_i\bigcap S|)!}$。转移时枚举值为 $i$ 的数选了 $k$ 个，那么 $f(i,j)=\sum\limits_{k=0}^{\min(j,cnt[i])}\frac{cnt[i]!}{(cnt[i]-k)!}\binom{cnt[i]}kf(i-1,j-k)$. 由于 $cnt[i]$ 之和为 $n$，$dp$ 复杂度是 $O(n^2)$。

这样的话，最终的答案就是 $\frac{n!+\sum\limits_{i=1}^n(-1)^i(n-i)!f(n,i)}{\prod\limits_{i=1}^ncnt[i]!}$，实际上 $f(n,0)=1$，所以也可以化简为  $\frac{\sum\limits_{i=0}^n(-1)^i(n-i)!f(n,i)}{\prod\limits_{i=1}^ncnt[i]!}$.

# 参考代码

```cpp
#include <iostream>
#include <cstdio>

using namespace std;

typedef long long ll;

const int N=2010;
const int mod=1e9+9;

int qpow(int x,int y);

int n,cnt[N],fac[N],inv[N],f[N][N],c[N][N],ans;

int main()
{
    int i,j,k,x;

    cin>>n;

    fac[0]=inv[0]=1;
    for (i=1;i<=n;++i)
    {
        cin>>x;
        ++cnt[x];
        fac[i]=(ll)fac[i-1]*i%mod;
        inv[i]=qpow(fac[i],mod-2);
    }

    c[0][0]=1;
    for (i=1;i<=n;++i)
    {
        c[i][0]=1;
        for (j=1;j<=i;++j) c[i][j]=(c[i-1][j-1]+c[i-1][j])%mod;
    }

    f[0][0]=1;
    for (i=1;i<=n;++i)
    {
        for (j=0;j<=n;++j)
        {
            for (k=0;k<=cnt[i]&&k<=j;++k)
            {
                f[i][j]=(f[i][j]+(ll)fac[cnt[i]]*inv[cnt[i]-k]%mod*f[i-1][j-k]%mod*c[cnt[i]][k])%mod;
            }
        }
    }

    for (i=0;i<=n;++i) ans=(ans+(ll)(i&1?-1:1)*fac[n-i]*f[n][i]+mod)%mod;
    for (i=1;i<=n;++i) ans=(ll)ans*inv[cnt[i]]%mod;

    cout<<ans;

    return 0;
}

int qpow(int x,int y)
{
    int out=1;
    while (y)
    {
        if (y&1) out=(ll)out*x%mod;
        x=(ll)x*x%mod;
        y>>=1;
    }
    return out;
}
```

