+++
title = "UER #8 —— 通信题：打雪仗"
date = "2018-12-22T23:03:25+08:00"
categories = ["题解"]
tags = ["UOJ", "通信题"]
description = ""
aliases = ["/post/UER-8-游记-——-通信题：打雪仗", "/UER-8-游记-——-通信题：打雪仗"]
+++


[T1题目链接](https://uoj.ac/contest/47/problem/454)

大意：Alice 有一个长度为 $2n$ 的 $01$ 串 $s_{1..2n}$，Bob 有 $n$ 个下标 $p_{1..n}$，Alice 和 Bob 只能用 $01$ 通信，需要在每人各 $m$ 个 bit 内使 Bob 输出 $s_{p_1..p_n}$ .

$n=1000$, $m=1350$。

<!-- more -->

# Part 1 通信题

> 只是说一下我做了这道题后对通信题的理解，可能有误。

## 赛时：通信题是啥？？？

作为一个从未做过通信题的选手，遇到这题自然是百度了一下“通信题”。如果你尝试一下，会搜到《移动通信试题库》。

尝试搜索 “通信题 OI”——OI是什么意思? - 问通信专家；"通信题 CSDN"——通信原理考试题-CSDN下载。

ok，只能自己看样例程序了。

于是我比赛的第一个小时就在对着样例代码懵逼中度过了.....

然后发现我sb了，忘记了一件事：标准输入是会等待输入的！![](https://i.loli.net/2018/10/23/5bcead9b66b11.gif) 可能是我 OI 题做傻了，以为输入一定要一连串不停地输入...导致我一直没有理解为什么两个程序之间可以来回通信...

（~~上面那段话纯属我sb了，请跳过不看~~）

## 通信题是……

根据我的理解，通信题就是：两个程序，分别从文件读入数据，从标准输入读入另一个程序的标准输出。最后其中一个程序按要求输出到文件。

（好像跟题面描述的差不多...）

那就说说我sb了而卡住的地方好了..一个程序 getchar() 的时候会暂停执行，直到另一个程序输出，就跟手动输入数据时等待回车一样。

## 关于调试

广二机房是 win7，不太方便复制...

于是只好手动输入了，只不过感觉 copy 两个程序的输出看它们相互配合着工作，还是蛮有趣的。

# Part 2 解法

其实我 $5min$ 就想到怎么做了..（但好像做法比最短解那些神仙做法麻烦的多？）其实也不是很难写，但由于第一次写通信题不太习惯，各种细节写挂，最后写了一个小时才A...

## 解决问题

可以想到这样一个做法（如果想不到也没关系..看懂就好了）：选择一个区间 $[l,r]$ ，Bob 用一个长度为 $r-l+1$ 的 $01$ 串告诉 Alice 这个区间内每个位置是否是一个下标，对于每个下标 Alice 告诉 Bob 对应的值；对于不在 $[l,r]$ 内的其它部分，Alice 把所有值（不管是不是一个下标）都告诉 Bob .

## 优化通信数

考虑一下，这样做需要的 bit 数是多少：

Bob 给 Alice 的：首先 Bob 要告诉 Alice $l$ 和 $r$ , 用二进制表示，需要 $22$ 个 bit；其次，Bob 要询问 $[l,r]$ ，需要 $r-l+1$ 个 bit 。

Alice 给 Bob 的：首先 Alice 要回答 Bob 在 $[l,r]$ 内的询问，需要 “ $[l,r]$ 内下标个数” 个 bit；其次，Alice 要告诉 Bob 除了 $[l,r]$ 其它区域的所有值，需要 $n-(r-l+1)$ 个 bit 。

那么，我们需要最小化：$\max(r-l+23,[l,r]\text{ 内下标个数}+n-r+l-1)$ 。

用 $[l,l+len)$ 来表示会简洁一些，所以下文都使用这种方式，即需要最小化：$\max(len+22,[l,l+len)\text{ 内下标个数}+n-len)$ 。

预处理前缀和即可快速算出 $[l,l+len)$ 内的下标个数，$O(n^2)$ 枚举区间取最小值即可。

## 参考代码

### Bob

```cpp
#include <iostream>
#include <fstream>
#include <cstdio>
#include <algorithm>
#include <vector>

using namespace std;

ifstream fin("bob.in");
ofstream fout("bob.out");

char rd() //读入一个bit
{
    return getchar();
}

void wt(int x) //输出一个bit
{
    putchar(x+'0');
    fflush(stdout);
}

int n,m,p[1010],rk[4010],pre[4010],minn=0x7fffffff,l,len;
char ans[1010];

int main()
{
    int i,j,t;

    fin>>n>>m;

    for (i=1;i<=n;++i) //读入并记录是第几个下标（便于存答案），并且复制一份拼在后面，这样如果询问的区间跨过首尾可以方便地处理
    {
        fin>>p[i];
        rk[p[i]]=rk[p[i]+2*n]=i;
    }

    for (i=1;i<=4*n;++i) //预处理前缀和
    {
        pre[i]=pre[i-1]+(rk[i]>0);
    }

    for (i=1;i<=2*n;++i) //枚举找最优方案
    {
        for (j=1;j<=2*n;++j)
        {
            t=max(pre[i+j-1]-pre[i-1]+2*n-j,j+22);
            if (t<minn)
            {
                minn=t;
                l=i;
                len=j;
            }
        }
    }

    for (i=10;i>=0;--i) //告诉Alice l和len
    {
        wt(bool((1<<i)&l));
    }

    for (i=10;i>=0;--i)
    {
        wt(bool((1<<i)&len));
    }

    for (i=l;i<l+len;++i) //询问区间
    {
        if (rk[i])
        {
            wt(1);
            ans[rk[i]-1]=rd(); //存答案
        }
        else
        {
            wt(0);
        }
    }

    for (i=l+len;i<l+2*n;++i) //读取剩余部分
    {
        if (rk[i])
        {
            ans[rk[i]-1]=rd();
        }
        else
        {
            rd();
        }
    }

    fout<<ans;

    return 0;
}
```

### Alice

```cpp
#include <iostream>
#include <fstream>
#include <cstdio>

using namespace std;

ifstream fin("alice.in");

int rd() //为了方便，两个程序中rd()和wt()的char/int是反的
{
    return getchar()-'0';
}

void wt(char x)
{
    putchar(x);
    fflush(stdout);
}

int n,m;
char s[4010];

int main()
{
    int i,l=0,len=0;

    fin>>n>>m>>(s+1);

    for (i=2*n+1;i<=4*n;++i) //复制一遍放在后面
    {
        s[i]=s[i-2*n];
    }

    for (i=0;i<=10;++i) //读入l和len
    {
        l=l*2+rd();
    }

    for (i=0;i<=10;++i)
    {
        len=len*2+rd();
    }

    for (i=l;i<l+len;++i) //回答询问
    {
        if (rd())
        {
            wt(s[i]);
        }
    }

    for (i=l+len;i<l+2*n;++i) //告诉Bob剩下的部分
    {
        wt(s[i]);
    }

    return 0;
}
```

# Part 3 证明

取询问区间为 $[l,l+1318]$ ，这样的话 Bob 发给 Alice  的 bit 数就为 $1319+22=1341$ .

区间 $[l,l+1318]$ 内下标的期望个数为 $\frac{1319}2$ ，所以一定存在某个区间使得下标个数小于等于 $659$ ，再加上剩余部分 $681$ ，Alice 发给 Bob 的 bit 数就为 $1340$ 。

事实上，我提交的评测记录里通信次数最多的就是 $1341+1340$ 。

# Part 4 优化

只取 $len=1325$ ，少枚举一维，可以优化到时间复杂度 $O(n)$ ；少传 $11$ 个 bit，可以优化到最大通信次数 $1336+1337$ 。证明从略。 

[提交记录](https://uoj.ac/submission/309171)