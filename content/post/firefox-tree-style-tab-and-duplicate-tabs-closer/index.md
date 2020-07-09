+++
title = 'Firefox 的 Tree Style Tab 和 Duplicate Tabs Closer 插件'
date = 2020-07-09T13:00:46+08:00
draft = false
categories = ['技术']
tags = ['浏览器']
+++

你在使用浏览器时是否有这样的烦恼？

-   到底开多少个标签页？
    1.  开很多浏览器标签页 ⇒ 找不到标签页在哪，切换麻烦。
    2.  开很少浏览器标签页 ⇒ 需要频繁地打开、关闭标签页。
-   标签页之间层次混乱，不同类别的标签页混在一起。
-   从不同的地方点链接打开同一个网址，关了一个忘记关另一个。
-   ……

使用 Firefox 的 Tree Style Tab 和 Duplicate Tabs Closer 插件，就可以解决上述烦恼。

<!--more-->

{{% img tree-style-tab png %}}

（环境：Arch Linux, KDE Plasma, Dark Breeze）

## 安装 Tree Style Tab

1.  安装扩展：[Tree Style Tab – 下载 🦊 Firefox 扩展](https://addons.mozilla.org/zh-CN/firefox/addon/tree-style-tab/)
2.  打开侧栏（默认快捷键为 F1）。
3.  （可选）在 Firefox 的首选项中开启“恢复先前的浏览状态”。这能使长期打开很多标签页变得有意义。
4.  （可选）隐藏传统标签栏：[How to hide tab bar (tabstrip) in Firefox 57+ Quantum - Super User](https://superuser.com/a/1268734)
5.  （可选）使用 Firefox 的“定制”功能，在地址栏左侧的按钮的左侧添加弹性空白，使侧栏与空白等宽。

## 安装 Duplicate Tabs Closer

1.  安装扩展：[Duplicate Tabs Closer – 下载 🦊 Firefox 扩展](https://addons.mozilla.org/zh-CN/firefox/addon/duplicate-tabs-closer/)
2.  点击扩展图标→设置按钮进行配置。

    我的配置：

    ![](/post_img/firefox-tree-style-tab-and-duplicate-tabs-closer/duplicate-tabs-closer-settings.png)

    其中，“ホワイトリスト”是白名单，内容是正则表达式，表示哪些标签页会被排除在外（重复了也不会被关闭）（我感觉这个东西应该叫黑名单 :new_moon_with_face:），而 `^(?!.*github.com).*$` 匹配的是所有不包含 `github.com` 的网址，也就是说只有网址包含 `github.com` 的重复标签页才会被自动关闭。我设置这个规则是因为 GitHub 上的很多页面是我一直打开着的，而我又经常会从邮箱里点进已打开的页面。

## 使用 Tree Style Tab 和 Duplicate Tabs Closer

### 随心所欲地新建标签页

开不开标签页？开！

以后这个标签页还可能会用到，要留着吗？留！

[Firefox 67 的更新](https://www.ghacks.net/2019/03/01/firefox-67-automatically-unload-unused-tabs-to-improve-memory/) 使得开很多标签页不会占用过多的内存，Tree Style Tab 则使得开很多标签页不会变得杂乱无章，Duplicate Tabs Closer 使得你不会打开很多重复的标签页。所以，你在打开新标签页时基本上不需要担心打开的标签页过多（当然，如果真的太多（上百个），或者树的节点的度数太大，就另说了）。

### 折叠，然后批量关闭

若一棵子树处于折叠状态，你可以一键关掉整棵子树。

{{% fold "这在搜索解决方案时非常有用" %}}
{{% video src="close-tabs" width="240px" %}}
{{% /fold %}}
