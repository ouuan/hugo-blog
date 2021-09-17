+++
title = '实质条件与全称量词'
date = 2021-08-08T16:26:55+08:00
draft = false
categories = ['数学']
tags = ['逻辑']
+++

反对论证不代表反对论点。

但是... $\neg (p \Rightarrow q) \Rightarrow \neg q$。难道反对论证就是反对论点？

<!--more-->

{{% warning %}}
这篇文章的观点在严格的形式逻辑中是不正确的：上离散数学的时候被教育了，不带 forall 的 $x^2 = 1$ 它就不是个命题...
{{% /warning %}}

## $p \Rightarrow (q \Rightarrow p)$

如果一个命题是真的，那么无论给你什么条件都可以推出这个命题是真的。比如，“已知 x=3，请证明 1+1=2”，这显然可以做到。

当然，这个定理也可以严格证明。

## $\neg (p \Rightarrow q) \Rightarrow \neg q$

我们来看看上面那个定理的逆否命题 :thinking:

如果 $p$ 不能推出 $q$，那么 $q$ 是假命题...确实是这样的，但是好像有哪不对..

这不就是在说，如果论证是错的，论点就是错的？究竟哪出了问题？

## 隐含的 forall

实际上，我们在说 $p(x) \Rightarrow q(x)$ 时，省略了 $\forall x$。

例如，当我们说“若 $x^2 = 1$，则 $x=1$”时，我们表达的并不是 $x^2=1\Rightarrow x=1$，而是 $\forall x\in\mathbb{R}, x^2=1 \Rightarrow x=1$。这样的话，当我们说这个论证是错误的，我们实际上是在说 $\exists x\in\mathbb{R}$ 使得 $x\neq 1$，这样就完全正确了。如果没有 forall，$x^2=1\Rightarrow x=1$ 是错误的就可以推出 $x\neq 1$，但 $x$ 是可以为 $1$ 的。
