# options API

## setup

**注意：不要在setup中使用`this`**

### 参数

```typescript
// setup(props, {attrs, slots, emit}) {
setup(props, context) {
  console.log(props.message)
  console.log(context.attrs.id)
  console.log(context.slots)
  console.log(context.emit)
}
```

- 第一个参数：props 

  ```js
  // 注意：props是响应式的，但不能直接进行解构，例如 { index } = props，原因为 会破坏Proxy响应式特性
  // 如果确实需要解构，则需要使用toRefs
  import { toRefs } from 'vue'
  setup(props) {
    const { title } = toRefs(props)
    console.log(title.value)
  }
  
  // 此外，如果props中某个值是可选的，则可以没有传入，此时需要用toRef
  import { toRef } from 'vue'
  setup(props) {
    const title = toRef(props, 'title')
    console.log(title.value)
  }
  ```

- 第二个参数：context

  ​	包含3个属性：

  1. attrs：所有的非prop的attribute；
  2. slots：父组件传递过来的插槽（主要用于render函数）；
  3. emit：组件内部需要发出事件时会用到emit
  4. expose：一般用不到，结合返回的h渲染函数使用，看官方文档
  
  ```js
  setup(props, context) {
    console.log(context.attrs) // Attribute (非响应式对象，等同于 $attrs)
    console.log(context.slots) // 插槽 (非响应式对象，等同于 $slots)
    console.log(context.emit) // 触发事件 (方法，等同于 $emit)
    console.log(context.expose) // 暴露公共 property (函数)
  }
  
  // context是普通的JavaScript对象，并不是响应式的，可以进行解构
  setup(props, { attrs, slots, emit, expose }) {
    ...
  }
  // attrs、slots是有状态的对象，会随组件本身的更新而更新，避免对它们进行解构，始终以attrs.x、slots.x的方式引用property。请注意，与props不同，attrs、slots的property是非响应式的。如果打算根据attrs、slots的更改应用副作用，那么应该在onBeforeUpdate生命周期中执行。
  ```
  

### 返回值

```typescript
// 类似于 data{ return {...} } 中返回的内容，setup的返回值可以在模板template中被使用，也可以返回一个执行函数来代替在methods中定义的方法
// 注意：返回值不是响应式的，改变值时页面不会发生刷新，因此，需要使用reactive或ref
<template>
	<div>{{counter}}</div>
</template>

setup() {
	let counter = 100
	const increment = () => { counter++ }
	
	return {
		title: "Hello",
		counter,
		increment
	}
}
```

## reactive

```typescript
<template>
	<div>{{state.counter}}</div>
</template>

import { reactive } from 'vue'

setup() {
  // reactive传入的值只能是对象或数组
  const state = reactive({
    counter: 100
  })
}
```

注意：reactive会进行深层的转化，另一个：shallowReactive

## ref

```typescript
<template>
	<div>{{counter.value}}</div>
	<div>{{counter}}</div> // vue会自动解包，在template中使用时可以省略.value
	<div>{{info.counter.value}}</div> // 注意：ref的解包只有浅层
</template>

import { ref } from 'vue'

setup() {
  let counter = ref(100)
  const increment = () => {
    counter.value++ // 需要.value
  }
  return {
    counter,
    increment
  }
}
```

## readonly

```typescript
<template>
	<div>{{counter.value}}</div>
</template>

import { readonly } from 'vue'

setup() {
  const info1 = {name: "why"}
  const readonlyInfo1 = readonly(info1)
  
  const info2 = reactive({
    name: "why"
  })
  const readonlyInfo2 = readonly(info2)
  
  const info3 = ref("why")
  const readonlyInfo3 = readonlt(info3)
  
  const updateState = () => {
    readonlyInfo1.name = "aaa" // 报错
    info1.name = ... // 可以
  }
  
  return {
    updateState
  }
}
```

注意：readonly会进行深层的转化，如果想要只有最外层是readonly，则使用 shallowReadonly

## toRefs

用ES6的解构语法对reactive返回的对象进行解构，之后无论是修改结构后的变量，还是修改reactive 返回的state对象，数据都不再是响应式的

```typescript
const state = reactive({
  name: "why",
  age: 18
})

const { name, age } = state // name 和 age 不再是响应式的

// toRefs可以将reactive返回的对象中的属性都转成ref，再次进行结构出来的本身都是 ref 的
const { name, age } = toRefs(state)
const changeAge = () => {
  age.value++
}
// 这种做法相当于已经在state.name和ref.value之间建立了链接，任何一个修改都会引起另外一个变化

// 如果只转换reactive对象中的其中一个属性为ref, 那么可以使用toRef
const name = toRef(state, 'name')
const changeName = () => state.name = "why"
```



## computed

```typescript
import { computed } from 'vue'

setup() {
  // 方式一：接收一个getter函数，并为 getter 函数返回的值
  const fullName = computed(() => firstName + lastName)
  fullName.value = "xxx" // 注意：返回一个不变的 ref 对象
  
  // 方式二：接收一个具有 get 和 set 的对象，返回一个可变的（可读写）ref 对象
  const fullName = computed({
		get: () => {
			return firstName.value + " " + lastName.value
		},
		set: newValue => {
			const names = newValue.split(" ");
			firstName.value = names[0]
			lastName.value = names[1]
		}
	})
}
```



## watchEffect & watch

### watchEffect

watchEffect用于自动收集响应式数据的依赖 

watch需要手动指定侦听的数据源

```typescript
import { watchEffect } from 'vue'

// 1.watchEffect传入的函数会被立即执行一次，收集的依赖发生变化时，会再次执行
const name = ref("why")
const age = ref(17)
watchEffect(() => {
  // 2.没有指定具体的侦听对象，watchEffect在第一次立即执行时会自动收集依赖的对象
  console.log("执行~", name.value, age.value)
})
```

```typescript
// 停止侦听
const stopWatch = watchEffect(() => {
  console.log("执行~", name.value, age.value)
})

if (...) {
	stopWatch() // 停止侦听
}
```

```typescript
// 清除副作用
// 在我们给watchEffect传入的函数被回调时，其实可以获取到一个参数：onInvalidate
const stopWatch = watchEffect((onInvalidate) => {
  onInvalidate(() => { // 接收一个函数，在侦听器执行或销毁时会执行一次这个函数
    // 在这个函数中清除额外的副作用
    // 例如发送的但还没有完成的网络请求
  })
  console.log("执行~", name.value, age.value)
})
```

```typescript
// 执行时机
// 默认情况下，组件的更新会在副作用函数执行之前
watchEffect(() => {
  console.log()
}, {
  flush: "pre" // 默认值是，在元素 挂载 或者 更新 之前 进行执行
  flush: "post" // 改变执行的时机
  flush: "sync" // 将强制效果始终同步触发。然而，这是低效的，应该很少需要。
})
```

### watch

watch需要手动执行具体要侦听谁，并且不会自动立即执行一次

```typescript
import { watch } from 'vue'

setup() {
  const info = reactive({name: "why", age: 18})
  const name = ref("kobe")

  watch(info, (newValue, oldValue) => { 
    // 变化前后的值
  })
  
  /* 第一个参数的类型：
  		1.reactive类型对象：newValue和oldValue也会是reactive对象，处理方法：见下方
  				
  		2.ref对象：newValue和oldValue是ref的value值
  		
  		3.数组：用于侦听多个数据，newValue和oldValue也会变成数组
  				watch([info, name], (newValue, oldValue) => { 
  				
  		4.函数：可以进行解构，用于获取普通的对象
  				watch(() => [...info], (newValue, oldValue) => { 
  */
  
  // reactive类型的参数如果想获取正确的值：
  watch(() => {
    return {...info}
  }, (newValue, oldValue) => {
    ...
  })
  
  // 第3个参数
  watch(() => state, (newValue, oldValue) => {
    ...
  }, {
    deep: true, // 是否深度侦听，如果第一个参数传入的是一个reactive对象，则会默认深度侦听
    immediate: true // 是否立即执行
  })
}
```



## ref

```typescript
<div ref="titleRef">title</div>

import { ref } from "vue"

setup() {
  const titleRef = ref(null)
  return {
    titleRef
  }
}
```



## 生命周期

多个同名的生命周期会进行合并

beforeCreate和created移出，相关操作直接放到setup中即可

```typescript
<script>
  import { onMounted, onUpdated, onUnmounted }
	export default {
    setup() {
      onMounted(() => {...})
      onUpdated(() => {...})
      onUnmounted(() => {...})
    }
  }
</script>
```



## provide & inject

```typescript
// 在父组件中
<script>
import { provide, readonly, ref, reactive } from 'vue'
export default {
  setup() {
    let counter = ref(100) // 可以变为响应式，值发生变化时，子组件会自动刷新
    let info = reactive({ // 可以变为响应式
      name: "why",
      age: 10
    })
    // 注意：尽量在父组件中改变provide提供的值，类似于props的单向数据流，如果确实需要在子组件中改变属性的值，可以暴露对应的方法
    const changeInfo = () => { info.name = "aaa" }
    
    // 如果要确保值不会被子组件修改，可以在共享时用readonly包裹
    provide("counter", readonly(counter)
    provide("info", readonly(info)
    provide("changeInfo", changeInfo) // 如果需要修改可响应的数据，最好是在数据提供的位置来修改，将修改方法进行共享，在后代组件中进行调用
  }
}
</script>


// 在后代组件中
<script>
import { inject } from 'vue'

export default {
  setup() {
    // inject第一个参数为属性的name，第二个参数为默认值(可选)
    const counter = inject("counter", 100000) // 可以指定默认值
		const info = inject("info")
    // 使用传入的方法修改inject的值，而不是在子组件中直接修改，类似于单向数据流
    const changeInfo = inject("changeInfo")

    return {
      counter,
      info,
      changeInfo
    }
  }
}
</script>
```



## 使用hook

```typescript
// 抽取
import { ref } from 'vue'
export default function useCounter() {
	const counter = ref(0) ;
  
	const increment = () => counter.value++ 
	const decrement = () => counter.value--
  
	return {
		counter，
		increment，
		decrement
	}
}
```

```typescript
// 使用
<script>
  import useCounter from '...'
	
	export default {
    setup() {
      const { counter， increment， decrement } = useCounter()
    }
    
    return {
    	counter， 
    	increment， 
    	decrement
  	}
    
    return {
    	...useCounter() // 或者直接在return中使用展开运算符，不推荐这种写法，阅读性比较差
  	}
  }
</script>
```



## 自定义指令

需要对DOM元素进行底层操作，就会用到自定义指令

自定义指令分为两种： 

- 自定义局部指令：组件中通过 directives 选项，只能在当前组件中使用； 
- 自定义全局指令：app的 directive 方法，可以在任意组件中被使用；

```typescript
// 局部指令
<input type="text" v-focus> // 以v-开头

<script>
	export default {
		directives: {
      focus: {
        mounted(el, bindings, vnode, preVnode) { // 4个形参
          el.focus()
          
          // 可以在自定义指令中使用生命周期
          /*
          	created：在绑定元素的attribute或事件监听器被应用之前调用。在指令需要附加在普通的 v-on 事件监听器调用前的事件监听器中时，这很有用。
            beforeMount：当指令第一次绑定到元素并且在挂载父组件之前调用。
            mounted：在绑定元素的父组件被挂载前调用。
            beforeUpdate：在更新包含组件的 VNode 之前调用。
            updated：在包含组件的 VNode 及其子组件的 VNode 更新后调用。
            beforeUnmount：在卸载绑定元素的父组件之前调用
            unmounted：当指令与元素解除绑定且父组件已卸载时，只调用一次。
          */
        }
      }
    }
	}
</script>


// 动态参数 v-mydirective:[argument]="value"
<p v-pin:[direction]="200"> // 传入arg和value
  page
</p>
const app = Vue.createApp({
  data() {
    return {
      direction: 'right'
    }
  }
})
app.directive('pin', {
  mounted(el, binding) { // 生命周期函数
    el.style.position = 'fixed'
    const s = binding.arg || 'top' // arg
    el.style[s] = binding.value + 'px' // value
  }
})

// 传入多个值，使用对象字面量
<div v-demo="{ color: 'white', text: 'hello!' }"></div>
app.directive('demo', (el, binding) => {
  console.log(binding.value.color)
  console.log(binding.value.text)
})
```

```typescript
// 全局指令
// main.js
const app = createApp(app)

app.directive("focus", {
  mounted(el) {
    el.focus()
  }
})

app.mount("#app")
```

指令的生命周期：

- created：在绑定元素的 attribute 或事件监听器被应用之前调用；
- beforeMount：当指令第一次绑定到元素并且在挂载父组件之前调用；
- mounted：在绑定元素的父组件被挂载后调用；
- beforeUpdate：在更新包含组件的 VNode 之前调用；
- updated：在包含组件的 VNode 及其子组件的 VNode 更新后调用；
- beforeUnmount：在卸载绑定元素的父组件之前调用；
- unmounted：当指令与元素解除绑定且父组件已卸载时，只调用一次；

```typescript
// 自定义指令中使用参数和修饰符
// 在第2个形参bindings中获取
<button v-why:info.aaa.bbb="{name: 'coderwhy', age: 18}">{{counter}}</button>
```

```typescript
// 练习：格式化时间戳
import dayjs from 'dayjs';

export default function(app) {
	app.directive("format-time", {
		created(el, bingings) {
      bingings.format = "YYYY-MM-DD HH:mm:ss"
			if (bingings.value) {
				bingings.format = bingings.value;
			}
		},
		mounted(el) {
			const textContent= el.textContent;
			let timestamp = parseInt(el, textContent);
			if (textContent.length === 10) {
				timestamp = timestamp * 1000;
			}
			console.log(timestamp);
			el.textContent = dayjs(timestamp).format(bingings.format);
		}
	});
}
```

## teleport

类似于React的portals，将某个元素挂载到其他的组件树上

两个属性： 

- to：指定将其中的内容移动到的目标元素，可以使用选择器；
- disabled：是否禁用 teleport 的功能；

```typescript
<template>
  <div class="my-app">
    // <teleport to="#aaa"> // 元素选择器
    <teleport to="body">
      <h2>why</h2>
		</teleport>
	</div>
</template>

// 和组件结合使用
<template>
  <div class="my-app">
    <teleport to="body">
      <hello-world message="aaaa"></hello-world>
		</teleport>
	</div>
</template>

// 多个teleport
<template>
  <teleport to="#why">
  	<h2>bbb</h2>
	</teleport>
  <teleport to="#why">
  	<hello-world message="aaaa"></hello-world>
	</teleport>
</template>
```

## 插件

插件可以完成的功能没有限制

通常向Vue全局添加一些功能时，会采用插件的模式，它有两种编写方式： 

- 对象类型：一个对象，但是必须包含一个 install 的函数，该函数会在安装插件时执行；
- 函数类型：一个function，这个函数会在安装插件时自动执行；

```typescript
// 对象类型
export default {
	name: "why",
	install(app, options) {
		console.log("插件被安装:", app, options);
		console.log(this.name) ; 
	}
}

// 函数类型
export default function(app, options) {
	console.log("插件被安装:"，app, options);
}

// main.js
import myPluginObject from '...'

const app = create(App)
app.use(myPluginObject) // 通过app.use注册插件
```



# 路由vue-router

## 基本使用

```javascript
// router/index.js

import { createRouter, createWebHistory, createWebHashHistory } from 'vue-router'

// import Home from '...'
import About from '...'
// const About = () => import('...')

const routes = [
  {
    path: '/',
    redirect: '/home'
  },
  {
    path: '/home',
    name: 'home',
    component: () => import('...'),
    meta: {
      age: 18
    }
  },
  {
    path: '/about',
    name: 'about',
    component: About
  },
  {
    path: '/:pathMatch(.*)', // 当路径没有匹配到路由时，显示404报错之类的页面
    // path: '/:pathMatch(.*)*', // 会将输入的路径转化为数组，默认是一个字符串
    // 通过$route.params.pathMatch 获取路径
    component: () => import('...')
  },
  
  // 将匹配以 `/user-` 开头的所有内容
  { path: '/user-:afterUser(.*)', component: ... },
]

const router = new VueRouter({
	routes,
  history: createWebHistory()
  // history: createWebHashHistory()
})

export default router
```

```javascript
// main.js
import { createApp } from 'vue'
import router from "./router"
import App from './App.vue'

const app = createApp(App)
app.use(router).mount('#app')
```

```javascript
// App.vue
<template>
	<div class="app">
    <router-link to="/home">首页</router-link>
    <router-link to="/about">关于</router-link>

    <router-view></router-view>
	</div>
</template>
```

## router-link

属性：

**to：**一个字符串或一个对象

**replace：**replace不会留下history记录，所以指定replace的情况下，后退键返回不能返回到上一个页面中

**active-class：**路由匹配成功时，会给当前的活跃路由设置一个默认名为router-link-active的class，用于设置样式，注意：不是精准匹配的，只要是包含关系，就会被添加，这与exact-active-class不同

**exact-active-class：**精准匹配时，应用于渲染的<a>的class，默认是router-link-exact-active

```javascript
<router-link to="/about" replace active-class="aaa" exact-active-class="bbb">关于</router-link>

<style>
  .aaa {
    color: red;
  }
</style>
```

## 动态路由

```javascript
{
	path: '/user/:userid',
  // path: '/user/:userid/:name' // 可以匹配多个参数
	component: User
}

<router-link :to="'/user/' + userid">用户</router-link>

<h2>{{$route.params.userid}}</h2>

import { useRoute } from 'vue-router'

// 参数值会被设置到 this.$route.params
setup() {
  const route = useRoute()
  console.log(route.params.username)
}
```

## 路由嵌套

```javascript
const routes = [
  {
  	path: '/home',
    component: () => import('...'),
    children: [
      {
        path: '', // 子路由的重定向
        redirect: '/home/message'
      },
      {
        path: 'news', // 不需要写 '/'
        component: () => import('...')
      }
    ]
  }
]
```

```javascript
// Home.vue
<template>
	<div>
		<router-link to="/home/news">新闻</router-link> // to需要写子路由的完整路径
		<router-view></router-view>
	</div>
</template>
```

## 代码跳转路由(编程式导航) 

```javascript
import { useRouter, useRoute } from 'vue-router'

setup() {
  const router = useRouter() // 使用useRouter

  const aaa = () => {
    router.push('/profile') // 字符串
  }
  const bbb = () => {
    router.replace({ // 对象
      path: '/about',
      query: {
        name: 'why',
        age: '18'
      }
    })
  }
  
  return {
    aaa
  }
}

router.go(1)
router.go(-1)
router.forward()


// 命名的路由
router.push({ name: 'user', params: { userId: '123' }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})

/* 如果提供了path，params会被忽略，上述例子中的query并不属于这种情况。取而代之的是下面例子的做法，需要提供路由的name或手写完整的带有参数的path： */
const userId = '123'
router.push({ name: 'user', params: { userId }}) // -> /user/123
router.push({ path: `/user/${userId}` }) // -> /user/123
// 这里的 params 不生效
router.push({ path: '/user', params: { userId }}) // -> /user
```

## 使用v-slot

### router-link

router-link的tag属性被移出，取而代之的是使用v-slot

```typescript
<router-link to="/home">
	<h2>首页</h2>
	<button>哈哈哈</button>
</router-link>

<router-link to="/home">
	<nav-bar title="首页"/> // 组件
</router-link>

<router-link to="/home" v-slot="props"> // 使用参数
	<button>{{props.href}}</button>
</router-link>
/*
	href：解析后的 URL
	route：解析后的规范化的route对象
	navigate：触发导航的函数
	isActive：是否匹配的状态，可用于添加样式
	isExactActive：是否是精准匹配的状态
*/


/*
	上面的写法，内容默认会被包裹在一个a元素中，如果不想要a元素包裹，可以使用custom：
    <router-link to="/home" custom>
      <button>哈哈哈</button>
    </router-link>
  但需要注意的是：没有a元素后是无法跳转的，完全需要自定义，例如监听点击，使用router.push等，但也有更渐变的方法：
  	<router-link to="/home" v-slot="props" custom>
      <button @click="props.navigate">哈哈哈</button>
    </router-link>
*/
```

### router-view

router-view也提供一个插槽，可以用于 transition 和 keep-alive 组件来包裹你的路由组件：

Component：要渲染的组件；

route：解析出的标准化路由对象；

```typescript
<router-view v-slot="props">
	<transition name="why">
		<keep-alive>
			<component :is="props.Component"></component>
		</keep-alive>
	</transition>
</router-view>


.router-link-active {
	color: Ored;
}
.why-enter-from,
.why-leave-to {
	opacity: 0;
}
.why-enter-active,
.why-leave-active {
	transition: opacity 1s ease-in;
}
```



## 导航守卫

建议看文档

#### 全局守卫

**参数**：

- to：进入的Route对象
- from：离开的Route对象

**返回值**： 

- false：取消导航

- 不返回 / undefined：进行默认导航

- 返回一个路由地址：

  - [ ] 一个string类型的路径

  - [ ] 一个对象，包含path、query、params等

**next（不推荐使用）**

- Vue2中通过next函数来决定如何进行跳转，但在Vue3中是通过返回值来控制的，不再推荐使用next函数

```javascript
// 全局前置守卫
router.beforeEach((to, from) => {
	
})

// 全局后置钩子
router.afterEach((to, from) => {
  
})
```

#### 路由独享的守卫

可以在路由配置上直接定义 `beforeEnter` 守卫： 

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from) => { // 与全局前置守卫的参数一样
        
      }
    }
  ]
})
```

 #### 组件内的守卫

可以在路由组件内直接定义以下路由导航守卫：

`beforeRouteEnter`、`beforeRouteUpdate`、`beforeRouteLeave`：

```javascript
import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router'

export default {
  setup() {
    // 与 beforeRouteLeave 相同，无法访问 `this`
    onBeforeRouteLeave((to, from) => {
      const answer = window.confirm(
        'Do you really want to leave? you have unsaved changes!'
      )
      if (!answer) return false // 取消导航并停留在同一页面上
    })

    const userData = ref()

    // 与 beforeRouteUpdate 相同，无法访问 `this`
    onBeforeRouteUpdate(async (to, from) => {
      //仅当 id 更改时才获取用户，例如仅 query 或 hash 值已更改
      if (to.params.id !== from.params.id) {
        userData.value = await fetchUser(to.params.id)
      }
    })
  },
}
```



## 动态添加路由

动态路由主要通过两个函数实现。`router.addRoute()` 和 `router.removeRoute()`。

某些情况下需要动态的来添加路由： 比如根据用户不同的权限，注册不同的路由；这个时候可以使用addRoute

路由的其他方法补充：
 	router.hasRoute()：检查路由是否存在
 	router.getRoutes()：获取一个包含所有路由记录的数组

```typescript
// 动态添加一级路由
const categoryRoute = {
	path: '/category',
	component: () => import('../pages/Category.vue')
}
router.addRoute(categoryRoute)

// 动态添加二级路由
const homeMomentRoute = {
	path: 'moment'
	component: () => import('../pages/HomeMoment.vue')
}
router.addRoute('home', homeMomentRoute)

```

```typescript
// 动态删除路由
/*
	删除路由有以下3种方式：
			方式1：添加一个name相同的路由进行覆盖
			方式2：通过removeRoute方法，传入路由的名称
			方式3：通过addRoute方法的返回值回调
*/

router.addRoute({ path: '/about', name: 'about', component: About })
// 这将会删除之前已经添加的路由，因为它们具有相同的名字且名字必须是唯一的
router.addRoute({ path: '/other', name: 'about', component: Home })

router.addRoute({ path: '/about', name: 'about', component: About })
// 删除路由
router.removeRoute('about')

const removeRoute = router.addRoute(routeRecord)
removeRoute() // 删除路由，如果存在的话
```



# Vuex

## 基本使用

使用store存储数据，使用getters对state中的数据进行操作，使用mutation改变数据（mutation必须是同步函数），使用actions进行异步操作改变数据例如网络请求

```typescript
// store/index.js
import { createStore } from 'vuex'

import { INCREMENT_N } from './mutation-types' // 使用常量

const store = createStore({
  state() {
    return {
      counter: 100,
      age: 18,
      books: [
        {name: "Vuejs", price: 200, count: 3},
        {name: "Node", price: 100, count: 2}
      ],
      discount: 0.6
    }
  },
  mutations: { // mutations用于改变state中保存的值
    increment(state) {
      state.counter++
    },
    [INCREMENT_N](state, n) {
      state.counter + n
    },
    abc(state, payload) {
      console.log(payload.name, payload.age)
    }
  },
  getters: { // getters用于对state中保存的值进行操作
    currentDiscount(state) {
      return state.discount * 0.9
    },
    totalPrice(state, getters) {
      let totalPrice = 0
      for (const book of state.books) {
        totalPrice += book.count * book.price
      }
      return totalPrice * getters.currentDiscount
    },
    aaa(state, getters) { // 本身无法接收传递的参数
      return function(bbb) { // 可以接收参数返回一个函数以作出某些限制
        if (bbb...) {
        	console.log(bbb)
        }
      }
    }
  },
  actions: {
    incrementAction(context, payload) {
      // 异步操作
      console.log(payload.messagee) // 传递的参数
      return new Promise((resolve, reject) => { // 可以返回promise
        axios.get("...").then(res => {
        	context.commit("increment", res.data)
          resolve({name: "why", age: 18})
      	}).catch(err => {
          reject(err)
        })
      })
      
    }
  }
})

export default store
```

```typescript
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import store from './store'

createApp(App).use(store).mount('#app')
```

```typescript
// 在组件中使用
<template>
  // state
  <h2>{{ $store.state.counter }}</h2> // 不使用mapState
	<h2>{{ sCounter }}</h2> // 使用别名
	<button @click="increment">+1</button>
	<button @click="incrementN(n)">+n</button>
	
	// getters
	<h2>{{ $store.getters.totalPrice }}</h2> // 会将函数自动执行以获取return的返回值
	<h2>{{ $store.getters.aaa(20) }}</h2> // 如果想要传递参数
	<h2>{{ sCurrentDiscount }}</h2>

	// mutations
	<button @click="increment">+1</button>
	<button @click="increment_n(123)">+123</button>
	<button @click="def">+1</button> // 使用别名
	
	// actions
	<button @click="doIncrementAction">+1</button>
</template>

<script>
  import { useStore, mapState, mapGetters, mapMutations, mapActions } from 'vuex'
	import { computed, onMounted } from 'vue'

	import useState from ',,,' // 导入封装的mapState
	import useGetters from ',,,' // 导入封装的useGetters

	import { INCREMENT_N } from './mutation-types' // 使用常量

  export default {
    setup() {
      const store = useStore()
      
      const state = useState()
      
      /* state */
      const sCounter = computed(() => store.state.counter) // 不使用mapState，纯computed的简化
      const storeStateFns = mapState(["counter", "age"]) // 不需要别名时，可以使用数组
      // 建议：使用自己编写的mapState辅助函数，见下方的 mapState的封装 篇
      
      /* getters */
      const sCurrentDiscount = computed(() => store.getters.currentDiscount)
      // 建议：使用自己编写的mapGetters辅助函数，见下方的 mapGetters的封装 篇
      
      /* mutations */
      // 使用mapMutations 注意：mutations在return中可以直接进行...解构
      const storeMutations = mapMutations(['increment', INCREMENT_N, 'abc'])
      const storeMutations = mapMutations({
        increment_n: INCREMENT_N
        def: 'abc' // 别名 def是别名，'abc'是真实的值
      })
      // 不使用mapMutations
      increment() {
        store.commit("increment")
      }
      [INCREMENT_N]() {
        store.commit("incrementN", 123)
      }
      abc() {
        store.commit("incrementN", {
        	name: "why",
          age: 18
        })
      }
      abc() { // 另一种提交mutation的方式
        store.commit({
          type: INCREMENT_N // 使用常量
        	name: "why",
          age: 18
        })
      }
      
      /* actions */
      onMounted(() => {
        doIncrementAction() {
          store.dispatch("incrementAction", {message: "我是参数"})
        }.then(res => {
          console.log(res)
        }).catch(err => {
          console.log(err)
        })
      })
      doIncrementAction() { // 另一种提交action的方式
        type: "incrementAction",
        message: "我是参数"
      }
      const actions1 = mapActions(["decrement"]) // mapActions不需要封装
      const actions2 = mapActions({
        add: "increment",
        sub: "decrement"
      })
      
      return() {
        sCounter
        sCurrentDiscount
        
        ...storeMutations // mutations可以直接进行解构
        ...actions1
      }
    }
  }
</script>
```

## map函数的封装

### mapState的封装

```typescript
// 默认情况下，Vuex并没有提供非常方便的使用mapState的方式，这里我们封装一个hook：
// useState.js
import { useStore, mapState } from 'vuex'
import { computed } from 'vue'

export function useState(mapper) {
  // 拿到stote
  const store = useStore()
  
  // 获取到对应的对象的functions: {name: function, age: function}
  const stateFns = mapState(mapper)
  
  // 对数据进行转换
  const state = {}
  Object.keys(stateFns).forEach(fnKey => {
  	state[fnKey] = computed(stateFns[fnKey].bind({$store: store}))
    // mapState 本质也是 this.$store.state.xxx
    // 上面的stateFns[fnKey]是没有this的，因此就没办法获取this.$store，所以要bindthis
  })
  return state
}
```

```typescript
// 在组件中使用
<template>
	<h2>{{ sCounter }}</h2>
	<h2>{{ sAge }}</h2>
</template>

<script>
	import { useState } from '../hooks/useState'
	
	export default {
    setup() {
      // 使用
      const stoteState = useState(["counter", "age"]) 
      const storeState = useState({
        sCounter: state => state.counter,
        sAge: state => state.age
      })
      
      return {
        ...stoteState // 解构
      }
    }
  }
</script>
```

### mapGetters的封装

```typescript
// 默认情况下，Vuex并没有提供非常方便的使用mapGetters的方式，这里我们封装一个hook：
// useGetters.js
import { useStore, mapGetters } from 'vuex'
import { computed } from 'vue'

export function useGetters(mapper) {
  // 拿到stote
  const store = useStore()
  
  // 获取到对应的对象的functions: {name: function, age: function}
  const stateFns = mapGetters(mapper)
  
  // 对数据进行转换
  const state = {}
  Object.keys(stateFns).forEach(fnKey => {
  	state[fnKey] = computed(stateFns[fnKey].bind({$store: store}))
    // mapState 本质也是 this.$store.state.xxx
    // 上面的stateFns[fnKey]是没有this的，因此就没办法获取this.$store，所以要bindthis
  })
  return state
}
```

```typescript
// 在组件中使用
<template>
	<h2>{{ sCounter }}</h2>
	<h2>{{ sAge }}</h2>
</template>

<script>
	import { useGetters } from '../hooks/useGetters'
	
	export default {
    setup() {
      // 使用
      const stoteGetters = mapGetters(["currentDiscount", "totalPrice"]) 
      const storeGetters = mapGetters({
        sCurrentDiscount: state => state.counter,
        sTotalPrice: state => state.age
      })
      
      return {
        ...stoteGetters // 解构
      }
    }
  }
</script>
```

### 整合

mapState的封装代码和mapGetters的封装代码有大量的重复代码，因此两个函数可以进行整合

```typescript
import { useStore, mapState, mapGetters } from 'vuex'

function useMapper(mapper, mapFn) {
  // 拿到stote
  const store = useStore()
  
  // 获取到对应的对象的functions: {name: function, age: function}
  const stateFns = mapFn(mapper)
  
  // 对数据进行转换
  const state = {}
  Object.keys(stateFns).forEach(fnKey => {
  	state[fnKey] = computed(stateFns[fnKey].bind({$store: store}))
    // mapState 本质也是 this.$store.state.xxx
    // 上面的stateFns[fnKey]是没有this的，因此就没办法获取this.$store，所以要bindthis
  })
  return state
}

export function useState(mapper) {
  return useMapper(mapper, mapState)
}

export function useGetters(mapper) {
  return useMapper(mapper, mapGetters)
}
```

```typescript
// 使用 useState.js
import { mapState, createNamespacedHelpers } from 'vuex'
import { useMapper } from './useMapper'

export function useGetters(moduleName, mapper) {
  let mapperFn = mapState
  if (typeof moduleName === 'string' && moduleName.length>0) {
    mapperFn = createNamespacedHelpers(moduleName).mapState
  } else {
    mapper = moduleName
  }
  
  return useMapper(mapper, mapperFn)
}
```

```typescript
// 使用 useGetters.js
import { mapGetters, createNamespacedHelpers } from 'vuex'
import { useMapper } from './useMapper'

export function useGetters(moduleName, mapper) {
  let mapperFn = mapState
  if (typeof moduleName === 'string' && moduleName.length>0) {
    mapperFn = createNamespacedHelpers(moduleName).mapGetters
  } else {
    mapper = moduleName
  }
  
  return useMapper(mapper, mapperFn)
}
```

### mapMutations

```typescript
const mutations = mapMutations(['increment', 'decrement', ADD_ NUMBER]) // 使用常量
const mutations2 = mapMutations({
	addNumber: ADD_ NUMBER // 使用常量
})
```

## 定义常量

```typescript
// 定义常量
// mutation-types.js
export const ADD_NUMBER = 'add_number'

// 定义mutation
[ADD_NUMBER](state, payload) {
  state.counter += payload.count
}

// 提交mutation
$store.commit({
	type: ADD_NUMBER,
  count: 100
})

/* 也可以导出一个对象，在对象里定义常量 */
```

## module

### 基本使用

Vuex允许将 store 分割成模块（module），每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块。

```typescript
// store/index.js
import { createStore } from 'vuex'
import homeModule from './modules/home'
import homeModule from './modules/user'
const store = createStore({
  state() { return { rootCounter: 10 } },
  actions: {},
  modules: {
    homeModule,
    // home: homeModule
    homeModule
  }
})

// store/modules/home.js
const homeModule = {
  state() { return { homeCounter: 10 } },
  mutations: {},
  actions: {},
  getters: {}
}
export default homeModule

// store/modules/user.js
const userModule = {
  state() { return { userCounter: 10 } },
  mutations: {},
  actions: {},
  getters: {}
}
export default userModule

// 使用
<h2>{{ $store.state.rootCounter }}</h2>
<h2>{{ $store.state.homeModule.homeCounter }}</h2>
<h2>{{ $store.state.userModule.userCounter }}</h2>
```

```typescript
// 对于模块内部的 mutation 和 getter，接收的第一个参数是模块的局部状态对象
mutations: {
	changeName(state) {
		state.name = "why"
	}
},
getters: {
	info(state, getters, rootState) {
		return `name: ${state.name} age:${state.age} height:${state.height}`
	}
}
actions: {
	changeNameAction({state, commit, rootState}) {
		commit("changeName", "kobe")
  }
}
```

### 命名空间

当module没有使用命名空间时，getters等会被合并，即直接通过$store.getters.something获取即可，并且如果根store和module中的getters、mutations、actions如果重名，则提交时会全部调用

```typescript
const homeModule = {
  namespaced: true, // 使用命名空间
  state() {},
  getters: {},
  ...
}
  
// 使用
<h2>{{ $store.getters["homeModule/something"] }}</h2> // 模块名称与在根store中映射配置有关

// 派发action
store.commit("increment") // 默认只会派发根store下的action
store.commit("home/increment") // 命名空间

// 模块内部的mutation
// 对于模块内部的mutation，会有3个参数，第3个参数表示是否是根store下的mutation
commit("increment", payload, {root: true})
// 模块内部的getter
// 对于模块内部的 mutation 和 getter，接收的第一个参数是模块的局部状态对象
theGetter(state, getters, rootState, rootGetters) {
  
}
// 模块内部的action
// 下面的参数是解构的，实际上只有一个参数
theAction({commit, dispatch, state, rootState, getters, rootGetters}) {
  // 会有3个参数，第3个参数表示是否是根store下的action
  dispatch("incrementAction", payload, {root: true})
}
```

### 使用辅助函数

```typescript
// 推荐第三种
// 方式一：通过完整的模块空间名称来查找
computed: {
  ...mapState({
    a: state => state.some.nested.moldule.a
    b: state => state.some.nested.moldule.b
  })
},
methods: {
  ...mapActions([
    "some/nested/module/foo", // -> this['some/nested/module/foo']()
    "some/nested/module/bar", // -> this['some/nested/module/bar']()
  ])  
}
```

```typescript
// 推荐第三种
// 方式二：第一个参数传入模块空间名称，后面写上要使用的属性

// 使用自己封装的map函数
import { useState, useGetters } from '../hooks/index' // 见目录Vuex/map函数的封装

setup() {
  // {homeCounter: function}
  const state = useState("home", ["homeCounter"])
  const getters = useGetters("home", ["doubleHomeCounter"])
  
  const mutations = mapMutations(["increment"])
  const actions = mapActions(["incrementAction"])
	
  return {
    ...state,
    ...getters,
    ...mutations,
    ...actions
  }
}
```

```typescript
// 常用
// 方式三：通过 createNamespacedHelpers 生成一个模块的辅助函数
const { mapState, mapActions } = createNamespacedHelpers('some/nested/module')

export default {
  computed: {
    // 在 'some/nested/module' 中查找
    ...mapState({
      a: state => state.a,
      b: state => state.b,
    })
  },
  methods: {
    // 在 'some/nested/module' 中查找
    ...mapActions([
      'foo',
      'bar',
    ])
  }
}
```



## Vuex和TypeScript的结合

Vuex本身对TypeScript的支持不是很好，例如进行类型限制，因此，可以使用Pinia等第三方库，也可以使用以下的方案：

```typescript
// store/login/types.ts
export interface ILoginState {
  token: string
  userInfo: any
  userMenus: any
}
```

```typescript
// store/login/login.ts
import { Module } from 'vuex'

import {
  accountLoginRequest,
  requestUserInfoById,
  requestUserMenusByRoleId
} from '@/service/login/login'
import localCache from '@/utils/cache'
import router from '@/router'

import { IAccount } from '@/service/login/type'
import { ILoginState } from './types'
import { IRootState } from '../types'

const loginModule: Module<ILoginState, IRootState> = {
  namespaced: true,
  state() {
    return {
      token: '',
      userInfo: {},
      userMenus: []
    }
  },
  getters: {},
  mutations: {
    changeToken(state, token: string) {
      state.token = token
    },
    changeUserInfo(state, userInfo: any) {
      state.userInfo = userInfo
    },
    changeUserMenus(state, userMenus: any) {
      state.userMenus = userMenus
    }
  },
  actions: {
    async accountLoginAction({ commit }, payload: IAccount) {
      // 1.实现登录逻辑
      const loginResult = await accountLoginRequest(payload)
      const { id, token } = loginResult.data
      commit('changeToken', token)
      localCache.setCache('token', token)

      // 2.请求用户信息
      const userInfoResult = await requestUserInfoById(id)
      const userInfo = userInfoResult.data
      commit('changeUserInfo', userInfo)
      localCache.setCache('userInfo', userInfo)

      // 3.请求用户菜单
      const userMenusResult = await requestUserMenusByRoleId(userInfo.role.id)
      const userMenus = userMenusResult.data
      commit('changeUserMenus', userMenus)
      localCache.setCache('userMenus', userMenus)

      // 4.跳到首页
      router.push('/main')
    },
    loadLocalLogin({ commit }) {
      const token = localCache.getCache('token')
      if (token) {
        commit('changeToken', token)
      }
      const userInfo = localCache.getCache('userInfo')
      if (userInfo) {
        commit('changeUserInfo', userInfo)
      }
      const userMenus = localCache.getCache('userMenus')
      if (userMenus) {
        commit('changeUserMenus', userMenus)
      }
    }
  }
}

export default loginModule
```

```typescript
// store/types.ts
import { ILoginState } from './login/types.ts'

export interface IRootState {
  name: string
  age: number
}

export interface IRootWithModule {
  login: ILoginState,
  // ...其他模块
}

export type IStoreType = IRootState & IRootWithModule
```

```typescript
// 使用自己定义的store
// store/index.ts
import { createStore, Store, useStore as useVuexStore } from 'vuex'

import login from './login/login.ts' // login module store

import { IRootState, IStoreType } from './types.ts'

const store = createStore<IRootState>({
  state() {
    return {
      name: 'coderwhy',
      age: 18
    }
  },
  mutations: {},
  getters: {},
  actions: {},
  modules: {
    login // login module store
  }
})

export function useStore(): Store<IStoreType> {
  return useVuexStore()
}

export default store
```

# 实现mini Vue

**功能：**

- 功能一：h函数，用于返回一个VNode对象；
- 功能二：mount函数，用于将VNode挂载到DOM上；
- 功能三：patch函数，用于对两个VNode进行对比，决定如何处理新的VNode；

## 渲染系统模块

```js
// 1.通过h函数来创建一个vnode
const vnode = h('div', {class: "aaa"}, {
  h("h2", null, "当前计数：100"),
  h("button", {onClick: function() {}}, "+1")
})
// 2.通过mount函数，将vnode挂载到div#app上
mount(vnode, document.querySelector("#app"))

// 3.创建新的vnode，用于演示diff算法
const vnode1 = h('div', {class: "bbb"}, "哈哈哈")

patch(vnode, vnode1) // 传入后进行diff算法，然后更新界面

// h函数用于创建虚拟DOM节点
const h = (tag, props, children) => {
  // vnode -> JavaScript对象 -> {}
  // vnode本质上就是一个js对象
  return {
    tag,
    props,
    children
  }
}

/* ---------------------------------------- */

// mount函数用于将虚拟DOM节点挂载到真实DOM上
const mount = (vnode, container) => {
  // vnode -> element

  // 1.创建出真实的原生节点，并且在vnode上保留el
  const el = vnode.el = document.createElement(vnode.tag)

  // 2.处理props
  if (vnode.props) {
    for (const key in vnode.props) {
      const value = vnode.props[key]
      if (key.startsWith("on")) { // 如果属性是事件监听器（以'on'开头）
        // 对事件监听的判断
        el.addEventListener(key.slice(2).toLowerCase(), value) // 一般是使用的驼峰绑定事件，所以需要转换为小写
      } else {
        // 设置DOM元素的属性
        el.setAttribute(key, value)
      }
    }
  }

  // 3.处理children  ，处理虚拟节点的子节点
  if (vnode.children) {
    if (typeof vnode.children === 'string') { // 如果子节点是字符串
      el.textContent = vnode.children // 直接将字符串作为文本内容添加到DOM元素中
    } else { // 如果子节点是数组
      // 递归地挂载每一个子虚拟节点
      vnode.children.forEach(item => {
        mount(item, el) // 递归调用
      })
    }
    // 如果是对象...（这里不再判断）
    // 如果是其他的...
  }

  // 4.将创建的DOM元素添加到容器元素中
  container.appendChild(el)
}


const patch = (n1, n2) => {
  if (n1.tag !== n2.tag) { // 判断两个节点的类型是否一样，如div和span
    // 如果类型都不一样，则直接移除旧的添加新的
    const n1ElParent = n1.el.parentElement // 先获取父元素
    n1ElParent.removeChild(n1.el) // 移除n1
    mount(n2, n1ElParent) // 直接挂载n2
  } else { // 类型相同
    // 1.去除element对象，并且在n2中进行保存
    const el = n2.el = n1.el // 这里是对象的引用，因此会共同修改

    // 2.处理props
    const oldProps = n1.props || {}
    const newProps = n2.props || {}
    // 2.1.获取所有的newProps添加到el
    for (const key in newProps) {
      const oldValue = oldProps[key]
      const newValue = newProps[key]
      if (newValue !== oldValue) { // 只有当属性或事件不相同时，才需要设置
        if (key.startsWith("on")) { // 如果属性是事件监听器（以'on'开头）
          // 对事件监听的判断
          el.addEventListener(key.slice(2).toLowerCase(), newValue) // 一般是使用的驼峰绑定事件，所以需要转换为小写
        } else {
          // 设置DOM元素的属性
          el.setAttribute(key, newValue)
        }
      }
    }

    // 2.2.删除旧的props
    for (const key in oldProps) {
      if (!(key in newProps)) { // 如果旧的props不在新的props中，则删除旧的
        if (key.startsWith("on")) { // 如果属性是事件监听器（以'on'开头）
          // 对事件监听的判断
          const value = oldProps[key]
          el.removeEventListener(key.slice(2).toLowerCase(), value) // 一般是使用的驼峰绑定事件，所以需要转换为小写
        } else {
          // 设置DOM元素的属性
          el.removeAttribute(key)
        }
      }
    }

    // 3.处理children
    const oldChildren = n1.children || []
    const newChildren = n2.children || []

    if (typeof newChildren === 'string') { // 是字符串，直接进行替换
      el.innerHTML = newChildren
    } else { // 数组
      if (typeof oldChildren === 'string') {
        // 旧的children是字符串，新的children是数组
        el.innerHTML = ""
        newChildren.forEach(item => {
          mount(item, el) // 挂载
        })
      } else {
        // 旧的children和新的children 都是数组
        // oldChildren: [v1, v2, v3]
        // newChildren: [v1, v4, v5, v6]
        // 这里不考虑使用key进行比较

        // 以较短的数组进行比较和替换，较长的数组的剩余部分则直接进行添加/删除
        const commonLength = Math.min(oldChildren.length, newChildren.length)
        for (let i=0; i<commonLength; i++) {
          patch(oldChildren[i], newChildren[i])
        }

        // 判断是较长的数组的剩余部分是直接进行添加还是删除
        if (newChildren.length > oldChildren.length) {
          // 对剩余的节点直接进行挂载
          newChildren.slice(oldChildren.length).forEach(item => {
            mount(item, el)
          })
        }
        if (newChildren.length < oldChildren.length) {
          // 对剩余的节点直接进行删除
          oldChildren.slice(newChildren.length).forEach(item => {
            el.removeChild(item.el)
          })
        }
      }
    }
  }
}
```

## 可响应式系统模块

```js
// Dep类用于管理一个依赖，能够收集和触发响应式效果
class Dep {
  constructor() {
    thistore.subscribers = new Set() // 使用Set存储订阅者，保证唯一性
  }

  depend() { // 添加一个依赖
    if (activeEffect) {
      this.subscribers.add(activeEffect)
    }
  }

  notify() { // 通知所有订阅者
    this.subscribers.forEach(effect => {
      effect() // 执行每一个effect函数
    });
  }
}

// 当前激活的effect函数
let activeEffect = null

// watchEffect用于注册一个响应式效果
// 注册副作用
// 当调用watchEffect时，传入的函数（副作用函数）被执行一次。
// 在执行过程中，任何被这个副作用函数访问的响应式数据都会将该副作用函数收集为依赖。
function watchEffect(effect) {
  activeEffect = effect // 设置当前激活的effect
  effect() // 主动先执行一次，触发数据劫持中的get方法，进行依赖收集
  activeEffect = null // 重置activeEffect
}

// WeakMap的key是一个对象而不是字符串，是弱引用，当之后target被赋值为null时，就会被垃圾回收机制回收，对应的WeakMap中的这个对象对应的值也会被回收，而不需要手动清理，相当于可以通过管理外面的作为键值对的键的对象，来管理WeakMap中的键值对
const targetMap = new WeakMap()

// 根据目标对象和属性获取对应的依赖对象
function getDep(target, key) {
  // 1.根据对象(target)取出对应的Map对象
  let depsMap = targetMap.get(target)
  if (!depsMap) {
    depsMap = new Map()
    targetMap.set(target, depsMap)
  }

  // 2.取出具体的dep对象
  let dep = depsMap.get(key)
  if (!dep) {
    dep = new Dep()
    depsMap.set(key, dep)
  }
  return dep
}

// vue2的方式
function reactive(raw) { // 数据劫持，当监听的对象发生改变时
  Object.keys(raw).forEach(key => {
    const dep = getDep(raw, dep)
    const value = raw[key]

    Object.defineProperty(raw, key, {
      get() {
        // 在watchEffect中主动先执行一次函数时，就会触发这里的get()，就会添加进依赖中
        dep.depend() // 自动收集响应式依赖
        return value
      },
      set(newValue) {
        if (value !== newValue) {
          value = newValue
          dep.notify() // 当值发生变化时，自动进行通知更新操作
        }
      }
    })
  })
  return raw
}
// vue3的方式
function reactive(raw) { // 数据劫持，当监听的对象发生改变时
  return new Proxy(raw, {
    // 当属性被访问时（get操作），会触发依赖收集，即将当前的副作用函数（如果有）添加到该属性的依赖列表中。
    get(target, key, receiver) {
      const dep = get(target, key)
      dep.depend() // 收集依赖
      return target[key]
    },
    // 当属性被修改时（set操作），会触发依赖通知，即执行该属性的所有依赖函数。
    set(target, key, newValue) {
      const dep = getDep(target, key)
      target[key] = newValue
      dep.notify()
    }
  })
}

const info = reactive({counter: 100, name: "why"})
const foo = reactive({height: 1.88})

watchEffect(function() {
  console.log("effect1:", info.counter * 2, info.name)
})
watchEffect(function() {
  console.log("effect2:", info.counter * info.counter)
})
watchEffect(function() {
  console.log("effect3:", info.counter + 10, info.name)
})
watchEffect(function() {
  console.log("effect4:", foo.height)
})

info.counter++
info.name = "lilei"
```



## 应用程序入口模块

```js
function createApp(rootComponent) {
  return {
    mount(selector) {
      const container = document.querySelector(selector)
      let isMounted = false
      let oldVNode = null

      watchEffect(() => {
        if (!isMounted) {
          oldVNode = rootComponent.render()
          this.mount(rootComponent.render(), container)
          isMounted = true
        } else {
          const newVNode = rootComponent.render()
          patch(oldVNode, newVNode)
          oldVNode = newVNode
        }
      })
    }
  }
}
```

# Vue3面试题

## Setup

1. **`setup`函数是什么时候执行的？**
   - 在组件实例创建时同步执行的，发生在`beforeCreate`之后、`created`钩子之前。在`setup`函数中，可以定义响应式的数据和函数，它们将用于组件的其余部分
2. **`setup`函数与Vue2的`data`和`methods`有什么不同？**
   - `setup`提供了一个更加函数式的方式来声明组件逻辑，这样有利于代码的复用和组织，特别是在复杂组件中
3. **在`setup`中如何访问组件的props或者发出事件？**
   - 在`setup`中，可以使用`defineProps`和`defineEmits`来声明和使用props以及发出事件
4. **`setup`函数中可以使用生命周期钩子吗？**
   - 是的，`setup`函数中可以使用生命周期钩子。但是，你需要使用Vue 3中特有的生命周期钩子函数，这些钩子函数以`on`为前缀，比如`onMounted`、`onUpdated`等。这些钩子必须在`setup`函数中同步注册，因为它们依赖于内部的全局状态来定位当前组件实例
5. **Vue3中如何处理计算属性和侦听器？**
   - 可以使用`computed`来定义计算属性，使用`watch`和`watchEffect`来定义侦听器。`watch`用于观察一个或多个响应式引用或函数的返回值，并在值变化时运行一个回调。`watchEffect`则会立即执行一次，并在其依赖的响应式状态变化时重新执行

### 参数

```vue
// setup
<script setup>
const props = defineProps(['foo']) // 数组形式
defineProps({ // 对象形式
  title: String,
  likes: Number
})

console.log(props.foo)
</script>
```

```vue
// 没有使用setup
export default {
  props: ['foo'],
  setup(props) {
    // setup() 接收 props 作为第一个参数
    console.log(props.foo)
  }
}
```

```js
// 校验
defineProps({
  // 基础类型检查
  // （给出 `null` 和 `undefined` 值则会跳过任何类型检查）
  propA: Number,
  propB: [String, Number], // 多种可能的类型
  propC: {
    type: String,
    required: true // 必传，且为 String 类型
  },
  propD: {
    type: Number,
    default: 100 // Number 类型的默认值
  },
  propE: {
    type: Object, // 对象类型的默认值
    default(rawProps) { // 接收组件所接收到的原始 prop 作为参数
      return { message: 'hello' } // 对象或数组的默认值，必须从一个工厂函数返回
    }
  },
  propF: {
    validator(value) { // 自定义类型校验函数
      return ['success', 'warning', 'danger'].includes(value)
    }
  },
  propG: {
    type: Function, // 函数类型的默认值
    default() {
      return 'Default function' // 不像对象或数组的默认，这不是一个工厂函数，这会是一个用来作为默认值的函数
    }
  },
  author: Person // 也可以是自定义的类或构造函数，会通过instanceof Person来校验的值是否是 Person 类的一个实例
})
```

注意：`defineProps()` 的参数**不可以访问 `<script setup>` 中定义的变量**，因为在编译时整个表达式都会被移到外部的函数中。

## ref && reactive

二者的区别：

1. **数据类型：**
   - `ref()`通常用于基本数据类型（如字符串、数字、布尔值），但也可以包装对象。
   - `reactive()`则是为了创建响应式的对象或数组。
2. **模板引用：**
   - 使用`ref()`时，在模板中引用时需要使用`.value`来访问实际的值。
   - `reactive()`返回的对象或数组可以直接被模板引用，不需要`.value`。
3. **嵌套对象：**
   - `ref()`在嵌套对象中不会使内部对象自动变为响应式，除非它们也被`ref()`或`reactive()`包装。
   - `reactive()`会深层次地使嵌套的对象变为响应式。
4. **响应式解包：**
   - `ref()`总是返回一个带有`.value`属性的响应式引用对象。
   - `reactive()`则返回原始对象的响应式代理。

```js
const obj = ref({
  nested: { count: 0 }
})
obj.value.nested.count++ // 需要.value

const state = reactive({ count: 0 })
state.count++ // 不需要.value
```



- **`ref`:** 当你使用`ref`来包装一个值时，Vue实际上创建了一个对象，这个对象有一个`value`属性。当你访问或修改这个`value`属性时，Vue能够通过getter和setter来追踪这些改变，并自动更新DOM。即使你用`ref`包装一个对象，访问和修改也必须通过`.value`，因为这是`ref`对象的约定。
- **`reactive`:** 另一方面，当你使用`reactive`来包装一个对象时，Vue使用`Proxy`来创建这个对象的代理。`Proxy`可以拦截对对象属性的所有操作，包括属性读取和写入。这样，当你操作`reactive`返回的代理对象时，你直接与原始对象的属性进行交互，无需`.value`属性。因为`Proxy`可以捕获对对象的任何修改，Vue能够在没有显式属性（如`.value`）的情况下追踪这些修改。

`ref`在Vue中被设计为一个引用类型，因为Vue的响应式系统需要一个通用的包装器来处理基础类型的值，比如字符串和数字。基础类型的值是不可变的，当你尝试修改一个基础类型的值时，实际上是创建了一个新的值。Vue需要一个容器来持有这个值，这样就能够追踪其变化。

如果`ref`不使用`.value`，就无法区分是要操作响应式引用本身还是其包裹的内部值。`reactive`不需要`.value`因为它作用于对象和数组，这些是引用类型，在它们身上可以直接修改属性或元素而不会丢失响应性。`ref`需要`.value`来明确你是想获取或设置包装的值，而不是操作响应式引用对象本身。

```js
// 注意，reactive 返回的是一个原始对象的 Proxy，它和原始对象是不相等的
const raw = {}
const proxy = reactive(raw)
console.log(proxy === raw) // false  代理对象和原始对象不是全等的
```



==reactive() 的局限==

1. **有限的值类型**：只能用于对象类型 (对象、数组和如 `Map`、`Set` 这样的集合类型)。不能持有如 `string`、`number` 或 `boolean` 这样的原始类型。

2. **不能替换整个对象**：Vue 的响应式跟踪是通过属性访问实现的，因此必须始终保持对响应式对象的相同引用。意味不能轻易地“替换”响应式对象，否则会与第一个引用的响应性连接将丢失：

   ```js
   let state = reactive({ count: 0 })
   
   // 上面的 ({ count: 0 }) 引用将不再被追踪
   state = reactive({ count: 1 })
   ```

3. **不能解构**：将响应式对象的原始类型属性解构为本地变量时，或将该属性传递给函数，将丢失响应性连接：

   ```js
   const state = reactive({ count: 0 })
   
   // 当解构时，count 已经与 state.count 断开连接
   let { count } = state
   
   // 该函数接收到的是一个普通的数字
   // 并且无法追踪 state.count 的变化，必须传入整个对象以保持响应性
   callSomeFunction(state.count)
   ```

由于这些限制，建议使用 `ref()` 作为声明响应式状态的主要 API。（ref包裹的对象在解构时也会失去响应式，因为当ref传入对象时，实际是调用的reactive）



==面试题==

1. **面试题：请解释Vue 3中`ref`和`reactive`的区别。**

   **答案：**`ref`用于定义一个响应式的引用，适用于基本数据类型和对象，访问或修改它的值需要使用`.value`属性。`reactive`则用于创建一个响应式的对象或数组，不需要使用`.value`属性，直接修改对象的属性即可实现响应式。

2. **面试题：为什么在模板中直接使用`ref`时不需要`.value`？**

   **答案：**Vue 的模板编译器会自动解包`ref`，这意味着在模板中使用时，编译器会生成代码来访问`.value`属性。这样，开发者就可以像使用普通变量一样使用`ref`。

3. **面试题：如果我有一个响应式对象，我应该使用`ref`还是`reactive`？为什么？**

   **答案：**如果是复杂的对象或数组），使用`reactive`，可以让直接操作对象的属性而保持响应性。如果需要在多个地方保持对原始对象的引用或者需要响应式的基本数据类型，`ref`更合适。

4. **面试题：如何将一个`reactive`对象转换为`ref`？**

   **答案：**你可以使用Vue的`toRef`函数将`reactive`对象的某个属性转换为`ref`。这样可以保持对原始`reactive`对象属性的响应性连接。例如：`const nameRef = toRef(reactiveObject, 'name')`。



关于自动解包：在模板渲染上下文中，只有顶级的 ref 属性才会被解包。

```vue
<template>
  <!-- 直接访问nestedObject的属性，不需要.value -->
  <div>{{ nestedObject.text }}</div>
  
  <!-- 访问nestedObject内部的nestedRef时，需要使用.value -->
  <div>{{ nestedObject.nestedRef.value }}</div>
</template>

<script setup>
import { reactive, ref } from 'vue'

const nestedObject = reactive({
  text: 'Hello', // 普通属性
  nestedRef: ref('I am a nested ref') // ref属性
})
</script>
```



## toRef && toRefs

toRef：将一个对象属性转换为ref响应式数据

```js
import { reactive, toRef } from 'vue';

// 创建响应式对象
const state = reactive({
  name: 'Vue',
  age: 3
});

// 直接解构，会失去响应性
const { age } = state;

// 使用 toRef 保持响应性
const ageRef = toRef(state, 'age');

// 现在 ageRef 是一个响应式引用
console.log(ageRef.value); // 输出 3

// 修改 state 的 age 属性
state.age = 4;

// ageRef 会同步更新，因为它是响应式的
console.log(ageRef.value); // 输出 4
```

toRefs：`toRefs` 用于将 `reactive` 对象中的所有属性转换为响应式引用对象（`ref`），这样即使在解构或扩展操作后，每个属性也能保持其响应性。使用场景主要是当你需要在解构或传递多个响应式属性时保持它们的响应性。

```js
import { reactive, toRefs } from 'vue';

const state = reactive({
  name: 'Vue',
  age: 3
});

// 转换为响应式引用对象
const stateRefs = toRefs(state);

// 解构 stateRefs，每个属性都保持响应性
const { name, age } = stateRefs;

// 修改原始响应式对象的属性
state.name = 'Vue 3';
state.age = 4;

// name 和 age 引用将同步更新
console.log(name.value); // 输出 'Vue 3'
console.log(age.value); // 输出 4
```





## 计算属性

1. **在Vue中通常用于什么？**

   用于基于现有数据派生新数据。与方法不同，计算属性是基于它们的响应式依赖进行缓存的。只有当依赖项发生变化时，计算属性才会重新计算。

2. **计算属性和方法有什么不同？**

   计算属性是缓存的，只有当它的依赖发生变化时才会重新计算。而方法会在每次重新渲染时调用。

3. **计算属性能否接收参数？**

   不可以。计算属性是基于它们的依赖自动计算值的，它们不接受参数。

4. **如何在计算属性中监听多个值的变化？**

   你可以在计算属性中返回一个基于多个组件状态的新值。Vue的响应性系统会追踪这些状态，一旦其中一个依赖变化，计算属性就会重新计算。

5. **如何在Vue 3的`setup`函数中使用计算属性？**

   ```vue
   <script setup>
   import { reactive, computed } from 'vue'
     
   const author = reactive({
     name: 'John Doe',
     books: [
       'Vue 2 - Advanced Guide',
       'Vue 3 - Basic Guide',
     ]
   })
   
   const publishedBooksMessage = computed(() => {
     return author.books.length > 0 ? 'Yes' : 'No'
   })
   </script>
   ```



```js
// 如果一个计算属性中没有用到响应式依赖，那么这个计算属性永远不会更新
const now = computed(() => Date.now())
```

## watch && watchEffect

`watchEffect`不提供旧值和新值。它主要是用来自动追踪其函数内部引用的响应式状态，并在这些状态改变时执行副作用（side effect）。如果你需要基于旧值与新值来决定你的业务逻辑，那么你应该使用`watch`。



## ref 模板引用

使用了 `<script setup>` 的组件是**默认私有**的：一个父组件无法访问到一个使用了 `<script setup>` 的子组件中的任何东西，除非子组件在其中通过 `defineExpose` 宏显式暴露

```vue
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

// 像 defineExpose 这样的编译器宏不需要导入
defineExpose({
  a,
  b
})
</script>
```







## nextTick







## 指令

### Vue指令

v-text

v-html

v-show

v-if

v-else

v-else-if

v-for

v-on

v-bind

v-model

v-slot

v-pre

v-once

v-memo

v-cloak

### 自定义指令





















## Vite







## 其他

- 当你修改了响应式状态时，DOM 的自动更新不是同步的。会在“next tick”更新周期中缓冲所有状态的修改，以确保不管进行了多少次状态修改，每个组件都只会被更新一次。

  ```js
  import { nextTick } from 'vue'
  
  async function increment() {
    count.value++
    await nextTick() // 要等待 DOM 更新完成后再执行额外的代码，可以使用 nextTick() 全局 API
    // 现在 DOM 已经更新了
  }
  ```

- 虽然`reactive()`是专门用来创建响应式对象的，但是`ref()`也可以用来包裹一个对象，使该对象成为响应式的。当你用`ref()`来包裹一个对象时，Vue会在内部通过`reactive()`来处理这个对象，使其成为响应式的。

- vue3为什么用Proxy代替defineProperty

  `Proxy` 可以拦截并监听到对象的所有操作，包括属性的添加和删除，而 `defineProperty` 仅能作用于单个属性，并不能监听属性的添加和删除。此外，`Proxy` 在处理复杂或大型对象时性能消耗更低。

  Vue2使用`defineProperty`因为2013年时只能用这种方式。由于该API存在一些局限性，比如对于数组的拦截有问题，为此vue需要专门为数组响应式做一套实现。另外不能拦截那些新增、删除属性；最后`defineProperty`在初始化时需要递归遍历待处理的对象才能对它进行完全拦截，增加了初始化的时间。

  以上两点在Proxy迎刃而解，不仅可以对数组实现拦截，还能对Map、Set实现拦截；另外Proxy的拦截也是懒处理行为，如果用户没有访问嵌套对象，那么也不会实施拦截，这就让初始化的速度和内存占用都改善了。

  当然Proxy是有兼容性问题的，IE完全不支持，所以如果需要IE兼容就不合适

- Teleport

  用于一些弹出框、提示框组件

  注意：虽然挂载到了指定的组件，但是父组件仍然是当前的组件，因此，样式和事件仍然是在当前组件中生效的

- setup中如何获得组件实例？

  直接访问组件实例通常是为了处理一些特殊情况或者兼容性问题，比如需要使用一些 Vue 2.x 中的选项 API 才有的功能，或者处理一些非标准的边缘情况。因

  ```js
  import { getCurrentInstance } from 'vue';
  
  export default {
    setup() {
      const instance = getCurrentInstance();
      console.log(instance); // 将输出组件实例
  
      // instance.ctx来访问组件的上下文，instance.proxy 会给你提供组件实例的代理访问
  
      return {
        // 返回的数据
      };
    }
  };
  ```

- v-if和v-for哪个优先级更高？

  1. 实践中**不应该把v-for和v-if放一起**
  2. 在**vue2中**，**v-for的优先级是高于v-if**，把它们放在一起，输出的渲染函数中可以看出会先执行循环再判断条件，哪怕我们只渲染列表中一小部分元素，也得在每次重渲染的时候遍历整个列表，这会比较浪费；另外需要注意的是在**vue3中则完全相反，v-if的优先级高于v-for**，所以v-if执行时，它调用的变量还不存在，就会导致异常
  3. 通常有两种情况下导致我们这样做：
     - 为了**过滤列表中的项目** (比如 `v-for="user in users" v-if="user.isActive"`)。此时定义一个计算属性 (比如 `activeUsers`)，让其返回过滤后的列表即可（比如`users.filter(u=>u.isActive)`）。
     - 为了**避免渲染本应该被隐藏的列表** (比如 `v-for="user in users" v-if="shouldShowUsers"`)。此时把 `v-if` 移动至容器元素上 (比如 `ul`、`ol`)或者外面包一层`template`即可。
  4. 文档中明确指出**永远不要把 `v-if` 和 `v-for` 同时用在同一个元素上**，显然这是一个重要的注意事项。
  5. 源码里面关于代码生成的部分，能够清晰的看到是先处理v-if还是v-for，顺序上vue2和vue3正好相反，因此产生了一些症状的不同，但是不管怎样都是不能把它们写在一起的。

- setup和created谁先执行？ setup中为什么没有beforeCreate和created？

  `setup` 在 `beforeCreate` 和 `created` 之前执行。因为 `setup` 是组合式 API 的入口点，用于替代 Vue2 的 `data`, `computed`, `methods`, 和大部分生命周期。

  由于 `setup` 在组件实例创建之前执行，此时响应式数据和观察者还没有设置好，所以没必要有 `beforeCreate` 。而 `created` 在组合式 API 中也不必要，因为在 `setup` 中就可以执行它的所有任务，包括数据的响应式声明和其他逻辑的初始化。

- 在Vue中实现权限管理

  一般是**页面权限**和**按钮权限**

  分后端和前端两种方案：

  前端方案会**把所有路由信息在前端配置**，通过路由守卫要求用户登录，用户**登录后根据角色过滤出路由表**。比如我会配置一个`asyncRoutes`数组，需要认证的页面在其路由的`meta`中添加一个`roles`字段，等获取用户角色之后取两者的交集，若结果不为空则说明可以访问。此过滤过程结束，剩下的路由就是该用户能访问的页面，**最后通过`router.addRoutes(accessRoutes)`方式动态添加路由**即可。

  后端方案会**把所有页面路由信息存在数据库**中，用户登录的时候根据其角色**查询得到其能访问的所有页面路由信息**返回给前端，前端**再通过`addRoutes`动态添加路由**信息按钮权限的控制通常会**实现一个指令**，例如`v-permission`，**将按钮要求角色通过值传给v-permission指令**，在指令的`moutned`钩子中可以**判断当前用户角色和按钮是否存在交集**，有则保留按钮，无则移除按钮。

  纯前端方案的优点是实现简单，不需要额外权限管理页面，但是维护起来问题比较大，有新的页面和角色需求就要修改前端代码重新打包部署；服务端方案就不存在这个问题，通过专门的角色和权限管理页面，配置页面和按钮权限信息到数据库，应用每次登陆时获取的都是最新的路由信息，可谓一劳永逸！

- 说一说你对vue响应式理解？

  响应式就是**能够使数据变化可以被检测并对这种变化做出响应的机制**。

  MVVM框架的一个核心问题是连接数据层和视图层，通过**数据驱动**应用，数据变化，视图更新，就需要对数据做响应式处理，一旦数据发生变化就做出更新处理。

  以vue为例，通过数据响应式加上虚拟DOM和patch算法，开发人员只需要关心业务数据，不用接触DOM操作。

  vue2的数据响应式会根据数据类型来做不同处理，**对象采用Object.defineProperty()\**的方式定义数据拦截，当数据被访问或发生变化时，并作出响应；如果是\**数组则通过覆盖数组对象原型的7个变更方法**，使这些方法可以额外的做更新通知，从而作出响应。但实际也存在一些缺点：比如初始化时递归遍历会造成性能损失；新增或删除属性时需要用户使用Vue.set/delete这样特殊的api才能生效；对于es6中新产生的Map、Set数据结构不支持等问题。

  vue3利用Proxy代理要响应化的数据，它有很多好处，编程体验是一致的，不需要使用特殊api，初始化性能和内存消耗都得到了大幅改善；另外由于响应化的实现代码抽取为独立的reactivity包，可以更灵活的使用它，第三方的扩展开发起来更加灵活了。

- 说说你对虚拟 DOM 的理解？

  1. 虚拟dom本身是一个 `JavaScript` 对象，通过不同的属性去描述一个视图结构。

  2. 引入vdom好处：

     **将真实元素节点抽象成 VNode，减少直接操作 dom 次数，从而提高程序性能**

     - 直接操作 dom 有限制，比如：diff、clone 等操作，一个真实元素上有许多的内容，如果直接对其进行 diff 操作，会去额外 diff 一些没有必要的内容；同样的，如果需要进行 clone 那么需要将其全部内容进行复制，也没必要
     - 频繁的dom操作容易引起页面的重绘和回流，但是通过抽象 VNode 进行中间处理，可以有效减少直接操作dom的次数，从而减少页面重绘和回流

     **方便实现跨平台**

- vue3有哪些新特性

  1、compositionAPI

  2、setup

  3、teleport

  4、Fragments

  5、自定义渲染器

  6、v-bind实现CSS变量

  7、Suspense

  8、优化了框架性能：基于Proxy的响应式系统、更好的TS支持、编译器优化：静态提升、patchFlags、block等、虚拟DOM重写

- 说说key的作用

  `key` 可以提高渲染列表的效率，特别是当列表数据发生变化时（比如排序操作），通过 `key` 确定哪些元素是仅仅移动了位置，而不是销毁后重新创建。这样可以减少不必要的 DOM 操作，提升性能。同时，`key` 还有助于维持组件状态和内部组件生命周期，特别是在复用节点时。

- nextTick

  `nextTick` 允许你在下一个 DOM 更新循环之后执行某些代码。在内部，`nextTick` 使用了微任务。

  Vue有个异步更新策略，意思是如果数据变化，Vue不会立刻更新DOM，而是开启一个队列，把组件更新函数保存在队列中，在同一事件循环中发生的所有数据变更会异步的批量更新。这一策略导致我们对数据的修改不会立刻体现在DOM上，此时如果想要获取更新后的DOM状态，就需要使用nextTick。

  传入的callback会被添加到队列刷新函数(flushSchedulerQueue)的后面，这样等队列内部的更新函数都执行完毕，所有DOM操作也就结束了，callback自然能够获取到最新的DOM值。

- Vue 子组件和父组件创建和挂载顺序

  创建过程自上而下，挂载过程自下而上；即：

  - parent created
  - child created
  - child mounted
  - parent mounted

  Vue创建过程是一个递归过程，先创建父组件，有子组件就会创建子组件，因此创建时先有父组件再有子组件；子组件首次创建时会添加mounted钩子到队列，等到patch结束再执行，可见子组件的mounted钩子是先进入到队列中的，因此等到patch结束执行这些钩子时也先执行。

- 从0到1自己构架一个vue项目，说说有哪些步骤、哪些重要插件、目录结构你会怎么组织

  从0创建一个项目我大致会做以下事情：项目构建、引入必要插件、代码规范、提交规范、常用库和组件

  目前vue3项目我会用vite或者create-vue创建项目

  接下来引入必要插件：路由插件vue-router、状态管理vuex/pinia、ui库我比较喜欢element-plus和antd-vue、http工具我会选axios

  其他比较常用的库有vueuse，nprogress，图标可以使用vite-svg-loader

  下面是代码规范：结合prettier和eslint即可

  最后是提交规范，可以使用husky，lint-staged，commitlint

- Vue实例挂载的过程中发生了什么?

  挂载过程指的是app.mount()过程，这个过程中整体上做了两件事：**初始化**和**建立更新机制**

  初始化会创建组件实例、初始化组件状态，创建各种响应式数据

  建立更新机制这一步会立即执行一次组件更新函数，这会首次执行组件渲染函数并执行patch将前面获得vnode转换为dom；同时首次执行渲染函数会创建它内部响应式数据之间和组件更新函数之间的依赖关系，这使得以后数据变化时会执行对应的更新函数。

- Vue3做了哪些编译优化

  1. **静态提升**：会检测模板中不变的静态内容，并将其提升到渲染函数之外，这样在重渲染组件时不会重新创建这些节点。
  2. **预字符串化**：对于完全静态的内容，编译器会预先将其转换为字符串，避免了创建节点的成本。
  3. **缓存事件处理函数**：自动缓存一些模板中的事件处理函数，以减少不必要的重新渲染。
  4. **Block Tree**：这是一种新的内部数据结构，可以更有效地跟踪动态节点的变化，优化了 Virtual DOM 的更新过程。

- ref和reactive的异同

  `ref`接收内部值（inner value）返回响应式`Ref`对象，`reactive`返回响应式代理对象

  从定义上看`ref`通常用于处理单值的响应式，`reactive`用于处理对象类型的数据响应式

  两者均是用于构造响应式数据，但是`ref`主要解决原始值的响应式问题

  ref返回的响应式数据在JS中使用需要加上`.value`才能访问其值，在视图中使用会自动脱ref，不需要`.value`；ref可以接收对象或数组等非原始值，但内部依然是`reactive`实现响应式；reactive内部如果接收Ref对象会自动脱ref；使用展开运算符(...)展开reactive返回的响应式对象会使其失去响应性，可以结合toRefs()将值转换为Ref对象之后再展开。

  reactive内部使用Proxy代理传入对象并拦截该对象各种操作（trap），从而实现响应式。ref内部封装一个RefImpl类，并设置get value/set value，拦截用户对值的访问，从而实现响应式。

- vue-loader是什么?它有什么作用?

  1. `vue-loader`是用于处理单文件组件（SFC，Single-File Component）的webpack loader
  2. 因为有了`vue-loader`，我们就可以在项目中编写SFC格式的Vue组件，我们可以把代码分割为<template>、<script>和<style>，代码会异常清晰。结合其他loader我们还可以用Pug编写<template>，用SASS编写<style>，用TS编写<script>。我们的<style>还可以单独作用当前组件。
  3. webpack打包时，会以loader的方式调用`vue-loader`
  4. `vue-loader`被执行时，它会对SFC中的每个语言块用单独的loader链处理。最后将这些单独的块装配成最终的组件模块。

  `vue-loader` 的作用包括：

  - 解析 `.vue` 文件，抽取其中的每个语言块。
  - 应用其它的 webpack loader 对语言块进行处理，例如 `babel-loader` 用于 `<script>` 块，`css-loader` 用于 `<style>` 块。
  - 允许使用其他预处理器，例如 Pug（用于模板）、TypeScript（用于脚本）、SCSS/Sass（用于样式）。
  - 支持 `<style>` 中的 CSS Modules 和 Scoped CSS，为组件样式提供局部作用域。
  - 热重载（Hot Module Replacement），在开发过程中实时更新修改的组件，无需完全刷新页面。

- v-memo

  `v-memo` 是 Vue3 引入的新指令，用于优化渲染性能。类似于 `React.memo`。接收一个数组，数组中的元素是依赖值，只有当依赖值发生变化时，才会重新渲染组件或元素。

- 关于异步组件

  因为异步路由的存在，我们使用异步组件的次数比较少，因此还是有必要两者的不同。

  在大型应用中，我们需要分割应用为更小的块，并且在需要组件时再加载它们。

  我们不仅可以在路由切换时懒加载组件，还可以在页面组件中继续使用异步组件，从而实现更细的分割粒度。

  使用异步组件最简单的方式是直接给defineAsyncComponent指定一个loader函数，结合ES模块动态导入函数import可以快速实现。我们甚至可以指定loadingComponent和errorComponent选项从而给用户一个很好的加载反馈。另外Vue3中还可以结合Suspense组件使用异步组件。

  异步组件容易和路由懒加载混淆，实际上不是一个东西。异步组件不能被用于定义懒加载路由上，处理它的是vue框架，处理路由组件加载的是vue-router。但是可以在懒加载的路由组件中使用异步组件。

  ```js
  import { defineAsyncComponent } from 'vue'
  // defineAsyncComponent定义异步组件
  const AsyncComp = defineAsyncComponent(() => {
    // 加载函数返回Promise
    return new Promise((resolve, reject) => {
      // ...可以从服务器加载组件
      resolve(/* loaded component */)
    })
  })
  // 借助打包工具实现ES模块动态导入，defineAsyncComponent定义了一个高阶组件，返回一个包装组件。包装组件根据加载器的状态决定渲染什么内容。
  const AsyncComp = defineAsyncComponent(() =>
    import('./components/MyComponent.vue')
  )
  ```

- 使用vue渲染大量数据时应该怎么优化?

  在大型企业级项目中经常需要渲染大量数据，此时很容易出现卡顿的情况。比如大数据量的表格、树。

  处理时要根据情况做不通处理：

  - 可以采取分页的方式获取，避免渲染大量数据
  - [vue-virtual-scroller](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FAkryum%2Fvue-virtual-scroller)等虚拟滚动方案，只渲染视口范围内的数据
  - 如果不需要更新，可以使用`v-once`方式只渲染一次
  - 通过[v-memo](https://link.juejin.cn?target=https%3A%2F%2Fvuejs.org%2Fapi%2Fbuilt-in-directives.html%23v-memo)可以缓存结果，结合`v-for`使用，避免数据变化时不必要的VNode创建
  - 可以采用懒加载方式，在用户需要的时候再加载数据，比如tree组件子树的懒加载

  总之，还是要看具体需求，首先从设计上避免大数据获取和渲染；实在需要这样做可以采用虚表的方式优化渲染；最后优化更新，如果不需要更新可以v-once处理，需要更新可以v-memo进一步优化大数据更新性能。其他可以采用的是交互方式优化，无线滚动、懒加载等方案。

- router-link和router-view是如何起作用的?

  vue-router中两个重要组件`router-link`和`router-view`，分别起到路由导航作用和组件内容渲染作用

  使用中router-link默认生成一个a标签，设置to属性定义跳转path。实际上也可以通过custom和插槽自定义最终的展现形式。router-view是要显示组件的占位组件，可以嵌套，对应路由配置的嵌套关系，配合name可以显示具名组件，起到更强的布局作用。

  router-link组件内部根据custom属性判断如何渲染最终生成节点，内部提供导航方法navigate，用户点击之后实际调用的是该方法，此方法最终会修改响应式的路由变量，然后重新去routes匹配出数组结果，router-view则根据其所处深度deep在匹配数组结果中找到对应的路由并获取组件，最终将其渲染出来。

- Vue3性能提升在哪些方面

  分别从代码、编译、打包三方面介绍vue3性能方面的提升

  代码层面性能优化主要体现在全新响应式API，基于Proxy实现，初始化时间和内存占用均大幅改进；

  编译层面做了更多编译优化处理，比如静态提升、动态标记、事件缓存，区块等，可以有效跳过大量diff过程；

  打包时更好的支持tree-shaking，因此整体体积更小，加载更快

- history模式和hash模式

  区别只在url形式

  hash模式在地址栏显示的时候是已哈希的形式：#/xxx，这种方式使用和部署简单，但不会被搜索引擎处理，seo有问题；history模式则建议用在大部分web项目上，但是它要求应用在部署时做特殊配置，服务器需要做回退处理，否则会出现刷新页面404的问题。

  底层实现上其实hash是一种特殊的history实现。

  ```js
  // hash
  // 浏览器里的形态：http://xx.com/#/about
  // history
  // 浏览器里的形态：http://xx.com/about
  ```

- 递归组件

  递归组件是一种在其模板中调用自己的组件，通常用于处理树形结构的数据，如菜单、评论列表等。为了让组件能够递归地调用自己，它必须有一个名字。

  ```vue
  <template>
    <ul>
      <li v-for="item in treeData" :key="item.id">
        {{ item.name }}
        <my-tree-component v-if="item.children" :tree-data="item.children"/>
      </li>
    </ul>
  </template>
  
  <script>
  export default {
    name: 'MyTreeComponent',
    props: {
      treeData: Array
    }
  }
  </script>
  ```

  在这个例子中，`MyTreeComponent` 是一个递归组件，它接受一个树状结构的数据作为 prop，并为每个节点渲染一个列表项。如果该节点有子节点，它将继续渲染子树。

- key可以设为随机数吗

  将 `key` 设置为随机数是不推荐的，因为这会迫使 Vue 在每次渲染时都无法识别节点是否可以复用，导致重复地销毁和创建元素，这样会丧失 `key` 本身用来优化 Virtual DOM 渲染性能的意义。正确的做法是使用一个唯一且稳定的值，如列表项的 id 或者是顺序索引（如果列表顺序不变的话）。这样 Vue 才能高效地更新和复用 DOM 元素。

- 谈一谈对 MVVM 的理解

  MVVM（Model-View-ViewModel）是一种架构模式，主要用于简化用户界面的开发：

  - **Model** 表示数据模型，包含业务逻辑以及数据状态，但不包含任何与表现层相关的信息。
  - **View** 是用户界面，展示数据（Model）并触发事件（如用户输入）。
  - **ViewModel** 作为视图的抽象，负责转换 Model 中的数据使之易于管理和展示。它提供命令以响应 View 的请求，并且可以监听 Model 的变化，实现数据的双向绑定。

  Vue.js 是 MVVM 的例子，其中 Vue 实例充当 ViewModel，负责连接 View（模板）和 Model（响应式数据）。通过响应式和模板数据绑定，ViewModel 使得数据状态的更改可以自动传递到 View，同时也可以将 View 中的事件和状态变化同步到 Model。

  优点

  - 降低了代码的耦合性，提高了视图或者逻辑的重用性，视图和数据层都是独立的，不必因为谁而改变谁。
  - 自动更新DOM，利用双向绑定，关注业务逻辑

  缺点

  - bug调试困难，界面出现异常，可能是view或者model层发生错误，并且数据绑定的声明是无法进行打断点debug，可以用扩展的工具进行调试
  - 一个大的模块中model也会很大，会占用更多的内存，相应viewmodel的构建和维护也是成本

  核心：双向绑定

  优点

  - **低耦合性**：视图（UI）和模型（业务逻辑）之间的解耦。
  - **可重用性**：可以独立于视图重用业务逻辑。
  - **可测试性**：由于业务逻辑和 UI 分离，可以更容易地编写测试代码。
  - **双向数据绑定**：自动同步视图和模型的变化，减少DOM操作。

  缺点

  - **复杂性**：对于简单的场景，MVVM 可能会增加不必要的复杂性。
  - **性能问题**：双向绑定机制可能会引入性能瓶颈，特别是在大量数据和频繁更新时。
  - **学习曲线**：初学者需要时间理解和掌握其概念和实现。

- Proxy 只会代理对象的第一层

  `Proxy` 只能直接代理它被应用到的对象的顶层属性。如果对象的属性值也是对象，那么这些嵌套对象的属性不会自动被代理。要使嵌套对象也具有响应式，需要递归地对这些嵌套对象应用 `Proxy`。这也是 Vue 3 中 `reactive` 函数的工作方式，它会递归地将一个对象的所有嵌套对象转换成响应式对象。

- 解释 hash 模式和 history 模式的实现原理

  后面 hash 值的变化，不会导致浏览器向服务器发出请求，浏览器不发出请求，就不会刷新页面；通过监听 hashchange 事件可以知道 hash 发生了哪些变化，然后根据 hash 变化来实现更新页面部分内容的操作。

  history 模式的实现，主要是 HTML5 标准发布的两个 API，pushState 和 replaceState，这两个 API 可以在改变 URL，但是不会发送请求。这样就可以监听 url 变化来实现更新页面部分内容的操作。

  两种模式的区别：

  - 首先是在 URL 的展示上，hash 模式有“#”，history 模式没有
  - 刷新页面时，hash 模式可以正常加载到 hash 值对应的页面，而 history 没有处理的话，会返回 404，一般需要后端将所有页面都配置重定向到首页路由
  - 在兼容性上，hash 可以支持低版本浏览器和 IE

- vue2 和 vue3 的 *diff* 算法分别说一下？

  Vue 2 使用的是传统的虚拟 DOM diff 算法：

  - 当数据发生变化时，渲染函数会基于新的数据生成新的虚拟节点 (vnodes)。
  - 新旧 vnodes 进行比对（diffing），按照深度优先和同层比较的策略对比差异。
  - 识别需要更新的部分，并对真实 DOM 进行最小量更新。

  Vue 3 采用优化的 diff 算法，引入了 Fragment, Teleport, Suspense 等新的内置组件类型，并对静态节点和动态节点做了更明显的区分，优化了 diff 过程：

  - 引入了静态提升，将不改变的 DOM 内容提升到渲染函数之外，减少了比对次数。
  - 对于动态绑定了 key 的列表进行更高效的 diff。
  - 通过块（block）的概念，保持动态节点的结构信息，使得 diff 过程中能够更快地定位到动态节点，提高了 diff 效率。
  - 使用了 Proxy 代替了 `Object.defineProperty`，使得依赖追踪更加精准和高效。

  这些优化使得 Vue 3 在性能上有了显著提升，尤其是在处理大规模更新和动态内容时。

- vueRouter 有哪几种导航守卫？

  1. **全局守卫**：适用于任何路由变化。
     - `beforeEach`: 在导航触发之前全局前置守卫。
     - `beforeResolve`: 在导航被确认之前，同时在所有组件内守卫和异步路由组件被解析之后。
     - `afterEach`: 在导航确认之后全局后置钩子。
  2. **路由独享的守卫**：在路由配置上直接定义。
     - `beforeEnter`: 仅在某个路由下生效。
  3. **组件内的守卫**：在路由组件内直接定义。
     - `beforeRouteEnter`: 在渲染该组件的对应路由被 confirm 前调用。
     - `beforeRouteUpdate`: 在当前路由改变，但是该组件被复用时调用。
     - `beforeRouteLeave`: 导航离开该组件的对应路由时调用。

- v-if 与 v-show 的区别

  1、手段

  v-if 是动态的向 DOM 树内添加或者删除 DOM 元素

  v-show 是通过设置 DOM 元素的 display 样式属性控制显隐

  2、编译过程

  v-if 切换有一个局部编译/卸载的过程，切换过程中合适地销毁和重建内部的事件监听和子组件

  v-show 只是简单的基于 css 切换

  3、编译条件

  **v-if 是惰性的，如果初始条件为假，则什么也不做。只有在条件第一次变为真时才开始局部编译**

  **v-show 是在任何条件下(首次条件是否为真)都被编译，然后被缓存，而且 DOM 元素保留**

  4、性能消耗

  v-if 有更高的切换消耗

  v-show 有更高的初始渲染消耗

  5、使用场景

  v-if 适合运营条件不大可能改变

  v-show 适合频繁切换

- Vue 组件的data为什么必须是函数

  组件中的 data 写成一个函数，数据以函数返回值形式定义。这样每复用一次组件，就会返回一份新的 data，类似于给每个组件实例创建一个私有的数据空间，让各个组件实例维护各自的数据。而单纯的写成对象形式，就使得所有组件实例共用了一份 data，就会造成一个变了全都会变的结果。

- computed的实现原理

  1. 声明 `computed` 属性，Vue 会为该属性创建一个观察者对象，存储了 `computed` 属性的 getter 函数。
  2. 首次访问 `computed` 属性时，观察者执行其 getter 函数并将结果缓存下来。
  3. 在 getter 函数执行过程中，会访问响应式数据，此时响应式数据的依赖收集机制会将该 `computed` 属性的观察者添加到数据的依赖列表中。
  4. 当依赖的响应式数据发生变化时，会通知 `computed` 属性的观察者，使得缓存的值失效。
  5. 下次再访问该 `computed` 属性时，由于缓存值已失效，会重新计算并缓存新值。

- 说一下Vue complier的实现原理

  1. **解析**：编译器首先将模板字符串解析为 AST（抽象语法树）。这个过程包括标记化（tokenization），将模板字符串分割成标记；然后进行语法分析（parsing），将标记转换成 AST。**使用正则对`template`模板进行解析，将标签、指令、属性等转化为AST抽象语法树**
  2. **优化**：编译器遍历 AST，并标记出所有静态根节点，不会在重新渲染时变化，对这部分进行优化处理。**遍历AST，对静态节点进行标记和提升，优化runtime的性能**
  3. **代码生成**：编译器将 AST 转换为 JS 渲染函数的代码字符串。这个渲染函数是响应式系统的依赖，当响应式数据变化，函数会被重新执行，生成新的虚拟 DOM。**将AST转化为render函数字符串**

  

  Vue 3 编译器还引入例如静态提升、内联事件处理和片段分组，进一步提高了运行时的效率。

- SPA首页加载慢怎么解决

  1. **代码分割**：如 Webpack 提供的代码分割功能，按路由或组件拆分代码，实现懒加载。
  2. **服务端渲染（SSR）**：使用像 Nuxt.js 框架可以在服务器端预渲染页面，加快首屏加载速度。
  3. **静态资源优化**：压缩 JavaScript、CSS 文件，使用图片懒加载等。
  4. **利用缓存**：通过配置 HTTP 缓存策略，减少重复资源的下载。
  5. **使用 CDN**：通过 CDN 分发静态资源，减少加载时间。
  6. **性能监测与分析**：利用 Chrome DevTools 等工具进行性能分析，查找瓶颈并进行针对性优化。
  7. **预加载和预取**：对关键资源使用 `<link rel="preload">` 预加载，对非关键资源使用 `<link rel="prefetch">` 预取。

- SessionStorage和Vuex

  最大的区别：vuex在页面刷新后就会重置，而sessionStorage在页面刷新后仍然存在（sessionStorage在页面关闭后重置）

  - 如果数据需要跨组件共享，且希望数据在页面刷新后不保留，可以使用 `Vuex`。
  - 如果数据仅在页面会话期间使用，不跨组件或跨页面共享，可以使用 `sessionStorage`。

  许多使用 `sessionStorage` 的场景也可以用 `Vuex` ，取决于具体需求：

  - 使用 `sessionStorage` 当想要在浏览器的单个会话中保留状态，而且这些状态不需要跨会话持久化。比如，用户在填写长表单的过程中关闭了浏览器标签，你可能希望用户重新打开标签时表单内容仍然存在。
  - 使用 `Vuex` 当状态需要在多个组件间共享，且你可能希望在应用的不同部分对这些状态进行管理和反应。例如，用户登录信息可能需要在多个组件中访问和验证。

  如果只是在页面会话中临时存储数据，并且这些数据在用户关闭标签后不需要保留，那么 `sessionStorage` 是一个更合适的选择。而对于应用的整体状态管理，尤其是涉及到多个组件之间状态共享的情况，`Vuex` 更为合适。

- keep-alive，有哪几种使用方式

  1. **基本用法**：直接包裹组件或者视图，用于缓存组件状态。

     ```
     vueCopy code<keep-alive>
       <YourComponent />
     </keep-alive>
     ```

  2. **结合路由使用**：在 Vue 路由中，可以将 `<router-view>` 包裹在 `<keep-alive>` 中，缓存页面组件。

     ```
     vueCopy code<keep-alive>
       <router-view />
     </keep-alive>
     ```

  3. **使用 include 和 exclude**：可以指定哪些组件缓存或不缓存。

     ```
     vueCopy code<keep-alive include="ComponentA,ComponentB">
       <router-view />
     </keep-alive>
     ```

  4. **配合动态组件**：在动态组件 `<component :is="currentView">` 周围使用 `<keep-alive>`。

     ```
     vueCopy code<keep-alive>
       <component :is="currentView" />
     </keep-alive>
     ```

- 为什么要有nextTick

  因为vue中将DOM的更新放在一个批量更新的事件中，而不是数据改变后立即更新DOM，这样就导致到数据改变后如果我们立即获取DOM元素，此时DOM元素还没有更新，导致获取到的DOM元素的数据是还没有更新前的，是不准确的

- **为什么要store.commit('add')才能触发事件执行呢？ 可不可以进行直接调用mutation函数进行操作呢？**

  在 Vuex 中，`commit` 用于触发同步的 `mutation` 方法，这是 Vuex 提供的一种模式，确保状态变更是可追踪和规范的。直接调用 `mutation` 函数并不符合 Vuex 的设计原则，因为这样做会绕过 Vuex 的状态管理机制，使得状态变更难以追踪。在 Vuex 的架构中，所有的状态变更都应该通过 `commit` 来触发，这样可以保持状态变更的清晰和一致性，并且可以在开发工具中跟踪状态变更的历史。store类里根本没有mutation方法，只能通过调用commit方法来执行mutation里的函数列表。

  **为什么不可以直接对state存储的状态进行修改，只能通过调用mutation函数的方式修改呢？**

  Vuex 通过强制限制对 store 的修改方式来确保状态的可追踪性。只有通过 mutation 函数才能修改 store 中的状态，这样可以轻松地跟踪状态的变化，也可以避免无意中从不同的组件中直接修改 store 导致的代码难以维护和调试的问题。

  **为什么存在异步调用的函数需要`store.dispatch('asyncAdd')`函数才能完成呢？可以直接调用`store.commit('asyncAdd')`嘛？如果不可以，为什么呢？**

  `mutation` 必须同步执行，这是因为任何时候对状态的变更都应该是可追踪的。异步操作会打破这个规则，因为它们可能会在外部的回调函数中改变状态，这样就无法准确地追踪到状态是何时何地发生变化的。因此，异步逻辑都应该包含在 `action` 之内。当异步操作完成后，你可以在 `action` 中调用 `commit` 来同步更新状态。这样做既可以处理异步操作，又可以保持状态变更的可追踪性。

- scoped 的原理

  给每个组件生成一个hash值，同一个组件内的所有样式加上这个hash值

  .button[data-v-123456] { ... }

  ```html
  在样式后面加上[]是CSS选择器的一部分，被称为属性选择器（Attribute Selector）。允许您选择具有特定HTML属性的元素，并且只选择那些具有匹配属性值的元素。
  
  例如：
  <button data-v-123456>Click me</button>
  <button data-v-789012>Click me too</button>
  您可以使用属性选择器来选择具有特定data-v属性值的按钮元素，如下所示：
  
  [data-v-123456] {
    background-color: red;
  }
  [data-v-789012] {
    background-color: blue;
  }
  ```

  在组件渲染时，Vue 会确保只有具有相同哈希值的元素才会应用这些样式规则。这是因为属性选择器`[data-v-123456]`会匹配带有相同哈希值的元素。

  关键点在于，Vue 在内部处理了这个机制，它会自动修改样式规则的选择器，而不需要在HTML元素上添加额外的属性。这个哈希值的生成和应用是Vue.js编译器和运行时的一部分，开发者不需要手动干预。

  所以，虽然HTML元素本身不会添加哈希，但Vue.js确保了样式规则只会应用于与组件相匹配的元素，从而实现了样式的隔离，而无需在HTML元素上添加额外的标记。

- vue3和vue2的自定义指令有什么区别？

  主要是在生命周期方面：

  Vue2 中，自定义指令的生命周期钩子函数包括：

  - `bind`：只调用一次，当指令第一次绑定到元素时调用。
  - `inserted`：被绑定元素插入父节点时调用。
  - `update`：所在组件的 VNode 更新时调用，但可能发生在其子 VNode 更新之前。
  - `componentUpdated`：指令所在组件的 VNode 及其子 VNode 全部更新后调用。
  - `unbind`：只调用一次，指令与元素解绑时调用。

  Vue3 引入了 Composition API，对自定义指令的生命周期进行了调整：

  - `created`：新的钩子，类似于 Vue 2 中的 `bind`，但它发生在绑定元素的父组件被挂载之前。
  - `beforeMount`：类似于 Vue 2 中的 `bind`，在指令第一次绑定到元素并在挂载父组件之前调用。
  - `mounted`：类似于 Vue 2 中的 `inserted`，在绑定元素的父组件被挂载后调用。
  - `beforeUpdate`：新的钩子，在所在组件的 VNode 更新之前调用。
  - `updated`：类似于 Vue 2 中的 `componentUpdated`，在指令所在组件的 VNode 及其子 VNode 全部更新后调用。
  - `beforeUnmount`：新的钩子，类似于 Vue 2 中的 `unbind`，但它发生在绑定元素的父组件卸载之前。
  - `unmounted`：类似于 Vue 2 中的 `unbind`，在绑定元素的父组件卸载后调用。

  其他的就是性能优化，由于 Vue 3 的整体架构改进，自定义指令的性能也得到了提升

- Vue 为什么要将 nextTick 放在微任务队列(的末尾)？

  首先，nextTick属于微任务 这种说法并不是很准确，Vue会首先进行判断，如果支持放在微任务则会放在微任务，如果不支持则会被放到宏任务的末尾。

  浏览器事件循环的执行顺序是：当前宏任务 -> 所有微任务 -> DOM重新(如果需要) -> 下一个循环

  **注意：Vue中的DOM更新是指将DOM 将反映出最新的状态[但页面并没有刷新]，而上述的“DOM重新(如果需要)”是指浏览器重新绘制页面[重绘和回流]**

  Vue 尝试使用 JavaScript 的微任务队列（例如 `Promise.then`、`MutationObserver`）来执行这些 DOM 更新。因此，当数据变化时，实际的 DOM 更新操作被放入微任务队列。

  `nextTick` 把回调函数加入到与 DOM 更新同一个微任务队列的末尾

  **`nextTick` 作为微任务**

  当 `nextTick` 是微任务，确保了在当前事件循环的所有 Vue DOM 更新之后，但在浏览器的重绘和回流之前，执行。这样，可以在 `nextTick` 的回调中安全地访问更新后的 DOM，而不会触发额外的渲染周期。

  **`nextTick` 如果是宏任务**

  如果 `nextTick` 是宏任务，那么它将被推迟到当前事件循环的所有微任务和浏览器渲染之后执行（下一个时间循环的宏任务中）。这意味着：

  1. **可能错过当前渲染周期**：`nextTick` 的回调将错过当前的渲染周期，只能在下一个宏任务中执行。
  2. **更多的渲染周期**：如果在 `nextTick` 的回调中进行了数据修改，这可能导致额外的渲染周期，因为它将发生在下一个事件循环中。
  3. **效率问题**：这可能导致效率问题，因为在获取最新 DOM 状态和可能的进一步 DOM 操作之间可能存在延迟。

- 在vue中用CDN引入以减少包体积，但如何使用CDN引入的内容

  1、使用script进行引入 <script src="https://unpkg.com/vue@next"></script>；

  2、对webpack或vite进行配置

- vite生产包是没有区别的，因为都是要全部打包到一起部署到服务器上，而vite开发包是可以用到哪个部分再加载哪个部分

  1. **基于 ES 模块的服务器：**
     - Vite 利用现代浏览器对 ES 模块（ESM）的支持，直接在浏览器中服务模块化的 JS 代码。意味着在开发环境中，Vite 可以快速提供文件，无需预先打包。而 Webpack 在开发环境中通常会创建一个完整的打包文件。
  2. **冷启动优化：**
     - 首次启动项目时，Vite 只需要对实际需要的文件进行处理和加载，而不是整个应用。而 Webpack 会处理和打包所有文件，这会使初始启动时间变长。 
  3. **热模块替换（HMR）：**
     - Vite 的热模块替换通常比 Webpack 的更快，因为 Vite 只需要重新请求改变了的模块，而 Webpack 是重新打包整个应用。
  4. **预构建优化：**
     - Vite 对依赖项进行预构建，在第一次启动时将所有的 node_modules 中的依赖预打包成 ESM，这样在后续启动和热更新时，这些依赖就已经是优化过的，减少了处理时间。
  5. **原生 ESM 支持：**
     - 由于 Vite 使用原生的 ES 模块，充分利用浏览器的模块缓存机制，而 Webpack 生成的是自定义格式的模块系统，需要浏览器进行更多的处理。
  6. **按需编译：**
     - **在开发环境中**：
     - Vite 实现了按需编译，只有在请求时才编译模块。
     - 而 Webpack 则启动时就会处理所有模块，哪怕模块并未立即用到。

- vue2为什么不数据劫持数组：对象的键值对数量一般来说比较少，而数组的元素一般来说数量会更多，有时候甚至会很多，如果每一个都去劫持的话会非常消耗性能（尤雨溪回答的大概意思）数组本质上也是对象，新增的属性并不会自动进行数据劫持

- Vue3中的错误捕获（React使用的是错误边界）

  #### 组件级错误处理

  **errorCaptured 钩子**：这是一个特殊的生命周期钩子，用于捕获组件树中后代组件抛出的错误。这个钩子在错误向上传播到应用的根部之前被调用。

  ```js
  export default {
    errorCaptured(err, instance, info) {
      // 处理错误
      // err：捕获的错误
      // instance：发生错误的组件实例
      // info：包含错误来源信息的字符串
      return false; // 阻止错误继续向上传播
    }
  }
  ```

  #### 全局错误处理

  **全局异常处理器**：可以通过应用实例的 `app.config.errorHandler` 设置全局异常处理器。

  ```js
  const app = Vue.createApp({ ... });
  
  app.config.errorHandler = (err, instance, info) => {
    // 全局处理错误
  };
  ```

- 持久化存储

  Vuex 使用插件 vuex-persistedstate

  Pinia 使用插件 pinia-plugin-persist

  本质上都是将数据保存到LocalStorage中，在初始化或加载页面时从LocalStorage中读取，在数据变化时将新数据保存到LocalStorage中

- Vue3和React在diff算法上的差别

  Vue3

  1. **静态树优化**：Vue 3 的编译器会在编译阶段尝试检测模板中的静态内容，并在后续的更新中跳过这些内容。这减少了不必要的虚拟 DOM 节点比较和重新渲染。
  2. **响应式系统**：Vue 3 使用基于 Proxy 的响应式系统，这比 Vue 2 的基于 Object.defineProperty 的系统更高效。这意味着 Vue 3 可以更快地响应状态变化，并触发组件的更新。
  3. **Fragment 和 Teleport**：Vue 3 支持 Fragment 和 Teleport 等新概念，允许开发者对视图进行更灵活的结构化，从而提高 diff 算法的效率。
  4. **列表渲染优化**：和 React 类似，Vue 3 在处理列表时也使用 `key` 属性来跟踪节点的身份，从而优化重渲染过程。
  5. **细粒度更新**：Vue 3 的 Composition API 允许开发者更精细地控制响应式状态和它们如何影响组件，这可以减少不必要的组件更新。

  React

  1. **元素类型比较**：React 在比较两个虚拟 DOM 元素时，首先检查它们的类型。如果类型不同，React 会销毁旧的 DOM 节点，并构建新的节点。如果类型相同，React 会保留 DOM 节点，并仅更新改变的属性。
  2. **列表中的 Key**：在处理列表时，React 强烈推荐为每个列表项提供一个唯一的 `key` 属性。这使得 React 能够在重新渲染列表时，正确地识别和复用 DOM 节点，从而减少不必要的元素创建和销毁。
  3. **子元素比较**：React 会递归地比较虚拟 DOM 树的所有子元素。一旦在某一层发现差异，它将会更新这一层以及所有子层级的 DOM。
  4. **组件级别比较**：React 将组件作为比较的单元。当一个组件的 props 或 state 发生变化时，该组件及其子组件会进行重新渲染。
  5. **单向数据流**：React 遵循单向数据流的原则，这意味着数据的变化会从上至下传递，并引起相应的界面更新。

- 

- 

- 

- 

- 

- 











