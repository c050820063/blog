---
title: Vue3 study
date: 2020-06-04 14:25:16
tags:
---

## 组件 API（Composition API）

组件 API 旨在通过将组件属性中当前可用的机制公开为 JavaScript 函数来解决这个问题。 Vue 核心团队将组件 API 描述为 “一组基于函数的附加 API，可以灵活地组合组件逻辑。” 用组件 API 编写的代码更具有可读性，并且其背后没有任何魔力，因此更易于阅读和学习。

让我们通过一个用了新的组件 API 的组件的简单示例，来了解其工作原理。

``` bash
<template>
  <button @click="increment">
    Count is: {{ count }}, double is {{ double }}, click to increment.
  </button>
</template>

<script>
import { ref, computed, onMounted } from 'vue'

export default {
  setup() {
    const count = ref(0)
    const double = computed(() => count.value * 2)

    function increment() {
      count.value++
    }

    onMounted(() => console.log('component mounted!'))

    return {
      count,
      double,
      increment
    }
  }
}
</script>

```

- import { ref, computed, onMounted } from 'vue'

需要使用 ref 创建响应性引用，用 computed 建立计算属性，并用 onMounted 访问安装的生命周期 hook

- setup 是用来干嘛的呢？

简而言之，它只是一个将属性和函数返回到模板的函数而已。我们在这里声明所有响应性属性、计算属性、观察者和生命周期 hook，然后将它们返回，以便可以在模板中使用它们。

我们不从 setup 函数返回的内容在模板中将会不可用。

- const count = ref(0)

根据上面的内容，我们声明了带有 ref 函数的名为 count 的响应属性。它可以包装任何原语或对象并返回其响应性引用。传递的元素的值将会保留在所创建引用的 value 属性中。例如，如果你想访问 count 引用的值，则需要明确要求 count.value。

``` bash
const double = computed(() => count.value * 2)

function increment() {
  count.value++
}
```

这个是声明计算属性 computed

- onMounted(() => console.log('component mounted!'))

这个是访问生命周期 hook

## 用组件 API 进行代码重用

新的组件 API 具有更多优点。考虑一下代码重用。目前如果我们要在其他组件之间共享一些代码，则有两个可用的选择：mixins 和作用域插槽（ scoped slots）。但是两者都有缺点。

假设我们要提取 counter 中的功能并在其他组件中重用。在下面，你可以看到如何将其与可用的 API 和新的组件 API 结合使用：

- mixins

``` bash
import CounterMixin from './mixins/counter'

export default {
  mixins: [CounterMixin]
}
```

mixins 的最大缺点在于我们对它实际上添加到组件中的行为一无所知。这不仅使代码变得难以理解，而且还可能导致名称与现有属性和函数发生冲突。

- 作用域插槽

``` bash
<template>
  <Counter v-slot="{ count, increment }">
     {{ count }}
    <button @click="increment">Increment</button> 
  </Counter> 
</template>
```

通过使用作用域插槽，我们确切地知道可以通过 v-slot 属性访问了哪些属性，因此代码更容易理解。这种方法的缺点是我们只能在模板中访问它，并且只能在 Counter 组件作用域内使用。

- Composition API

``` bash
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

``` bash
const { commit, dispatch } = useStore()
```

[更多信息](https://vue-composition-api-rfc.netlify.com/)
[great repository](https://github.com/LinusBorg/composition-api-demos)

## 全局挂载/配置 API 更改

- Vue2
``` bash
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
``` bash
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

[more](https://github.com/vuejs/rfcs/pull/29)

## 片段（Fragments）
Vue 3 中添加了片段功能
Vue2 中无发创建这样的组件
``` bash
<template>
  <div>Hello</div>
  <div>World</div>
</template>
```
原因是代表任何 Vue 组件的 Vue 实例都需要绑定到单个 DOM 元素中。创建具有多个 DOM 节点的组件的唯一方法是创建一个没有基础 Vue 实例的功能组件。

React同样有这个问题， React使用了一个名为 Fragment 的虚拟元素
``` bash
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
``` bash
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
``` bash
<input v-bind="property />
```

在组件内使用v-model，v-model 只是传递 value 属性和侦听 input 事件的捷径。把上面的例子重写为以下语法，将具有完全相同的效果：
``` bash
<input 
  v-bind:value="property"
  v-on:input="property = $event.target.value"
/>
```
也可以通过 model 属性更改默认属性及事件

``` bash
model: {
  prop: 'checked',
  event: 'change'
}
```
但是一个组件只能有一个v-model, 但是在Vue3中可以给v-model属性名

- Vue3

``` bash
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

``` bash
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
``` bash
const MyDirective = {
  bind(el, binding, vnode, prevVnode) {},
  inserted() {},
  update() {},
  componentUpdated() {},
  unbind() {}
}
```

- Vue3
``` bash
const MyDirective = {
  beforeMount(el, binding, vnode, prevVnode) {},
  mounted() {},
  beforeUpdate() {},
  updated() {},
  beforeUnmount() {}, // new
  unmounted() {}
}
```

[参考](https://segmentfault.com/a/1190000020933028)

