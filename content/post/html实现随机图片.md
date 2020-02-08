+++
title = "html实现随机图片"
date = "2018-12-12T23:42:17+08:00"
categories = ["技术"]
tags = ["HTML/JavaScript"]
description = ""
aliases = ["/post/html实现随机图片", "/html实现随机图片"]
+++


> 注：暂时还不知道如何制作能被引用的随机图片，只能查看图片，而不能通过类似于`![](图片地址)`的方式查看。

> [demo](https://ouuan.github.io/randpic/people/)
>
> 欢迎投稿图片：[投稿地址](https://github.com/ouuan/ouuan.github.io/issues/19)

<!-- more -->

## 核心代码

```html
<script type="text/javascript" src="imagelist.json"></script>
<script type="text/javascript">
    var r=Math.floor(Math.random()*images.length)
    document.write("<img src="+images[r]+">")
</script>
```

图片列表保存在 `imagelist.json` 内。

## 参考示例

`https://ouuan.github.io/randpic/people/index.html`：

```html
<html>
    <head>
        <script type="text/javascript" src="imagelist.json"></script>
        <title>随机图片-人物类</title>
        <link rel="icon" type="image/png" sizes="32x32" href="/images/favicon32.png">
        <link rel="icon" type="image/png" sizes="16x16" href="/images/favicon16.png">
        <style>
            img
            {
                width: 100%;
                height: auto;
            }
        </style>
    </head>
    <body>
        <script type="text/javascript">
            var r=Math.floor(Math.random()*images.length)
            document.write("<img src="+images[r]+">")
        </script>
    </body>
</html>
```

`https://ouuan.github.io/randpic/people/imagelist.json`：

```json
var images=
[
  "https://z4a.net/images/2018/12/12/70469686_p0.png",
  "https://z4a.net/images/2018/12/12/69212051_p0.jpg",
  "https://z4a.net/images/2018/12/12/64660644_p0.jpg",
  "https://z4a.net/images/2018/12/12/61438972_p0.jpg",
  "https://z4a.net/images/2018/12/12/60141148_p0.png",
  "https://z4a.net/images/2018/12/12/1200296-20170715113653118-1762611401.jpg",
  "https://z4a.net/images/2018/12/12/71631241_p0.jpg",
  "https://z4a.net/images/2018/12/12/f0Q5-g62pXkZ5lT3cS1hc-rs.jpg",
  "https://z4a.net/images/2018/12/12/36224612_p0.jpg",
  "https://z4a.net/images/2018/12/12/64702477_p0.jpg",
  "https://z4a.net/images/2018/12/12/64670588_p0.jpg",
  "https://z4a.net/images/2018/12/12/61815260_p0.jpg",
  "https://i.loli.net/2018/12/12/5c10a02b0831b.jpg",
  "https://i.loli.net/2018/12/12/5c1119665c83a.jpg",
  "https://i.loli.net/2018/12/12/5c111a8bed8e8.jpg",
  "https://i.loli.net/2018/12/12/5c111ab43f7cf.jpg",
  "https://i.loli.net/2018/12/12/5c111ade38590.jpg",
  "https://i.loli.net/2018/12/12/5c111b8240591.png"
]
```

## UPD

研究了一下 js 后写了一下图片缩放：（代码很丑，毕竟是靠百度学了一个小时写出来的）（大括号不换行是因为sublime缩进写着写着就炸了，只好在网上格式化了一下）

```html
<html>
    <head>
        <script type="text/javascript" src="imagelist.json">
        </script>
        <title>
            随机图片-人物类
        </title>
        <link rel="icon" type="image/png" sizes="32x32" href="/images/favicon32.png">
        <link rel="icon" type="image/png" sizes="16x16" href="/images/favicon16.png">
    </head>
    <body style="margin: 0px; background: #0e0e0e;">
        <script type="text/javascript">
            var cur = 1;
            var xx = 0;
            var yy = 0;
            var rx = 0;
            var ry = 0;
            var nw;
            var nh;
            var mw;
            var mh;
            var w;
            var h;
            function setSize() {
                if (cur == 1) {
                    var p = document.getElementsByTagName("img")[0];
                    nw = p.naturalWidth;
                    nh = p.naturalHeight;
                    mw = window.innerWidth;
                    mh = window.innerHeight;
                    if (nw > mw || nh > mh) {
                        p.style = "cursor: zoom-in";
                    } else {
                        p.style = "cursor: auto";
                    }
                    if (nw * mh > nh * mw) {
                        h = nh * mw / nw;
                        w = mw;
                    } else {
                        w = nw * mh / nh;
                        h = mh;
                    }
                    p.style.width = w;
                    p.style.height = h;
                    p.style.marginTop = (mh - h) / 2;
                    p.style.marginLeft = (mw - w) / 2;
                } else {
                    if (nw > mw || nh > mh) {
                        p.style = "cursor: zoom-out";
                    } else {
                        p.style = "cursor: auto";
                    }
                }
            }
            function picLoaded() {
                setSize();
                window.onresize = function() {
                    setSize();
                }
            }
            function BigSmall() {
                mw = window.innerWidth;
                mh = window.innerHeight;
                if (nw > mw || nh > mh) {
                    if (cur == 1) {
                        cur = 2;
                        var p = document.getElementsByTagName("img")[0];
                        p.style = "cursor: zoom-out";
                        var e = event || window.event;
                        mw = window.innerWidth;
                        mh = window.innerHeight;
                        if (nw * mh > nh * mw) {
                            h = nh * mw / nw;
                            w = mw;
                        } else {
                            w = nw * mh / nh;
                            h = mh;
                        }
                        if (nw <= mw) {
                            xx = (mw - nw) / 2;
                            rx = 0;
                        } else {
                            xx = 0;
                            rx = (nw - mw) * (e.clientX - (mw - w) / 2) / w;
                        }
                        if (nh <= mh) {
                            yy = (mh - nh) / 2;
                            ry = 0;
                        } else {
                            yy = 0;
                            ry = (nh - mh) * (e.clientY - (mh - h) / 2) / h;
                        }
                        p.style.marginTop = yy;
                        p.style.marginLeft = xx;
                        document.body.scrollLeft = rx;
                        document.body.scrollTop = ry;
                    } else {
                        cur = 1;
                        setSize();
                    }
                }
            }
            var r = Math.floor(Math.random() * images.length);
            document.write("<img src=" + images[r] + " onload=\"picLoaded()\" onclick=\"BigSmall()\">");
        </script>
    </body>
</html>
```
