---
title: "Rust Error Handling"
date: 2021-02-05
draft: false
categories: [Programming Language]
tags: [Rust]
description: 
slug: ""
featured_image:
author: 1uvu
---

Rust 将错误组合成两个主要类别：**可恢复错误**（*recoverable*）和 **不可恢复错误**。本部分内容的重点在于：1）辨别两种类型的错误；2）根据需要选择错误处理的方式，从错误中恢复还是停止执行。

## panic! 与不可恢复的错误

`panic!` 是一个宏，执行它时，程序会打印错误信息，接着**展开**（*unwinding*）并清理栈数据，即 *Rust* 会回溯栈并清理它遇到的每一个函数的数据；当然，也可以选择不展开不清理，而是直接**终止**程序，将栈数据交给 OS 来释放，这样可以减小程序的体积。

修改 `Cargo.toml` 配置

```toml
[profile.release]
panic = 'abort'
```

`panic!` 中的 *backtrace* 是一个执行到目前位置所有被调用的函数的列表。可通过设置 `RUST_BACKTRACE=1` 环境变量来得到一个 backtrace。

```powershell
RUST_BACKTRACE=1 cargo run
```

## Result 与可恢复的错误

大部分错误其实并没有严重到无法恢复需要停止执行的地步，因此可以通过捕获 `Result` 枚举类型来从错误中恢复。

```rust
enum Result<T, E> {  // T, E 都是泛型参数
    Ok(T),
    Err(E),
}
```

通常使用 `match` 来对 `Result` 做模式匹配，但是为了更加方便，`Result` 还定义了许多辅助方法，如 `unwarp()`、`except()` 方法等。

- 如果 `Result` 值是成员 `Ok`，`unwrap` 会返回 `Ok` 中的值。如果 `Result` 是成员 `Err`，`unwrap` 会为我们调用 `panic!`。
- `expect` 类似于 `unwrap` ，同时它还允许我们选择 `panic!` 的错误信息，且相比 `unwrap`，`expect` 显示的错误信息更加详细。

另外，Rust 还提供**传播**（*propagating*）错误机制来便于向调用者提供错误信息，并自行选择错误处理方式，需要注意如下几点：

- 使用 `Result<T, E> {<你的代码>}` 来定义方法要返回的 `Result`，**此处需要指定 `T` 和 `E` 的具体类型**。接着在代码块中需要返回错误的位置返回 `Err(e)` 即可，具体可以查看示例代码。
- 上述的返回 `Err(e)` 通常要使用 `match`，因此为了**消除大量的样板代码**，Rust 提供 `?` 运算符来**快速地返回可能存在的错误（Result 类型）**，只需将它放置于可能存在 `Err(e)` 的**语句末尾**即可，甚至可以在 `?` 之后直接使用**链式方法调用**来进一步缩短代码，链式调用的理解请查看代码示例 ***9-7*** 和 ***9-8***。
- `?` 只可用于返回 `Result` 或者其它实现了 `std::ops::Try` 的类型的**函数**中。

`match` 和 `?` 的作用机制还有一点不同：

> `?` 运算符所使用的错误值被传递给了 `from` 函数，它定义于标准库的 `From` trait 中，其用来**将错误从一种类型转换为另一种类型**。当 `?` 运算符调用 `from` 函数时，收到的错误类型被转换为定义为当前函数返回的错误类型。

## panic! 还是不 panic!

【待补充】

