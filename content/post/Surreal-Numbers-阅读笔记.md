+++
title = "Surreal Numbers 阅读笔记"
date = "2019-06-26T15:05:09+08:00"
categories = ["知识点"]
tags = ["博弈论", "超现实数"]
description = ""
aliases = ["/post/Surreal-Numbers-阅读笔记", "/Surreal-Numbers-阅读笔记"]
+++


今天模拟赛遇到了一道需要超现实数的题目，赛后在阅读 [Matrix67 的博客](http://www.matrix67.com/blog/archives/6333) 时听说了唐纳德所著的《Surreal Numbers》（中译：研究之美）这本书，于是就阅读了一下。

大约会把书里的定理证一遍吧..

学习超现实数的时候请假装自己不知道关于数字的一切知识，并且不要把定义的名字真的当回事（某些定义有着熟悉的名字，但可能与我们熟知的意义相同，也可能不同）。

[小说下载](/post_doc/[研究之美].（美）高德纳.扫描版.pdf)

<!--more-->

本文可能比较咕，不知道什么时候能填完坑...

## Conway's rules

（~~翻译挺神仙的~~）

> 创生二道，大小诸数盖由此出。

1.	凡数，皆合于前创二数之集，其位左者，无一大于或似于其位右者。

2.	甲数小于或似于乙数，当且仅当甲数之左集中无一大于或似于乙数，且乙数之右集中无一小于或似于甲数。


> Conway 检视二道，连呼妙哉！此二道真妙绝。

## Definitions

### Symbols

比较运算符上画一道斜线表示不满足该运算符。

$x\le y$ 表示 $x$ 小于或似于 $y$。

$x\ge y$ 表示 $y\le x$。

$x\equiv y$ 表示 $x$ 似于 $y$，即 $x\le y$ 且 $y\le x$。

根据下文会介绍的定理 (T4)，“不小于或似于” 即 "大于且不似于"，所以可以定义 $x<y$ 表示 $x\not\ge y$，$x>y$ 表示 $x\not\le y$。

$A\le x$（$A$ 是一个集合，$x$ 是一个数）表示 $A$ 中任意一个元素都 $\le x$。（其它运算符类似）

$x\le A$（$A$ 是一个集合，$x$ 是一个数）表示 $A\ge x$。（其它运算符类似）

$A\le B$（$A$ 和 $B$ 都是集合）表示 $A$ $\le$ $B$ 中任意一个元素。（其它运算符类似）

### Number

一个数 $x$ 可以表示为 $(X_L,X_R)$ 的形式，其中 $X_L$ 表示 $x$ 的左集，$X_R$ 表示 $x$ 的右集。即 $x=(X_L,X_R)$。

$x_L$ 表示 $X_L$ 中的一个元素，$x_R$ 表示 $X_R$ 中的一个元素。

### Rule #1

$$x_L\not\ge x_R$$

### Rule #2

$$x\le y\Leftrightarrow X_L\not\ge y\land Y_R\not\le x$$

## Theorems

### T1

$$x\le y\land y\le z\Rightarrow x\le z$$

{{% admonition note "证明" true %}}

假设该命题不成立，即存在 $x\le y,y\le z,x\not\le z$。

$\because x\not\le z$

$\therefore \exists\\ x_L\ge z\lor\exists\\ z_R\le x$

当 $x_L\ge z$ 时

​	$\because x\le y$

​	$\therefore x_L\not\ge y$

​	$\therefore y\le z,z\le x_L,y\not\le x_L$

当 $z_R\le x$ 时

​	$\because y\le z$

​	$\therefore z_R\not\le y$

​	$\therefore z_R\le x,x\le y,z_R\not\le y$

综上，无论是哪种情形，都会得到新的一组不满足原命题的数，而这组数的其中一个数会比原来的三个数中的一个创造的早，新的这组数的另外两个数就是原来的三个数中另外两个数。这样的话，若出现了一组不满足原命题的数，创造时间就会不断向前追溯，而追溯是有尽头的，因此这种情形不可能出现。

证毕。

{{% /admonition %}}

### T2 

$$X_L\le x\le X_R$$

{{% admonition note "证明" true %}}



{{% /admonition %}}

### T3

$x\le x$

{{% admonition note "证明" true %}}



{{% /admonition %}}

### T4

$$x\not\le y\Rightarrow y\le x$$

{{% admonition note "证明" true %}}



{{% /admonition %}}

### T5

$$x<y\land y\le z\Rightarrow x<z$$

{{% admonition note "证明" true %}}



{{% /admonition %}}

### T6

$$x\le y\land y<z\Rightarrow x<z$$

{{% admonition note "证明" true %}}



{{% /admonition %}}

### T7

$$Y_L<x<Y_R\Rightarrow x\equiv(x_L\cup Y_L,x_R\cup Y_R)$$

{{% admonition note "证明" true %}}



{{% /admonition %}}

（本文咕咕中……）
