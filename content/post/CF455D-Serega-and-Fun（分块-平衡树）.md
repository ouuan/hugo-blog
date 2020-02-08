+++
title = "CF455D Serega and Fun（分块 / 平衡树）"
categories = ["题解"]
tags = ["分块", "平衡树", "数据结构"]
date = "2019-12-04T15:20:52+08:00"
description = ""
aliases = ["/post/CF455D-Serega-and-Fun（分块-平衡树）", "/CF455D-Serega-and-Fun（分块-平衡树）"]
+++


## 题目链接

[CF](https://codeforces.com/contest/455/problem/D)

## 题意简述

给你一个序列，在线地支持两个操作：

1. 将一个区间循环移位。

2. 查询一个区间中某个数出现的次数。

序列长度、查询个数都不超过 $10^5$，时限 $\texttt{4s}$ 。

<!--more-->

## O((n+m)sqrtn) 做法

大致思路：分成若干块，每块维护块内每个数的出现次数（这导致空间复杂度是 $O(n\sqrt n)$），以及这一块对应的序列（相当于块状链表）。

具体来说，有至少四种大同小异的做法：

1. 算法：
	- 每次循环移位时只将给定区间的末尾移至给定区间的开头，这样的话每块的大小会经常改变，每根号次循环移位需要重构一次。

	- 每次循环移位时除了将给定区间的末尾移至给定区间的开头，还将区间内每一块的末尾移至下一块的开头，这样的话每块的大小总是不变的。

2. 数据结构：
	- 使用链表维护每一块，找到需要插入、删除的位置可以做到 $O(\sqrt n)$，插入、删除可以做到 $O(1)$ ，换块（末尾删除、头部插入）可以做到 $O(1)$ 。

	- 使用双端队列维护每一块，找到需要插入、删除的位置可以做到 $O(1)$，插入、删除可以做到 $O(\sqrt n)$ ，换块（末尾删除、头部插入）可以做到 $O(1)$ 。

## O(n+mlog^2n) 做法

使用一棵平衡树维护整个序列，再使用 $n$ 棵平衡树分别维护值为 $i$ 的数之间的相对位置。

即，$t_0$ 中的元素是序列中的每个数，$t_i$ ($1\le i\le n$) 中的元素是所有大小为 $i$ 的数，用于比较的键值是这个数在序列中的位置。

$t_0$ 的维护是经典问题，而维护 $t_i$ 时需要查找位置不超过/不小于给定值的最靠右/最靠左元素，找这个的时候需要利用 $t_0$ 来查询一个数在序列中的位置，具体实现时需要记录 $t_i$ 的每个节点在 $t_0$ 中对应的节点。

由于在 $t_i$ 中调用了 $t_0$，复杂度就是 $O(n+m\log^2 n)$ 。

有点难写，我写了 5KB，调了一年..CF 上有 [2.3KB 的提交](https://codeforces.com/contest/455/submission/7410439)（没仔细看，但应该是一样的做法..）。

然后这题还成功劝退结构体数组选手，让我改用指针了..最后内存占用比分块还大。

## 参考代码

{{% admonition note "nsqrtn 做法（链表，每次换块）" true %}}

```cpp
#include <iostream>
#include <cstdio>
#include <list>
#include <algorithm>

using namespace std;

const int N = 1e5 + 5;
const int B = 1000;

int n, m, a[N], bl[N], bll[N], blr[N], cnt[N / B + 5][N];
list<int> lst[N / B + 5];

int main()
{
    scanf("%d", &n);

    for (int i = 1; i <= n; ++i) scanf("%d", a + i);

    for (int i = 1, l = 1, r; l <= n; l = r + 1, ++i)
    {
        r = min(n, l + B - 1);
        bll[i] = l;
        blr[i] = r;
        for (int j = l; j <= r; ++j)
        {
            bl[j] = i;
            ++cnt[i][a[j]];
            lst[i].push_back(a[j]);
        }
    }

    scanf("%d", &m);

    int ans = 0;

    while (m--)
    {
        int type, l, r;
        scanf("%d%d%d", &type, &l, &r);
        l = (l + ans - 1) % n + 1;
        r = (r + ans - 1) % n + 1;
        if (l > r) swap(l, r);
        if (type == 1)
        {
            auto it = lst[bl[r]].begin();
            for (int i = bll[bl[r]]; i < r; ++i) ++it;
            int tmp = *it;
            --cnt[bl[r]][tmp];
            ++cnt[bl[l]][tmp];
            lst[bl[r]].erase(it);
            it = lst[bl[l]].begin();
            for (int i = bll[bl[l]]; i < l; ++i) ++it;
            lst[bl[l]].insert(it, tmp);
            for (int i = bl[l]; i < bl[r]; ++i)
            {
                int t = lst[i].back();
                lst[i].pop_back();
                lst[i + 1].push_front(t);
                --cnt[i][t];
                ++cnt[i + 1][t];
            }
        }
        else
        {
            int k;
            scanf("%d", &k);
            k = (k + ans - 1) % n + 1;
            ans = 0;
            if (bl[l] == bl[r])
            {
                auto it = lst[bl[l]].begin();
                for (int i = bll[bl[l]]; i <= r; ++i)
                {
                    if (i >= l) ans += *it == k;
                    ++it;
                }
            }
            else
            {
                auto it = lst[bl[l]].begin();
                for (int i = bll[bl[l]]; i <= blr[bl[l]]; ++i)
                {
                    if (i >= l) ans += *it == k;
                    ++it;
                }
                it = lst[bl[r]].begin();
                for (int i = bll[bl[r]]; i <= r; ++i)
                {
                    ans += *it == k;
                    ++it;
                }
                for (int i = bl[l] + 1; i < bl[r]; ++i) ans += cnt[i][k];
            }
            printf("%d\n", ans);
        }
    }

    return 0;
}
```

{{% /admonition %}}

{{% admonition note "nlog^2n 做法（Splay）" true %}}

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <queue>
#include <algorithm>

using namespace std;

struct Node;

int n, m;
vector<int> a;
vector<Node *> rt;
vector<queue<Node *> > q; // q[i] 存的是值为 i 的数在 t_0 中对应的节点
Node *empty, *lsen, *rsen;

struct Node
{
    int val, siz;
    Node *pa, *mir; // mir 是在 t_0 中对应的节点
    vector<Node *> ch;
    Node(Node *_pa = empty, int _val = 0, Node *_mir = empty)
    {
        siz = 1;
        pa = _pa;
        mir = _mir;
        val = _val;
        ch.resize(2, empty);
    }
};

void pushup(Node *x)
{
    x->siz = x->ch[0]->siz + x->ch[1]->siz + 1;
}

void rotate(Node *x)
{
    Node *y = x->pa;
    Node *z = y->pa;
    int k = x == y->ch[1];
    z->ch[y == z->ch[1]] = x;
    x->pa = z;
    y->ch[k] = x->ch[k ^ 1];
    x->ch[k ^ 1]->pa = y;
    x->ch[k ^ 1] = y;
    y->pa = x;
    pushup(y);
    pushup(x);
}

void Splay(Node *x, Node *goal)
{
    while (x->pa != goal)
    {
        Node *y = x->pa;
        Node *z = y->pa;
        if (z != goal) rotate((x == y->ch[1]) ^ (y == z->ch[1]) ? x : y);
        rotate(x);
    }
}

void makeroot(Node *& root, Node *x)
{
    Splay(x, empty);
    root = x;
}

Node* kth(Node *root, int x)
{
    Node *u = root;
    while (1)
    {
        if (u->ch[0]->siz >= x) u = u->ch[0];
        else if (u->ch[0]->siz + 1 == x) return u;
        else
        {
            x -= u->ch[0]->siz + 1;
            u = u->ch[1];
        }
    }
}

int rk(Node *& root, Node *x)
{
    makeroot(root, x);
    return x->ch[0]->siz + 1;
}

void link(Node *& root, Node *x, Node *y, int type = 0)
{
    makeroot(root, y);
    if (root->ch[type] != empty)
    {
        Node *u = kth(y->ch[0], y->ch[0]->siz);
        makeroot(root, u);
    }
    y->ch[type] = x;
    x->pa = y;
    pushup(y);
    if (y->pa != empty) pushup(y->pa);
}

void cut(Node *& root, Node *x)
{
    makeroot(root, x);
    Node *u = kth(x->ch[0], x->ch[0]->siz);
    makeroot(root, u);
    u->ch[1] = x->ch[1];
    x->ch[1]->pa = u;
    x->ch[0] = x->ch[1] = x->pa = empty;
    x->siz = 1;
    pushup(u);
}

Node* rkle(Node *root, int x)
{
    Node *u = root;
    while (1)
    {
        if (u->ch[1] != empty && rk(rt[0], kth(u->ch[1], 1)->mir) <= x) u = u->ch[1];
        else if (rk(rt[0], u->mir) <= x) return u;
        else u = u->ch[0];
    }
}

Node* rkge(Node *root, int x)
{
    Node *u = root;
    while (1)
    {
        if (u->ch[0] != empty && rk(rt[0], kth(u->ch[0], u->ch[0]->siz)->mir) >= x) u = u->ch[0];
        else if (rk(rt[0], u->mir) >= x) return u;
        else u = u->ch[1];
    }
}

Node* split(Node *& root, int l, int r)
{
    Node *x = kth(root, l);
    Node *y = kth(root, r + 2);
    makeroot(root, x);
    Splay(y, x);
    return y->ch[0];
}

Node* get(Node *& root, int p)
{
    Node *x = rkle(root, p);
    Node *y = rkge(root, p + 2);
    makeroot(root, x);
    Splay(y, x);
    return y->ch[0];
}

Node* build1(vector<int>::iterator l, vector<int>::iterator r, Node *pa)
{
    if (l == r) return empty;
    auto mid = l + (r - l) / 2;
    Node *cur = new Node(pa, *mid);
    cur->ch[0] = build1(l, mid, cur);
    q[*mid].push(cur);
    cur->ch[1] = build1(mid + 1, r, cur);
    pushup(cur);
    return cur;
}

void build1(Node *& root, vector<int>& a)
{
    root = build1(a.begin(), a.end(), empty);
    lsen = new Node();
    link(root, lsen, kth(root, 1));
    rsen = new Node();
    link(root, rsen, kth(root, root->siz), 1);
}

Node* build2(int l, int r, int val, Node *pa)
{
    if (l == r) return empty;
    int mid = (l + r) >> 1;
    Node *cur = new Node(pa, val);
    cur->ch[0] = build2(l, mid, val, cur);
    cur->mir = q[val].front();
    q[val].pop();
    cur->ch[1] = build2(mid + 1, r, val, cur);
    pushup(cur);
    return cur;
}

void build2(Node *& root, int size, int val)
{
    if (size == 0) return;
    root = build2(0, size, val, empty);
    Node *lsentry = new Node(empty, 0, lsen);
    link(root, lsentry, kth(root, 1));
    Node *rsentry = new Node(empty, 0, rsen);
    link(root, rsentry, kth(root, root->siz), 1);
}

int main()
{
    empty = new Node();
    empty->siz = 0;
    empty->ch[0] = empty->ch[1] = empty->pa = empty->mir = empty;

    scanf("%d", &n);

    a.resize(n, 0);

    for (int i = 0; i < n; ++i) scanf("%d", &a[i]);

    rt.resize(n + 1);
    q.resize(n + 1);

    build1(rt[0], a);

    for (int i = 1; i <= n; ++i) build2(rt[i], q[i].size(), i);

    scanf("%d", &m);

    int ans = 0;

    while (m--)
    {
        int type, l, r;
        scanf("%d%d%d", &type, &l, &r);
        l = (l + ans - 1) % n + 1;
        r = (r + ans - 1) % n + 1;
        if (l > r) swap(l, r);
        if (type == 1)
        {
            if (l == r) continue;
            Node *x = split(rt[0], l, r);
            Node *y = kth(x, 1);
            Node *z = kth(x, x->siz);
            int k = z->val;
            x = get(rt[k], r);
            cut(rt[k], x);
            Node *qaq = rkle(rt[k], l);
            Node *qwq = rkge(rt[k], l + 1);
            makeroot(rt[k], qaq);
            Splay(qwq, qaq);
            link(rt[k], x, qwq);
            cut(rt[0], z);
            link(rt[0], z, y);
        }
        else
        {
            int k;
            scanf("%d", &k);
            k = (k + ans - 1) % n + 1;
            if (rt[k] == 0)
            {
                ans = 0;
                puts("0");
                continue;
            }
            Node *x = rkle(rt[k], l);
            Node *y = rkge(rt[k], r + 2);
            makeroot(rt[k], x);
            Splay(y, x);
            ans = y->ch[0]->siz;
            printf("%d\n", ans);
        }
    }

    return 0;
}
```

{{% /admonition %}}
