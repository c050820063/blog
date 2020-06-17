---
title: 'TS Day1'
date: 2020-05-25 22:46:57
tags:
---

## 安装

1. npm i -g typescript
2. npm init -y
3. tsc --init
4. npm i -D @types/node
<!-- more -->
## 基础类型定义

### string
``` bash
const name:string = 'joke'
```

### number
``` bash
const age:number = 16
const stature:number = 178.5
const nan:number = NaN
```

### boolean
``` bash
const b:boolean = true
```

### enum 枚举
- 人: 男人、女人、中性
``` bash
  enum REN{nan, nv, yao}
  console.log(REN.yao) // 2

  enum REN{nan='男人', nv='女人', yao='妖'}
  console.log(REN.yao) // 妖
```

### any
``` bash
  let t:any = 0
  t = 'any'
  t = true
```

### null undefined


