+++
title = "AGC007F Shik and Copying String（贪心，实现）"
categories = ["题解"]
tags = ["贪心", "实现"]
date = "2019-08-16T23:34:10+08:00"
description = ""
+++


## 题目链接

[洛谷](https://www.luogu.org/problem/AT2173)

[AtCoder](https://atcoder.jp/contests/agc007/tasks/agc007_f)

## 题意简述

给你初始串 $S_0$ 和目标串 $T$，每一步操作可以将当前串 $S_i$ 变成 $S_{i+1}$，其中：

$$S_{i+1}[j]=\begin{cases}S_i[1]&j=1\\\\S_i[j]\text{ 或 }S_{i+1}[j-1]&j>1\end{cases}$$

求最少需要几次操作可以将当前串变为 $T$。

串长 $10^6​$。

<!--more-->

这题题解真的难写..之前觉得别人的题解写的不清楚，然而自己也写的不是很清楚...

## 简要做法

首先，这个过程可以用折线表示：

![zx](/post_img/AGC007F-Shik-and-Copying-String（贪心，实现）/zx.png)

（如果您在色觉方面存在障碍，还请见谅。）

可以发现，每条折线都尽量靠右是最优的，一旦画不下了，就加一行。

现在问题变成了如何高效地维护这一贪心。

当 $S_0=T$ 时，先特判掉，输出 $0​$。

由于每次拐点都会往左下移动一格，我们可以用队列来维护当前折线的每个拐点（折线往右拐的点，也就是 $S_i[j]=S_i[j-1]$ 的 $j-1$ 这个点）（不包括最后一行的拐点），其中靠近队首表示靠下（离 $T$ 较近）的拐点，靠近队尾表示靠上（靠近 $S_0$）的拐点。

详见代码（因为这题文字写出来不如代码好理解）：

## 参考代码

```cpp
#include <iostream>
#include <cstdio>
#include <string>
#include <queue>
#include <algorithm>

using namespace std;

int n, ans;
string s, t;
queue<int> q;

int main()
{
    cin >> n >> s >> t;

    if (s == t) // 特判两串相等
    {
        puts("0");
        return 0;
    }

    int up = n - 1, down = n - 1;

    while (down >= 0)
    {
        while (down && t[down - 1] == t[down]) --down; // 找到当前折线在最后一行最左的位置
        while (up >= 0 && (up > down || s[up] != t[down])) --up; // 找到当前折线在第一行最左的位置
        if (up < 0) // 如果第一行没有对应的字符，输出无解
        {
            puts("-1");
            return 0;
        }
        while (!q.empty() && q.front() - q.size() >= down) q.pop(); // 把当前折线不会碰到的部分弹出
        if (up != down) q.push(up); // 如果当前折线真的是“折线”而不是竖直下来不拐弯，就把 S1 的拐点压入队列
        ans = max(ans, (int)q.size() + 1); // 后文会解释为什么这样更新答案
        --down;
    }

    cout << ans;

    return 0;
}
```

## 补充说明

这个维护拐点的方式应该画画图就能明白。

最后剩下一个问题：为什么是这样更新答案？

换句话说：为什么答案是拐点个数的历史最大值？（加一是因为没有维护最后一行的拐点）

如果没有这个 pop 操作，应该是很显然的。但 pop 操作破坏了“队列中每个元素对应除最后一行外每一行最左位置”这个性质。

这里需要一个引理：

> 除了最后一行的拐点，其它拐点一定位于连续的前几行。

我们可以归纳地证明：

- 对于最右的那条折线，显然成立。
- 对于之后的每条折线，一定是先贴着上一条折线，再直接往下到最后一行。由于上一条折线满足引理，如果中途有一段没有拐点而后又出现拐点，中途的那一段就没有紧贴上一条折线。

有了这个引理，就可以~~感性理解~~说明为什么有 pop 操作的情况下答案依然是拐点个数的历史最大值了。