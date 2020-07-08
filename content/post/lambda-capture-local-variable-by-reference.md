+++
title = 'C++ lambda 使用引用捕获局部变量的陷阱'
date = 2020-07-08T14:08:38+08:00
draft = false
categories = ['技术']
tags = ['C++']
+++

C++ NB $\times$

C++ UB $\sqrt{}$

<!--more-->

这是一篇流水帐。

### 学JS

前一天，我学习了 [JavaScript 中的闭包](https://zh.javascript.info/closure)。

### 写题

当天，我在写一道需要使用 [SAM](/post/后缀自动机sam学习笔记/) 的题。

### 想写闭包

我想到了昨天学的闭包，于是想“要是用 JS 就能写一个优美的迭代器用于访问 SAM 了”。
   
（想象中的代码：）

```cpp
function<int(char)> makeIterator()
{
    int cur = 0;
    return [this, &cur](char nxt) { return cur = t[cur].ch[nxt - 'a']; };
}
```

### 测试了一下，发现它 work 了！

（Arch Linux，g++ 10.1.0，`-std=c++11 -O2`）
    
```cpp
auto makeCounter = []
{
    int cnt = 0;
    return [&cnt] { return ++cnt; };
};

auto counter1 = makeCounter();
auto counter2 = makeCounter();

wtb(counter1());
wtb(counter1());
wtb(counter2());
wtb(counter1());
wtb(counter2());
wtb(counter2());
```

输出：

```
1
2
1
3
2
3
```

### 发现函数返回时还是会调用析构函数

```cpp
#include <iostream>
#include <functional>

using namespace std;

class Test
{
public:
    Test(int id) : m_id(id), m_value(new int(0)) { cout << "constructor: " << m_id << endl; }
    
    ~Test() { cout << "destructor: " << m_id << endl; }
    
    void add(int delta) { *m_value += delta; }
    
    int value() const { return *m_value; }

private:
    int m_id;
    int *m_value;
};

int main()
{
    auto makeCounter = [](int id) {
        Test cnt(id);
        return [&cnt] {
            cnt.add(1);
            return cnt.value();
        };
    };
    
    auto counter1 = makeCounter(1);
    auto counter2 = makeCounter(2);
    
    cout << counter1() << endl;
    cout << counter1() << endl;
    cout << counter2() << endl;
    cout << counter1() << endl;
    
    return 0;
}
```

输出：

```
constructor: 1
destructor: 1
constructor: 2
destructor: 2
1
2
1
3
```

### 甚至会轻而易举地 UB

（其它代码不变）

```cpp
auto makeCounter = [](int id) {
    Test cnt(id);
    auto ret = [&cnt] {
        cnt.add(1);
        return cnt.value();
    };
    cnt.add(1);
    return ret;
};
```

```
constructor: 1
destructor: 1
constructor: 2
destructor: 2
2
-72537979
-72537978
-72537977
```

### 甚至 [最开始](#测试了一下发现它-work-了) 那个版本，只要不开 O2 就会

输出：

```
1
2
3
4
5
6
```

### 发现有博客讲这个

[Modern C++中lambda表达式的陷阱_尘中远的程序开发记录](https://blog.csdn.net/czyt1988/article/details/80149695)

### 得出结论

当你以为 C++ NB 的时候，实际上它可能在 UB。

C++ 的局部变量确实是会在函数返回/语句块结束时销毁的，但销毁后由于 UB 可能还能用（

总之不要在局部变量销毁后还能执行的 lambda 中以引用捕获局部变量。
