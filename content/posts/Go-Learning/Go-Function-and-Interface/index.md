---
title: "Go Function and Interface"
date: 2021-03-23T18:30:04+08:00
aliases: []
categories: 
 - Programming language
tags:
 - Golang
description: 
featured_image:
draft: false
author: 1uvu
---



## 方法和接口

Go 中没有类，但是却可以为任意**自定义的类型**绑定其方法，只需为函数指定其**接收者** (T type)，type 为自定义的类型名，T 为指定的任意变量名。这意味着任意 type 类型的变量在实例化后，都可以直接向变量的使用者提供这个方法。

**具有接收者的函数变成了方法**。

自定义类型的方式有很多：自定义结构体或是直接使用 type 关键字使用其它任意类型作为新的类型，只需为其指定一个独特的类型名称即可。

### 方法

函数的接收者既可以是**类型的副本**，也就是实例化的变量本身，也可以是**指针**，也称为指针接收者。

关于两种接收者，唯一的不同是，方法可以通过传入的指针接收者**改变接收者本身的值**。

另外需要注意的是，如果是指针接收者，那么在调用这个方法时，**既可以传入其副本，也可以传入指针**；对于值为接收者的方法也一样。这是因为一种称为**指针重定向**的机制。

```go
/*
定义略...
*/

// 指针接收者
var v Vertex
// Go 编译器会自动将其解释为 (&v).Scale(5)
v.Scale(5)  // OK
p := &v
p.Scale(10) // OK

// 值接收者
fmt.Println(v.Abs()) // OK
// Go 编译器会自动将其解释为 (*p).Abs()
fmt.Println(p.Abs()) // OK
```

据上所述，在为类型定义方法时，通常**更为推荐传入值的指针**，来避免进行过多的复制操作；当然，如果为了避免在方法中对值本身的误修改，也可传入值的副本。

### 接口

首先，接口是一种类型，它是由一组函数签名定义的集合。使用 `type MyInterface interface {...}` 语句块来定义。

所以接口也可任意的作为参数等来**进行传递**。

需要注意的是，为类型实现接口的方式与为函数传入接收者是十分类似的 (上面也提到了)，而唯一的不同是，接口实现对于传入的接收者要求严格，**只传入了指针接收者，那么只有变量的指针才可以使用已实现了的接口方法**。

由于接口要求接口实现时要**实现所有方法**，那么在某种类型实现了接口定义的**所有方法**后，这种类型即**隐式实现**了接口，**无需显式说明**。

通过 `t := i.(T)` 来查看接口底层值的类型的方式，称为**类型断言**。通常用于 switch 语句块中对不同类型进行分支处理。

**Go 中有许多接口都需要深入学习**，比如 sort.Sort 包中的 data 接口，如果需要对一种自定义的类型调用 sort.Sort() 方法，均需要实现 data 接口的方法，如：

```go
package main

import (
	"fmt"
	"sort"
)

type T []int

func (t T) Len() int           { return len(t) }
func (t T) Swap(i, j int)      { t[i], t[j] = t[j], t[i] }
func (t T) Less(i, j int) bool { return t[i] < t[j] }

func main() {
	myArr := T{1, 7, 3, 4}
	sort.Sort(myArr)
	fmt.Println(myArr)
}

```

