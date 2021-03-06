+++
title = '功利主义下排队问题的数学原理'
date = 2021-04-04T00:44:54+08:00
draft = false
categories = ['随笔']
tags = ['食堂']
+++

本文以功利主义为指导思想，通过适当的数学建模，对一些排队过程中经常遇到的问题进行了探讨。

<!--more-->

## 排队问题

顾名思义，排队问题就是研究排队的过程。对于一些大家在生活中就能遇到，解释起来比较麻烦、反而难以理解的概念，就不在这里加以定义，以免引起读者不适。

**等待时间**：每个人自进入队伍到退出队伍的这段时间（的长度）。

**等待代价**：等待时间越久，等待代价越高；等待时间越久，单位时间所需代价就越高。换言之，代价是关于时间的单调递增的凹函数。并且，我们假定它“凹”的程度不大，所以等待时间可以作为等待代价的近似。

**排队段**：一段极长的时间，在这段时间中队伍一直非空。换言之，就是从第一个人开始排队（ta 前面没有人）直到最后一个人排完队（ta 后面没有人）的这段时间。

**操作用时**：一个人在退出队伍后进行某些操作（例如，从饭盒中拿出一碗饭）所用的时间，这个操作导致了后面一个人需要等待这个刚出队的人操作完成才能出队。为了简化问题，我们一般认为每个人的操作用时是相同的。

在下面的问题中，你的目标是最小化排队过程中所有人的等待代价之和。

## 总等待时间的一些计算方式

### 整体角度

不难看出，总等待时间是队伍中人数关于时间的积分。

### 个体进入队伍产生的影响

当一个人进入队伍时，队伍总等待时间的增量是其自己的等待时间加上同一排队段中在其之后出队的人数与其操作用时的乘积。换言之，你拿一碗饭的用时使排在你后面的每个人都等了这么久。

## 几例实际生活中的排队问题

### 先进先出

先来后到，天经地义，有道理可言吗？其实，由于代价是凹函数，先进先出是最优的选择，证明如下：

你现在在队伍的最后，我们考虑你与你前面一个人互换出队顺序是否更优。

记在你入队之前你前面的人已经等待了 $a$ 秒，在你入队时你们两个前面所有人出队并完成操作还需 $b$ 秒，每人操作需要 $c$ 秒，代价函数为 $f(x)$。

不换的话，代价为：$f(a + b) + f(b + c)$

换的话，代价为：$f(a + b + c) + f(b)$

$\because f(x)$ 是单调递增的凹函数

$\therefore f(a + b + c) - f(a + b) > f(b + c) - f(b)$

$\therefore$ 不换更优

若不是先进先出，可以等价为如上所述的若干次相邻两人交换，由于每次交换都更劣，总体也更劣。

### 离队首还远的话，允许他人横穿

其实这并不是一个严格的“排队”问题，因为这里引入了新情景 —— 有非排队者要横穿队伍。我不是在说出于道德你应该“舍己为人” —— 我是在说，如果你离队首还远，允许他人横穿，有百利而无一害。因为，还没排到你，你往前走并不会减少你的等待时间，却会增加要借过的那个人的等待时间。

但是，如果你已经接近队首了，让他人横穿可能增大你的操作用时，而这样做的代价是恐怖的：你后面的所有人都要承担这一代价！因此，若马上就要排到你了，还是尽量不要让其它事情耽搁，而应尽力减少你的操作用时。

### 千钧一发之际 —— 队伍是空的，而你身后有人

我们来想象一下：队伍是空的，你身后有一大波人 —— 你成功地成为了这一餐第一个拿到饭的人！此时，你有两个选择：

1.  正常速度走，你手还没碰到饭，后面一个人已经排在你身后了；
2.  快步向前，当你成功拿到饭后，队伍空出来了 0.01 秒，你身后的人成了一个新的排队段里第一个拿到饭的人。

在第二个选择中，你仅仅只是早了不到 5 秒拿到饭，但世界却发生了天翻地覆的变化 —— 一整个排队段的所有人都不需要等待你的操作用时！

让这一念之差所造成的后果显得更加糟糕的，或许是这样一个事实：这不仅是一个突变点，还是一个极劣点。因为，只要在同一个排队段内，来的越晚，你的操作用时所影响到的人就越少，所以如果你是一个冗长的排队段的队首，就意味着你选择了最糟糕的一个位置，而更好的选择应该是在教室里写几分钟作业再来（当然，前提是你有高效利用碎片化时间的能力），最佳的选择则是在不影响他人的情况下第一个拿到饭，即上面所说的第二种选择。
