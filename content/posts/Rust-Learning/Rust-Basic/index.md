---
title: "Rust Basic"
date: 2021-01-24
draft: false
categories: 
 - Programming Language
tags: 
 - Rust
description: 
slug: ""
featured_image:
author: 1uvu
---

## 前言

作为一门安全、高效的编程语言，其参考了很多其它语言的优秀设计思想，**Rust** 获得了越来越多资深开发者的热爱。在对安全性和效率都十分看重的领域，如自动化测试、操作系统、编译器、区块链技术、数据库等，其应用十分广泛。

但作为一门尚且年轻的语言，尽管其社区维护全面友好，但碍于陡峭的学习曲线，如果想要更好地掌握 Rust，要求同时熟悉 C 系语言和 FP 系语言的特性和使用，这在语言学习的开始就劝退了大部分人。

因此，为了坚持将 Rust 学习应用贯彻到底，基于 Rust 官方中英文文档，我把自己学习和实践的过程以笔记的形式整理下来。

笔记中主要提取文档中的重点内容，记录对 Rust 的深入理解，而不去计较百科皆知的基础知识。整个笔记的顺序主要按照官方[中](https://kaisery.github.io/trpl-zh-cn/title-page.html)[英](https://doc.rust-lang.org/book/)文文档来组织，在重要的部分，我会给出个人的一些整理。

**建议边读边看本笔记。**

开发环境：Win10 + Intellij IDEA + Rust nightly

## 入门指南

### 安装

在按照官方文档安装之前，需要本机安装好通用 C++ 开发环境（使用 Visual Studio Installer）

在这里要熟悉 Rust 生态中 *rustup* 的几个常用用法，如下

`rustup `

```powershell
rustup install <toolchain> # toolchain 即 release channel，主要有三种 stable，beta 和 nightly，注意此处还要指定版本，如 nightly-2019-01-17
rustup default <toolchain> # 切换默认 toolchain
rustup update # 更新 toolchain
rustup self uninstall # 卸载当前 toolchain
rustup doc # 查看本地文档
```

### Hello, World!

需要了解到 Rust 程序需要先编译再运行，即 Rust 是一种 **预编译静态类型**（*ahead-of-time compiled*）语言

### Hello, Cargo!

本节需要熟悉 [*cargo*](https://doc.rust-lang.org/cargo/) 的常用用法及其初始化的项目结构，*Cargo.toml* 的大致结构

`cargo`

```powershell
cargo new <project name> # 创建项目，以 Cargo 来管理
cargo check # 只检查能否编译，不产生可执行二进制文件
cargo build # 构建产生可执行二进制文件，不运行
cargo run # 构建并运行
cargo build --release # 构建并发布可执行二进制文件，此过程会对代码进行优化，但会增加构建时间
cargo update # 基于 toml 中的 dependencies 更新标准库版本和处理依赖，前提是对 toml 进行了修改
```

`项目结构`

- `.git/`
- `src/main.rs`
- `.gitignore`
- `Cargo.toml`

`Cargo.toml`

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
edition = "2018"

[dependencies]
```

此配置文件格式使用的是 toml 格式，而至于具体详细的字段，不是初学所要考虑的事情，建议遇到一个就查一个，这是交给搜索引擎做的事儿。

## 猜猜看游戏教程

这一节包括了简单程序可能会涉及到的大部分东西：

- 变量定义：`let`
- 变量可变：`mut`
- 类型自动推导
- 标准库引入：`use`
- 命名空间：`::`
- 标准输入：`io.stdio()`
- 可变引用：`&mut guess`，**引用**（*reference*）
- 基本错误处理：`except()`
- 打印字符串宏：`println!()`
- 输出格式化/占位符：`{}` 代表标准打印格式化，`{:?}` 或 `{:#?}` 代表 Debug 打印，这与要待输出类型的 `Display trait` 还是 `Debug trait` 有关。
- 堆数据定义：`String::new()`
- 关联函数：`::`，即函数本身不需要指定生成一个实例就可以使用，在 Java 等面向对象语言中称为**静态函数**（*static function*）或**静态方法**（*static method*）
- Result 枚举：成员包括 `Ok` 和 `Err`，`read_line()` 方法返回 `io::Result` 枚举值传递给 except() 方法
- 扩展库引入：**库**又称为 *crate*，来自于 [Crates.io](https://crates.io) ，在 *Cargo.toml* 中的依赖中加入库的**语义化版本**（*Semantic Versioning*）（*Semver*）
- Cargo.lock 文件：确保构建是可重现的，cargo 构建时检查其依赖版本来决定是否重新下载依赖和编译依赖库
- trait：Rust 核心之一，部分类似于 Java 中的接口，但是又很大不同，现在不用纠结。
- Ordering 枚举：`std::cmp::Ordering`，三个值 `Ordering::Less`，`Ordering::Greater`，`Ordering::Equal`
- => 语法：模式匹配成功时执行之后的代码
- match 的分支和模式匹配：match 有多个**分支**（*arms*），一个分支对应一个**模式**（*pattern*）
- 隐藏机制：允许使用同名的变量**隐藏** （*shadow*）之前的变量
- 循环：`loop`，`for`，`while`，`break`
- _ 通配符值

## 常见编程概念

此处介绍了与其他语言十分类似的概念

### 变量与可变性

- Rust 中**变量默认是不可变的**，***let mut*** 来定义**可变变量**（*mutable variable*）
- **不可变变量**与（*immutable variable*）**常量**（*constants*）的区别
  - 使用 `let` 声明不可变变量，使用 `const` 声明常量而不是 `let`
  - 声明常量时必须注明其数据类型
  - 常量可在任意作用域内声明，包括**全局作用域**，常量名**英文字母全大写**
  - 在声明它的作用域之中，常量在整个程序生命周期中都有效
- 变量的**隐藏** （*shadow*）：重复使用 `let` 来多次隐藏一个变量，复用变量名

### 数据类型

Rust 是**静态类型**（*statically typed*）的语言，即编译时就要知道或是可以推导出所有变量的类型

- **标量**（*scalar*）类型：一个单独的值，四种基本的标量类型：整型、浮点型、布尔类型和字符类型
- 除 byte 以外的所有数字字面值允许使用类型后缀，例如 `57u8`
- 使用 `_` 来分割数字字面值
- float 64 和 float 32 速度几乎一样
- char 大小为 **4 个字节**，即可以表示一个 Unicode 字符
- **复合类型**（*Compound types*）：
  - **元组**（*tuple*）：可以使用**模式匹配**（pattern matching）来**解构**（destructure）元组值，如 `let tup: (i32, f64, u8) = (500, 6.4, 1);`，也可使用 `tup.0` 来访问元组元素。
  - **数组**（*array*）：与元组不同，数组中的每个元素的类型必须相同，且Rust 中的数组是固定长度的：一旦声明，它们的长度不能增长或缩小，即声明时要指定长度和类型，如 `let a = [3; 5];`。允许变长的“数组”称为**向量**（*Vector*）。并且，Rust 编译器不允许访问数组之外的数据。

### 函数

- 函数定义时指定的**参数**（*parameters*）也称为**形参**，向函数传入的参数值称为**实参**，也称为**参数**（*arguments*）
- 每个形参都必须指定其数据类型
- 返回值的数据类型通过 `->` 来指定
- **语句**（*Statements*）是执行一些操作但不返回值的指令，如赋值语句。**表达式**（*Expressions*）计算并产生一个值。注意区分
- 函数最后一行的表达式，如不加 `;` 会被认为是返回值

### 注释

普通注释使用 `//`，另外还有**文档注释**

### 控制流

- **分支结构**（*branching construct*）
  - 任何**代码块**（被 `{}` 包裹的部分）都是可以返回值的
  - `if-else if-else` 的条件表达式必须返回 *bool* 值，而不能是数字
  - `match` 
- **循环**（*loops*）：`loop`、`while` 和 `for`，及 `break`、`continue`、`iter()` 和`Range` 类型（如 `(1..4)`）



Rust 官方文档基础知识部分到此结束。