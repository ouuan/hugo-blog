+++
title = "CF1260F Colored Tree（点分治，差分，基数排序）"
categories = ["题解"]
tags = ["点分治", "差分", "基数排序", "数据结构", "概率期望", "分治"]
date = "2019-11-29T14:00:53+08:00"
description = ""
aliases = ["/post/CF1260F-Colored-Tree（点分治，差分，基数排序）", "/CF1260F-Colored-Tree（点分治，差分，基数排序）"]
+++


## 题目链接

[CF](https://codeforces.com/contest/1260/problem/F)

## 题意简述

给你一棵树，每个点的颜色为一个区间内的整数，一种染色方案的权值是所有同色无序点对的距离之和，求所有不同染色方案的权值之和。

点数、颜色数均不超过 $10^5$ 。

<!--more-->

## 简要做法

首先是 $O(n^2)$ 做法：把计算总和转化为计算期望，然后就可以枚举点对，计算同色概率，乘上距离，最后把期望乘上方案数就是答案。进行 $n$ 次 DFS 而不是每次求 LCA 就可以做到 $O(n^2)$ 。

然后是 $O(n\log n\log c)$（$c$ 为颜色数）做法：点分治，令 $f(u, v)=\max(0, \min(r_u, r_v)-\max(l_u, l_v)+1)$, 点 $u$ 的贡献是 $\sum_{v}f(u, v)(dep_u+dep_v)(r_u-l_u+1)^{-1}(r_v-l_v+1)^{-1}$，即 $\left(dep_u\cdot(r_u-l_u+1)^{-1}\right)\cdot\left(\sum_{v}f(u, v)(r_v-l_v+1)^{-1}\right)+(r_u-l_u+1)^{-1}\cdot\left(\sum_{v}f(u, v)dep_v(r_v-l_v+1)^{-1}\right)$ ，用以颜色为下标、支持区间加/查询区间和的线段树维护 $\sum_{v}(r_v-l_v+1)^{-1}$ 以及 $\sum_{v}dep_v(r_v-l_v+1)^{-1}$ 即可。

如果你写的是其它 $O(n\log n\log c)$ 做法，如链分治（dsu on tree），或者你的常数比较小（比如使用树状数组而不是线段树），你可能就过了。

否则，你可能 [TLE on 6](https://codeforces.com/contest/1260/submission/65895673) 或者 [TLE on 14](https://codeforces.com/contest/1260/submission/65896002) 之类的。

但是，大常数选手并不是没有活路的，因为这题有 $O(n\log n)$ 做法。

其中一种，是使用差分代替上述做法中的线段树。但如果在点分治内进行比较排序，复杂度是 $O(n\log^2 n)$（尽管瓶颈部分的常数很小），解决方法是在点分治的外部进行一次排序，然后在内部就可以线性地将每棵子树的修改与询问划分开来。

“划分”本质上是一个双关键字排序，第一关键字为所在子树的编号，第二关键字为修改/询问原本的排序方式，利用基数排序的思想，由于第二关键字已经有序，对第一关键字进行稳定排序即可。使用计数排序可以做到线性复杂度，总复杂度就是 $O(n\log n)$ 。

## 参考代码

{{% admonition note "nlognlogc" true %}}

```cpp
																																																	#ifndef OUUAN
																																																	#pragma GCC optimize("Ofast,no-stack-protector,unroll-loops,fast-math")
																																																	#endif
																																																	#include<bits/stdc++.h>
 
//#define int LoveLive
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
																																																	using namespace std;typedef long long LoveLive;typedef pair<int,int>pii;typedef vector<int>vi;typedef long double ld;const double inf=1e121;const double eps=1e-10;mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());int randint(int l,int r){int out=rng()%(r-l+1)+l;return out>=l?out:out+r-l+1;}
																																																	#ifdef int
																																																	const int INF=0x3f3f3f3f3f3f3f3fll;
																																																	#else
																																																	const int INF=0x3f3f3f3f;typedef long long ll;
																																																	#endif
																																																	#ifdef FAST_IOSTREAM
																																																	#define br cout<<'\n'
																																																	#define sp cout<<' '
																																																	long long read(){long long x;cin>>x;return x;}template<typename T>void read(T&x){cin>>x;}template<typename T>void write(const T&x){cout<<x;}
																																																	#else
																																																	#define br putchar('\n')
																																																	#define sp putchar(' ')
																																																	template<typename T>typename enable_if<!is_integral<T>::value,void>::type read(T&x){cin>>x;}long long read(){char c;long long out=0,f=1;for(c=getchar();!isdigit(c)&&c!='-';c=getchar());if(c=='-'){f=-1;c=getchar();}for(;isdigit(c);c=getchar())out=(out<<3)+(out<<1)+c-'0';return out*f;}template<typename T>typename enable_if<is_integral<T>::value,T>::type read(T&x){char c;T f=1;x=0;for(c=getchar();!isdigit(c)&&c!='-';c=getchar());if(c=='-'){f=-1;c=getchar();}for(;isdigit(c);c=getchar())x=(x<<3)+(x<<1)+c-'0';return x*=f;}char read(char&x){for(x=getchar();isspace(x);x=getchar());return x;}double read(double&x){scanf("%lf",&x);return x;}ld read(ld&x){scanf("%Lf",&x);return x;}template<typename T>typename enable_if<!is_integral<T>::value,void>::type write(const T&x){cout<<x;}template<typename T>typename enable_if<is_integral<T>::value,void>::type write(const T&x){if(x<0){putchar('-');write(-x);return;}if(x>9)write(x/10);putchar(x%10+'0');}void write(const char&x){putchar(x);}void write(const double&x){printf("%.10lf",x);}void write(const ld&x){printf("%.10Lf",x);}
																																																	#endif
																																																	template<typename T,typename...Args>void read(T&x,Args&...args){read(x);read(args...);}template<typename OutputIt,typename=typename enable_if<is_same<output_iterator_tag,typename iterator_traits<OutputIt>::iterator_category>::value||(is_base_of<forward_iterator_tag,typename iterator_traits<OutputIt>::iterator_category>::value&&!is_const<OutputIt>::value)>::type>void read(OutputIt __first,OutputIt __last){for(;__first!=__last;++__first)read(*__first);}template<typename InputIt,typename=typename enable_if<is_base_of<input_iterator_tag,typename iterator_traits<InputIt>::iterator_category>::value>::type>void wts(InputIt __first,InputIt __last){bool isFirst=true;for(;__first!=__last;++__first){if(isFirst)isFirst=false;else sp;write(*__first);}br;}template<typename InputIt,typename=typename enable_if<is_base_of<input_iterator_tag,typename iterator_traits<InputIt>::iterator_category>::value>::type>void wtb(InputIt __first,InputIt __last){for(;__first!=__last;++__first){write(*__first);br;}}template<typename T>void wts(const T&x){write(x);sp;}template<typename T>void wtb(const T&x){write(x);br;}template<typename T>void wte(const T&x){write(x);exit(0);}template<typename T,typename...Args>void wts(const T&x,Args...args){wts(x);wts(args...);}template<typename T,typename...Args>void wtb(const T&x,Args...args){wts(x);wtb(args...);}template<typename T,typename...Args>void wte(const T&x,Args...args){wts(x);wte(args...);}template<typename T>inline bool up(T&x,const T&y){return x<y?x=y,1:0;}template<typename T>inline bool dn(T&x,const T&y){return y<x?x=y,1:0;}template<typename T>inline bool inRange(const T&x,const T&l,const T&r){return!(x<l)&&!(r<x);}template<typename valueType,typename tagType>class segmentTreeNode{public:int id,left,right;valueType val;tagType tag;};template<typename valueType, typename tagType, valueType(*merge)(valueType,valueType), void(*update)(segmentTreeNode<valueType,tagType>&,tagType)>class segmentTree{private:std::vector<segmentTreeNode<valueType,tagType>>nodes;int leftRange,rightRange;valueType valueZero;tagType tagZero;void pushup(int cur){nodes[cur].val=merge(nodes[cur<<1].val,nodes[cur<<1|1].val);}void pushdown(int cur){update(nodes[cur<<1],nodes[cur].tag);update(nodes[cur<<1|1],nodes[cur].tag);nodes[cur].tag=tagZero;}void build(int cur,int l,int r,const std::vector<valueType>&initValue){nodes[cur].id=cur;nodes[cur].left=l;nodes[cur].right=r;nodes[cur].tag=tagZero;if(l==r-1)nodes[cur].val=initValue[l-leftRange];else{build(cur<<1,l,(l+r)>>1,initValue);build(cur<<1|1,(l+r)>>1,r,initValue);pushup(cur);}}void init(const std::vector<valueType>&_initValue,const valueType&_valueZero,const tagType&_tagZero){valueZero=_valueZero;tagZero=_tagZero;nodes.resize((rightRange-leftRange)<<2);build(1,leftRange,rightRange,_initValue);}void modify(int cur,int l,int r,int L,int R,const tagType&tag){if(l>=R||r<=L)return;if(L<=l&&r<=R)update(nodes[cur],tag);else{pushdown(cur);modify(cur<<1,l,(l+r)>>1,L,R,tag);modify(cur<<1|1,(l+r)>>1,r,L,R,tag);pushup(cur);}}valueType query(int cur,int l,int r,int L,int R){if(l>=R||r<=L)return valueZero;if(L<=l&&r<=R)return nodes[cur].val;pushdown(cur);return merge(query(cur<<1,l,(l+r)>>1,L,R), query(cur<<1|1,(l+r)>>1,r,L,R));}public:segmentTree(){}segmentTree(int _leftRange,int _rightRange,const std::vector<valueType>&_initValue,const valueType&_valueZero,const tagType&_tagZero){leftRange=_leftRange;rightRange=_rightRange;init(_initValue,_valueZero,_tagZero);}segmentTree(int size,const std::vector<valueType>&_initValue,const valueType&_valueZero,const tagType&_tagZero){leftRange=1;rightRange=size+1;init(_initValue,_valueZero,_tagZero);}void modify(int l,int r,const tagType&tag){modify(1,leftRange,rightRange,l,r,tag);}void modify(int p,const tagType&tag){modify(p,p+1,tag);}valueType query(int l,int r){return query(1,leftRange,rightRange,l,r);}valueType query(int p){return query(p,p+1);}};class maxFlow{private:typedef long long ll;std::queue<int>q;std::vector<int>head,cur,nxt,to,dep;std::vector<ll>cap;public:maxFlow(int _n=0){init(_n);}void init(int _n){head.clear();head.resize(_n+1,0);nxt.resize(2);to.resize(2);cap.resize(2);}void init(){init(head.size()-1);}void add(int u,int v,ll w){nxt.push_back(head[u]);head[u]=to.size();to.push_back(v);cap.push_back(w);}void Add(int u,int v,ll w){add(u,v,w);add(v,u,0);}void del(int x){cap[x<<1]=cap[x<<1|1]=0;}bool bfs(int s,int t){dep.clear();dep.resize(head.size(),-1);dep[s]=0;q.push(s);while(!q.empty()){int u=q.front();q.pop();for(int i=head[u];i;i=nxt[i]){int v=to[i];ll w=cap[i];if(w>0&&dep[v]==-1){dep[v]=dep[u]+1;q.push(v);}}}return ~dep[t];}ll dfs(int u,ll flow,int t){if(dep[u]==dep[t])return u==t?flow:0;ll out=0;for(int&i=cur[u];i;i=nxt[i]){int v=to[i];ll w=cap[i];if(w>0&&dep[v]==dep[u]+1){ll f=dfs(v,std::min(w,flow-out),t);cap[i]-=f;cap[i ^ 1]+=f;out+=f;if(out==flow)return out;}}return out;}ll maxflow(int s,int t){ll out=0;while(bfs(s,t)){cur=head;out+=dfs(s,0x7fffffffffffffffll,t);}return out;}ll getflow(int x)const{return cap[x<<1|1];}};struct customHash{static uint64_t splitmix64(uint64_t x){x+=0x9e3779b97f4a7c15;x=(x ^(x>>30))*0xbf58476d1ce4e5b9;x=(x ^(x>>27))*0x94d049bb133111eb;return x ^(x>>31);}size_t operator()(uint64_t x)const{static const uint64_t FIXED_RANDOM=rng();return splitmix64(x+FIXED_RANDOM);}};
 
typedef pair<bool, pii> pbii;

const int W = 100000;
const int mod = (1e9 + 7//, 998244353
);

vector<vi> g;
vector<bool> vis;
int n, tsiz, rt, ans;
vi l, r, siz, wt, inv;

int qpow(int x, int y)
{
    int out = 1;
    while (y)
    {
        if (y & 1) out = (ll) out * x % mod;
        x = (ll) x * x % mod;
        y >>= 1;
    }
    return out;
}

int modadd(int x, int y)
{
    return (x += y) >= mod ? x - mod : x;
}

pii merge(pii x, pii y)
{
    return pii(modadd(x.fi, y.fi), modadd(x.se, y.se));
}

void update(segmentTreeNode<pii, pbii>& u, pbii x)
{
    if (x.first)
    {
        u.tag = pbii(true, pii(0, 0));
        u.val = pii(0, 0);
    }
    else if (x.se == pii(0, 0)) return;
    u.tag.se = merge(u.tag.se, x.se);
    u.val.fi = (u.val.fi + (ll) x.se.fi * (u.right - u.left)) % mod;
    u.val.se = (u.val.se + (ll) x.se.se * (u.right - u.left)) % mod;
}

segmentTree<pii, pbii, merge, update> t(W, vector<pii>(W, pii(0, 0)), pii(0, 0), pbii(false, pii(0, 0)));

void getroot(int u, int pa)
{
    siz[u] = wt[u] = 1;
    for (auto v : g[u])
    {
        if (v == pa || vis[v]) continue;
        getroot(v, u);
        siz[u] += siz[v];
        up(wt[u], siz[v]);
    }
    up(wt[u], tsiz - siz[u]);
    if (!rt || wt[u] < wt[rt]) rt = u;
}

void calc(int u, int pa, int dep)
{
    auto res = t.query(l[u], r[u] + 1);
    ans = (ans + (ll) inv[u] * dep % mod * res.fi + (ll) inv[u] * res.se) % mod;
    for (auto v : g[u])
    {
        if (v == pa || vis[v]) continue;
        calc(v, u, dep + 1);
    }
}

void insert(int u, int pa, int dep)
{
    t.modify(l[u], r[u] + 1, pbii(false, pii(inv[u], (ll) inv[u] * dep % mod)));
    for (auto v : g[u])
    {
        if (v == pa || vis[v]) continue;
        insert(v, u, dep + 1);
    }
}

void solve(int u)
{
    vis[u] = true;
    t.modify(1, W + 1, pbii(true, pii(0, 0)));
    t.modify(l[u], r[u] + 1, pbii(false, pii(inv[u], 0)));
    for (auto v : g[u])
    {
        if (vis[v]) continue;
        calc(v, u, 1);
        insert(v, u, 1);
    }
    for (auto v : g[u])
    {
        if (vis[v]) continue;
        rt = 0;
        tsiz = siz[v];
        getroot(v, u);
        solve(rt);
    }
}

signed main()
{
    #ifdef FAST_IOSTREAM
    cin.sync_with_stdio(false);
    cin.tie(0);
    #endif

    read(n);

    l.resize(n + 1);
    g.resize(n + 1);
    vis.resize(n + 1, false);
    r = wt = siz = inv = l;

    For (i, 1, n)
    {
        read(l[i], r[i]);
        inv[i] = qpow(r[i] - l[i] + 1, mod - 2);
    }

    For (i, 2, n)
    {
        int u, v;
        read(u, v);
        g[u].pb(v);
        g[v].pb(u);
    }

    tsiz = n;
    getroot(1, 0);
    solve(rt);

    for (int i = 1; i <= n; ++i) ans = (ll) ans * (r[i] - l[i] + 1) % mod;

    cout << ans;

    return 0;
}
```

{{% /admonition %}}

{{% admonition note "nlogn#1" true %}}

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <algorithm>

using namespace std;

typedef long long ll;

const int mod = 1e9 + 7;

int qpow(int x, int y)
{
    int out = 1;
    while (y)
    {
        if (y & 1) out = (ll) out * x % mod;
        x = (ll) x * x % mod;
        y >>= 1;
    }
    return out;
}

vector<bool> vis;
int n, tsiz, rt, ans;
vector<vector<int> > g;
vector<int> siz, wt, bel, dep, inv;

struct Modification
{
    int id, p, inv, l;
    Modification(int _id, int _p, int _inv, int _l): id(_id), p(_p), inv(_inv), l(_l) {}
    bool operator<(const Modification& b) const { return p < b.p; }
    int k1() const { return inv; }
    int b1() const { return p == l ? (ll) (1 - l + mod) * k1() % mod : (ll) (mod - p + 1) * k1() % mod; }
    int k2() const { return (ll) inv * dep[id] % mod; }
    int b2() const { return p == l ? (ll) (1 - l + mod) * k2() % mod : (ll) (mod - p + 1) * k2() % mod; }
};

struct Query
{
    int id, p, inv;
    Query(int _id, int _p, int _inv): id(_id), p(_p), inv(_inv) {}
    bool operator<(const Query& b) const { return p < b.p; }
    void calc(int x1, int x2, int type) const { ans = (ans + (ll) type * inv * ((ll) dep[id] * x1 % mod + x2) % mod + mod) % mod; }
};

void getroot(int u, int pa)
{
    siz[u] = wt[u] = 1;
    for (auto v : g[u])
    {
        if (v == pa || vis[v]) continue;
        getroot(v, u);
        siz[u] += siz[v];
        wt[u] = max(wt[u], siz[v]);
    }
    wt[u] = max(wt[u], tsiz - siz[u]);
    if (!rt || wt[u] < wt[rt]) rt = u;
}

void calc(const vector<Modification>& ms, const vector<Query>& qs, int type)
{
    int k1 = 0, b1 = 0, k2 = 0, b2 = 0, i = 0;
    for (auto q : qs)
    {
        while (i < ms.size() && ms[i].p <= q.p)
        {
            k1 = (k1 + ms[i].k1()) % mod;
            b1 = (b1 + ms[i].b1()) % mod;
            k2 = (k2 + ms[i].k2()) % mod;
            b2 = (b2 + ms[i].b2()) % mod;
            ++i;
        }
        q.calc(((ll) k1 * q.p + b1) % mod, ((ll) k2 * q.p + b2) % mod, type);
    }
}

void dfs(int u, int pa)
{
    for (auto v : g[u])
    {
        if (v == pa || vis[v]) continue;
        dep[v] = dep[u] + 1;
        bel[v] = bel[u];
        dfs(v, u); 
    }
}

void solve(int u, vector<Modification>& ms, vector<Query>& qs)
{
    vis[u] = true;
    vector<vector<Query> > qson(1, vector<Query>());
    vector<vector<Modification> > mson(1, vector<Modification>());
    for (auto v : g[u])
    {
        if (vis[v]) continue;
        mson.push_back(vector<Modification>());
        qson.push_back(vector<Query>());
        bel[v] = mson.size() - 1;
        dep[v] = 1;
        dfs(v, u);
    }
    bel[u] = dep[u] = 0;
    calc(ms, qs, 1);
    for (auto m : ms) mson[bel[m.id]].push_back(m);
    for (auto q : qs) qson[bel[q.id]].push_back(q);
    vector<Modification>().swap(ms); // free memory, the memory complexity is O(n) instead of O(nlogn) because of these two lines
    vector<Query>().swap(qs);
    for (auto v : g[u])
    {
        if (vis[v]) continue;
        calc(mson[bel[v]], qson[bel[v]], -1);
        tsiz = siz[v];
        rt = 0;
        getroot(v, u);
        solve(rt, mson[bel[v]], qson[bel[v]]);
    }
}

int main()
{
    scanf("%d", &n);

    g.resize(n + 1);
    siz.resize(n + 1, 0);
    vis.resize(n + 1, false);
    wt = bel = dep = inv = siz;

    vector<Modification> ms;
    vector<Query> qs;

    int mul = (mod + 1) / 2;

    for (int i = 1; i <= n; ++i)
    {
        int l, r;
        scanf("%d%d", &l, &r);
        mul = (ll) mul * (r - l + 1) % mod;
        inv[i] = qpow(r - l + 1, mod - 2);
        ms.push_back(Modification(i, l, inv[i], l));
        ms.push_back(Modification(i, r + 1, mod - inv[i], l));
        qs.push_back(Query(i, l - 1, mod - inv[i]));
        qs.push_back(Query(i, r, inv[i]));
    }

    for (int i = 1; i < n; ++i)
    {
        int u, v;
        scanf("%d%d", &u, &v);
        g[u].push_back(v);
        g[v].push_back(u);
    }

    sort(ms.begin(), ms.end());
    sort(qs.begin(), qs.end());

    tsiz = n;
    getroot(1, 0);
    solve(rt, ms, qs);

    cout << (ll) ans * mul % mod;

    return 0;
}
```

{{% /admonition %}}

{{% admonition note "nlogn#2" true %}}

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <algorithm>

using namespace std;

typedef long long ll;

const int mod = 1e9 + 7;

static unsigned fast_mod(uint64_t x)
{
#if !defined(_WIN32) || defined(_WIN64)
    return x % mod;
#endif
    // Optimized mod for Codeforces 32-bit machines.
    // x must be less than 2^32 * m for this to work, so that x / m fits in a 32-bit integer.
    unsigned x_high = x >> 32, x_low = (unsigned) x;
    unsigned quot, rem;
    asm("divl %4\n"
        : "=a" (quot), "=d" (rem)
        : "d" (x_high), "a" (x_low), "r" (mod));
    return rem;
}

int modadd(int x, int y)
{
    return (x += y) >= mod ? x - mod : x;
}

int qpow(int x, int y)
{
    int out = 1;
    while (y)
    {
        if (y & 1) out = fast_mod((ll) out * x);
        x = fast_mod((ll) x * x);
        y >>= 1;
    }
    return out;
}

vector<bool> vis;
int n, tsiz, rt, ans;
vector<vector<int> > g;
vector<int> siz, wt, bel, dep, inv, cnt, ma, mb, qa, qb;

struct Modification
{
    int id, p, inv, l;
    Modification(int _id, int _p, int _inv, int _l): id(_id), p(_p), inv(_inv), l(_l) {}
    bool operator<(const Modification& b) const { return p < b.p; }
    int k1() const { return inv; }
    int b1() const { return p == l ? fast_mod((ll) (1 - l + mod) * k1()) : fast_mod((ll) (mod - p + 1) * k1()); }
    int k2() const { return (ll) inv * dep[id] % mod; }
    int b2() const { return p == l ? fast_mod((ll) (1 - l + mod) * k2()) : fast_mod((ll) (mod - p + 1) * k2()); }
};

vector<Modification> ms;

struct Query
{
    int id, p, inv;
    Query(int _id, int _p, int _inv): id(_id), p(_p), inv(_inv) {}
    bool operator<(const Query& b) const { return p < b.p; }
    void calc(int x1, int x2, int type) const { ans = fast_mod(ans + type * fast_mod((ll) inv * (fast_mod((ll) dep[id] * x1 + x2))) + mod); }
};

vector<Query> qs;

void getroot(int u, int pa)
{
    siz[u] = wt[u] = 1;
    for (auto v : g[u])
    {
        if (v == pa || vis[v]) continue;
        getroot(v, u);
        siz[u] += siz[v];
        wt[u] = max(wt[u], siz[v]);
    }
    wt[u] = max(wt[u], tsiz - siz[u]);
    if (!rt || wt[u] < wt[rt]) rt = u;
}

void calc(int l, int r, int type)
{
    int k1 = 0, b1 = 0, k2 = 0, b2 = 0, i = l;
    for (int j = l; j <= r; ++j)
    {
        Query& q = qs[qa[j]];
        while (i <= r)
        {
            Modification& m = ms[ma[i]];
            if (m.p > q.p) break;
            k1 = modadd(k1, m.k1());
            b1 = modadd(b1, m.b1());
            k2 = modadd(k2, m.k2());
            b2 = modadd(b2, m.b2());
            ++i;
        }
        q.calc(fast_mod((ll) k1 * q.p + b1), fast_mod((ll) k2 * q.p + b2), type);
    }
}

void dfs(int u, int pa)
{
    cnt[bel[u]] += 2;
    for (auto v : g[u])
    {
        if (v == pa || vis[v]) continue;
        dep[v] = dep[u] + 1;
        bel[v] = bel[u];
        dfs(v, u); 
    }
}

void solve(int u, int l, int r)
{
    vis[u] = true;
    int soncnt = bel[u] = dep[u] = 0;
    cnt[0] = l + 1;
    for (auto v : g[u])
    {
        if (vis[v]) continue;
        bel[v] = ++soncnt;
        cnt[bel[v]] = cnt[bel[v] - 1];
        dep[v] = 1;
        dfs(v, u);
    }
    calc(l, r, 1);
    vector<int> cnt2(cnt.begin(), cnt.begin() + soncnt + 1);
    for (int i = r; i >= l; --i) mb[cnt2[bel[ms[ma[i]].id]]--] = ma[i];
    for (int i = l; i <= r; ++i) ma[i] = mb[i];
    for (int i = 0; i <= soncnt; ++i) cnt2[i] = cnt[i];
    for (int i = r; i >= l; --i) qb[cnt2[bel[qs[qa[i]].id]]--] = qa[i];
    for (int i = l; i <= r; ++i) qa[i] = qb[i];
    for (int i = 0; i <= soncnt; ++i) cnt2[i] = cnt[i];
    for (auto v : g[u])
    {
        if (vis[v]) continue;
        calc(cnt2[bel[v] - 1] + 1, cnt2[bel[v]], -1);
        tsiz = siz[v];
        rt = 0;
        getroot(v, u);
        solve(rt, cnt2[bel[v] - 1] + 1, cnt2[bel[v]]);
    }
}

int main()
{
    scanf("%d", &n);

    g.resize(n + 1);
    siz.resize(n + 1, 0);
    vis.resize(n + 1, false);
    wt = bel = dep = inv = cnt = siz;

    int mul = (mod + 1) / 2;

    for (int i = 1; i <= n; ++i)
    {
        int l, r;
        scanf("%d%d", &l, &r);
        mul = fast_mod((ll) mul * (r - l + 1));
        inv[i] = qpow(r - l + 1, mod - 2);
        ms.push_back(Modification(i, l, inv[i], l));
        ms.push_back(Modification(i, r + 1, mod - inv[i], l));
        qs.push_back(Query(i, l - 1, mod - inv[i]));
        qs.push_back(Query(i, r, inv[i]));
    }

    for (int i = 1; i < n; ++i)
    {
        int u, v;
        scanf("%d%d", &u, &v);
        g[u].push_back(v);
        g[v].push_back(u);
    }

    ma.resize(n * 2);
    mb = qa = qb = ma;
    for (int i = 0; i < n * 2; ++i) ma[i] = qa[i] = i;

    sort(ma.begin(), ma.end(), [](int x, int y) {
        return ms[x] < ms[y];
    });
    sort(qa.begin(), qa.end(), [](int x, int y) {
        return qs[x] < qs[y];
    });

    tsiz = n;
    getroot(1, 0);
    solve(rt, 0, 2 * n - 1);

    cout << fast_mod((ll) ans * mul);

    return 0;
}
```

{{% /admonition %}}