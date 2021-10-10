+++
title = '使用 auto-merge 解决 dependabot'
date = 2021-10-10T09:33:17+08:00
draft = false
categories = ['技术']
tags = ['GitHub', 'dependabot']
+++

用过 dependabot 的同学，尤其是使用着一些更新频繁的依赖的同学，一定都体验过被 dependabot 支配的恐惧。使用 GitHub 的 auto-merge 功能可以很好地解决这一问题。

<!--more-->

## dependabot 的问题

1.  每个依赖都有一个 PR；
2.  dependabot 会通过 rebase 保持与 base branch up-to-date。

这两点合在一起，就变得非常恐怖了。每合并一个 PR，其它所有 PR 都会更新。除了 $O(n^2)$ 的邮件量，还有对 rebase 和 CI 的漫长等待，需要你每隔几分钟去点一下 merge 按钮，非常耗时间。

## Auto-merge

[Pull request auto-merge is now generally available | GitHub Changelog](https://github.blog/changelog/2021-02-04-pull-request-auto-merge-is-now-generally-available/)

## 使用 auto-merge

1.  在 Settings → Branches 中添加 Branch protection rule
2.  开启 "Require status checks to pass before merging" 和 "Require branches to be up to date before merging"，以及其它你想要的选项。
3.  在 Settings → Options 中开启 "Allow auto-merge"
4.  在每个 PR 中点 "Enable auto-merge"。
5.  等待 dependabot 的 PR 一个接一个地合上！
