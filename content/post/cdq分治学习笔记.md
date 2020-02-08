+++
title = "cdq分治学习笔记"
date = "2019-03-26T18:49:44+08:00"
categories = ["算法"]
tags = ["离线算法", "cdq分治"]
description = ""
aliases = ["/post/cdq分治学习笔记", "/cdq分治学习笔记"]
+++


cdq分治也是咕了好久了..最近总算把它学了。

<del>cdq分治是一种离线算法，可以代替一些复杂的数据结构，降低代码难度，减小常数。</del>废话大家都知道。

<!--more-->

本文未完待续（cdq分治的其它应用，如维护凸壳，待填坑）。

# 简介

感觉cdq分治不如叫“ex归并排序”，就是**以操作的时间作为初始顺序，在递归处理的过程中按位置归并排序**。

更一般地说，对于一个二维偏序 $P(i,j)=P_1(a_i,a_j)\land P_2(b_i,b_j)​$，位置 $i​$ 的修改对位置 $j​$ 的询问（询问为类前缀和形式，区间询问需拆成两个前缀询问）有影响当且仅当 $P(i,j)=true​$，cdq分治就是以其中一维为初始顺序，对另一维进行归并排序的过程中计算左区间里修改的总和，将左区间修改的影响应用到右区间。

学会了之后就会发现，cdq分治的确就是这样，已经描述的很清楚了，然而在没学会的时候估计是看不懂上面这段话的..所以结合具体题目来看一看吧。

# 例题

## [**P3374** 【模板】树状数组 1](https://www.luogu.org/problemnew/show/P3374) 

<del>树状数组裸题！</del>冷静，我们来用ex归并排序做..（嗯，我决定就这么叫它了）

按照我们上面说的，我们把操作存下来，询问拆成两个前缀和相减，初始值视作修改，需要存的信息有操作的种类（修改、询问的左端点减一、询问的右端点），操作的位置（$p$、$l-1$、$r$）以及修改加上的值/询问的编号。如果写法正常的话你已经以操作的时间作为初始顺序了..

然后，写个归并排序，按操作的位置排序，同一个位置的修改要放在询问的前面。然后，在归并排序的过程中，遇到左区间里的修改就更新左区间修改的总和，遇到右区间里的询问就用记录的“左区间修改的总和”更新这个询问的答案。

{{% admonition note "具体见代码" true %}}

```cpp
#include <iostream>
#include <cstdio>

using namespace std;

typedef long long ll;

const int N=500010;

struct Node
{
    int type,p,val; //type为2表示修改，type为-1表示左端点减一，type为1表示右端点
    bool operator<(const Node& b) const { return p==b.p?type>b.type:p<b.p; }
} q[N<<2],tmp[N<<2];

void solve(int l,int r);

int n,m,tot,qtot;
ll ans[N];

int main()
{
    int i,op,x,y;

    scanf("%d%d",&n,&m);

    for (i=1;i<=n;++i) //初始值视作修改
    {
        scanf("%d",&x);
        q[++tot].type=2;
        q[tot].p=i;
        q[tot].val=x;
    }

    for (i=1;i<=m;++i)
    {
        scanf("%d%d%d",&op,&x,&y);
        if (op==1)
        {
            q[++tot].type=2;
            q[tot].p=x;
            q[tot].val=y;
        }
        else //询问拆成两个前缀和相减
        {
            q[++tot].type=-1;
            q[tot].p=x-1;
            q[tot].val=++qtot;
            q[++tot].type=1;
            q[tot].p=y;
            q[tot].val=qtot;
        }
    }

    solve(1,tot+1);

    for (i=1;i<=qtot;++i) printf("%lld\n",ans[i]);

    return 0;
}

void solve(int l,int r)
{
    if (l==r-1) return;
    int i,j,k,mid;
    ll sum=0;
    i=k=l;
    j=mid=(l+r)>>1;
    solve(l,mid);
    solve(mid,r);
    while (i<mid&&j<r)
    {
        if (q[i]<q[j])
        {
            if (q[i].type==2) sum+=q[i].val; //记录左区间里的修改之和
            tmp[k++]=q[i++];
        }
        else
        {
            if (q[j].type!=2) ans[q[j].val]+=q[j].type*sum; //将左区间里的修改应用到右区间里的询问
            tmp[k++]=q[j++];
        }
    }
    while (i<mid) tmp[k++]=q[i++];
    while (j<r)
    {
        if (q[j].type!=2) ans[q[j].val]+=q[j].type*sum;
        tmp[k++]=q[j++];
    }
    for (i=l;i<r;++i) q[i]=tmp[i];
}
```

{{% /admonition %}}

之前说过ex归并排序本质上是一个二维偏序限制了修改对询问的影响，所以也可以先按位置排序再按时间排序。只不过..这样写很奇怪，很麻烦，常数又大。然而为了理解ex归并排序的本质，我还是写了份这个做法..

{{% admonition note "一种奇怪的写法" true %}}

```cpp
#include <iostream>
#include <cstdio>
#include <algorithm>

using namespace std;

typedef long long ll;

const int N=500010;

struct Node
{
    int type,tim,p,val;
    Node(int _type=0,int _tim=0,int _p=0,int _val=0):type(_type),tim(_tim),p(_p),val(_val){}
} q[N<<2],tmp[N<<2];

void solve(int l,int r);

int n,m,tot,qtot;
ll ans[N];

int main()
{
    int i,j,op,x,y;

    scanf("%d%d",&n,&m);

    for (i=1;i<=n;++i)
    {
        scanf("%d",&x);
        q[++tot]=Node(2,0,i,x);
    }

    for (i=1;i<=m;++i)
    {
        scanf("%d%d%d",&op,&x,&y);
        if (op==1) q[++tot]=Node(2,i,x,y);
        else
        {
            q[++tot]=Node(-1,i,x-1,++qtot);
            q[++tot]=Node(1,i,y,qtot);
        }
    }

    sort(q+1,q+tot+1,[](const Node& x,const Node& y){return x.p==y.p?x.type>y.type:x.p<y.p;});

    solve(1,tot+1);

    for (i=1;i<=qtot;++i) printf("%lld\n",ans[i]);

    return 0;
}

void solve(int l,int r)
{
    if (l==r-1) return;
    int i,j,k,mid;
    ll sum=0;
    i=k=l;
    j=mid=(l+r)>>1;
    solve(l,mid);
    solve(mid,r);
    while (i<mid&&j<r)
    {
        if (q[i].tim<q[j].tim)
        {
            if (q[i].type==2) sum+=q[i].val;
            tmp[k++]=q[i++];
        }
        else
        {
            if (q[j].type!=2) ans[q[j].val]+=q[j].type*sum;
            tmp[k++]=q[j++];
        }
    }
    while (i<mid) tmp[k++]=q[i++];
    while (j<r)
    {
        if (q[j].type!=2) ans[q[j].val]+=q[j].type*sum;
        tmp[k++]=q[j++];
    }
    for (i=l;i<r;++i) q[i]=tmp[i];
}
```

{{% /admonition %}}

# [**P3810** 【模板】三维偏序（陌上花开）](https://www.luogu.org/problemnew/show/P3810) 

有两种做法，一种是cdq分治套树状数组，需要注意的有两点，一是清空树状数组可以用时间戳，二是 $a,\,b,\,c$ 都相等的元素要合并。

{{% admonition note "参考代码" true %}}

```cpp
#include <iostream>
#include <cstdio>
#include <algorithm>

using namespace std;

const int N=100010;
const int K=200010;

struct Node
{
    int a,b,c,w,f;
} a[N],b[N];

void solve(int l,int r);
void add(int p,int x);
int query(int p);

int n,k,d[N],BIT[K],vis[K],tim,tot;

int main()
{
    int i;

    scanf("%d%d",&n,&k);

    for (i=1;i<=n;++i)
    {
        scanf("%d%d%d",&b[i].a,&b[i].b,&b[i].c);
        b[i].w=1;
    }

    sort(b+1,b+n+1,[](const Node& x,const Node& y){return x.a==y.a?(x.b==y.b?x.c<y.c:x.b<y.b):x.a<y.a;});

    for (i=1;i<=n;++i)
    {
        if (b[i].a!=b[i+1].a||b[i].b!=b[i+1].b||b[i].c!=b[i+1].c) a[++tot]=b[i];
        else b[i+1].w+=b[i].w;
    }

    solve(1,tot+1);

    for (i=1;i<=tot;++i) d[a[i].f+a[i].w]+=a[i].w;

    for (i=1;i<=n;++i) printf("%d\n",d[i]);

    return 0;
}

void solve(int l,int r)
{
    if (l==r-1) return;
    int i,j,k,mid;
    i=k=l;
    j=mid=(l+r)>>1;
    solve(l,mid);
    solve(mid,r);
    ++tim;
    while (i<mid&&j<r)
    {
        if (a[i].b<=a[j].b)
        {
            add(a[i].c,a[i].w);
            b[k++]=a[i++];
        }
        else
        {
            a[j].f+=query(a[j].c);
            b[k++]=a[j++];
        }
    }
    while (i<mid) b[k++]=a[i++];
    while (j<r)
    {
        a[j].f+=query(a[j].c);
        b[k++]=a[j++];
    }
    for (i=l;i<r;++i) a[i]=b[i];
}

void add(int p,int x)
{
    for (;p<=k;p+=(p&-p))
    {
        if (vis[p]!=tim)
        {
            BIT[p]=0;
            vis[p]=tim;
        }
        BIT[p]+=x;
    }
}

int query(int p)
{
    int out=0;
    for (;p;p-=(p&-p)) if (vis[p]==tim) out+=BIT[p];
    return out;
}
```

{{% /admonition %}}

还有一种做法是cdq分治套cdq分治：

{{% admonition note "参考代码" true %}}

```cpp
#include <iostream>
#include <cstdio>
#include <algorithm>

using namespace std;

const int N=100010;

struct Node
{
    int a,b,c,d,w,id;
} a[N],b[N],c[N];

void solve(int l,int r);
void solve2(int l,int r);

int n,k,d[N],tot,ans[N];

int main()
{
    int i;

    scanf("%d%d",&n,&k);

    for (i=1;i<=n;++i)
    {
        scanf("%d%d%d",&b[i].a,&b[i].b,&b[i].c);
        b[i].w=1;
        b[i].id=i;
    }

    sort(b+1,b+n+1,[](const Node& x,const Node& y){return x.a==y.a?(x.b==y.b?x.c<y.c:x.b<y.b):x.a<y.a;});

    for (i=1;i<=n;++i)
    {
        if (b[i].a!=b[i+1].a||b[i].b!=b[i+1].b||b[i].c!=b[i+1].c) a[++tot]=b[i];
        else b[i+1].w+=b[i].w;
    }

    solve(1,tot+1);

    for (i=1;i<=tot;++i) d[ans[a[i].id]+a[i].w]+=a[i].w;

    for (i=1;i<=n;++i) printf("%d\n",d[i]);

    return 0;
}

void solve(int l,int r)
{
    if (l==r-1) return;
    int i,j,k,mid;
    i=k=l;
    j=mid=(l+r)>>1;
    solve(l,mid);
    solve(mid,r);
    while (i<mid&&j<r)
    {
        if (a[i].b<=a[j].b)
        {
            a[i].d=a[i].w;
            b[k++]=a[i++];
        }
        else
        {
            a[j].d=0;
            b[k++]=a[j++];
        }
    }
    while (i<mid)
    {
        a[i].d=a[i].w;
        b[k++]=a[i++];
    }
    while (j<r)
    {
        a[j].d=0;
        b[k++]=a[j++];
    }
    for (i=l;i<r;++i) a[i]=b[i];
    solve2(l,r);
}

void solve2(int l,int r)
{
    if (l==r-1) return;
    int i,j,k,mid,sum=0;
    i=k=l;
    j=mid=(l+r)>>1;
    solve2(l,mid);
    solve2(mid,r);
    while (i<mid&&j<r)
    {
        if (b[i].c<=b[j].c)
        {
            sum+=b[i].d;
            c[k++]=b[i++];
        }
        else
        {
            if (!b[j].d) ans[b[j].id]+=sum;
            c[k++]=b[j++];
        }
    }
    while (i<mid) c[k++]=b[i++];
    while (j<r)
    {
        if (!b[j].d) ans[b[j].id]+=sum;
        c[k++]=b[j++];
    }
    for (i=l;i<r;++i) b[i]=c[i];
}
```

{{% /admonition %}}

# cdq分治求偏序对的本质

（下文中“偏序问题”即求满足偏序关系的数对个数。而”高维偏序“实际上是多个严格弱序的并。非严格偏序与之类似，主要是在代码上有些细节改变。）

大家知道，二维偏序可以先按一维排序后用普通的归并排序解决，那为什么“三维偏序”不可以呢？

首先，按其中一维排序相当于降了一维，问题就变成了“为什么一维偏序可以用普通的归并排序解决，而二维偏序不可以”。

原因就在于，两个偏序关系的并，不一定具有不可比性的传递性。（Strict Weak Ordering 相关内容参见[我的另一篇博客](https://ouuan.github.io/%E6%B5%85%E8%B0%88%E9%82%BB%E9%A1%B9%E4%BA%A4%E6%8D%A2%E6%8E%92%E5%BA%8F%E7%9A%84%E5%BA%94%E7%94%A8%E4%BB%A5%E5%8F%8A%E9%9C%80%E8%A6%81%E6%B3%A8%E6%84%8F%E7%9A%84%E9%97%AE%E9%A2%98/#%E4%B8%A5%E6%A0%BC%E5%BC%B1%E5%BA%8F)）

可以证明，两个严格弱序的并一定是一个严格偏序，但不一定是一个严格弱序。而 cdq 分治可以将多个严格弱序的并进行降维，每一次 cdq 分治都标记出哪些位置会对其它位置有贡献，并按某一维排序。

上面说的有点乱..简单概括一下。排序可以降维，只有严格弱序能排序，高维偏序不一定是严格弱序，cdq 分治在排序的过程中标记了元素之间如何贡献答案。所以 cdq 分治就可以解决高维偏序问题了..

所以，我们可以写出一份 cdq 分治求 $k$ 维偏序对的代码：

题意简述：第一行 $n,\,k$，后 $n$ 行每行 $k$ 个数 $a_{i,1..k}$，对每个 $i$ 求 $\forall d\in[1,k],\,a_{i,d}<a_{j,d}$ 的 $j$ 个数。

当然，$n$ 要足够大，否则会被暴力艹。只不过理论上来说，如果维数是常数，复杂度就比暴力更优...

[随便造的一个模板题](<https://www.luogu.org/problemnew/show/U66865>)

{{% admonition note "代码" true %}}

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <algorithm>

using namespace std;

const int N=10005;
const int K=15;

void solve(int l,int r,int d);

int n,k,ans[N];

struct Node
{
    int w[K],type,id;
    bool operator<(const Node& y) const
    {
        if (w[0]!=y.w[0]) return w[0]<y.w[0];
        for (int i=1;i<k;++i) if (w[i]!=y.w[i]) return w[i]>y.w[i]; //如果是非严格偏序都应该顺着排，严格偏序除了第一维都应该倒着排。这是由于相等元素可以/不可以转移。
        return false;
    }
} a[K][N],tmp[N];

int main()
{
    int i,j;

    scanf("%d%d",&n,&k);

    for (i=1;i<=n;++i)
    {
        a[0][i].id=i;
        for (j=0;j<k;++j) scanf("%d",a[0][i].w+j);
    }

    sort(a[0]+1,a[0]+n+1);

    solve(1,n+1,1);

    for (i=1;i<=n;++i) printf("%d\n",ans[i]);

    return 0;
}

void solve(int l,int r,int d)
{
    if (l==r-1) return;
    int i,j,p,mid,sum=0;
    i=p=l;
    j=mid=(l+r)>>1;
    solve(l,mid,d);
    solve(mid,r,d);
    while (i<mid&&j<r)
    {
        if (a[d-1][i].w[d]<a[d-1][j].w[d])
        {
            a[d][p]=tmp[p]=a[d-1][i++];
            if (d>1&&a[d][p].type!=1) a[d][p].type=0;
            else
            {
                a[d][p].type=1;
                if (d==k-1) ++sum;
            }
            ++p; 
        }
        else
        {
            a[d][p]=tmp[p]=a[d-1][j++];
            if (d>1&&a[d][p].type!=2) a[d][p].type=0;
            else
            {
                a[d][p].type=2;
                if (d==k-1) ans[a[d][p].id]+=sum;
            }
            ++p;
        }
    }
    while (i<mid)
    {
        a[d][p]=tmp[p]=a[d-1][i++];
        if (d>1&&a[d][p].type!=1) a[d][p].type=0;
        else a[d][p].type=1;
        ++p;
    }
    while (j<r)
    {
        a[d][p]=tmp[p]=a[d-1][j++];
        if (d>1&&a[d][p].type!=2) a[d][p].type=0;
        else
        {
            a[d][p].type=2;
            if (d==k-1) ans[a[d][p].id]+=sum;
        }
        ++p;
    }
    for (i=l;i<r;++i) a[d-1][i]=tmp[i];
    if (d<k-1) solve(l,r,d+1);
}
```

{{% /admonition %}}