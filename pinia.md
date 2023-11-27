# Pinia

## 与Vuex的区别

- 移除了 `mutations` ，所有操作都放到 `actions` 中。（mutations被用在devtools进行数据追踪，但现在这在pinia中不再是问题）

- 无需创建自定义复杂包装器来支持 TypeScript，所有内容都是类型化的，并且 API 的设计方式尽可能利用 TS 类型推断。

- 不再有 *modules* 的嵌套结构。但仍然可以通过在另一个 Store 中导入和 *使用* 来隐式嵌套 Store。

  

- **设计理念**：Pinia 是为 Composition API 设计的，它提供了一种现代、简洁和类型安全的方式来管理 Vue 应用的状态。Vuex 可以与 Options API 或 Composition API 一起使用，但它不是专门为 Composition API 设计的。

- **类型安全**：Pinia 通过 TypeScript 的类型推导提供了很好的类型安全支持，使得在 TypeScript 项目中使用 Pinia 可以得到良好的类型检查和代码补全。Vuex 在类型安全方面的支持不如 Pinia，可能需要额外的类型定义和断言。

- **API 设计**：Pinia 的 API 设计更简洁、更现代，它利用了 Vue 3 和 Composition API 的新特性，例如 `ref` 和 `reactive`。Vuex 的 API 设计相对传统，可能需要更多的样板代码和配置。

- **Store 定义**：Pinia 允许你为每个 Store 定义一个单独的文件，每个 Store 都有自己的 state、getters、actions 和 mutations，这使得代码组织更清晰，更易于管理。Vuex 通常需要你在一个单独的文件中定义所有的 state、getters、actions 和 mutations，或者通过模块系统将它们分割到多个文件中。

- **Store 使用**：在 Pinia 中，你可以直接导入并使用 Store，不需要通过 `this.$store` 或 `useStore()`。在 Vuex 中，你通常需要通过 `this.$store` 或 `useStore()` 访问 Store。

- **状态持久化和服务器端渲染（SSR）**：Pinia 提供了对状态持久化和服务器端渲染的内置支持，使得在 SSR 场景和持久化状态方面更容易使用。Vuex 可能需要额外的配置和插件来支持状态持久化和服务器端渲染。

- **开发者体验**：Pinia 提供了对状态持久化和服务器端渲染的内置支持，使得在 SSR 场景和持久化状态方面更容易使用。Vuex 可能需要额外的配置和插件来支持状态持久化和服务器端渲染。



## 基本使用

`npm install pinia`



```js
// main.js

import { createPinia } from 'pinia';
import { createApp } from 'vue';
import App from './App.vue';

const pinia = createPinia();

const app = createApp(App);
app.use(pinia);
app.mount('#app');
```



```js
// 创建一个新的 store 文件，并定义你的 store。

// stores/userStore.js
import { defineStore } from 'pinia';

export const useUserStore = defineStore('user', { // 第一个参数是应用程序中 store 的唯一 id
  state: () => ({
    name: '',
    age: 30,
  }),
  getters: { // 或者直接返回所有数据，在具体的组件中进行数据操作
    getAgePlusTen: state => {
      return state.age + 10
    },
    getAgePluasTenAgain: state => { // 访问其他的getter
      return this.getAgePlusTen + 10
    },
    getOther: state => { // 访问其他store
      const otherStore = useOtherStore()
      creturn otherStore.something + 1
    }
    ageInYears: state => { // 可以返回一个函数，以实现传递参数
      return (years) => state.age + years; 
    }
  },
  actions: {
    updateName(newName) {
      this.userData = await api.post({ login, password }) // 异步操作，这里是网络请求
      this.name = newName; // 可以通过this访问
    },
    incrementAge() {
      this.age++;
    },
  },
});

// 或者
export const useCounterStore = defineStore('counter', () => {
  const count = ref(0)
  function increment() {
    count.value++
  }

  return { count, increment }
})
```



```vue
// 在组件中使用

<template>
  <div>
    <p>Name: {{ user.name }}</p>
    <p>Age: {{ user.age }}</p>
    <button @click="user.updateName('Jane Doe')">Update Name</button>
    <button @click="aaa">一些操作</button>
  </div>
</template>

<script setup>
import { useUserStore } from '@/stores/userStore';

const user = useUserStore();
    
user.updateName('初始化名称')
    
function aaa() {
  user.$patch({ // $patch 方法用于部分更新 store 的状态
    name: user.name + 'aaa',
    age: 40
  })
    
  cartStore.$patch((state) => { // 也可以传入一个函数来批量修改
    state.items.push({ name: 'shoes', quantity: 1 })
    state.hasChanged = true
  })
}
</script>
```



## State

### 不要直接结构 & storeToRefs

`storeToRefs` 是 Pinia 的一个实用功能，它可以将一个 store 的所有属性转换为单独的 refs。这使得你可以在组件中直接访问 store 的每个属性，而不是通过 store 对象。这是一种使得模板和组件代码更清晰、更简洁的方法，特别是当你需要在模板中访问多个 store 属性时。

```js
// src/stores/userStore.js
import { defineStore } from 'pinia';

export const useUserStore = defineStore({
  id: 'user',
  state: () => ({
    name: 'John Doe',
    age: 30,
  }),
  actions: {
    updateName(newName) {
      this.name = newName;
    },
    incrementAge() {
      this.age++;
    },
  },
});
```

```vue
// YourComponent.vue
<template>
  <div>
    <p>Name: {{ name }}</p>
    <p>Age: {{ age }}</p>
    <button @click="updateName('Jane Doe')">Update Name</button>
    <button @click="incrementAge">Increment Age</button>
  </div>
</template>

<script setup>
import { useUserStore } from '@/stores/userStore';
import { storeToRefs } from 'pinia';

// store 是一个用reactive 包裹的对象，这意味着不需要在getter 之后写.value，但是，就像setup 中的props 一样，我们不能对其进行解构：
const { name, age } = store // 直接解构会破坏响应式

// 可以通过解构赋值并使用storeToRefs将这些 refs 分配给本地变量，然后在模板和组件逻辑中直接使用这些变量，而不再需要store.xxx的方式
const { name, age } = storeToRefs(useUserStore());
// 如果你直接改变 age 的值，它将改变 useUserStore 中的 age 值。这是因为 storeToRefs 返回的是对 store 中状态的 ref 引用，而不是状态的副本。因此，当你改变 age ref 的值时，它会反映到 store 的状态中。这是 Vue 的响应式系统和 ref API 的工作方式。
    
</script>
```

### $reset

```js
const store = useStore()
store.$reset() // $reset() 方法将状态 重置 到其初始值
```

### $state

```js
store.$state = { counter: 666, name: 'Paimon' } // 通过将其 $state 属性设置为新对象来替换 Store 的整个状态
```

### $subscribe

在 Pinia 中，`$subscribe` 用于监听 store 的变化。`$subscribe` 方法接受一个回调函数作为参数，该回调函数将在每次 store 状态改变时被调用。

与常规的 `watch()` 相比，使用 `$subscribe()` 的优点是 *subscriptions* 只会在 *patches* 之后触发一次。

```js
// src/stores/userStore.js
import { defineStore } from 'pinia';

export const useUserStore = defineStore({
  id: 'user',
  state: () => ({
    name: 'John Doe',
    age: 30,
  }),
  actions: {
    updateName(newName) {
      this.name = newName;
    },
    incrementAge() {
      this.age++;
    },
  },
});
```

```vue
// YourComponent.vue
<template>
  <div>
    <p>Name: {{ user.name }}</p>
    <p>Age: {{ user.age }}</p>
    <button @click="user.updateName('Jane Doe')">Update Name</button>
    <button @click="user.incrementAge">Increment Age</button>
  </div>
</template>

<script setup>
import { useUserStore } from '@/stores/userStore';
import { onUnmounted } from 'vue';

const user = useUserStore();

user.$subscribe((mutation, state) => {
  console.log('Store mutation:', mutation);
  console.log('Mutation type:', mutation.type);  // e.g., "updateName"
  console.log('Mutation payload:', mutation.payload);  // e.g., "Jane Doe"
  console.log('Store state:', state);
});
// 第一个参数mutation包含两个信息：
// 1、type: 描述变化类型的字符串。例如，如果你在 action 或 setter 中更新了一个属性，type 可能会是该属性的名称。
// 2、payload: 该变化的额外数据。例如，如果你更新了一个属性，payload 可能会是该属性的新值。
    
// 默认情况下，state subscriptions 绑定到添加它们的组件（如果 store 位于组件的 setup() 中）。当组件被卸载时，它们将被自动删除。如果要在卸载组件后保留它们，请将 { detached: true } 作为第二个参数传递给 detach 当前组件的 state subscription：
user.$subscribe((mutation, state) => {
  // 一些操作
}, { deep: true });
</script>
```

## Getters



## Actions

`$onAction` 允许监听 store 中的 actions，来执行某些逻辑，比如日志记录或调试。接受一个回调函数作为参数，接收到关于正在执行的 action 的信息，包括名称和参数。

```js
const unsubscribe = someStore.$onAction(
  ({
    name, // action 的名字
    store, // store 实例
    args, // 调用这个 action 的参数
    after, // 在这个 action 执行完毕之后，执行这个函数
    onError, // 在这个 action 抛出异常的时候，执行这个函数
  }) => {
    console.log(111) // 这将在 `store` 上的操作执行之前触发

    // 如果 action 成功并且完全运行后，after 将触发，它将等待任何返回的 promise
    after(result => { console.log(222) })

    // 如果 action 抛出或返回 Promise.reject ，onError 将触发
    onError(error => { console.warn(333) })
  }
)

// 可以手动移除订阅，但默认情况下会在组件销毁时自动移动，不需要手动
// unsubscribe()

// 如果要在卸载组件后保留它们，请将 true 作为第二个参数
const someStore = useSomeStore()
someStore.$onAction(callback, true) // 此订阅将在组件卸载后保留
```



## Plugins

[插件 | Pinia 中文文档 (web3doc.top)](https://pinia.web3doc.top/core-concepts/plugins.html)

用于扩展或增强store，例如添加新的方法或响应 store 的变化。你可以创建自己的插件，或使用社区提供的插件。

在创建 Pinia 实例时通过 `plugins` 选项传递一个插件数组。每个插件都是一个函数，它接收一个 `store` 参数，并可以返回一个 cleanup 函数。

你需要在pinia实例创建前添加plugins，否则将不会被应用。

```js
import { createPinia } from 'pinia'

// 为安装此插件后创建的每个store添加一个名为 `secret` 的属性
// 这可能在不同的文件中
function SecretPiniaPlugin() {
  return { secret: 'the cake is a lie' }
}

const pinia = createPinia()

pinia.use(SecretPiniaPlugin) // 将插件提供给 pinia

pinia.use({
  pinia // 使用 `createPinia()` 创建的 pinia
  app // 使用 `createApp()` 创建的当前应用程序（仅限 Vue 3）
  store // 插件正在扩充的 store
  options // 定义存储的选项对象传递给`defineStore()`
} => {
  // .,.
})
```

```js
// 在另一个文件中
const store = useStore()
store.secret // 'the cake is a lie'
```

