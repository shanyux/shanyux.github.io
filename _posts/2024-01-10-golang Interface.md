---
layout:     post
title:      golang []interface{} 遇到的问题
date:       2024-01-10
author:     KevinShan
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Golang
---

`interface{}` 是一个空的 interface 类型，根据前文的定义：一个类型如果实现了一个 interface 的所有方法就说该类型实现了这个 interface，空的 interface 没有方法，所以可以认为所有的类型都实现了 `interface{}` 。如果定义一个函数参数是 `interface{}` 类型，这个函数应该可以接受任何类型作为它的参数。

```go
func doSomething(v interface{}){      
}
```

如果函数的参数 v 可以接受任何类型，那么函数被调用时在函数内部 v 是不是表示的是任何类型？并不是，虽然函数的参数可以接受任何类型，并不表示 v 就是任何类型，在函数 doSomething 内部 v 仅仅是一个 interface 类型，之所以函数可以接受任何类型是在 go 执行时传递到函数的任何类型都被自动转换成 `interface{}`

但是 `interface{}` 类型的 slice 不可以就可以接受任何类型的 slice

```go
func printAll(vals []interface{}) { //1  
	for _, val := range vals {  
		fmt.Println(val)  
	}  
}  
  
func main(){  
	names := []string{"stanley", "david", "oscar"}  
	printAll(names)  
}
```

上面的代码是按照我们的假设修改的，执行之后竟然会报 `cannot use names (type []string) as type []interface {} in argument to printAll` 错误。go 不会对 类型是 `interface{}` 的 slice 进行转换。因为 `interface{}` 会占用两个字长的存储空间，一个是自身的 methods 数据，一个是指向其存储值的指针，也就是 interface 变量存储的值，因而 slice []interface{} 其长度是固定的 `N*2` ，但是 []T 的长度是 `N*sizeof(T)` ，两种 slice 实际存储值的大小是有区别的。

一个指针类型可以通过其相关的值类型来访问值类型的方法，但是反过来不行。即，一个 `*Dog` 类型的值可以使用定义在 `Dog` 类型上的 `Speak()` 方法，而 `Cat` 类型的值不能访问定义在 `*Cat` 类型上的方法。。Go 中的所有东西都是按值传递的。每次调用函数时，传入的数据都会被复制。对于具有值接收者的方法，在调用该方法时将复制该值。如果是按 pointer 调用，go 会自动进行转换，因为有了指针总是能得到指针指向的值是什么，如果是 value 调用，go 将无从得知 value 的原始值是什么，因为 value 是份拷贝。**go 会把指针进行隐式转换得到 value，但反过来则不行**。
##### 
引用
1. [理解 Go interface 的 5 个关键点 | 三月沙](https://sanyuesha.com/2017/07/22/how-to-understand-go-interface/)
2. [理解Golang中的interface和interface{} - maji233 - 博客园](https://www.cnblogs.com/maji233/p/11178413.html)
