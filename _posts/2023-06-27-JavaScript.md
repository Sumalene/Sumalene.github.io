---
layout: single
title: "[JavaScript]==和==之间的区别"
categories: [Web, JavaScript]
last_modified_at: 2022-08-16
excerpt: "==和===，隐式类型转换,显式类型转换之间有什么区别？"
header:
  teaser: https://uploadstatic.mihoyo.com/bh3-wiki/2023/03/03/75216984/84bc5ae219025aa7b1a41f737b0d4861_1254439676903324710.png
---

<br>

在阅读《Inside JavaScript》第三章时，我了解了平等运算符（==）和匹配运算符（==）之间的区别。
让我们来看看ECMAScript的规范

<br><br><br>

# <span style="background-color:#FFF3B1">== 和 ===</span>

首先，**==**是一个**强制的平等运算符**。如果操作数在比较时是不同的类型，它允许**强制转换。因此，如果你想比较的操作数是不同的类型，它们将首先被转换，然后再进行比较。
**===*是一个**严格的平等**操作符，如果操作数属于不同的类型，它不允许强制转换；也就是说，如果操作数属于不同的类型，它们在比较时不会改变其类型。

```jsx
console.log(1 == "1"); // true
console.log(1 === "1"); // false
```

对于平等运算符，**两个操作数的类型不同，所以我们将它们转换为同一类型（数字）并确定它们相等**。严格的平等运算符确定这两个值不相等，因为没有进行类型转换。
因此，大多数JavaScript编码指南不建议使用==运算符进行比较，因为它可能会给类型转换带来错误的结果。

<br><br>

## 1. IsLooselyEqual

==运算符遵循什么算法进行比较？

`ECMAScript` 如果你看一下规范，它遵循这个算法。

![Untitled](https://user-images.githubusercontent.com/72294509/190646744-8b5b8708-f01e-4514-aaf4-3202fb313200.png)

- 如果x和y是同一类型，使用===运算符比较算法（`isStrictlyEqual`）。
- 如果x和y分别是`null`和`undefined`，则返回**true**。
- 当比较字符和数字时，使用`ToNumber`在比较前将类型强制为数字。
- 当比较一个字符和一个`BigInt`时，通过`StringToBigInt`将字符转换为`BigInt`，然后进行比较。(7)
- 如果x或y是一个 "boolean"，在通过 "ToNumber "将其类型强制转换成数字后进行比较。
- 如果x或y是一个原始类型（`String, Number, BigInt`），而另一个操作数是一个引用类型的`Object`，通过`ToPrimitive`将`Object`胁迫为原始类型，然后进行比较。
- 如果x或y是一个`BigInt`，另一个操作数是一个`Number`，根据两个条件返回一个值（13-a, 13-b）。
- 如果上述条件没有返回一个`true'值，则返回`false'。


<br><br>
