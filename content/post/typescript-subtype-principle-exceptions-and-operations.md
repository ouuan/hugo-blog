+++
title = 'TypeScript 中子类型判定的基本原则，特例，以及相关操作'
date = 2021-08-22T12:28:25+08:00
draft = false
categories = ['技术']
tags = ['TypeScript']
+++

类型是 TypeScript 中的基本概念，而两个类型之间是否存在子类型关系则决定了许多操作是否合法。多数类型的相关规则是非常符合直觉的，有些规则是符合逻辑而不一定符合直觉的，而少数规则则是例外。本文试图归纳一下子类型判定的基本原则，特例，以及相关操作。

<!--more-->

注：本文所述的是所有 strict 选项都启用时的情况。

由于 TypeScript 的类型检查是 [不完备（unsound）](https://www.typescriptlang.org/docs/handbook/type-compatibility.html#a-note-on-soundness) 的，而这些不完备的特例是根据实际开发需要而制定的，没什么规律，本文在这些特例处可能会有一些错误，如果发现可在评论指出。

# 子类型的基本原则

子类型的基本原则可以推导出绝大多数的规则，但是如果不透彻理解，可能会觉得由它推导出的一些规则是反直觉的。

## 类型即集合，集合即能力

在 TypeScript 中，除 `any` 外的所有类型都可以看作一个集合，而一个集合是通过“这个集合内所有元素都能做到的事情”，即这个类型的“能力”，来描述的。例如，`number` 类型能够完成数字相关的操作，`{ x : number }` 类型能够访问类型为 `number` 的 `x` 属性，`(x : string) => number` 类型能够接收一个 `string` 类型的参数，以及在这之后任意数量个任意类型的参数（在 JavaScript 中，多余的实参会被忽略），并返回一个 `number` 类型的值。

## 子类型即子集合，子集合即能力强

既然类型是集合，那么子类型就是子集合。这在一些情形，例如 union 的一部分、原始值的字面量中，很好理解。

在一些更加复杂的情形中，以“类型的能力”来进行理解，就十分重要了。具有更多的能力的类型，就是子类型。

### 对象类型的子类型

因为对象类型具有“访问某个特定属性”的能力，能访问更多属性的对象类型是子类型。

能访问同一个属性时，如果每个共有属性都是子类型，整个对象才可能是子类型。

如果一个属性是可选的，那么能够访问这个属性的能力就“更弱”，所以属性非可选的对象类型是属性可选的对象类型的子类型。

### 函数类型的子类型

由于函数可以接收比参数列表的长度更多的参数个数，而对多余的参数没有任何要求，参数列表短意味着能任意接受的参数更多，所以参数列表更短的函数类型是子类型。

因为函数类型具有“接收特定类型的参数，返回特定类型的值”的能力，对于同一个位置的参数，如果 A 函数类型的参数是 B 函数类型参数的子类型，反而是 B 可能是 A 的子类型，因为 B 能接收的参数的类型更多，其“接收参数的能力”更强。

可选参数 [比较特殊](#函数的可选参数与-rest-参数)。

返回值是子类型，函数类型才能是子类型。

## 简要梳理

类型宽泛，能表示的值多，能力弱；

类型具体，能表示的值少，能力强；

对象属性用来提供能力，属性能力越强，对象能力越强；

函数参数用来要求能力，参数能力越弱，函数能力越强。

# 子类型的一些特例

## `any`

`any` 无法被视作一个集合。任何类型都可以赋值给 `any`，而 `any` 可以赋值给除了 `never` 外的任何类型。

使用 `any` 是危险的，它的很多行为类似于禁用了类型检查。

## `unknown` 与 `never`

`unknown` 是全集，它不具有任何“能力”（无法访问任何属性，无法被调用……）。

`never` 是空集，也不具有任何“能力”（这是“集合越小，能力越强”的一个反例）。

## `void` 与 `undefined`

`void` 和 `undefined` 表示的都是 `undefined` 这个 JavaScript 类型。`undefined` 被规定为 `void` 的子类型，而在函数中能够体现出它们的主要区别：

-   在判定 A 函数类型是 B 函数类型的子类型时，如果 B 函数类型的返回值是 `void`，A 函数类型的返回值可以是任何类型。例如，`(number) => number` 是 `(number) => void` 的子类型，但不是 `(number) => undefined` 的子类型。这样的话，即使调用 callback 时不使用其返回值，也可以传入一个有返回值的函数作为 callback。
-   如果一个函数没有显式指定返回值类型，并且没有 `return` 语句，其返回值的类型会被推导为 `void`。
-   如果一个函数的返回值被显式指定为 `void`，这个函数可以没有 `return` 语句，也可以返回 `void`/`undefined`/`any`/`never` 类型的值。而当一个函数的返回值被显式指定为 `undefined` 时，这个函数必须有 `return` 语句，可以返回 `undefined`/`any`/`never` 类型的值，不能返回 `void` 类型的值。
-   如果一个处于末尾的参数的类型在化简后依然在字面上包含 `void`（可以是 `void`/`number | void`，但不能是 `unknown`/`unknown | void`），调用时可以不传入这个参数；如果不包含 `void` 但包含 `undefined`，可以传入 `undefined` 但是不能不传这个参数。不知道是故意设计成这样的还是意外..反正这个特性没啥用，应该使用可选参数而非 `| void` 来表示可以不传入参数。

## `readonly`

在对象类型中，可以使用 `readonly` 来表示一个属性不可被修改。这个 `readonly` 属性不仅不影响运行时行为，也不影响子类型判定：`{ readonly a : number }` 和 `{ a : number }` 是可以互相赋值的。

但是，`ReadonlyArray` / `readonly Type[]` 与 `readonly` 的对象属性不同，只读数组是不能赋值给可修改数组的。

## 函数的可选参数与 rest 参数

为了方便，涉及到可选参数和 rest 参数时，TypeScript 使用的是不完备（unsound）的子类型判定。这样的设计是为了避免不必要的麻烦，而据说这样不完备的类型检查一般不会在实际开发中带来问题。具体规则我暂时还没有完全搞清楚，可以参考 [reference](https://www.typescriptlang.org/docs/handbook/type-compatibility.html#optional-parameters-and-rest-parameters) / 自己试一试。一个比较简单的情形是，参数可选和参数必填之间是可以互相赋值的，前提是必填参数的类型有 `| undefined`。

## class

类有两个类型：“类自己”的类型，和类的实例的类型。一般来说，“类自己”用构造函数来表示：`new () => Class` 或者 `{ new () : Class }`；`static` 成员也是“类自己”的一部分。而类的实例的类型就是 `Class`。

在比较类的实例的类型之间是否是子类型时，对于 `public` 的成员以及只有一方有的 `private`/`protected` 成员，像普通对象一样比较。对于共有的 `private`/`protected` 成员，如果有子类型关系，这个成员在两个类中被定义的“位置”（关系最近的定义或重载了这个成员的基类或者自身）需要是一样的。如果一方是 `public` 而另一方不是，只有 `A` 是 `B` 的派生类时 `A` 才会是 `B` 的子类型。

# 与子类型相关的操作

## 赋值

在声明一个变量时，它会有一个或显式指定或隐式推导的 *声明类型*。在赋值时，所赋的值必须是这个声明类型的子类型。

赋值和子类型并不完全等同，具体来说，[在与 `any` / `enum` 有关时可能不等同](https://www.typescriptlang.org/docs/handbook/type-compatibility.html#subtype-vs-assignment)。

## 函数传参

传参可以看作将实参赋值给形参。

## type guard

进行类型判断时，会相应地使变量在局部变为符合条件的子类型。

## `extends`

`extends` 这个关键字表明了一种子类型关系。在 `class` 和 `interface` 中，这种关系相当于“继承”。而在 generic constraint / conditional type 中，`A extends B` 表示 `A` 是 `B` 的子类型（`B` 不需要是对象，也可以是原始类型）。

## union（并集）和 intersection（交集）

`A | B` 是 `A` 和 `B` 的并集，具有 `A` 和 `B` 的共同能力。

`A & B` 是 `A` 和 `B` 的交集，如果交集非空，得到的类型同时具有 `A` 的能力和 `B` 的能力，否则得到 `never`。

比较令人迷惑的一点是，当类型取并集时，能力反而取交集；当类型取交集时，能力反而取并集。

和基本的子类型判定一样，“并集”与“交集”已经准确地描述了它们的行为，但是这里还是说一下对于对象和函数而言的具体行为：

### 对象的并集和交集

对象的并集的属性是两方的共有属性，其中每个属性的类型为两方类型的并集。

对象的交集的属性是在至少一方出现的属性，其中共有属性的类型为两方类型的交集。

### 函数的交集和并集

如果有两个函数类型 `A` 和 `B`，不妨令 `A` 的参数列表长度小于或等于 `B` 的，那么：

-   `A | B` 的参数列表长度为 `B` 的参数列表长度，其中前 `A` 的参数列表长度个参数的类型为两方类型的交集，后面几个参数的类型为 `B` 的参数列表中的类型；返回值为两方返回值的并集。
-   `A & B` 相当于函数重载：如果实参符合 `A` 的参数列表，返回类型就是 `A` 的返回类型；如果实参不符合 `A` 的参数列表而符合 `B` 的参数列表，返回类型就是 `B` 的返回类型。
    
这两种看起来有些不对称的行为可以理解成逻辑表达式。例如，有两个函数类型：

```typescript
type A = (a : 1 | 2, b : { x : number, y : number }) => void;
type B = (a : 2 | 3, b : { x : number, z : number }) => void;
```

那么，`A | B` 要同时满足 `A` 和 `B` 的限制，对于参数列表，就是在说：

```javascript
((a === 1 || a === 2) && (typeof b.x === "number" && typeof b.y === "number"))
&&
((a === 2 || a === 3) && (typeof b.x === "number" && typeof b.z === "number"))
```

将顺序换一下，就是对每个参数分别进行限制：

```javascript
((a === 1 || a === 2) && (a === 2 || a === 3)) &&
((typeof b.x === "number" && typeof b.y === "number") && (typeof b.x === "number" && typeof b.z === "number"))
```

最后结果就是，`A | B` 的类型为 `(a : 2, b : { x : number, y : number, z : number }) => void`。


而 `A & B` 是满足 `A` 的限制或 `B` 的限制：

```javascript
((a === 1 || a === 2) && (typeof b.x === "number" && typeof b.y === "number"))
||
((a === 2 || a === 3) && (typeof b.x === "number" && typeof b.z === "number"))
```

由“或”所连接的两个括号不能直接拆开，所以就表现为“函数重载”，而非对每个参数独立地进行限制。

## type assertion

type assertion 的语法为 `expression as Type` 或 `<Type>expression`，即类型检查层面的“类型转换”（不影响运行时行为）。

这一语法要求 `expression` 的类型与 `Type` 有子类型关系，但是这一关系的方向是任意的，所以这个语法可以用来在类型检查阶段“赋予一个值它并不具有的能力”，这是很危险的。尤其是，`expression as any as Type` 这种语法允许进行任意的类型转换。

一般来说，使用 type assertion 将类型转为更宽泛的类型是没有必要的，而转为更具体的类型是危险的，所以尽量不要使用这种语法——除了 `as const`。

`as const` 主要有两个作用：

1.  在类型检查层面禁止修改对象的属性（但是这不会改变运行时的行为，并且仍然可以修改属性的属性）。
2.  将原始类型自身或具有原始类型的属性的类型设为字面量而不是这个原始类型。例如，`const a = 'GET'` 不能作为 `(method : 'GET' | 'POST') => void` 的参数，而 `const a = 'GET' as const` 可以。

例如：`{ a : 233 }` 的类型为 `{ a : number }`，而 `{ a : 233 } as const` 的类型为 `{ readonly a : 233 }`；`[2, 3] as const` 会得到类型为 `readonly [2, 3]` 的 tuple。

如果真的需要将一个值转化为更具体的类型，可以考虑使用 type guard。

# 亲手做实验！

不管是读博客、[handbook](https://www.typescriptlang.org/docs/handbook/intro.html) 还是 [reference](https://www.typescriptlang.org/docs/handbook/type-compatibility.html)，都比不上亲手操作一下。[TS Playground](https://www.typescriptlang.org/play) 就是一个做实验的好地方。
