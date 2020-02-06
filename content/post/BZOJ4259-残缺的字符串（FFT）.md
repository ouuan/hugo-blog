+++
title = "BZOJ4259 残缺的字符串（FFT）"
categories = ["题解"]
tags = ["字符串", "FFT"]
date = "2019-03-19T16:27:51+08:00"
description = ""
+++


# 题目链接

[洛谷](https://www.luogu.org/problemnew/show/P4173)

[darkbzoj](https://darkbzoj.tk/problem/4259)

# 题意简述

带通配符的单模式串、单文本串匹配。

字符串长度 $3\times10^5$。

<!--more-->

# 简要做法

这题是利用字符串的距离函数来匹配字符串，使用 FFT 优化距离函数的计算。

将通配符的值设为零，定义 $dis(s,t)=\sum(s[i]-t[i])^2s[i]t[i]​$，那么两个等长的字符串匹配当且仅当它们的距离为零。

为了方便卷积，将题目中的 $A$ 串翻转​得到串 $A'​$。

推一波式子：

$\begin{aligned}dis(A,B[i..i+m-1])&=\sum\limits_{j=0}^{m-1}(A'[m-j-1]-B[i+j])^2A'[m-j-1]B[i+j]\\\\&=\left(\sum\limits_{j=0}^{m-1}A'[m-j-1]B^3[i+j]\right)-2\left(\sum\limits_{j=0}^{m-1}(A')^2[m-j-1]B^2[i+j]\right)+\left(\sum\limits_{j=0}^{m-1}(A')^3[m-j-1]B[i+j]\right)\end{aligned}$

设 $f(i)=\sum\limits_{j=0}^{i}A'[i-j]B^3[j]​$（即它们的卷积），将 $A'[m..n-1]​$ 设为 $0​$，那么 $\sum\limits_{j=0}^{m-1}A'[m-j-1]B^3[i+j]=f(i+m-1)​$。

可以类似地计算另外两项，就可以求出所有 $dis(A,B[i..i+m-1])​$，看看哪些等于零就好了。

# 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <cmath>
#include <vector>
#include <cstring>
#include <algorithm>

using namespace std;

const int N=1<<19;
const double PI=acos(-1);

struct cp
{
    double re,im;
    cp(double _re=0.0,double _im=0.0) { re=_re; im=_im; }
    cp operator+(const cp& b) const { return cp(re+b.re,im+b.im); }
    cp operator-(const cp& b) const { return cp(re-b.re,im-b.im); }
    cp operator*(const cp& b) const { return cp(re*b.re-im*b.im,re*b.im+im*b.re); }
} a[N],a2[N],a3[N],b[N],b2[N],b3[N],c1[N],c2[N],c3[N];

void fft(cp* A,int type);

int n,m,lim,L,r[N];
char s[N],t[N];
vector<int> ans;

int main()
{
    int i;

    scanf("%d%d%s%s",&m,&n,s,t);

    for (i=0;i<n;++i)
    {
        if (m-i>0&&s[m-i-1]!='*')
        {
            a[i].re=s[m-i-1]-'a'+1;
            a2[i].re=a[i].re*a[i].re;
            a3[i].re=a[i].re*a2[i].re;
        }
        if (t[i]!='*')
        {
            b[i].re=t[i]-'a'+1;
            b2[i].re=b[i].re*b[i].re;
            b3[i].re=b[i].re*b2[i].re;
        }
    }

    for (lim=0,L=1;L<=n;++lim,L<<=1);
    for (i=1;i<L;++i) r[i]=(r[i>>1]>>1)|((i&1)<<(lim-1));

    fft(a,1);
    fft(a2,1);
    fft(a3,1);
    fft(b,1);
    fft(b2,1);
    fft(b3,1);

    for (i=0;i<L;++i)
    {
        c1[i]=a[i]*b3[i];
        c2[i]=a2[i]*b2[i];
        c3[i]=a3[i]*b[i];
    }

    fft(c1,-1);
    fft(c2,-1);
    fft(c3,-1);

    for (i=m-1;i<n;++i)
    {
        if (c1[i].re-2*c2[i].re+c3[i].re<0.5)
        {
            ans.push_back(i-m+2);
        }
    }

    printf("%d\n",int(ans.size()));
    for (i=0;i<ans.size();++i) printf("%d ",ans[i]);

    return 0;
}

void fft(cp* A,int type)
{
    int i,j,k;
    for (i=1;i<L;++i) if (i<r[i]) swap(A[i],A[r[i]]);
    for (i=1;i<L;i<<=1)
    {
        cp w1(cos(PI/i),type*sin(PI/i));
        for (j=0;j<L;j+=2*i)
        {
            cp w(1,0);
            for (k=j;k<i+j;++k,w=w*w1)
            {
                cp t=A[k+i]*w;
                A[k+i]=A[k]-t;
                A[k]=A[k]+t;
            }
        }
    }
    if (type==-1) for (i=0;i<L;++i) A[i].re=A[i].re/L;
}
```

