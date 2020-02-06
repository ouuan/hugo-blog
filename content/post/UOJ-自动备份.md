+++
title = "UOJ 自动备份"
date = "2019-09-27T12:01:05+08:00"
categories = ["技术"]
tags = ["OJ"]
description = ""
+++


作为一个刚刚上线两周就回滚了无数次数据的 OJ，备份自然是很重要的~

<!--more-->

## 一些脚本

### commit.sh

```bash
#!/bin/bash

time=$(date "+%Y%m%d%H%M%S")
echo ${time}
docker commit uoj uoj_backup_${time}
docker images --format "{{.ID}}: {{.CreatedSince}}" | grep 'day' | cut -d : -f 1 | xargs docker image rm
```

commit，然后删掉比较久远（在 CREATED 中显示为 XX days ago）的镜像节约空间（不然在自动备份下过不了几天硬盘就爆了）。

### save.sh

```bash
#!/bin/bash

time=$(date "+%Y%m%d%H%M%S")
echo ${time}
docker commit uoj uoj_backup_${time}
docker save -o /home/ouuan/uoj/uoj_backup_${time}.tar uoj_backup_${time}
docker images --format "{{.ID}}: {{.CreatedSince}}" | grep 'day' | cut -d : -f 1 | xargs docker image rm
```

先 commit，然后存为 tar。即使系统挂了，也能从文件恢复。

### new.sh

```bash
#!/bin/bash

time=$(date "+%Y%m%d%H%M%S")
echo ${time}
docker commit uoj uoj_backup_${time}
docker stop uoj
docker rm uoj
docker image ls
echo "Please enter the version (after uoj_backup_): "
read version
docker run --name uoj -dit -p 23333:80 -p 3690:3690 --cap-add SYS_PTRACE "uoj_backup_$version"
```

便捷地从镜像创建新容器，在删除当前容器之前 commit。

## 计划任务

输入命令 `sudo crontab -e`。

（可能会先让你选择一个文本编辑器）然后输入：

```bash
30 * * * * bash path/commit.sh
0 22 * * * bash path/save.sh
```

`path` 就是放脚本文件的路径。

前面五项分别是 分钟 / 小时 / 日期 / 月份 / 星期，在符合条件时就会执行后面的命令。

上面的例子会在 `xx:30` 时 commit 一次，在每天晚上 10 点 save 一次。