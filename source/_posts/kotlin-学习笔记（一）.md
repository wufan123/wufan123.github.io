---
title: kotlin-学习笔记（一）
date: 2017-06-08 10:15
categories: 
    - [Android]
tags: [kotlin]
---

## 什么是kotlin
一门现代多平台应用的静态编程语言
## 相对于java的优点
总的来说就是语法上比较简洁，更易书写和阅读，对空指针这类的错误异常更易把握，并很好兼容java
## 基础语法
列出几个较java不同的地方
- **包定义**，不要求包名符合文件路径，每个kotlin源文件都会默认引入kotlin一些相关包，然后根据平台的不同，引入一些平台包
- **字符串模板**，字符串中可以包含模板表达式，列如：
``` kotlin
val s = "abc"
val str = "$s.length is ${s.length}" // evaluates to "abc.length is 3"
```
- **空值检查**，可以通过？.来安全使用可能为空的对象。列如：
``` kotlin
val l = b?.length ?: -1
```

<!-- more -->

- **类型转换检查**
``` kotlin
val aInt: Int? = a as? Int//if a  is not Int，a is null
```
- **类型的自动转换**
``` kotlin
if (obj is String) {
        // `obj` is automatically cast to `String` in this branch
        return obj.length
    }
```
- **for循环和while循环**
``` kotlin
for (item in items) {
    println(item)
}
for (index in items.indices) {
    println("item at $index is ${items[index]}")
}
while (index < items.size) {
    println("item at $index is ${items[index]}")
    index++
}
```
- **when**表达式，类似java中switch
``` kotlin
when (obj) {
    1          -> "One"
    "Hello"    -> "Greeting"
    is Long    -> "Long"
    !is String -> "Not a string"
    else       -> "Unknown"
}
``` 
- **ranges**表达式，检查某值是否在某个范围内
``` kotlin
val x = 10
val y = 9
if (x in 1..y+1) {
    println("fits in range")
}
```
- **collections**的使用
``` kotlin
when {//hecking if a collection contains an object using in operator
    "orange" in items -> println("juicy")
    "apple" in items -> println("apple is fine too")
}
fruits//Using lambda
.filter { it.startsWith("a") }
.sortedBy { it }
.map { it.toUpperCase() }
.forEach { println(it) }
```



