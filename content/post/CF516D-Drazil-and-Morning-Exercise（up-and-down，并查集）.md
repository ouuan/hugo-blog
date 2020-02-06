+++
title = "CF516D Drazil and Morning Exercise（up and down，并查集）"
categories = ["题解"]
tags = ["up and down", "并查集"]
date = "2019-09-10T13:37:12+08:00"
description = ""
+++


## 题目链接

[CF](http://codeforces.com/contest/516/problem/D)

[洛谷](https://www.luogu.org/problem/CF516D)

## 题意简述

给定一棵 $n$ 个点带边权的树，定义 $d(u)$ 为树上离它最远的点到它的距离，$q$ 次询问，每次询问给定 $l$，求一个最大的树上连通块 $V'$ 的大小，满足 $\forall u, v\in V'$，$|d(u)-d(v)|\le l$。

$1\le n\le 10^5$, $1\le q\le 50$。

<!--more-->

## 简要做法

首先使用 up and down（两遍 dfs，第一遍求往下走的最远距离，第二遍求往上走的最远距离）求出 $d(u)$。

有一个性质：如果以 $d(u)$ 最小的 $u$ 为根，$\forall v, d(v)\ge d(parent(v))$。（下文中的“子树”都是以 $d$ 最小的点为根的。）

{{% admonition note "简单证明" true %}}

不难发现，我们只要证明 $u$ 的每个儿子 $v$ 都是子树 $v$ 中 $d$ 最小的，即可归纳地证明原命题。

假设子树 $v$ 中存在一个点 $w$，$d(w)<d(v)$，那么从 $v$ 出发的最长路的第一步一定是 $v$ 到 $w$ 的路径上的第一条边（否则的话，从 $w$ 出发可以走到 $v$ 再走从 $v$ 出发的最长路，就会导致 $d(w)>d(v)$），这样的话，从 $u$ 出发也可以先走到 $v$ 再走 $v$ 出发的最长路，这样的话 $d(u)>d(v)$，与题设矛盾。

{{% /admonition %}}

题目所求的连通块一定是 $d$ 的大小连续的一段，令所求连通块中 $d$ 最小且离根最近的点为 $u$，由上面的性质不难发现，所求连通块一定在子树 $u$ 中。

如果按 $d$ 从大到小枚举每个点，用并查集维护符合条件的点的连通性以及每块的大小，可以发现，删去一个点（一个 $d>d(u)+l$ 的点）并不影响连通性，只需要将其所在块的大小减一即可。（加入一个点就是合并子树，删去一个点是删去叶子。）

这样的话答案就是过程中最大的一块的大小。

总时间复杂度就是 $O(qn\alpha(n)+n\log n)$ ~~或 O(qn+nlogn)~~ 或 $O(qn\log n)$。

## 参考代码

（非常抱歉，使用了 CF 模板 qaq）

```cpp
#ifndef OUUAN
#pragma GCC optimize("Ofast,no-stack-protector,unroll-loops,fast-math")
#endif
#include<bits/stdc++.h>

#define int LoveLive
//#define FAST_IOSTREAM 1

#define For(i,l,r)for(int i=(l),i##end=(r);i<=i##end;++i)
#define FOR(i,r,l)for(int i=(r),i##end=(l);i>=i##end;--i)
#define SON(i,u)for(int i=head[u];i;i=nxt[i])
#define ms(a,x)memset(a,x,sizeof(a))
#define fi first
#define se second
#define pb emplace_back
#define pq priority_queue
#define isinf(x)(x>=INF?-1:x)
#define y1 why_is_there_a_function_called_y1
#define DEBUG(x)cerr<<(#x)<<":"<<x<<endl
using namespace std;
typedef long long LoveLive;typedef pair<int,int>pii;typedef vector<int>vi;
#ifdef int
const int INF=0x3f3f3f3f3f3f3f3fll;
#else
const int INF=0x3f3f3f3f;
#endif
const double eps=1e-9;mt19937 rng(chrono::steady_clock::now().time_since_epoch().
count());int randint(int l,int r){int out=rng()%(r-l+1)+l;return out>=l?out:out+
r-l+1;}
#ifdef FAST_IOSTREAM
#define br cout<<'\n'
#define sp cout<<' '
long long read(){long long x;cin>>x;return x;}template<typename T>void read(T&x){
cin>>x;}template<typename T>void write(const T&x){cout<<x;}
#else
#define br putchar('\n')
#define sp putchar(' ')
template<typename T>typename enable_if<!is_integral<T>::value,void>::type read(T
&x){cin>>x;}long long read(){char c;long long out=0,f=1;for(c=getchar();!isdigit
(c)&&c!='-';c=getchar());if(c=='-'){f=-1;c=getchar();}for(;isdigit(c);c=getchar())
out=(out<<3)+(out<<1)+c-'0';return out*f;}template<typename T>typename enable_if
<is_integral<T>::value,T>::type read(T&x){char c;T f=1;x=0;for(c=getchar();!isdigit
(c)&&c!='-';c=getchar());if(c=='-'){f=-1;c=getchar();}for(;isdigit(c);c=getchar())
x=(x<<3)+(x<<1)+c-'0';return x*=f;}char read(char&x){for(x=getchar();isspace(x);
x=getchar());return x;}double read(double&x){scanf("%lf",&x);return x;}template<
typename T>typename enable_if<!is_integral<T>::value,void>::type write(const T&x
){cout<<x;}template<typename T>typename enable_if<is_integral<T>::value,void>::type
write(const T&x){if(x<0){putchar('-');write(-x);return;}if(x>9)write(x/10);putchar
(x%10+'0');}void write(const char&x){putchar(x);}void write(const double&x){printf
("%.10lf",x);}
#endif
template<typename T,typename...Args>void read(T&x,Args&...args){read(x);read(args
...);}template<typename OutputIt,typename=typename enable_if<is_same<output_iterator_tag
,typename iterator_traits<OutputIt>::iterator_category>::value||(
is_base_of<forward_iterator_tag,typename iterator_traits<OutputIt>::iterator_category>::value
&&!is_const<OutputIt>::value)>::type>void read(OutputIt __first,OutputIt __last){for(;__first
!=__last;++__first)read(*__first);}template<typename InputIt,typename=typename enable_if
<is_base_of<input_iterator_tag,typename iterator_traits<InputIt>::iterator_category
>::value>::type>void wts(InputIt __first,InputIt __last){bool isFirst=true;for(;
__first!=__last;++__first){if(isFirst)isFirst=false;else sp;write(*__first);}br;}
template<typename InputIt,typename=typename enable_if<is_base_of<input_iterator_tag
,typename iterator_traits<InputIt>::iterator_category>::value>::type>void wtb(InputIt
__first,InputIt __last){for(;__first!=__last;++__first){write(*__first);br;}}template
<typename T>void wts(const T&x){write(x);sp;}template<typename T>void wtb(const
T&x){write(x);br;}template<typename T>void wte(const T&x){write(x);exit(0);}template
<typename T,typename...Args>void wts(const T&x,Args...args){wts(x);wts(args...);}
template<typename T,typename...Args>void wtb(const T&x,Args...args){wts(x);wtb(args
...);}template<typename T,typename...Args>void wte(const T&x,Args...args){wts(x);
wte(args...);}template<typename T>inline bool up(T&x,const T&y){return x<y?x=y,1
:0;}template<typename T>inline bool dn(T&x,const T&y){return y<x?x=y,1:0;}

const int N = 100010;
const int mod = 1000000007;

int head[N], nxt[N << 1], to[N << 1], edge[N << 1], cnt;
int n, f[N], siz[N], id[N], rid[N];

struct Element
{
    int fi, se;
    Element() { fi = 0; se = -INF; }
    void insert(int x)
    {
        if (x > fi)
        {
            se = fi;
            fi = x;
        }
        else if (x > se) se = x;
    }
    int get(int x)
    {
        if (x == fi) return se;
        return fi;
    }
} dis[N];

void dfsdn(int u, int fa)
{
    SON (i, u)
    {
        int v = to[i];
        int w = edge[i];
        if (v == fa) continue;
        dfsdn(v, u);
        dis[u].insert(dis[v].fi + w);
    }
}

void dfsup(int u, int fa)
{
    SON (i, u)
    {
        int v = to[i];
        int w = edge[i];
        if (v == fa) continue;
        dis[v].insert(dis[u].get(dis[v].fi + w) + w);
        dfsup(v, u);
    }
}

void add(int u, int v, int w)
{
    nxt[++cnt] = head[u];
    head[u] = cnt;
    to[cnt] = v;
    edge[cnt] = w;
}

int find(int x)
{
    return x == f[x] ? x : f[x] = find(f[x]);
}

void merge(int x, int y)
{
    if (find(x) == find(y)) return;
    if (siz[find(x)] < siz[find(y)]) swap(x, y);
    siz[find(x)] += siz[find(y)];
    f[find(y)] = find(x);
}

signed main()
{
    #ifdef FAST_IOSTREAM
    cin.sync_with_stdio(false);
    cin.tie(0);
    #endif

    read(n);

    For (i, 1, n - 1)
    {
        int u, v, w;
        read(u, v, w);
        add(u, v, w);
        add(v, u, w);
    }

    dfsdn(1, 0);
    dfsup(1, 0);

    For (i, 1, n) id[i] = i;
    sort(id + 1, id + n + 1, [](int x, int y){return dis[x].fi > dis[y].fi;});
    For (i, 1, n) rid[id[i]] = i;

    int q = read();

    while (q--)
    {
        int len = read();
        int l = 1;
        int ans = 0;
        For (i, 1, n)
        {
            f[i] = i;
            siz[i] = 1;
        }
        For (u, 1, n)
        {
            while (dis[id[l]].fi > dis[id[u]].fi + len) --siz[find(id[l++])];
            SON (i, id[u])
            {
                int v = to[i];
                if (rid[v] < u) merge(id[u], v);
            }
            up(ans, siz[find(id[u])]);
        }
        wtb(ans);
    }

    return 0;
}
```
