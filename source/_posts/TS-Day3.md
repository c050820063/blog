---
title: TS Day3
date: 2020-05-27 23:04:05
tags:
---

## 类
1. 声明与创建
``` bash
class Person {
  public sex: string
  protected name: string // 受保护属性，只能在类“Person”及其子类中访问
  private age: number // 私有属性，只能在类“Person”中访问
  public constructor(name: string, sex: string, age: number) {
    this.name = name
    this.sex = sex
    this.age = age
  }
  public sayHello() {
    console.log('hello')
  }
}

const person: Person = new Person('joke', '男', 18)
console.log(person) // { name: 'joke', sex: '男', age: 18 }
person.sayHello() // hello
```

2. 继承与重写
``` bash
class Nan extends Person {
  public height: number

  public constructor(name: string, sex: string, age: number, height: number) {
    super(name, sex, age)
    this.height = height
  }

  public interest() {
    console.log('看书')
  }

  public sayHello() {
    super.sayHello()
    console.log('world')
  }
}

const nan: Nan = new Nan('jc', '男', 22, 168)
console.log(nan) // { name: 'jc', sex: '男', age: 22, height: 168 }
nan.interest() // 看书
nan.sayHello() //hello world
```

## 接口
``` bash
interface Cat {
  name?: string,
  pinzhong: string,
  cry(): void
}

const cat: Cat = {
  name: 'nangua',
  pinzhong: 'gouzi',
  cry () {
    console.log('en')
  }
}
```