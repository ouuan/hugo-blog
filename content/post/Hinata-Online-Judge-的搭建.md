+++
title = "Hinata Online Judge 的搭建"
date = "2019-09-21T21:58:04+08:00"
categories = ["技术"]
tags = ["OJ", "工程"]
description = ""
+++


程序员就是代码重用的艺术家（

<!--more-->

这周回到学校，基于 [社区版 UOJ](https://github.com/UniversalOJ/UOJ-System) 搭了一个校内 OJ。

过程中现在还记得的遇到的问题，是重启后显示 "wrong database"，解决方案是 docker 里运行命令 `service mysql restart`。

一周里大部分时间都是在魔改，并且是 [开源](https://github.com/ouuan/Hinata-Online-Judge) 的。

这几天算是稍微体验了一下程序员的生活。会为精妙的代码重用而惊叹，也会为一个 bug 而苦苦求索。commit 中的两行代码，可能是若干小时搜索的结果，经过了数十次错误的尝试。

以前我对程序员的印象是「苦力活」，现在稍微有点改变了。诗人给同一个事物起不同的名字，数学家给不同的事物起相同的名字，程序员给不同的需求使用同一个函数。

回想起来，不到一年前，也就是我开始使用 hexo 博客之前，我还对 web 开发一无所知。只不过现在所会的那些东西都是通过阅读源码 + 需要什么就搜什么学到的，导致有很多不清楚的地方，这也是导致效率低下的重要原因。

总之，欢迎大家自由选取 Hinata Online Judge 的 [feature](https://github.com/ouuan/Hinata-Online-Judge/issues/1) copy 到自己的 UOJ 里！

然而代码太不规范了，所以暂时不太准备发 pr...也欢迎大家来帮我规范一下发 pr。