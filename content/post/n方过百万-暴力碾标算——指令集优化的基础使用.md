+++
title = "n方过百万 暴力碾标算——指令集优化的基础使用"
date = "2019-02-01T14:26:52+08:00"
categories = ["黑科技"]
tags = ["常数优化"]
description = ""
+++


感谢 yfz 和 mcfx 在 WC 营员交流上的分享！

然而只看那个课件来学习指令集好像略有困难..所以我来分享一下~~我自学一晚上的成果~~。

希望能帮助大家暴力过题，~~考场上再也写不出标算~~。

<!-- more -->

> 注：本文省略了无数个 `unsigned`，请自行把所有 `int` 视作 `unsigned int`，把所有 `long long` 视作 `unsigned long long`。

# 适用范围

## 环境

**不要尝试在正式OI竞赛中使用指令集优化。**

只适用于提供资瓷的 OJ，具体列表参照营员交流ppt：

![ojzc1](/post_img/n方过百万-暴力碾标算——指令集优化的基础使用/ojzc1.png)

![ojzc2](/post_img/n方过百万-暴力碾标算——指令集优化的基础使用/ojzc2.png)

sse2，avx 什么的都是指令集的名字。

## 作用

适用于**方便对连续内存空间进行批量处理**的题目。大约可以视作每 $8$ 个 int 为一个分块，块内进行赋值、修改等操作常数为 $1$，也就实现了常数/=$8$。当然如果是 long long 就只能除以四。

# 具体使用

## define & pragma & include

```cpp
#define __AVX__ 1
#define __AVX2__ 1
#define __SSE__ 1
#define __SSE2__ 1
#define __SSE2_MATH__ 1
#define __SSE3__ 1
#define __SSE4_1__ 1
#define __SSE4_2__ 1
#define __SSE_MATH__ 1
#define __SSSE3__ 1

#pragma GCC optimize("Ofast,no-stack-protector,unroll-loops,fast-math")
#pragma GCC target("sse,sse2,sse3,ssse3,sse4.1,sse4.2,avx,avx2,popcnt,tune=native")

#include <immintrin.h>
#include <emmintrin.h>
```

第一行是优化，如果你都用指令集了当然是能优化尽量优化。

第二行是告诉编译器你要使用指令集。

后面两个头文件是 C++ 将指令集封装成了函数，这样就不用在代码中内联汇编了。

## 变量类型

大约有 `__m256i` `__m256` `__m256d` 三种，分别存储 `long long`，`float` 和 `double`，实际上 `__m256i` 也可以用来存储 `int`。

## 指令使用

可以在[一个神奇的网站](https://software.intel.com/sites/landingpage/IntrinsicsGuide)查需要的指令，左边选指令集以及指令类型，右边是指令，点开指令可以查看函数原型以及伪代码。

这里列几条常用指令：

`__m256i _mm256_set_epi32 (int e7, int e6, int e5, int e4, int e3, int e2, int e1, int e0)`：参数是八个数，也就是一个“分块”里的数，注意是逆序的。返回值是一个含这八个数的“分块”。

`__m256i _mm256_set_epi64x (__int64 e3, __int64 e2, __int64 e1, __int64 e0)`：和上面一样，只不过是 $64$ 位整数，也就是 long long。

`__m256i _mm256_set1_epi32 (int a)`：相当于 `_mm256_set_epi32(a,a,a,a,a,a,a,a)`。

`__m256i _mm256_add_epi32 (__m256i a, __m256i b)`：把两个“分块”的对应位置分别相加，返回结果。

`__m256i _mm256_cmpeq_epi32 (__m256i a, __m256i b)`：判断两个“分块”的对应位置是否相等，若相等则返回的“分块”对应位置是 `0xffffffff`，否则是 `0`。

`__m256i _mm256_cmpgt_epi32 (__m256i a, __m256i b)`：和上面一样，只不过比较符是大于而不是相等。

`__m256i _mm256_and_si256 (__m256i a, __m256i b)`：返回两个“分块”的按位与，可以配合上面两条比较指令来使用。

## 访问数据

可以直接通过下标访问：

```cpp
#include <cstdio>

#pragma GCC optimize("Ofast,no-stack-protector,unroll-loops,fast-math")
#pragma GCC target("sse,sse2,sse3,ssse3,sse4.1,sse4.2,avx,avx2,popcnt,tune=native")

#include <immintrin.h>
#include <emmintrin.h>

__m256i a;

int main()
{
    a=_mm256_set_epi32(1,2,3,4,5,6,7,8);
    printf("%d",a[2]);
    return 0;
}
```

你们可以猜猜这个的结果是什么。

答案是..4。

为什么呢，首先 `_mm256_set_epi32` 的参数是逆序的，所以实际上存储的数顺序是 `8,7,6,5,4,3,2,1`。其次，`__m256i` 类型是存储 long long 的，所以直接通过下标访问实际上是在访问 long long，如果 `cout<<a[2]`就会返回 `12884901892`（$3\times2^{32}+4$）。所以，这句话实际上是在 `printf("%d",12884901892ll);`。

那么如何访问 `int`（甚至 `short`，如果题目允许这样就可以常数除以 $16$）呢？

其实搞个指针就可以了：

```cpp
a=_mm256_set_epi32(1,2,3,4,5,6,7,8);
int *b=(int *)&a;
printf("%d",b[2]);
```

这样子的输出就是 $6$ 了。

用这种方法就可以方便地处理序列问题了：

```cpp
#include <cstdio>

#pragma GCC optimize("Ofast,no-stack-protector,unroll-loops,fast-math")
#pragma GCC target("sse,sse2,sse3,ssse3,sse4.1,sse4.2,avx,avx2,popcnt,tune=native")

#include <immintrin.h>
#include <emmintrin.h>

int* b;
__m256i a[10],x;

int main()
{
    int i;

    b=(int *)&a;

    for (i=0;i<80;++i) scanf("%d",b+i);

    x=_mm256_set1_epi32(233);

    for (i=0;i<10;++i) a[i]=_mm256_add_epi32(a[i],x);

    for (i=0;i<80;++i) printf("%d ",b[i]);

    return 0;
}
```

上面是一个简单的示例，读入 $80$ 个数，然后输出它们加上 $233$ 的结果。

# 例题

~~这种东西为什么还会有例题啊。~~

[教主的魔法](https://www.luogu.org/problemnew/show/P2801)，这题比较简单（~~废话暴力当然简单~~）。

[【模板】线段树1](https://www.luogu.org/problemnew/show/P3372)，这题其实是最简单的，然而由于 dl 出题人把值域搞到了 long long，常数只能除以四，需要卡卡常，多提交几次才能过。

[[Ynoi2018]五彩斑斓的世界](https://www.luogu.org/problemnew/show/P4117)，神司怒艹lxl标算的课件例题。

[Simple Tree](http://uoj.ac/problem/435)，这个还要树剖，只不过也还好，神司是直接内嵌汇编写的，没有测过用函数能不能过..

然后以教主的魔法为例讲一下代码：

```cpp
#include <cstdio>

#pragma GCC optimize("Ofast,no-stack-protector,unroll-loops,fast-math")
#pragma GCC target("sse,sse2,sse3,ssse3,sse4.1,sse4.2,avx,avx2,popcnt,tune=native")

#include <immintrin.h>
#include <emmintrin.h>

const int N=1000010;

int n,m,tot,*a;
__m256i A[N>>3];
char op[10];

void modify(int l,int r,int x)
{
    while ((l&7)&&l<r) a[l++]+=x; //处理左边不是整块的部分，和分块的处理方法是一样的
    if (l==r) return;
    while (r&7) a[--r]+=x; //处理右边不是整块的部分
    if (l==r) return;
    __m256i t=_mm256_set1_epi32(x); //剩下的部分整块加上x
    for (l>>=3,r>>=3;l<r;++l) A[l]=_mm256_add_epi32(A[l],t);
}

int query(int l,int r,int x)
{
    int out=0;
    while ((l&7)&&l<r) out+=int(a[l++]>=x); //处理左边不是整块的部分
    if (l==r) return out;
    while (r&7) out+=int(a[--r]>=x); //处理右边不是整块的部分
    if (l==r) return out;
    __m256i t=_mm256_set1_epi32(1); //这个1是每个大于等于x的数的贡献
    __m256i ans=_mm256_set1_epi32(0); //这个ans是用来存答案的
    __m256i cp=_mm256_set1_epi32(x-1); //这个是用来比较的，题目中是大于等于，所以和x-1比较
    for (l>>=3,r>>=3;l<r;++l) ans=_mm256_add_epi32(ans,_mm256_and_si256(t,_mm256_cmpgt_epi32(A[l],cp))); //这个意会一下，作用是数当前块有几个大于x-1的数
    for (int i=0;i<4;++i) out+=(ans[i]&0xffffffff)+(ans[i]>>32); //最后统计答案，因为ans[i]是一个long long，所以要前32位和后32位分别统计
    return out;
}

int main()
{
    int i,l,r,x,aa[8];

    scanf("%d%d",&n,&m);

    a=(int*)&A;

    for (i=0;i<n;++i) scanf("%d",a+i);

    while (m--)
    {
        scanf("%s%d%d%d",op,&l,&r,&x);
        if (op[0]=='M') modify(l-1,r,x);
        else printf("%d\n",query(l-1,r,x));
    }

    return 0;
}
```