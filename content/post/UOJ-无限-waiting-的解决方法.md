+++
title = "UOJ 无限 waiting 的解决方法"
date = "2019-11-28T20:10:41+08:00"
categories = ["技术"]
tags = ["OJ"]
description = ""
aliases = ["/post/UOJ-无限-waiting-的解决方法", "/UOJ-无限-waiting-的解决方法"]
+++


本文讲的是自己搭建的 UOJ 如何解决无限 waiting，而不是 <http://uoj.ac> 如何解决无限 waiting（后者大概要联系 vfk..反正我是没遇到过）。

<!--more-->

这个问题困扰了我很久..

然后在若干次数据回滚后，我发现每次都是使用 git 后出现无限 waiting。进而发现是在 `git reset --hard` 后发生这种情况。

最后，我发现了问题的真正原因：git 把某些文件的权限改了，导致 docker 启动时无法运行 judger。

解决方法：

```bash
docker exec -it uoj chmod +x /opt/uoj/judger/judge_client
docker exec -it uoj chown local_main_judger: /opt/uoj/judger/log/judger.log
docker restart uoj
```

（当然，如果你改了 docker 容器名 / judger 用户名，甚至网页和 judger 不是同一个 docker，就不能照搬上面的命令了。）