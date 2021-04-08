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

### 类型判断
核心：Object.prototype.toString
``` bash
const isType = (type) => (obj) => Object.prototype.toString.call(obj) === `[object ${type}]`
```

### call、apply、bind
改变this指向

``` bash
Function.prototype.myCall = function() {
  let [thisArg, ...args] = [...arguments]
  thisArg = thisArg === 'null' ||
    thisArg === 'undefined'
      ? window
      : Object(thisArg)
  const fn = Symbol()
  thisArg[fn] = this
  const result = thisArg[fn](...args)
  delete thisArg[fn]
  return result
}

Function.prototype.myApply = function() {
  let [thisArg, args] = [...arguments]
  thisArg = thisArg === 'null' ||
    thisArg === 'undefined'
      ? window
      : Object(thisArg)
  const fn = Symbol()
  thisArg[fn] = this
  const result = thisArg[fn](...args)
  delete thisArg[fn]
  return result
}

Function.prototype.mybind = function(context, ...args) {
  return (...newArgs) => {
    return this.call(context, ...args, ...newArgs)
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
  return ret instanceof Object ? ret : obj
  # const o = Object.create(func.prototype); // 创建对象
  # const k = func.call(o); // 改变this指向，把结果付给k
  # if (k && k instanceof Object) { // 判断k的类型是不是对象
  #   return k; // 是，返回k
  # } else {
  #   return o; // 不是返回返回构造函数的执行结果
  # }
}
```

### 实现instanceof
``` bash
fuction myInstanceof (left, right) {
  left = left.__proto__
  const prototype = right.prototype
  while(true) {
    if (left === null) {
      return false
    }
    if (left === prototype) {
      return true
    }
    left = left.__proto__
  }
}

```

### leetcode
- [LRU 缓存](https://leetcode-cn.com/problems/lru-cache/)
``` bash
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity
    this.capacityMap = new Map()
  }

  get(key) {
    const isExist = this.capacityMap.has(key)
    if (!isExist) return -1
    const value = this.capacityMap.get(key)
    this.capacityMap.delete(key)
    this.capacityMap.set(key, value)
    return value
  }

  put(key, value) {
    if (this.capacityMap.has(key)) this.capacityMap.delete(key)
    this.capacityMap.set(key, value)
    if (this.capacityMap.size > this.capacity) {
      this.capacityMap.delete(this.capacityMap.keys().next().value)
    }
  }
}
```

- [反转链表](https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof/)
``` bash
const reverseList = function(head) {
  let p1 = head
  let p2 = null
  while(p1) {
    const { next, val } = p1
    p2 = { val, next: p2 }
    p1 = next
  }
  return p2
}
```

- [字典树](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)
``` bash
class Trie {
  constructor() {
    this.tries = {}
  }
  insert(word) {
    const res = [...word].reduce((tries, cur) => {
      return tries[cur] || (tries[cur] = {})
    }, this.tries)
    res.isExist = true
  }
  search(word) {
    let tries = this.tries
    for (let w of word) {
      tries = tries[w]
      if (!tries) {
        return false
      }
    }
    return !!tries.isExist
  }
  startsWith(prefix) {
    let tries = this.tries
    for (let w of prefix) {
      tries = tries[w]
      if (!tries) {
        return false
      }
    }
    return true
  }
}
```

- [给定数值的最小字符串](https://leetcode-cn.com/problems/smallest-string-with-a-given-numeric-value/)
``` bash
const getSmallestString = function(n, k) {
  let str = ''
  for (let i = n; i > 0; i--) {
    const temp = k - 26 * (i - 1)
    if (temp > 0) {
      str += String.fromCharCode(96 + temp) // 获取'a-z'
      k -= temp
    } else {
      str += 'a'
      k -= 1
    }
  }
  return str
}
```

- [反转二叉树](https://leetcode-cn.com/problems/invert-binary-tree/)
``` bash
const invertTree = function(root) {
  if (!root) return root
  const { left, right } = root
  const temp = left
  root.left = right
  root.right = temp
  invertTree(root.left)
  invertTree(root.right)
  return root
}
```

- [斐波那契额数列](https://leetcode-cn.com/problems/fibonacci-number/)
``` bash
const fib = function(n) {
  if (n < 2) return n
  let sum = 1
  let prev = 0
  let next = 0
  for (let i = 2; i <= n; i++) {
    prev = next
    next = sum
    sum = prev + next
  }
  return sum
}
```

- [数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)
``` bash
const findKthLargest = function(nums, k) {
  return nums.sort((a, b) => b - a)[k - 1]
}
```

- [字符的最短距离](https://leetcode-cn.com/problems/shortest-distance-to-a-character/)
``` bash
const shortestToChar = function(s, c) {
  const answer = [];
  const cArr = [];
  [...s].forEach((item, index) => {
    item === c && cArr.push(index)
  });
  [...s].forEach((_, index) => {
    let shortL = null;
    cArr.forEach(item => {
      const l = Math.abs(item - index);
      shortL = shortL === null || shortL > l
        ? l
        : shortL
    });
    answer.push(shortL);
  });
  return answer
};
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

### setState是同步还是异步
setState 只在合成事件和钩子函数中是“异步”的，在原生事件和 setTimeout 中都是同步的。
