+++
title = "CF786C Till I Collapse（根号分治，二分答案 / 主席树，调和级数）"
categories = ["题解"]
tags = ["根号分治", "二分答案", "主席树", "数颜色", "调和级数", "数据结构", "贪心"]
date = "2019-12-06T19:24:14+08:00"
description = ""
aliases = ["/post/CF786C-Till-I-Collapse（根号分治，二分答案-主席树，调和级数）", "/CF786C-Till-I-Collapse（根号分治，二分答案-主席树，调和级数）"]
+++


## 题目链接

[CF](https://codeforces.com/contest/786/problem/C)

## 题意简述

给你一个长为 $n$ 的数列，对于 $1\le k\le n$ 的所有整数 $k$，求出这个问题的答案：将数列划分成若干连续段，每段内最多有 $k$ 个 **不同** 的数，至少要划分成几段？

$n\le 10^5$ 。

<!--more-->

## 简要做法

首先是一个显而易见的贪心：分段时在一段内放更多的数一定不劣。这个结论在下面两个做法都会用到。

### 根号分治

根据上面的那个结论，可以得到一个 $k$ 固定时复杂度为 $O(n)$ 的做法。需要使用时间戳清空访问数组。

然后就可以进行根号分治：

取一个合适的值 $B$ 。

对于 $1\le k\le B$ ，直接做，时间复杂度为 $O(nB)$ 。

对于 $B+1\le k\le n$ ，答案不超过 $n/B$ ，可以对于每个答案二分求出 $k$ 的最小值和最大值，时间复杂度为 $O(n^2\log n/B)$ 。

取 $B=\sqrt{n\log n}$，总时间复杂度为 $O(n\sqrt{n\log n})$，可以通过本题。

### 主席树

每个 $k$ 的答案不超过 $n/k$，根据调和级数，可以得到总答案的级别为 $O(n\log n)$ ，所以只要能够快速求出一次分段的右端点，就能够快速求出总答案了。

这题依然可以沿用 [「SDOI2009」HH 的项链](https://www.luogu.com.cn/problem/P1972) 的树状数组做法，但由于要不断地求分段的右端点，需要使用主席树，再加上线段树二分，就可以 $O(\log n)$ 求出一段的右端点，总时间复杂度即为 $O(n\log^2n)$ 。

具体来说，$f_{l, i}\in\\{0, 1\\}$ ，$f_{l, i}=1$ 当且仅当 $i$ 是 $l$ 及其右侧第一个为 $a_i$ 的数，即 $i\ge l$ 且不存在 $j$ 使得 $l\le j\le i-1, a_j=a_i$ 。那么，$[l, r]$ 的颜色数就是 $\sum_{i=l}^rf_{l, i}$ 。使用 $n$ 棵主席树分别维护 $f_{1..n, i}$ 即可。

## 参考代码

{{% admonition note "根号分治" true %}}

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
																																																	using namespace std;typedef long long LoveLive;typedef pair<int,int>pii;typedef vector<int>vi;typedef long double ld;const double inf=1e121;const double eps=1e-10;mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());int randint(int l,int r){int out=rng()%(r-l+1)+l;return out>=l?out:out+r-l+1;}template<typename A,typename B>string to_string(pair<A,B>p);template<typename A,typename B,typename C>string to_string(tuple<A,B,C>p);template<typename A,typename B,typename C,typename D>string to_string(tuple<A,B,C,D>p);string to_string(const string&s){return '"'+s+'"';}string to_string(const char*s){return to_string((string)s);}string to_string(bool b){return(b?"true":"false");}string to_string(vector<bool>v){bool first=true;string res="{";for(int i=0;i<static_cast<int>(v.size());i++){if(!first){res+=",";}first=false;res+=to_string(v[i]);}res+="}";return res;}template<size_t N>string to_string(bitset<N>v){string res="";for(size_t i=0;i<N;i++){res+=static_cast<char>('0'+v[i]);}return res;}template<typename A>string to_string(A v){bool first=true;string res="{";for(const auto&x:v){if(!first){res+=",";}first=false;res+=to_string(x);}res+="}";return res;}template<typename A,typename B>string to_string(pair<A,B>p){return "("+to_string(p.first)+","+to_string(p.second)+")";}template<typename A,typename B,typename C>string to_string(tuple<A,B,C>p){return "("+to_string(get<0>(p))+","+to_string(get<1>(p))+","+to_string(get<2>(p))+")";}template<typename A,typename B,typename C,typename D>string to_string(tuple<A,B,C,D>p){return "("+to_string(get<0>(p))+","+to_string(get<1>(p))+","+to_string(get<2>(p))+","+to_string(get<3>(p))+")";}template<typename A,typename B,typename C,typename D,typename E>string to_string(tuple<A,B,C,D,E>p){return "("+to_string(get<0>(p))+","+to_string(get<1>(p))+","+to_string(get<2>(p))+","+to_string(get<3>(p))+","+to_string(get<4>(p))+")";}void debug_out(){cerr<<endl;}template<typename Head,typename...Tail>void debug_out(Head H,Tail...T){cerr<<" "<<to_string(H);debug_out(T...);}
																																																	#ifdef OUUAN
																																																	#define debug(...)cerr<<"["<<#__VA_ARGS__<<"]:",debug_out(__VA_ARGS__)
																																																	#else
																																																	#define debug(...)42
																																																	#endif
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
 
const int B = 1400;

int n, tim;
vi a, vis;

int solve(int k)
{
    int out = 0, p = 0;
    do
    {
        int cnt = 0;
        ++tim;
        ++out;
        while (p < n)
        {
            if (vis[a[p]] != tim)
            {
                if (++cnt > k) break;
                vis[a[p]] = tim;
            }
            ++p;
        }
    } while (p < n);
    return out;
}

signed main()
{
#ifdef FAST_IOSTREAM
    cin.sync_with_stdio(false);
    cin.tie(0);
#endif

    read(n);

    a.resize(n);
    vis.resize(n + 1, 0);

    read(a.begin(), a.end());

    For (i, 1, min(B, n)) wts(solve(i));

    int p = min(B, n) + 1;

    FOR (i, n / B + 1, 1)
    {
        int l = p - 1;
        int r = n;
        while (l < r)
        {
            int mid = (l + r + 1) >> 1;
            if (solve(mid) < i) r = mid - 1;
            else l = mid;
        }
        for (; p <= l; ++p) wts(i);
    }

    return 0;
}
```

{{% /admonition %}}

{{% admonition note "主席树" true %}}

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
																																																	using namespace std;typedef long long LoveLive;typedef pair<int,int>pii;typedef vector<int>vi;typedef long double ld;const double inf=1e121;const double eps=1e-10;mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());int randint(int l,int r){int out=rng()%(r-l+1)+l;return out>=l?out:out+r-l+1;}template<typename A,typename B>string to_string(pair<A,B>p);template<typename A,typename B,typename C>string to_string(tuple<A,B,C>p);template<typename A,typename B,typename C,typename D>string to_string(tuple<A,B,C,D>p);string to_string(const string&s){return '"'+s+'"';}string to_string(const char*s){return to_string((string)s);}string to_string(bool b){return(b?"true":"false");}string to_string(vector<bool>v){bool first=true;string res="{";for(int i=0;i<static_cast<int>(v.size());i++){if(!first){res+=",";}first=false;res+=to_string(v[i]);}res+="}";return res;}template<size_t N>string to_string(bitset<N>v){string res="";for(size_t i=0;i<N;i++){res+=static_cast<char>('0'+v[i]);}return res;}template<typename A>string to_string(A v){bool first=true;string res="{";for(const auto&x:v){if(!first){res+=",";}first=false;res+=to_string(x);}res+="}";return res;}template<typename A,typename B>string to_string(pair<A,B>p){return "("+to_string(p.first)+","+to_string(p.second)+")";}template<typename A,typename B,typename C>string to_string(tuple<A,B,C>p){return "("+to_string(get<0>(p))+","+to_string(get<1>(p))+","+to_string(get<2>(p))+")";}template<typename A,typename B,typename C,typename D>string to_string(tuple<A,B,C,D>p){return "("+to_string(get<0>(p))+","+to_string(get<1>(p))+","+to_string(get<2>(p))+","+to_string(get<3>(p))+")";}template<typename A,typename B,typename C,typename D,typename E>string to_string(tuple<A,B,C,D,E>p){return "("+to_string(get<0>(p))+","+to_string(get<1>(p))+","+to_string(get<2>(p))+","+to_string(get<3>(p))+","+to_string(get<4>(p))+")";}void debug_out(){cerr<<endl;}template<typename Head,typename...Tail>void debug_out(Head H,Tail...T){cerr<<" "<<to_string(H);debug_out(T...);}
																																																	#ifdef OUUAN
																																																	#define debug(...)cerr<<"["<<#__VA_ARGS__<<"]:",debug_out(__VA_ARGS__)
																																																	#else
																																																	#define debug(...)42
																																																	#endif
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
 
#define mid ((l + r) >> 1)
 
struct Node
{
    int val, ls, rs;
    bool l1;
    Node()
    {
        val = ls = rs = 0;
        l1 = false;
    }
};

vector<Node> t;

int modify(int u, int l, int r, int p, int x)
{
    int cur = t.size();
    t.push_back(t[u]);
    t[cur].val += x;
    if (p == l) t[cur].l1 = x == 1;
    if (l == r - 1) return cur;
    if (p < mid)
    {
        int ls = modify(t[u].ls, l, mid, p, x);
        t[cur].ls = ls;
    }
    else 
    {
        int rs = modify(t[u].rs, mid, r, p, x);
        t[cur].rs = rs;
    }
    return cur;
}

int query(int u, int l, int r, int k)
{
    if (l == r - 1) return l;
    if (t[t[u].ls].val < k || (t[t[u].ls].val == k && !t[t[u].rs].l1))
        return query(t[u].rs, mid, r, k - t[t[u].ls].val);
    else return query(t[u].ls, l, mid, k);
}

int n;
vi a, rt, pre;

int solve(int k)
{
    int p = 0, out = 0;
    while (p < n)
    {
        ++out;
        p = query(rt[p], 0, n, k) + 1;
    }
    return out;
}

signed main()
{
#ifdef FAST_IOSTREAM
    cin.sync_with_stdio(false);
    cin.tie(0);
#endif

    read(n);

    a.resize(n);
    rt.resize(n + 1, 0);
    pre.resize(n + 1, -1);

    read(a.begin(), a.end());

    t.resize(1);

    FOR (i, n - 1, 0)
    {
        rt[i] = modify(rt[i + 1], 0, n, i, 1);
        if (~pre[a[i]]) rt[i] = modify(rt[i], 0, n, pre[a[i]], -1);
        pre[a[i]] = i;
    }

    For (i, 1, n) wts(solve(i));

    return 0;
}
```

{{% /admonition %}}