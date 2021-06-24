---
title: Vue3 study
date: 2020-06-04 14:25:16
tags:
---

## 组件 API（Composition API）

组件 API 旨在通过将组件属性中当前可用的机制公开为 JavaScript 函数来解决这个问题。 Vue 核心团队将组件 API 描述为 “一组基于函数的附加 API，可以灵活地组合组件逻辑。” 用组件 API 编写的代码更具有可读性，并且其背后没有任何魔力，因此更易于阅读和学习。
<!-- more -->
让我们通过一个用了新的组件 API 的组件的简单示例，来了解其工作原理。

``` js
import { reactive, watch, toRefs, computed, watchEffect } from 'vue'
export default {
  setup () {
    const state = reactive({
      count: 0,
      doubleCount: computed(() => {
        return state.count * 2
      }),
      a: 1,
      watchCount: 0,
      watchCount1: 1
    })
    watchEffect(() => {
      console.log('watchEffect', state.count, state.a)
    }, {
      onTrack() {
        console.log('onTrack调用')   // 当反应性属性或ref作为依赖项被跟踪时
      },
      onTrigger() {
        console.log('ontrigger调用') // 当观察程序回调由依赖项的变异触发时
      }
    })
    watch(() => {
      return [state.watchCount, state.watchCount1]
    }, (val, prev) => {
      console.log(val, prev)
    })
    setTimeout(() => {
      state.watchCount++
    }, 1000)
    const addRef = () => {
      state.count++
    }
    return {
      // 将代理对象转换为纯对象。并对其每个key做包装，包装为ref
      ...toRefs(state),
      addRef,
    }
  }
}
```

### setup

setup() 函数是 Vue3 中，专门为组件提供的新属性。它为我们使用 vue3 的 Composition API 新特性提供了统一的入口。

- 执行时机

setup 函数会在 beforeCreate 之后、created 之前执行

- 接收 props 数据

1. 在 props 中定义当前组件允许外界传递过来的参数名称：
``` js
props: {
  p1: String
}
```
2. 通过 setup 函数的第一个形参，接收 props 数据：

``` js
setup(props) {
  console.log(props.p1)
}
```

- context
setup 函数的第二个形参是一个上下文对象，这个上下文对象中包含了一些有用的属性，这些属性在 vue 2.x 中需要通过 this 才能访问到，在 vue 3.x 中，它们的访问方式如下：

``` js
const MyComponent = {
  setup(props, context) {
    context.attrs
    context.slots
    context.parent
    context.root
    context.emit
    context.refs
  }
}
```
注意：在 setup() 函数中无法访问到 this

### reactive
reactive() 函数接收一个普通对象，返回一个响应式的数据对象。

- 基本语法
等价于 vue 2.x 中的 Vue.observable() 函数，vue 3.x 中提供了 reactive() 函数，用来创建响应式的数据对象，基本代码示例如下：

``` js
import { reactive } from 'vue'
 
// 创建响应式数据对象，得到的 state 类似于 vue 2.x 中 data() 返回的响应式对象
const state = reactive({ count: 0 })
```

- 定义响应式数据供 template 使用
1. 按需导入 reactive 函数：
``` js
import { reactive } from 'vue'
```

2. 在 setup() 函数中调用 reactive() 函数，创建响应式数据对象：
``` js
setup() {
     // 创建响应式数据对象
 const state = reactive({count: 0})
 
     // setup 函数中将响应式数据对象 return 出去，供 template 使用
 return state
}
```
3. 在 template 中访问响应式数据：

``` js
<p>当前的 count 值为：{{count}}</p>
```

### ref

- 基本语法
ref() 函数用来根据给定的值创建一个响应式的数据对象，ref() 函数调用的返回值是一个对象，这个对象上只包含一个 .value 属性：
``` js
import { ref } from 'vue'
 
// 创建响应式数据对象 count，初始值为 0
const count = ref(0)
 
// 如果要访问 ref() 创建出来的响应式数据对象的值，必须通过 .value 属性才可以
console.log(count.value) // 输出 0
// 让 count 的值 +1
count.value++
// 再次打印 count 的值
console.log(count.value) // 输出 1
```

- 在 template 中访问 ref 创建的响应式数据

1. 在 setup() 中创建响应式数据：
``` js
import { ref } from 'vue'
 
setup() {
const count = ref(0)
  return {
    count,
    name: ref('zs')
  }
}
```

2. 在 template 中访问响应式数据：
``` js
<template>
 <p>{{count}} --- {{name}}</p>
</template>
```

3. 在 reactive 对象中访问 ref 创建的响应式数据
当把 ref() 创建出来的响应式数据对象，挂载到 reactive() 上时，会自动把响应式数据对象展开为原始的值，不需通过 .value 就可以直接被访问，例如：
``` js
const count = ref(0)
const state = reactive({
  count
})
 
console.log(state.count) // 输出 0
state.count++            // 此处不需要通过 .value 就能直接访问原始值
console.log(count)       // 输出 1
```

- 注意：新的 ref 会覆盖旧的 ref，示例代码如下：
``` js
// 创建 ref 并挂载到 reactive 中
const c1 = ref(0)
const state = reactive({
  c1
})
 
// 再次创建 ref，命名为 c2
const c2 = ref(9)
// 将 旧 ref c1 替换为 新 ref c2
state.c1 = c2
state.c1++
 
console.log(state.c1) // 输出 10
console.log(c2.value) // 输出 10
console.log(c1.value) // 输出 0
```

### isRef
isRef() 用来判断某个值是否为 ref() 创建出来的对象；应用场景：当需要展开某个可能为 ref() 创建出来的值的时候，例如：
``` js
import { isRef } from 'vue'
 
const unwrapped = isRef(foo) ? foo.value : foo
```

### toRefs
toRefs() 函数可以将 reactive() 创建出来的响应式对象，转换为普通的对象，只不过，这个对象上的每个属性节点，都是 ref() 类型的响应式数据，最常见的应用场景如下：
``` js

import { toRefs } from 'vue'
 
setup() {
    // 定义响应式数据对象
    const state = reactive({
      count: 0
    })
    
    // 定义页面上可用的事件处理函数
    const increment = () => {
      state.count++
    }
    
    // 在 setup 中返回一个对象供页面使用
    // 这个对象中可以包含响应式的数据，也可以包含事件处理函数
    return {
      // 将 state 上的每个属性，都转化为 ref 形式的响应式数据
      ...toRefs(state),
      // 自增的事件处理函数
      increment
    }
}
```

页面上可以直接访问 setup() 中 return 出来的响应式数据：

``` js
<template>
  <div>
    <p>当前的count值为：{{count}}</p>
    <button @click="increment">+1</button>
  </div>
</template>
```

### computed
computed()函数用来创建计算属性，返回的是个ref实例

- 只读属性
``` js
template>
  <div class="wrapper">
      <p>count:{{refCount}}</p>
      <p>计算属性：{{computedCount}}</p>
      <button @click="refCount+=1">+1</button>
  </div>
</template>

<script>
import { ref,computed } from "vue";
export default {
  setup(){
      const refCount = ref(0)

      const computedCount = computed(()=> refCount.value+1)

  		//   computedCount=2
      return {
          refCount,
          computedCount
      }
  },
}
</script>
```

- 可读可写
``` js
<template>
  <div class="wrapper">
    <p>count:{{refCount}}</p>
    <p>计算属性：{{computedCount}}</p>
    <button @click="refCount+=1">+1</button>
  </div>
</template>

<script>
import { ref, computed } from "vue";
export default {
  setup() {
    const refCount = ref(0);

    //   const computedCount = computed(()=> refCount.value+1)

    //   computedCount=2

    const computedCount = computed({
      get: () => refCount.value + 1,
      set: a => {
        refCount.value = a + 112;
      }
    });
    computedCount.value = 33;
    return {
      refCount,
      computedCount
    };
  },
};
</script>
```

### watch

监视数据变化，做响应处理。组件创建，默认会执行一次，可配置项

- 基本用法

``` js
import { watch,ref, set } from "vue";
export default {
  setup(){
    const refCount = ref(0)

    watch(()=>console.log(refCount.value))

    setTimeout(()=>{
      refCount.value += 2;
    },1000)
  },
}
```

- 监听指定数据源
ref
``` js
const count = ref(2)
watch(count,(count,oldCount)=>{
  console.log(count,oldCount)
})
setTimeout(() => {
  count.value += 2;
}, 1000);
// 2 undefined
// 4 2
```
reactive
``` js
setup() {
  const state = reactive({ count: 0 });
  watch(() => state.count, (count, oldCount) => console.log(count, oldCount));
  setTimeout(() => {
    state.count += 2;
  }, 1000);
},
// 0 undefined
// 2 0
----------------------------------------
watch(
  () => state.count,
  (count, oldCount) =>{ console.log(count, oldCount)},
  {lazy:true,} //组件第一次创建不调用
);
```

- 监视多个数据源
ref
``` js
const count = ref(0)
const name = ref('yp1')

watch(
  [count,name],
  ([count,name],[oldCount,oldName])=>{
    console.log(count)
    console.log(name)
    console.log('------------')
    console.log(oldCount)
    console.log(oldName)
  },
  {
    lazy:true
  }
)
setTimeout(()=>{
  count.value++
  name.value = "yp2"
})
// 1
// yp2
// ---------------------
// 0
// yp1
```

reactive
``` js
const state = reactive({count:0,name:'yp1'})

watch(
  [()=>state.count,()=>state.name],
  ([count,name],[oldCount,oldName])=>{
    console.log(count)
    console.log(name)
    console.log("---------------------")
    console.log(oldCount)
    console.log(oldName)
  },
  {
    lazy: true
  }
)
setTimeout(()=>{
  state.count++
  state.name = "yp2"
})
// 1
// yp2
// ---------------------
// 0
// yp1
```

- 取消监视
在setup()函数内创建的watch监视，会在当前组件被销毁的时候自动停止。watch的返回值调用
两秒内点击按钮，取消了watch监听，可查 console

``` js
<template>
  <div class="wrapper">
    <div>{{count}}</div>
    <button @click="stopwatch">stop</button>
    
  </div>
</template>

<script>
import { watch, ref, reactive } from "vue";
export default {
  setup() {
    const count = ref(0)

    const stop = watch(()=>{
      console.log("监听到watch的变化"),
      console.log(count.value)
    })

    setTimeout(() => {
        count.value += 2
    }, 2000);

    const stopwatch = ()=>{
      stop()
    }

    return {
      count,
      stopwatch
    }
  },
};
</script>
```
- watch 清除无效的异步
当watch监视的值发生变化的时候，或者watch本身被stop()之后，我们希望能够清除那些无效的异步任务，此时，watch回调中提供一个cleanup registrator function来执行清除的工作，

eg watch 被重复执行了 或者被强制stop

实际场景一：打开a.html 会执行a.html的页面的3个ajax请求；但是在有两个ajax请求还没有发出去的时候，用户打开b.html，那剩下的两个请求我就不需要再接着请求了

实际场景二：输入框按键输值请求

``` js
<template>
  <div>
    <input type="text" v-model="kw">
  </div>
</template>

<script>
import { ref, watch } from "@vue/composition-api";
export default {
  name: "",
  setup() {
    const kw = ref("");

    const asyncprint = val => {
      return setTimeout(() => {
        console.log(val);
      }, 1000);
    };

    watch(
      kw,
      (kw, oldkw, callback) => {
        const timeId = asyncprint(kw);
        
        callback(()=>clearTimeout(timeId));
      },
      { lazy: true }
    );

    return {
      kw
    };
  },
};
</script>
```

## LifeCycle Hooks

vue2生命周期 | vue3生命周期
-|-
beforeCreate | setup |
created | setup |
beforeMount | onBeforeMount |
mounted | onMounted |
beforeUpdate | onBeforeUpdate |
updated | onUpdated |
beforeDestory | onBeforeDestory |
destroyed | onUnmounted |
errorCaptured | onErrorCaptured |

- 建议异步请求，在onMounted()/mounted()

``` js
import { onBeforeMount,onMounted, } from "@vue/composition-api";

setup(){

    onBeforeMount(()=>{
        console.log("onBeforeMount")
    });
    onMounted(()=>{
        console.log("onMounted")
    })
    
}
```

## provide & inject
- 共享普通数据
祖组件
``` js
import { provide } from '@vue/composition-api'

setup(){
  provide('themeColor','red')  // 键值
},
```

孙组件
``` js
import { inject } from '@vue/composition-api'

setup(){
  const color = inject('themeColor');

  return {
      color: color,
  }
},
```

## 用组件 API 进行代码重用

新的组件 API 具有更多优点。考虑一下代码重用。目前如果我们要在其他组件之间共享一些代码，则有两个可用的选择：mixins 和作用域插槽（ scoped slots）。但是两者都有缺点。

假设我们要提取 counter 中的功能并在其他组件中重用。在下面，你可以看到如何将其与可用的 API 和新的组件 API 结合使用：

- mixins

``` js
import CounterMixin from './mixins/counter'

export default {
  mixins: [CounterMixin]
}
```

mixins 的最大缺点在于我们对它实际上添加到组件中的行为一无所知。这不仅使代码变得难以理解，而且还可能导致名称与现有属性和函数发生冲突。

- 作用域插槽

``` js
<template>
  <Counter v-slot="{ count, increment }">
     {{ count }}
    <button @click="increment">Increment</button> 
  </Counter> 
</template>
```

通过使用作用域插槽，我们确切地知道可以通过 v-slot 属性访问了哪些属性，因此代码更容易理解。这种方法的缺点是我们只能在模板中访问它，并且只能在 Counter 组件作用域内使用。

- Composition API

``` js
function useCounter() {
  const count = ref(0)
  function increment () { count.value++ }

  return {
    count,
    incrememt
  }
}

export default {
  setup () {
    const { count, increment } = useCounter()
    return {
      count,
      increment
    }
  }
}
```
使用Composition API，我们不受模板和组件作用域的限制，并且能够确切地知道可以从 counter 访问哪些属性。另外我们可以受益于编辑器中可用的代码补全功能，因为 useCounter 只是一个返回某些属性的函数，因此编辑器可以帮助我们进行类型检查和建议。

这也是使用第三方库的更优雅的方式。例如，如果我们想使用 Vuex，则可以显式地使用 useStore 函数，而不是污染 Vue 原型（this.$store）。这种方法也消除了 Vue 插件的幕后魔力。

``` js
const { commit, dispatch } = useStore()
```

[更多信息](https://vue-composition-api-rfc.netlify.com/)
[great repository](https://github.com/LinusBorg/composition-api-demos)

## 全局挂载/配置 API 更改

- Vue2
``` js
import Vue from 'vue'
import App from './App.vue'

Vue.config.ignoredElements = [/^app-/]
Vue.use(/* ... */)
Vue.mixin(/* ... */)
Vue.component(/* ... */)
Vue.directive(/* ... */)

new Vue({
  render: h => h(App)
}).$mount('#app')
```

-Vue3
``` js
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

app.config.ignoredElements = [/^app-/]
app.use(/* ... */)
app.mixin(/* ... */)
app.component(/* ... */)
app.directive(/* ... */)

app.mount('#app')
```

Vue3 每个配置都限于使用 createApp 定义的某个 Vue 程序,它可以使你的代码更易于理解，并且不易出现由第三方插件引发的意外问题。Vue2中，如果某些第三方解决方案正在修改 Vue 对象，那么它可能会以意想不到的方式（尤其是全局混合）影响你的程序，而 Vue 3 则没有这个问题。

[参考](https://github.com/vuejs/rfcs/pull/29)

## 片段（Fragments）
Vue 3 中添加了片段功能
Vue2 中无发创建这样的组件
``` js
<template>
  <div>Hello</div>
  <div>World</div>
</template>
```
原因是代表任何 Vue 组件的 Vue 实例都需要绑定到单个 DOM 元素中。创建具有多个 DOM 节点的组件的唯一方法是创建一个没有基础 Vue 实例的功能组件。

React同样有这个问题， React使用了一个名为 Fragment 的虚拟元素
``` js
class Columns extends React.Component {
  render() {
    return (
      <React.Fragment>
        <td>Hello</td>
        <td>World</td>
      </React.Fragment>
    );
  }
}
```
尽管 Fragment 看起来像是普通的 DOM 元素，但它是虚拟的，根本不会在 DOM 树中渲染。这样我们就可以将组件功能绑定到单个元素中，而无需创建冗余的 DOM 节点。

在Vue2中可以使用[vue-fragments](https://vuejsdevelopers.com/2018/09/11/vue-multiple-root-fragments/)库使用这个标签，Vue3中可以直接使用

## Suspense

将被用在 Vue 3 中的另一个从 React 学来的功能是 Suspense 组件。

Suspense 能够暂停你的组件渲染，并渲染后备组件，直到条件满足为止。在 Vue London 期间，尤雨溪简短地谈到了这个主题，并向我们展示了可以期望的 API。事实证明，Suspense 只是带有插槽的组件：
``` js
<Suspense>
  <template >
    <Suspended-component />
  </template>
  <template #fallback>
    Loading...
  </template>
</Suspense>
```

Suspense可以挂起Loading内容将一直显示到Suspended-component完全渲染为止。挂起可以等待，直到该组件被下载（如果这是一个异步组件），或者在setup功能上执行一些异步操作。

## Multiple v-models

- Vue2
我们可以从表单元素上很好的了解 v-model：
``` js
<input v-bind="property />
```

在组件内使用v-model，v-model 只是传递 value 属性和侦听 input 事件的捷径。把上面的例子重写为以下语法，将具有完全相同的效果：
``` js
<input 
  v-bind:value="property"
  v-on:input="property = $event.target.value"
/>
```
也可以通过 model 属性更改默认属性及事件

``` js
model: {
  prop: 'checked',
  event: 'change'
}
```
但是一个组件只能有一个v-model, 但是在Vue3中可以给v-model属性名

- Vue3

``` js
<InviteeForm
  v-model:name="inviteeName"
  v-model:email="inviteeEmail"
/>
```

## Portals
Portals 是特殊的组件，用来在当前组件之外渲染某些内容。它也是在 React 中实现的功能之一。这就是 React 文档关于 Portals 的内容：

“Portals 提供了一种独特的方法来将子级渲染到父组件的 DOM 层次结构之外的 DOM 节点中。”

这种处理模式，是弹出式窗口以及通常显示在页面顶部的组件所使用的一种非常好的方法。通过使用 Portals，你可以确保没有任何主机组件 CSS 规则会影响你要显示的组件，并且可以避免用 z-index 进行的黑客攻击。

对于每个 Portal，我们需要为其指定目标位置，在该目标位置将渲染 Portals 内容。在Vue2中可以使用 [portal-vue](https://github.com/LinusBorg/portal-vue) 库。

``` js
<portal to="destination">
  <p>This slot content will be rendered wherever thportal-target with name 'destination'
    is located.</p>
</portal>

<portal-target name="destination">
  <!--
  This component can be located anywhere in your App.
  The slot content of the above portal component wilbe rendered here.
  -->
</portal-target>
```

## 新的自定义指令 API

自定义指令 API 在 Vue 3 中将略有变化，以便更好地与组件生命周期保持一致。这项改进应使 API 更加直观。

- Vue2
``` js
const MyDirective = {
  bind(el, binding, vnode, prevVnode) {},
  inserted() {},
  update() {},
  componentUpdated() {},
  unbind() {}
}
```

- Vue3
``` js
const MyDirective = {
  beforeMount(el, binding, vnode, prevVnode) {},
  mounted() {},
  beforeUpdate() {},
  updated() {},
  beforeUnmount() {}, // new
  unmounted() {}
}
```
## 挂载原型

- Vue2
``` js
import Vue from 'vue'

Vue.prototype.xxx = xxx
```

-Vue3
``` js
app.config.globalProperties.xxx = xxx
```

## 渲染组件

- Vue2
``` js
import Vue from 'vue'

const Component = Vue.extend(component)
new Component({
  el: document.createElement('div')
})
document.body.appendChild($vm.$el)
```

-Vue3
``` js
import { createVNode, render } from 'vue'

const div = document.createElement('div')
const $vm = createVNode(AlertComponent)
$vm.appContext = Vue.appContext
render($vm, div)
```


[参考](https://blog.csdn.net/weixin_44420276/article/details/101621169)
[参考](https://segmentfault.com/a/1190000020933028)