+++
title = "使用 testlib.h 写 lemon SPJ"
date = "2019-05-22T13:59:32+08:00"
categories = ["技术"]
tags = ["评测"]
description = ""
aliases = ["/post/使用-testlib-h-写-lemon-SPJ", "/使用-testlib-h-写-lemon-SPJ"]
+++


不用 testlib.h 是不可能写好 checker 的（逃

<!--more-->

大约这样就可以了：

```cpp
#include <fstream>
#include <cstdlib>
#include <cstdio>

using namespace std;

ofstream fscore;
char cmd[1000];

int main(int argc, char * argv[])
{
    fscore.open(argv[5]);
    
    int score = atoi(argv[4]);
    
    sprintf(cmd, "testlibchecker %s %s %s %s", argv[1], argv[2], argv[3], argv[6]);
    if (system(cmd)) fscore << 0 << endl;
    else fscore << score << endl;
    
    fscore.close();
    
    return 0;
}
```

注意 testlibchecker 实际使用时要使用绝对路径。