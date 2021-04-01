---
title: study
date: 2020-08-19 11:11:44
tags:
---

### 实现一个可以拖拽的DIV
<!-- more -->
js 版
``` bash
<div id="xxx"></div>

var dragging = false
var position = null

xxx.addEventListener('mousedown',function(e){
  dragging = true
  position = [e.clientX, e.clientY]
})

document.addEventListener('mousemove', function(e){
  if(dragging === false) return null
  const x = e.clientX
  const y = e.clientY
  const deltaX = x - position[0]
  const deltaY = y - position[1]
  const left = parseInt(xxx.style.left || 0)
  const top = parseInt(xxx.style.top || 0)
  xxx.style.left = left + deltaX + 'px'
  xxx.style.top = top + deltaY + 'px'
  position = [x, y]
})
document.addEventListener('mouseup', function(e){
  dragging = false
})
```
react ts版
``` bash
<div
  ref="container"
  className="container"
  onMouseDown={e => this.handleMouseDown(e)}
>
</div>

this.state = {
  dragging: false,
  position: [],
}
handleMouseDown(e: React.MouseEvent) {
  const position = [e.clientX, e.clientY]
  console.log('%c position =>', 'color:blue;font-size:18px', position)
  this.setState({
    dragging: true,
    position,
  })
  document.onmousemove = (e: MouseEvent) => this.handleMouseMove(e)
  document.onmouseup = () => this.handleMouseUp()
}
handleMouseMove(e: MouseEvent) {
  if (!this.state.dragging) return
  let position = this.state.position
  const x = e.clientX
  const y = e.clientY
  const deltaX = x - position[0]
  const deltaY = y - position[1]
  const container = (this.refs.container as HTMLElement)
  const left = parseInt(container.style.left || '0')
  const top = parseInt(container.style.top || '0')
  container.style.position = 'relative'
  container.style.left = left + deltaX + 'px'
  container.style.top = top + deltaY + 'px'
  position = [x, y]
  this.setState({
    position,
  })
}
handleMouseUp() {
  document.onmousemove = null
  document.onmouseup = null
  this.setState({
    dragging: false,
  })
}
```

### 防抖
``` bash
function debounce (fn, delay) {
  let timer = null
  return function(...args) {
    timer && clearTimeout(timer)
    const _this = this
    timer = setTimeout(function() {
      fn.apply(_this, args)
    }, delay)
  }
}
```

### 节流
``` bash
function throttle (fn, delay) {
  let flag = true
  let timer = null
  return function(...args) {
    if (!flag) return
    flag = false
    const _this = this
    timer && clearTimeout(timer)
    timer = setTimeout(function() {
      fn.apply(_this, args)
      flag = true
    }, delay)
  }
}
```

### 去重
``` bash
let unique_1 = arr => [...new Set(arr)];

function unique_2(array) {
  var res = array.filter(function (item, index, array) {
    return array.indexOf(item) === index;
  })
  return res;
}

let unique_3 = arr =>
  arr.reduce((pre, cur) =>
    pre.includes(cur)
      ? pre
      : [...pre, cur]
  , []);

function unique_4(array) {
  var obj = {};
  return array.filter(function (item, index, array) {
    return obj.hasOwnProperty(typeof item + item)
      ? false
      : (obj[typeof item + item] = true)
  })
}

```

### 柯里化函数
``` bash
const currying = (fn, ...args) =>
  fn.length > args.length
    ? (...arguments) => currying(fn, ...args, ...arguments)
    : fn(...args)
```

### 数组flat
``` bash
const flatDeep = (arr) => {
  return arr.reduce((res, cur) => {
    if (Array.isArray(cur)) {
      return [...res, ...cur]
    } else {
      return [...res, cur]
    }
  }, [])
}

function flatDeep2 (arr, d = 1) {
  return d > 0
    ? arr.reduce(
      (acc, val) => acc.concat(
        Array.isArray(val)
          ? flatDeep2(val, d - 1)
          : val
      ), [])
    : arr.slice();
};
```

### 深拷贝
``` bash
function forEach (array, iteratee) {
  let index = -1;
  const length = array.length;
  while (++index < length) {
    iteratee(array[index], index);
  }
  return array;
}

function deepClone (obj, map = new WeakMap()) {
  if (obj instanceof RegExp) return new RegExp(obj)
  if (obj instanceof Date) return new Date(obj)
  if (obj === null || typeof obj !== 'object') return obj
  if (map.has(obj)) return map.get(obj)
  const target = new obj.constructor()
  map.set(obj, target)
  const isArray = Array.isArray(obj)
  const keys = isArray ? undefined : Object.keys(obj)
  forEach(keys || obj, (value, key) => {
    if (keys) {
      key = value;
    }
    target[key] = deepClone(obj[key], map);
  })
  return target
}
```

### 实现一个对象类型的函数
核心：Object.prototype.toString
``` bash
const isType = (type) => (obj) => Object.prototype.toString.call(obj) === `[object ${type}]`
```

### 手写call、apply、bind
改变this指向

``` bash
Function.prototype.myCall = function() {
  let [thisArg, ...args] = [...arguments]
  thisArg = Object(thisArg) || window
  const fn = Symbol()
  thisArg[fn] = this
  const result = thisArg[fn](...args)
  delete thisArg[fn]
  return result
}

Function.prototype.myApply = function() {
  let [thisArg, args] = [...arguments]
  thisArg = Object(thisArg) || window
  const fn = Symbol()
  thisArg[fn] = this
  const result = thisArg[fn](...args)
  delete thisArg[fn]
  return result
}

Function.prototype.mybind = function(context, ...args) {
  return (...newArgs) => {
    return this.call(context,...args, ...newArgs)
  }
}
```

### 实现new操作
``` bash
const new = function () {
  const obj = Object.create(null) // 创建对象
  const Constructor = [].shift.call(arguments) // 取出构造函数
  obj.__proto__ = Constructor.prototype // 将对象的__proto__指向 构造函数的prototype
  const ret = Constructor.apply(obj, arguments) // 改变this指向
  return typeof ret === 'object' ? ret : obj
  # const o = Object.create(func.prototype); // 创建对象
  # const k = func.call(o); // 改变this指向，把结果付给k
  # if (k && k instanceof Object) { // 判断k的类型是不是对象
  #   return k; // 是，返回k
  # } else {
  #   return o; // 不是返回返回构造函数的执行结果
  # }
}
```



### 加减乘除
``` bash
// 获取小数个数
function getDecimalsLength (val) {
  let curVal
  try {
    curVal = val.toString().split('.')[1].length
  } catch (f) {
    curVal = 0
  }
  return curVal
}

// 浮点数相加
export function MathAdd (a, b) {
  if (!a && !b) return '0'
  let e
  let c = getDecimalsLength(a)
  let d = getDecimalsLength(b)
  return (
    (e = Math.pow(10, Math.max(c, d))),
    (MathMul(a, e) + MathMul(b, e)) / e
  )
}

// 浮点数相减
export function MathSub (a, b) {
  if (!a && !b) return '0'
  let e
  let c = getDecimalsLength(a)
  let d = getDecimalsLength(b)
  return (
    (e = Math.pow(10, Math.max(c, d))),
    (MathMul(a, e) - MathMul(b, e)) / e
  )
}

// 浮点数相乘
export function MathMul (a, b) {
  if (!a || !b) return '0'
  let c = 0
  let d = a.toString()
  let e = b.toString()
  c += getDecimalsLength(d)
  c += getDecimalsLength(e)
  return (Number(d.replace('.', '')) * Number(e.replace('.', ''))) / Math.pow(10, c)
}

// 浮点数相除
export function MathDiv (a, b) {
  if (!a || !b) return '0'
  let c
  let d
  let e = getDecimalsLength(a)
  let f = getDecimalsLength(b)
  return (
    (c = Number(a.toString().replace('.', ''))),
    (d = Number(b.toString().replace('.', ''))),
    MathMul(c / d, Math.pow(10, f - e))
  )
}
```