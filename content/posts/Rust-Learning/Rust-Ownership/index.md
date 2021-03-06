---
title: "Rust Ownership"
date: 2021-01-24
draft: false
categories: [Programming Language]
tags: [Rust]
description: 
slug: ""
featured_image:
author: 1uvu
---

## 什么是所有权

所有权是 Rust 的核心功能也是最与众不同的特性，它让 Rust 无需垃圾回收（garbage collector）即可保障内存安全，对语言的其他部分有着深刻的影响。

### 栈和堆

栈和堆都是代码在运行时可供使用的内存，但是它们的结构和使用方式不同。

- 栈以放入值的顺序存储值并以相反顺序取出值。这也被称作 **后进先出**（*last in, first out*）。增加数据叫做 **进栈**（*pushing onto the stack*），而移出数据叫做 **出栈**（*popping off the stack*）。
- 栈中的所有数据都必须占用已知且固定的大小。
- 堆上存储编译时大小未知或大小可能变化的数据。
- 操作系统寻找足够大的空闲内存，并返回其指针，称为**在堆上分配内存**（*allocating on the heap*），有时简称为 “**分配**”（allocating）。

因此，对于编程语言来说，如何对堆上的数据进行管理是需要解决的大问题，管理操作主要包括分配和回收。而对于不同编程语言所采取的回收机制，则体现了语言的核心设计思想，这对程序运行速度和效率的影响也十分关键，目前存在三种主流的堆数据回收机制：

- **垃圾回收**（*garbage collector*，*GC*）机制，在程序运行时不断地寻找不再使用的内存，如 Java，Golang 等。
- 程序员必须亲自分配和释放内存，如 C，CPP。
- 通过所有权系统管理内存，如 Rust。

### 所有权规则

- Rust 中的每一个值都有一个被称为其 **所有者**（*owner*）的变量。
- 值有且只有一个所有者。
- 当所有者（变量）离开作用域，这个值将被丢弃。

### 变量作用域

Rust 中，如果变量本身不受其它影响，那么此变量的作用域就是它当前所处的代码块。

- 当变量 `s` **进入作用域** 时，它就是有效的。
- 这一直持续到它 **离开作用域** 为止。

### String 类型

与上一章中的字符串标量类型不同，`String` 类型的变量是被分配在堆上的。

### 内存与分配

变量 `s`  离开它的作用域时，它所拥有的内存会被自动释放回收。

> 实际上，当变量离开作用域时，Rust 会为我们调用一个特殊的函数。这个函数叫做 `drop`，在这里 `String`的作者可以放置释放内存的代码。Rust 在结尾的 `}` 处自动调用 `drop`。在 C++ 中，这种 item 在生命周期结束时释放资源的模式有时被称作 **资源获取即初始化**（*Resource Acquisition Is Initialization，RAII*），而 Rust 显然在这种思想的基础上进行了拓展，可以理解为：**一种变量在退出作用域时回收内存的模式。**

下面介绍所有权需要解决的几个问题。

### 变量与数据交互的方式（一）：移动

传统的**浅拷贝**（*shallow copy*），会出现问题**二次释放**（*double free*）的错误。而在 Rust 中，在将一个**所有数据在堆上的变量**赋值给一个新变量时，**原来的变量会变成无效**，这个操作被称为 **移动**（*move*），这样就避免了二次释放问题。

而对于**不需要被分配在堆上**的数据类型来说，则**不存在移动这一操作**，即在将一个**所有数据在栈上的**变量赋值给一个新变量时，**原来的变量会依然有效**。以各种标量为例，这是因为这些标量没有实现 `Drop` 这一 *trait* 而实现了 `Copy` 这一 *trait*，反之，被分配在堆上的数据类型只实现了 `Drop` 这一 *trait* 而没有实现了 `Copy` 这一 *trait*。

总的来说，对于一种数据类型，`Copy` 和 `Drop` 的 *trait* 不能同时出现。

### 变量与数据交互的方式（二）：克隆

**深拷贝**（*deep copy*），Rust 中的**克隆**（*clone*）与其等价，只有**需要被分配在堆上**的数据类型（只实现了 `Drop` 这一 trait），才可以被克隆，且要注意Rust 永远也不会自动创建数据的 “深拷贝”。

另外，与克隆相区分，Rust 中还存在着**拷贝**（*copy*）这一方式，只有**不需要被分配在堆上**的数据类型才可以被拷贝（这个结论只是我的理解，也就是，对于标量这些数据类型来说，不存在深浅拷贝的区别的意思）。

### 所有权与函数

注意函数调用时，将变量作为实参传入函数后，变量的所有权移动给了实参，如果其不可 Copy，此变量就变得无效了，也可理解为此变量的作用域到此结束。

### 返回值与作用域

可以通过函数的返回值的移动来转移所有权，可以直接将实参返回来将其所有权重新移交给原变量。

当然这种返回参数的所有权的方法是不推荐的，由于函数参数的移动问题，就产生了引用。

## 引用与借用

引用是获取变量使用权但不获取其所有权的一种方式，引用指向的是被引用变量的地址，而不是内存中数据的地址，实际上我们拥有的是引用的所有权，所以 Drop 的也是引用变量而已。

引用用&，解引用用*（即直接访问原始变量，而不是指向原始变量的指针）。

借用是指：当没有定义引用变量，而是直接使用&var，将变量的引用作为参数传入函数时，此时就发生了借用。

### 可变引用

可变引用，使用mut来声明，可变引用的问题就是要避免由于可变引用所导致的**数据竞争**（*data race*）。引用之间的数据竞争主要包括如下三种行为情况（实际上就是读者和写者的数据同步问题）：

- 同一作用域同时存在一个以上对变量的可变引用。可以通过添加大括号，即通过分割作用域（实际上还是没有同时出现在同一作用域）作为一种临时替代的解决方案。
- 同一作用域同时存在可变引用和不可变引用。
- 没有进行同步数据访问的机制。

所以，Rust 编译器会检查上述三种情况，给出编译错误信息。

### 悬垂引用

**悬垂引用**（*Dangling References*）指的就是 **悬垂指针**（*dangling pointer*）问题：当释放了一段内存但是保留了指向此内存的指针。可以从多个角度来描述这一问题所导致的结果：

- 在分配一段内存时，此内存有可能已经有指针指向它了。
- 换句话说，悬垂指针所指向的内存可能会被分配给其他持有者（其他变量）。

在 Rust 中编译器确保引用永远也不会变成悬垂状态：当你拥有一些数据的引用，编译器确保数据不会在其引用之前离开作用域。即包含如下两方面内容：

- Rust 编译器不允许存在指向无效变量的引用，即引用在定义时，编译器会检查待引用的变量是否有效。
- Rust 编译器会通过控制变量的有效性來确保指向变量的引用也保持有效性，即引用定义后，编译器会保证变量始终是有效的。

本质上，悬垂引用问题就是引用的有效性问题。

### 引用的规则

概括一下之前对引用的讨论：

- 在任意给定时间，**要么** 只能有一个可变引用，**要么** 只能有多个不可变引用。
- 引用必须总是有效的。

## Slice 类型

除了堆上的数据类型，*slice*（无论是标量的 slice 还是其它数据类型的 slice）也没有所有权

### 字符串 Slice

以 `String` 为例，由于纯数字索引与 `String` 的状态本身完全没有关联，纯数字索引在作用域内，如果 `String` 改变了，那么索引对 `String` 的访问就会产生不同的结果，即索引与 `String` 中的数据不同步了。

*Slice* 可以解决这个问题，它是**对数据类型中一部分值的不可变引用**，在这个 `String` 的例子中它是 `&str` 类型，注意：标量字符串类型的分片也是 `&str` 类型。

而由于 *slice* 本身就是引用，所以在定义了 *slice* 后，对原本 `String` 的修改都会被 Rust 编译器检查到。



本章节中深入浅出地细致阐述了 Rust 所有权机制的来龙去脉，它体现了程序语言设计中对于安全性和易用性等的相互取舍。

其中涉及到了许多优秀的程序设计思想及和系统底层原理，相信无论是对于新手还是资深开发者来说，这一节的内容都让人受益匪浅。