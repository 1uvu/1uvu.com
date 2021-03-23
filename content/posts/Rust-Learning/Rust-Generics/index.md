---
title: "Rust Generics"
date: 2021-02-08
draft: false
categories: [Programming Language]
tags: [Rust]
description: 
slug: ""
featured_image:
author: 1uvu
---

此部分内容为原文档第十章《泛型、trait 和生命周期》中第一部分关于泛型的内容，将拆为三部分来学习。

不太精确地说，**泛型**就是一种抽象数据类型，关注数据类型的**行为**和其与其它类型的**关联**，而不关心数据类型的具体含义；换句话说，泛型可用来指代具有相同行为和关联的一系列具体的数据类型。

泛型是一种用来**处理重复**的技术。

本部分内容，将对泛型定义函数、结构体、枚举和方法进行相关介绍，最后讨论泛型对代码性能的影响。

需要提醒的是，本部分介绍的内容都比较简略（达到可以读懂开源代码的水平），更多进阶用法均需要边实践边查阅官方 api 文档。

## 在函数定义中使用泛型

使用泛型定义函数时，我们在**函数签名**中通常为**参数**和**返回值**指定数据类型的位置放置泛型。

示例：

```rust
fn foo<T>(list: &[T]) -> T {
// ...
}
```

函数 `foo` 有泛型类型 `T`。它有一个参数 `list`，它的类型是一个 `T` 值的 slice。`foo` 函数将会返回一个与 `T` 相同类型的值。

注意：

> 不同类型在传入泛型函数时，可能会出现 **trait 缺失**的错误。
>
> 如在函数中对泛型类型比较大小，则需要为该泛型类型**指定并实现** `std::cmp::PartialOrd`这一 trait。

## 结构体定义中的泛型

示例：

```rust
struct Point<T, U> {
    x: T,
    y: U,
}
```

## 枚举定义中的泛型

示例：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

## 方法定义中的泛型

上述的泛型结构体和泛型枚举，都可以在定义中定义结构体和枚举的泛型方法。

泛型结构体示例：

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

// 传入泛型方法的泛型并不一定是一致的，注意泛型名称的对应
impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x, // T
            y: other.y, // W
        }
    }
}
/* 如使用如下代码调用上面的泛型方法
fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
*/


// 可以单独为 Point<f32, f32> 实现方法
impl Point<f32, f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

## 泛型代码的性能

Rust 实现了泛型，使用泛型类型参数的代码相比使用具体类型并没有任何速度上的损失。

Rust 通过在编译时进行泛型代码的 **单态化**（*monomorphization*）来保证效率。单态化是一个通过**填充**编译时使用的**具体类型**，将通用代码**转换**为特定代码的过程。

如 `Option<T>` 中的 `T` 为 `i32` 时，Rust 会将泛型定义 `Option<T>` 展开为 `Option_i32` ，等同于，在代码中基于 `Option<T>` 添加了 `Option_i32` 结构体的实现。