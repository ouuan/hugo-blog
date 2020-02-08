---
title: "从 Hexo 到 Hugo"
date: 2020-02-06T23:07:17+08:00
draft: false
categories: ['技术']
tags: ['博客']
weight: 1
---

该博客的前身是自 2018 年 12 月搭建的 hexo 博客，于 2020 年 2 月搬迁至 hugo。

<!--more-->

## Motivation

这次搬迁最直接的原因，是我把 hexo 博客从 Windows 搬到 Linux 失败了..

之前魔改太多，主题里各种文件都改过，甚至 node_modules 里面都被我改过很多地方，之前因此都放弃升级 hexo 和 NexT 主题，而最近开始主用 Manjaro，于是博客必须得搬到 Linux 上，但无论是简单复制过来还是重新使用 npm 安装包都未能如愿。

其实说不定还有很多解决方法，但我之前就听说过 hugo，于是就决定干脆搬过来好了。

但为什么是搬到 hugo，而不是升级 hexo 呢？

从 hugo 本身来说，它最显著的优点，就是快：

1. 安装快。这在本地安装时倒影响不大，而且如果你要修改主题还得用 yarn 安装包，得装上几分钟。最主要的是，安装快使得 CI 快，使用 CI 发布 hugo 博客就会非常容易且快捷。
2. 构建快。hugo 比起 hexo 的构建速度不是快了一点点，hexo 需要几十秒才能构建完的博客，hugo 只需不到一秒。这点更加重要，它不仅加速了 CI，而且本地构建几乎没有延时，加上 server 预览使用 websocket，直接分屏当预览写博客体验良好。

但弊端也同样明显：没有插件，导致很多功能得自己写，或者复制别人的代码。只不过对于 hexo 都能改到 node_modules 里面的魔改玩家来说，这都不算什么（

只不过我一直用的是 2018 年 12 月那时的 hexo 和 NexT，这一年来 hexo 可能也有很多改进，所以我的比较不一定准确。

## Process

hugo 的配置步骤就不说了，网上很多教程，而且这个博客是 [开源](https://github.com/ouuan/hugo-blog) 的，可以自己看。稍微提醒一下，CI 配置中的 `$GITHUB_TOKEN` 需要在 travis.ci 上进行设置。

从 hexo 搬到 hugo 可以参考这篇博客：[Hugo 与 Hexo 的异同 | reuixiy](https://io-oi.me/tech/hugo-vs-hexo/) 。

我自己使用的脚本（fork 然后稍微改了一点）：[ouuan/hexo2hugo](https://github.com/ouuan/hexo2hugo) 。

hugo 还支持 [aliases](https://gohugo.io/content-management/urls/#aliases)，所以链接改动可以轻松处理。

几个搞了比较久的功能：

1. 站内搜索
   可以参考 [1d0901f](https://github.com/ouuan/hugo-blog/commit/1d0901fca6725480450581bb7bec28e0b2afc4d6)，如果主题不是 even 的话我这个应该就不太能直接用了，得改一下 `layout/_default/search.html` 还有 `static/js/search.js` 中的 `render` 函数。其实如果主题不是 even 的话建议以 [Client side searching for Hugo.io with Fuse.js](https://gist.github.com/eddiewebb/735feb48f50f0ddd65ae5606a1cb41ae) 为基础而不是以我魔改过的版本为基础修改，可以以我的作参考。

2. 代码高亮
   推荐使用 highlight.js 而不是 chroma。even 主题把 highlight.js 的 CSS 放到了自己的 `_code.scss` 里，所以光是加上一个 highlight.js 的 CSS 是不够的，还得在 `_code.scss` 里把 hljs 相关的删掉，如果是 dark theme 还得修改背景颜色。

3. 代码复制按钮
   可以参考 [8347acf](https://github.com/ouuan/hugo-theme-even/commit/8347acfe30f386f00dd81c843a879755377cccf5)。全靠搜索引擎学来的 CSS 果然不够..就这么点东西我搞了四个小时，主要是滚动条、字体大小、行高、padding 之类杂七杂八的问题。

## Issues

因为是用脚本批量改的，改完之后也没有一篇篇去检查，难免会有改错的地方以及一些死链，可以直接在评论区指出，多谢大家了。

如果有原博客链接没有 redirect 而是 404 了，也请务必告诉我。

如果需要原 hexo 博客的，可以看 [ouuan.github.io/hexo-archive](https://github.com/ouuan/ouuan.github.io/tree/hexo-archive) 。
