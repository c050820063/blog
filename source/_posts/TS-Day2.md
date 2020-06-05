---
title: TS Day2
date: 2020-05-26 22:16:21
tags:
---

## 函数的定义
- 固定参数
``` bash
  function add(a:number, b:number): number {
    return a + b
  }
```
<!-- more -->
- 固定某些参数 ?
``` bash
  function add(a:number, b:number, c:?number): number {
    let sum:number = a + b
    if (c) sum += c
    return sum
  }
```
- 不定参数
``` bash
  function add(...arg:number[]): number {
    let sum:number
    for (let i = 0; i < arg.length; i++) {
      sun += arg[i]
    }
    return sum
  }
```

## 引用类型(数组、字符串)
- 数组
``` bash
let arr1:number[] = [1, 2]
let arr2:Array<string> = ['j', 's']
let arr3:Array<boolean> = new Array(true, false)
// 元组
let arr4:[number, string] = [1, 'js']
```

- 字符串
1. 构造函数
``` bash
let s1:String = new String('s')
```
2. 字面量
``` bash
let s2:string = 's'
```
3. replace indexOf lastIndexOf
``` bash
let s3:string = 'hello world'
s3.replace('world', 're') // 'hello re'
s3.indexOf('world') // 6
s3.lastIndexOf('o') // 7
```

## 日期类型

``` bash
let d:Date = new Date()
```

## 正则
1. 声明
``` bash
let reg1:RegExp = new Reg('hello', 'gi') // 构造函数声明
let reg2:RegExp = /world/gi
```
2. test、exec方法
``` bash
reg1.test('hello world') // true
reg2.exec('hello world') // ["hello", index: 0, input: "hello world", groups: undefined]
```