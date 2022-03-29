# Pinia

> 官网地址：https://pinia.vuejs.org/introduction.html

Pinia 最初是在 2019 年 11 月左右重新设计使用 Composition API 的 Vue Store 外观的实验。从那时起，最初的原则仍然相同，但 Pinia 适用于 Vue 2 和 Vue 3，并且不需要你 使用组合 API。 除了安装和 SSR 之外，两者的 API 都是相同的。

Pinia 是 Vue 的存储库，它允许您跨组件/页面共享状态。如果您熟悉 Composition API，您可能会认为您已经可以使用简单的`export const state = reactive({})`. 这对于单页应用程序来说是正确的，但如果它是服务器端呈现的，则会**将您的应用程序暴露给安全漏洞。**但即使在小型单页应用程序中，您也可以从使用 Pinia 中获得很多好处：

- 开发工具支持
  - 跟踪actions, mutations的时间线
  - Stores 出现在使用它们的组件中
  - 更容易的调试
- 热模块更换
  - 在不重新加载页面的情况下修改stores 
  - 在开发时保持任何现有状态
- 插件：使用插件扩展 Pinia 功能
- 为 JS 用户提供适当的 TypeScript 支持或**自动完成功能**
- 服务器端渲染支持



### 基本实例

```js
// stores/counter.js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => {
    return { count: 0 }
  },
  // could also be defined as
  // state: () => ({ count: 0 })
  actions: {
    increment() {
      this.count++
    },
  },
})
```

在组件中使用：多种方式修改stores中的state

```js
import { useCounterStore } from '@/stores/counter'

export default {
  setup() {
    const counter = useCounterStore()

    counter.count++
    // with autocompletion ✨
    counter.$patch({ count: counter.count + 1 })
    // or using an action instead
    counter.increment()
  },
}
```

甚至可以使用函数（类似于组件 setup()）为更高级的用例定义 Store：

```js
export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)
  function increment() {
    count.value++
  }

  return { count, increment }
})
```

如果您仍然不熟悉 setup() 和 Composition API，请不要担心，Pania 还支持一组类似的地图助手，如 Vuex。您以相同的方式定义存储，但随后使用 mapStores()、mapState() 或 mapActions()：

```js
const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0 }),
  getters: {
    double: (state) => state.count * 2,
  },
  actions: {
    increment() {
      this.count++
    }
  }
})

const useUserStore = defineStore('user', {
  // ...
})

export default {
  computed: {
    // other computed properties
    // ...
    // gives access to this.counterStore and this.userStore
    ...mapStores(useCounterStore, useUserStore)
    // gives read access to this.count and this.double
    ...mapState(useCounterStore, ['count', 'double']),
  },
  methods: {
    // gives access to this.increment()
    ...mapActions(useCounterStore, ['increment']),
  },
}
```

以上示例，会发现更多关于pinia的map用法

### 完整的pinia使用示例：

以下是一个更完整的 API 示例，您将在 Pinia 中使用类型，即使在 JavaScript 中也是如此：

```ts
import { defineStore } from 'pinia'

export const todos = defineStore('todos', {
  state: () => ({
    /** @type {{ text: string, id: number, isFinished: boolean }[]} */
    todos: [],
    /** @type {'all' | 'finished' | 'unfinished'} */
    filter: 'all',
    // type will be automatically inferred to number
    nextId: 0,
  }),
  getters: {
    finishedTodos(state) {
      // autocompletion! ✨
      return state.todos.filter((todo) => todo.isFinished)
    },
    unfinishedTodos(state) {
      return state.todos.filter((todo) => !todo.isFinished)
    },
    /**
     * @returns {{ text: string, id: number, isFinished: boolean }[]}
     */
    filteredTodos(state) {
      if (this.filter === 'finished') {
        // call other getters with autocompletion ✨
        return this.finishedTodos
      } else if (this.filter === 'unfinished') {
        return this.unfinishedTodos
      }
      return this.todos
    },
  },
  actions: {
    // any amount of arguments, return a promise or not
    addTodo(text) {
      // you can directly mutate the state
      this.todos.push({ text, id: this.nextId++, isFinished: false })
    },
  },
})
```



### 与 Vuex 的比较

Pinia 最初是为了探索 Vuex 的下一次迭代会是什么样子，结合了 Vuex 5 核心团队讨论中的许多想法。最终，我们意识到 Pinia 已经实现了我们在 Vuex 5 中想要的大部分内容。

与 Vuex 相比，Pinia 提供了一个更简单的 API，具有更少的仪式，提供了 Composition-API 风格的 API，最重要的是，在与 TypeScript 一起使用时具有可靠的类型推断支持。

现在 Pinia 已经成为默认的状态管理解决方案，它和 Vue 生态系统中的其他核心库一样遵循 RFC 流程，其 API 也进入了稳定状态。

**与 Vuex 3.x/4.x 的比较**

> Vuex 3.x 是Vue 2的Vuex， 而 Vuex 4.x 是 Vue 3的

Pinia API 与 Vuex ≤4 有很大不同，即：

- *mutations*不再存在。他们经常被认为非常冗长。他们最初带来了 devtools 集成，但这不再是问题。
- 无需创建自定义复杂包装器来支持 TypeScript，所有内容都是类型化的，并且 API 的设计方式尽可能利用 TS 类型推断。
- 不再需要注入魔法字符串、导入函数、调用它们，享受自动完成功能！
- 无需动态添加stores，默认情况下它们都是动态的，您甚至都不会注意到。请注意，您仍然可以随时手动使用store进行注册，但因为它是自动的，您无需担心。
- 不再有*modules*的嵌套结构。您仍然可以通过在另一个商店中导入和使用商店来隐式嵌套商店，但 Pinia 通过设计提供平面结构，同时仍然支持商店之间的交叉组合方式。您甚至可以拥有stores的循环依赖关系。 
- 没有命名空间的模块。鉴于stores的扁平架构，“命名空间”stores是其定义方式所固有的，您可以说所有stores都是命名空间的。



### 安装

```sh
yarn add pinia
# or with npm
npm install pinia
```

如果在 Vue 2使用，还需要安装组合 API：@vue/composition-api。如果您使用 Nuxt，则应遵循[这些](https://pinia.vuejs.org/ssr/nuxt.html)说明。

如果使用的是Vue CLI，还可以使用这个[非官方的插件](https://github.com/wobsoriano/vue-cli-plugin-pinia)

创建一个pinia并且在app中使用：

```js
import { createPinia } from 'pinia'

app.use(createPinia())
```

在vue2中还需要安装插件的方式去注入创建的pinia

```js
import { createPinia, PiniaVuePlugin } from 'pinia'

Vue.use(PiniaVuePlugin)
const pinia = createPinia()

new Vue({
  el: '#app',
  // other options...
  // ...
  // note the same `pinia` instance can be used across multiple Vue apps on
  // the same page
  pinia,
})
```



## 核心概念

### Pinia是什么？

一个store（如 Pinia）是一个实体，它持有未绑定到组件树的状态和业务逻辑。换句话说，它托管全局状态。它有点像一个始终存在并且每个人都可以读取和写入的组件。它包含三个概念，state、getters 和actions，可以安全地假设这些概念等同于组件中的data、computed和methods。

### 定义Pinia

在深入了解核心概念之前，我们需要知道 store 是使用 defineStore() 定义的，并且它需要一个唯一的名称，作为第一个参数传递

```js
import { defineStore } from 'pinia'

// useStore could be anything like useUser, useCart
// the first argument is a unique id of the store across your application
export const useStore = defineStore('main', {
  // other options...
})
```

名称（也称为 id）是必需的，Pinia 使用此name将store连接到开发工具。将返回的函数命名为 use... 是可组合项之间的约定，以使其使用习惯。

### 使用Pinia

下面我们在setup()中定义一个Store，因为在useStore()被调用之前store不会被创建：

```js
import { useStore } from '@/stores/counter'

export default {
  setup() {
    const store = useStore()

    return {
      // you can return the whole store instance to use it in the template
      store,
    }
  },
}
```

也可以根据需要定义任意数量的Store，并且应该在不同的文件中定义每个Store以充分利用 pinia（例如自动允许您的包进行代码拆分和 TypeScript 推理）。

如果组件中不使用setup的方式，那么可以通过[map helper](https://pinia.vuejs.org/cookbook/options-api.html)的方式使用Pinia。

一旦 store 被实例化，就可以直接在 store 上访问 state、getters 和 actions 中定义的任何属性。

注意：store是一个被`reactive`包裹的对象，这意味在getters之后，不要写`.value`，但是就像在setup中的`props`一样，我们不要去解构它：

```js
export default defineComponent({
  setup() {
    const store = useStore()
    // 不能正常工作，因为解构阻断了响应式，就像从props中解构一样
    const { name, doubleCount } = store

    name // "eduardo"
    doubleCount // 2

    return {
      // name依然是'eduardo'
      name,
      // 依然是2
      doubleCount,
      // 下面这个是响应式的
      doubleValue: computed(() => store.doubleCount),
      }
  },
})
```

**Pinia的storeToRefs**

为了从store中提取属性同时保持其响应式，您需要使用 storeToRefs()。它将为每个反应属性创建参考。当您仅使用store中的状态但不调用任何操作时，这很有用。注意，可以直接从store中解构操作，因为它们也绑定到store本身：

```js
import { storeToRefs } from 'pinia'

export default defineComponent({
  setup() {
    const store = useStore()
    // `name` and `doubleCount` are reactive refs
    // This will also create refs for properties added by plugins
    // but skip any action or non reactive (non ref/reactive) property
    // name` and `doubleCount`是响应式的refs
    const { name, doubleCount } = storeToRefs(store)
    // the increment action can be just extracted
    // increment方法可以被提取使用
    const { increment } = store

    return {
      name,
      doubleCount
      increment,
    }
  },
})
```

### State

在 Pinia 中，状态被定义为返回初始状态的函数。这允许 Pinia 在服务器端和客户端工作。

```js
import { defineStore } from 'pinia'

const useStore = defineStore('storeId', {
  // arrow function recommended for full type inference
  state: () => {
    return {
      // all these properties will have their type inferred automatically
      counter: 0,
      name: 'Eduardo',
      isAdmin: true,
    }
  },
})
```

**访问state**：默认情况下，您可以通过 store 实例访问状态来直接读取和写入状态：

```js
const store = useStore()

store.counter++
```

**重置state：**可以通过调用 store 上的 $reset() 方法将state重置为其初始值：

```js
const store = useStore()

store.$reset()
```

**修改state：**除了直接用 store.counter++ 修改 store，你还可以调用 $patch 方法。它允许您使用部分状态对象同时应用多个更改：

```js
store.$patch({
  counter: store.counter + 1,
  name: 'Abalam',
})
```

但是，使用这种语法应用某些突变非常困难或代价高昂：任何集合修改（例如，从数组中推送、删除、拼接元素）都需要您创建一个新集合。**正因为如此，$patch 方法也接受一个函数来分组这种难以用补丁对象应用的突变：**

```js
cartStore.$patch((state) => {
  state.items.push({ name: 'shoes', quantity: 1 })
  state.hasChanged = true
})
```

这里的主要区别是 $patch() 允许您将多个更改分组到 devtools 中的一个条目中。请注意，对 state 和 $patch() 的直接更改出现在 devtools 中，并且可以进行time travelled（在 Vue 3 中还没有）。

**更换状态：**

可以通过将store的 $state 属性设置为新对象来替换store的整个状态：

```js
store.$state = { counter: 666, name: 'Paimon' }
```

还可以通过更改 pinia 实例的state来替换应用程序的整个state。这在 SSR 期间的[State hydration](https://pinia.vuejs.org/ssr/#state-hydration)。
