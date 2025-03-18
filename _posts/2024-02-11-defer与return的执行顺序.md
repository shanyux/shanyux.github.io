---
layout:     post
title:      defer与return的执行顺序
date:       2024-02-11
author:     KevinShan
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Golang
---

defer 的作用就是把defer关键字之后的函数执行压入一个栈中延迟执行，多个 `defer` 的执行顺序是后进先出LIFO，也就是先执行最后一个defer，最后执行第一个defer。

在此之前，先理解一下return返回值的运行机制：return并非原子操作，共分为**赋值**、**返回值**两步操作。

defer、return、返回值三者的执行是：return最先执行，先将结果写入返回值中（即赋值）；接着defer开始执行一些收尾工作；最后函数携带当前返回值退出（即返回值）。

所以结论是：第一步先return赋值，第二步再执行defer，第三步执行return返回。

但是在有名与无名的函数返回值的情况下会有些区别：

## 不带命名返回值

如果函数的返回值是无名的（不带命名返回值），则go语言会在执行return的时候会执行一个类似创建一个临时变量作为保存return值的动作。

我们看一个demo：

```go
package main
import "fmt"
func main() {
  t := test()
  fmt.Println(t)
}

func test() int { //无名返回
  i := 9
  defer func() {
    i++
    fmt.Println("defer1=", i)
  }()
  defer func() {
    i++
    fmt.Println("defer2=", i)
  }()
  return i
}
```

结果如下：

```
 defer2= 10 
 defer1= 11 
 9
```

分析一下：

如上例子，实际上一共执行了3步操作，

1）赋值，因为返回值没有命名，所以return 默认指定了一个返回值（假设为s），首先将i赋值给s，i初始值是9，所以s也是9

2）后续的defer操作因为是针对i, 进行的，所以不会影响s, 此后因为s不会更新，所以s不会变还是9

3）返回值，return s，也就是return 9  
相当于：  

```
var i int  
s := i  
return s
```

## 有名返回值

有名返回值的函数，由于返回值在函数定义的时候已经将该变量进行定义，在执行return的时候会先执行返回值保存操作，而后续的defer函数会改变这个返回值(虽然defer是在return之后执行的，但是由于使用的函数定义的变量，所以执行defer操作后对该变量的修改会影响到return的值。

来看demo：

```go
package main
import "fmt"
func main() {
  t := test()
  fmt.Println(t)
}
func test() (i int) { //有名返回i
  i = 9
  defer func() {
    i++
    fmt.Println("defer1=", i)
  }()
  defer func() {
    i++
    fmt.Println("defer2=", i)
  }()
  return i
}
```

结果如下：

```
defer2= 10 
defer1= 11 
11
```

分析一下：

s 就相当于命名的变量i，因为所有的操作都是基于命名变量i(s)，返回值也是i，所以每一次defer操作，都会更新返回值，执行完defer后，会返回最终i的值。

1）赋值，因为返回值有命名，return 指定了一个返回值i

2）后续的defer操作因为是针对i, 进行的，所以每一次defer操作，都会更新返回值i

3）返回值，return i，也就是return 11

引用
[Golang的defer与return的执行顺序本文使用Go版本是1.14 之前写过一篇文章Golang的defer预计 - 掘金](https://juejin.cn/post/7095631673865273352)
  
