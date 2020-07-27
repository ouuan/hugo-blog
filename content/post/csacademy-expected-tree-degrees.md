+++
title = 'CS Academy Round #10 - Expected Tree Degrees'
date = 2020-07-26T22:33:07+08:00
draft = false
categories = ['题解']
tags = ['概率期望', '计数dp', '生成函数', '矩阵']
+++

[题目链接](https://csacademy.com/contest/archive/task/expected-tree-degrees/)

{{% question %}}
有一棵 $n$ 个点的树，由如下方式生成：

对于每个点 $i$ ($i\ge 2$)，在 $1$ 到 $i-1$ 中等概率选择一个点作为点 $i$ 的父亲。

求每个点的度数的平方和的期望，要求绝对误差不超过 $10^{-6}$，$n\le 10^6$。
{{% /question %}}

<!--more-->

## 状态设计与转移

令 $f_{i, j}$ 表示 $i$ 个点的树中度数为 $j$ 的点的个数的期望。

那么：$f_{i, j} = f_{i - 1, j} - \frac{f_{i - 1, j}}{i - 1} + \frac{f_{i - 1, j - 1}}{i - 1} + [j=1]$

其中 $\frac{f_{i - 1, j}}{i - 1}$ 表示点 $i$ 的父亲在只有前 $i-1$ 个点时度数为 $j$ 的概率，$[j=1]$ 表示 `j == 1 ? 1 : 0`。

然而，直接 DP 复杂度是 $O(n^2)$，无法通过此题。有多种处理方式，见下文。

## 求答案的近似值

可以猜测 $j$ 比较大时 $f_{i, j}$ 非常小，可以忽略不计。实际上 $f_{10^6, 50}$ 只有大约 $10^{-14}$，所以只用计算 $j\le 50$ 的 $f_{i, j}$。

## 用生成函数和矩阵求答案的精确值

虽然上面这种求近似值的方法可以通过本题，但本题实际上是可以线性地求答案的精确值的。

首先，每个 $f_{i, 1..i}$ 可以视为一个生成函数，记作 $F_i$，也就是说 $F_i$ 的 $x^j$ 项的系数表示 $f_{i, j}$。

那么，转移方程可以表示为：

$$
\begin{aligned}
F_i &= F_{i - 1} - \frac{F_{i - 1}}{i - 1} + x\frac{F_{i - 1}}{i - 1}+x\\\\
    &= \frac{i-2+x}{i-1}F_{i - 1} + x
\end{aligned}
$$

然而，这样算出 $F_n$ 的每一项的系数依然是 $O(n^2)$ 的。

但是，我们要求的并不是 $F_n$ 的每一项的系数，而是每一项的系数乘上次数的平方的和。

注意到 $\\{1^2, 2^2, 3^2, \ldots\\}$ 这个数列可以用下面这个矩阵的幂计算：

$$
A=
\begin{bmatrix}
1&0&0\\\\
1&1&0\\\\
1&2&1
\end{bmatrix}
$$

$$
A
\begin{bmatrix}
1\\\\
i\\\\
i^2
\end{bmatrix}=
\begin{bmatrix}
1\\\\
(i+1)\\\\
(i+1)^2
\end{bmatrix}
$$

所以，把 $A$ 作为 $x$ 带入生成函数中计算即可得到答案。

{{% fold 参考代码 %}}
```cpp
// Problem: Expected Tree Degrees
// Contest: CS Academy
// URL: https://csacademy.com/contest/archive/task/expected-tree-degrees/
// Memory Limit: 128 MB
// Time Limit: 1000 ms
// Powered by CP Editor (https://github.com/cpeditor/cpeditor)

																																																	#ifndef OUUAN
																																																	#pragma GCC optimize("Ofast,no-stack-protector,unroll-loops,fast-math")
																																																	#endif
																																																	#include<bits/stdc++.h>

#define int ll
// #define FAST_IOSTREAM 1

																																																	#define For(i,l,r)for(int i=(l),i##end=(r);i<=i##end;++i)
																																																	#define FOR(i,r,l)for(int i=(r),i##end=(l);i>=i##end;--i)
																																																	#define SON(i,u)for(int i=head[u];i;i=nxt[i])
																																																	#define BE(x)(x).begin(),(x).end()
																																																	#define BE1(x)((x).begin()+1),(x).end()
																																																	#define fi first
																																																	#define se second
																																																	#define pb push_back
																																																	#define eb emplace_back
																																																	#define pq priority_queue
																																																	#define min minOfDifferentTypes
																																																	#define max maxOfDifferentTypes
																																																	#define y1 why_is_there_a_function_called_y1
																																																	using namespace std;typedef long long ll;typedef pair<int,int>pii;typedef vector<int>vi;typedef vector<vi>vvi;typedef vvi v2i;typedef vector<vvi>v3i;typedef vector<v3i>v4i;typedef vector<bool>vb;typedef vector<vb>vvb;typedef vvb v2b;typedef vector<vvb>v3b;typedef vector<v3b>v4b;typedef vector<pii>vpii;typedef vector<vpii>vvpii;typedef vvpii v2pii;typedef vector<vvpii>v3pii;typedef vector<v3pii>v4pii;typedef vector<double>vd;typedef vector<vd>vvd;typedef vvd v2d;typedef vector<v2d>v3d;typedef vector<v3d>v4d;const double inf=1e121;const double eps=1e-10;mt19937 rng(chrono::steady_clock::now().time_since_epoch().count());int randint(int l,int r){int out=rng()%(r-l+1)+l;return out>=l?out:out+r-l+1;}template<typename A,typename B>string to_string(pair<A,B>p);template<typename A,typename B,typename C>string to_string(tuple<A,B,C>p);template<typename A,typename B,typename C,typename D>string to_string(tuple<A,B,C,D>p);string to_string(const string&s){return'"'+s+'"';}string to_string(const char*s){return to_string((string)s);}string to_string(bool b){return(b?"true":"false");}string to_string(vector<bool>v){bool first=true;string res="{";for(int i=0;i<static_cast<int>(v.size());i++){if(!first){res+=", ";}first=false;res+=to_string(v[i]);}res+="}";return res;}template<size_t N>string to_string(bitset<N>v){string res="";for(size_t i=0;i<N;i++){res+=static_cast<char>('0'+v[i]);}return res;}template<typename A>string to_string(A v){bool first=true;string res="{";for(const auto&x:v){if(!first){res+=", ";}first=false;res+=to_string(x);}res+="}";return res;}template<typename A,typename B>string to_string(pair<A,B>p){return"("+to_string(p.first)+", "+to_string(p.second)+")";}template<typename A,typename B,typename C>string to_string(tuple<A,B,C>p){return"("+to_string(get<0>(p))+", "+to_string(get<1>(p))+", "+to_string(get<2>(p))+")";}template<typename A,typename B,typename C,typename D>string to_string(tuple<A,B,C,D>p){return"("+to_string(get<0>(p))+", "+to_string(get<1>(p))+", "+to_string(get<2>(p))+", "+to_string(get<3>(p))+")";}template<typename A,typename B,typename C,typename D,typename E>string to_string(tuple<A,B,C,D,E>p){return"("+to_string(get<0>(p))+", "+to_string(get<1>(p))+", "+to_string(get<2>(p))+", "+to_string(get<3>(p))+","+to_string(get<4>(p))+")";}void debug_out(){cerr<<endl;}template<typename Head,typename...Tail>void debug_out(Head H,Tail...T){cerr<<" "<<to_string(H);debug_out(T...);}template<typename T>struct is_pair{static const bool value=false;};template<typename T,typename U>struct is_pair<std::pair<T,U>>{static const bool value=true;};
																																																	#ifdef OUUAN
																																																	#define debug(...)cerr<<"["<<#__VA_ARGS__<<"]:",debug_out(__VA_ARGS__)
																																																	#else
																																																	#define debug(...)42
																																																	#endif
																																																	#ifdef int
																																																	const int INF=0x3f3f3f3f3f3f3f3fll;
																																																	#else
																																																	const int INF=0x3f3f3f3f;
																																																	#endif
																																																	#ifdef FAST_IOSTREAM
																																																	#define br cout<<'\n'
																																																	#define sp cout<<' '
																																																	#define fl cout.flush()
																																																	ll read(){ll x;cin>>x;return x;}template<typename T>void read(T&x){cin>>x;}template<typename T>void write(const T&x){cout<<x;}
																																																	#else
																																																	#define br putchar('\n')
																																																	#define sp putchar(' ')
																																																	#define fl fflush(stdout)
																																																	template<typename T>typename enable_if<!is_integral<T>::value&&!is_pair<T>::value,void>::type read(T&x){cin>>x;}ll read(){char c;ll out=0,f=1;for(c=getchar();!isdigit(c)&&c!='-';c=getchar()){}if(c=='-'){f=-1;c=getchar();}for(;isdigit(c);c=getchar())out=(out<<3)+(out<<1)+c-'0';return out*f;}template<typename T>typename enable_if<is_integral<T>::value,T>::type read(T&x){char c;T f=1;x=0;for(c=getchar();!isdigit(c)&&c!='-';c=getchar()){}if(c=='-'){f=-1;c=getchar();}for(;isdigit(c);c=getchar())x=(x<<3)+(x<<1)+c-'0';return x*=f;}char read(char&x){for(x=getchar();isspace(x);x=getchar()){}return x;}double read(double&x){scanf("%lf",&x);return x;}template<typename T>typename enable_if<!is_integral<T>::value&&!is_pair<T>::value,void>::type write(const T&x){cout<<x;}template<typename T>typename enable_if<is_integral<T>::value,void>::type write(const T&x){if(x<0){putchar('-');write(-x);return;}if(x>9)write(x/10);putchar(x%10+'0');}void write(const char&x){putchar(x);}void write(const double&x){printf("%.12lf",x);}
																																																	#endif
																																																	template<typename T>typename enable_if<is_pair<T>::value,void>::type read(T&x){read(x.fi);read(x.se);}template<typename T>typename enable_if<is_pair<T>::value,void>::type write(const T&x){write(x.fi);sp;write(x.se);}template<typename T,typename...Args>void read(T&x,Args&...args){read(x);read(args...);}template<typename OutputIt,typename=typename enable_if<is_same<output_iterator_tag,typename iterator_traits<OutputIt>::iterator_category>::value||(is_base_of<forward_iterator_tag,typename iterator_traits<OutputIt>::iterator_category>::value&&!is_const<OutputIt>::value)>::type>void read(OutputIt __first,OutputIt __last){for(;__first!=__last;++__first)read(*__first);}template<typename InputIt,typename=typename enable_if<is_base_of<input_iterator_tag,typename iterator_traits<InputIt>::iterator_category>::value>::type>void wts(InputIt __first,InputIt __last){bool isFirst=true;for(;__first!=__last;++__first){if(isFirst)isFirst=false;else sp;write(*__first);}br;}template<typename InputIt,typename=typename enable_if<is_base_of<input_iterator_tag,typename iterator_traits<InputIt>::iterator_category>::value>::type>void wtb(InputIt __first,InputIt __last){for(;__first!=__last;++__first){write(*__first);br;}}template<typename T>void wts(const T&x){write(x);sp;}template<typename T>void wtb(const T&x){write(x);br;}template<typename T>void wte(const T&x){write(x);exit(0);}template<typename T,typename...Args>void wts(const T&x,Args...args){wts(x);wts(args...);}template<typename T,typename...Args>void wtb(const T&x,Args...args){wts(x);wtb(args...);}template<typename T,typename...Args>void wte(const T&x,Args...args){wts(x);wte(args...);}template<typename T1,typename T2>inline bool up(T1&x,const T2&y){return x<y?x=y,1:0;}template<typename T1,typename T2>inline bool dn(T1&x,const T2&y){return y<x?x=y,1:0;}template<typename T1,typename T2,typename T3>inline bool inRange(const T1&x,const T2&l,const T3&r){return!(x<l)&&!(r<x);}template<typename T1,typename T2>inline auto minOfDifferentTypes(const T1&x,const T2&y)->decltype(x<y?x:y){return x<y?x:y;}template<typename T1,typename T2>inline auto maxOfDifferentTypes(const T1&x,const T2&y)->decltype(x<y?y:x){return x<y?y:x;}template<typename T1,typename T2,typename T3>inline T1&madd(T1&x,const T2&y,const T3&modulo){return x=(ll)(x+y+modulo)%modulo;}template<typename T1,typename T2,typename T3>inline T1&mmul(T1&x,const T2&y,const T3&modulo){return x=(ll)x*y%modulo;}inline int modadd(int x,int y,int modulo){return(x+y)>=modulo?x+y-modulo:x+y;}inline int isinf(int x){return x<INF?x:-1;}inline void yesno(bool x){wtb(x?"Yes":"No");}

/* ------------------------------------------------------------------------------------------------------------------- */

typedef long double ld;

struct Matrix
{
	ld a[3][3];
	
	Matrix() { memset(a, 0, sizeof(a)); }
	
	ld *operator[](int x) { return a[x]; }
	const ld *operator[](int x) const { return a[x]; }
	
	Matrix operator*(const Matrix &b) const
	{
		Matrix res;
		For (i, 0, 2)
		{
			For (j, 0, 2)
			{
				For (k, 0, 2)
				{
					res[i][j] += a[i][k] * b[k][j];
				}
			}
		}
		return res;
	}
	
	Matrix operator*(ld x) const
	{
		Matrix res;
		For (i, 0, 2)
		{
			For (j, 0, 2)
			{
				res[i][j] = a[i][j] * x;
			}
		}
		return res;
	}
	
	Matrix operator+(const Matrix &b) const
	{
		Matrix res;
		For (i, 0, 2)
		{
			For (j, 0, 2)
			{
				res[i][j] = a[i][j] + b[i][j];
			}
		}
		return res;
	}
};

signed main()
{
#ifdef FAST_IOSTREAM
	ios_base::sync_with_stdio(false);
	cin.tie(0);
	cout.tie(0);
#endif
	
	int n = read();
	
	Matrix sq;
	sq[0][0] = sq[1][0] = sq[1][1] = sq[2][0] = sq[2][2] = 1;
	sq[2][1] = 2;
	
	Matrix one;
	one[0][0] = one[1][1] = one[2][2] = 1;
	
	Matrix res = one;
	
	For (i, 2, n) res = res * (sq + one * (i - 2)) * (1.0l / (i - 1)) + sq;
	
	cout << fixed << setprecision(20) << res[2][0];
	
	return 0;
}
```
{{% /fold %}}

## 推公式求答案的精确值

在 [评论区](https://csacademy.com/contest/round-10/task/expected-tree-degrees/discussion/) 看到的：答案为 $6(n-1) - 4(1 + \frac{1}{2} + \frac{1}{3} + \cdots + \frac{1}{n - 1})$。

然而评论区除了这个公式啥都没有..我就自己试着把它推出来了..

考虑增加一个点对答案的贡献，分为两部分：

1.  新加的点度数为 $1$，贡献也为 $1$；
2.  若父亲的度数为 $x$，则父亲的贡献增加 $2x+1$（因为 $(x+1)^2-x^2=2x+1$）。

所以，在一棵有 $i$ 个点的树上增加一个点的期望贡献为：（$f_{i, j}$ 为 [上文中设计的状态](#状态设计与转移)）

$$
1 + \sum\limits_{j=0}^i (2j+1) \frac{f_{i, j}}{i}
$$

为了方便，在下文中设 $g(i)=\sum\limits_{j=0}^i (2j+1) f_{i, j}$。

这时再来观察一下转移方程：

1.  $f_{i-1, j}$，对 $g(i)$ 的贡献为 $g(i-1)$；
2.  $\frac{f_{i - 1, j - 1}}{i - 1} - \frac{f_{i - 1, j}}{i - 1}$，相当于把 $f_{i, j}$ 的 $\frac{1}{i-1}$ “移动”到了 $f_{i, j+1}$，从而使 $g$ 中的系数从 $2j+1$ 变成了 $2j+3$，即每一项的系数加二，所以对 $g(i)$ 的贡献为 $\frac{2}{i-1}\sum_{j=0}^{i-1}f_{i-1,j}=2$；
3.  $[j=1]$，对 $g(i)$ 的贡献为 $2j+1=3$。

所以，$g(i)=g(i-1)+5$。又因为由定义计算可得 $g(1)=1$，所以 $g(i)=5i-4$。

所以，在一棵有 $i$ 个点的树上增加一个点的期望贡献为 $1+\frac{g(i)}{i}=6-\frac{4}{i}$，累加一下就可以得到最上面那个公式了。

## 坑

不用 `long double` 可能会被卡精度..原因大概是题目要求的是 **绝对** 误差不超过 $10^{-6}$... 怎么还有人出题要求绝对精度误差的 /fad

只不过标算是求近似值，要是出题人会求精确值的做法就可以取模了 /fad
