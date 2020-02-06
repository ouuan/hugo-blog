+++
title = "C++11 enable_if 的使用"
date = "2019-07-03T01:24:13+08:00"
categories = ["技术"]
tags = ["泛型编程", "C++"]
description = ""
+++


今天想更新一下自己的 [CF 模板](https://github.com/ouuan/CF-template)，然后发现由于写法问题可能要给每种整型（int，long，long long，unsigned int，unsigned long long……）分别定义函数，于是尝试搜了一下有没有什么好的代码重用方式，发现了 enable_if，还挺好用的，但去网上搜教程可能比较难学..我乱搞了两三个小时才学会。于是就来分享一下..

<!--more-->

## 重载的匹配

### SFINAE

SFINAE 是 substitution failure is not an error 的缩写，即匹配失败不是错误。就是说，匹配重载的函数 / 类时如果匹配后会引发编译错误，这个函数 / 类就不会作为候选。这是一个 C++11 的新特性，也是 enable_if 最核心的原理。

### 完整的重载匹配顺序（SFINAE 下）

1. 找到候选函数，去掉其中会引发编译错误的。
2. 完全匹配 > 提升转换 > 标准转换 > 用户定义的转换。
	- 完全匹配：
		1. 值 ↔ 引用
		2. [] → \*
		3. type(argument-list) → (type \*)(argument-list)（函数指针）
		4. type → const / volatile type
		5. type \* → const type
		6. type \* → volatile type \*
	- 提升转换：char / shorts → int，float → double。
	- 标准转换：int → char，long → double。
	- 用户定义的转换：类中的构造函数，类型转换函数等。
3. 非模板函数优先于模板函数。
4. 寻找“最佳匹配”，我自己也不是很了解，可以参见 《C++ Primer Plus（第五版）》8.5.4 或上网搜索。

若经过以上过程仍有多个候选函数，则会引发二义性错误。

## enable_if 的原理

enable_if 的定义类似于下面的代码：（只有 Cond = true 时定义了 type）

```cpp
template<bool Cond, class T = void> struct enable_if {};
template<class T> struct enable_if<true, T> { typedef T type; };
```

这样的话，`enable_if<true, T>::type` 即为 `T`，而 `enable_if<false, T>::type` 会引发编译错误（在 SFINAE 下，即不将包含这一 enable_if 的函数 / 类作为候选）。

## enable_if 的使用

enable_if 可以在任何地方充当一个类型使用，可以有实际意义，也可以新增一个多余的仅用来 enable / unable 的参数。

```cpp
#include <iostream>
#include <type_traits>

using namespace std;

template<int a, int b>
typename enable_if<a + b == 233, bool>::type is233()
{
    return true;
}

template<int a, int b>
typename enable_if<a + b != 233, bool>::type is233()
{
    return false;
}

int main()
{
    cout << is233<1, 232>() << endl << is233<114514, 1919>();
    return 0;
}
```

只不过，大多数时候 enable_if 都用来判断模板参数的类型，此时一般要和 `is_integral` 等模板类结合使用。

有关 `is_integral` 等相关模板类的信息可以参见 [C++ Reference](http://www.cplusplus.com/reference/type_traits/)。

`is_integral<T>::value` 是一个布尔值，在 `T` 为整型时为真，否则为假。

```cpp
#include <iostream>
#include <type_traits>

using namespace std;

template<typename T, typename = typename enable_if<is_integral<T>::value, void>::type>
bool isodd(T x)
{
    return x % 2;
}

int main()
{
    cout << isodd(4) << endl << isodd('a');
    //cout << isodd("qwq"); -- compile error
    return 0;
}
```

一个 OutputIterator 的例子：

```cpp
template <typename OutputIt, typename = typename enable_if<is_same<output_iterator_tag, typename iterator_traits<OutputIt>::iterator_category>::value || (is_base_of<forward_iterator_tag, typename iterator_traits<OutputIt>::iterator_category>::value && !is_const<OutputIt>::value)>::type>
void read(OutputIt __first, OutputIt __last) { for (; __first != __last; ++__first) read(*__first); }
```