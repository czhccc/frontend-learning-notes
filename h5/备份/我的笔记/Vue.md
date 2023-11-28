# Vue

## 目录划分

```
node_modules	依赖
public
scr
	assets
		css
		img
	common		公共的js文件
		const.js	  公共的常量
		mixin.js	  混入
		utils.js	  公共的工具类
	components
		common   完全可以复用的组件，与当前项目无关
		content  可以复用但与当前项目有关的组件
	network	网络相关
	router	路由相关
	store   vuex状态管理相关
	views	
		xxx1  
		xxx2
		……
	App.vue
	main.js
	……
	vue.config.js	配置别名

```

## MVVM

什么是MVVM呢？

​		通常我们学习一个概念，最好的方式是去看维基百科(对，千万别看成了百度百科)

​		https://zh.wikipedia.org/wiki/MVVM

​		维基百科的官方解释，我们这里不再赘述。

**Vue的MVVM：**

![](Vue.assets/16.png)

**View层：**

- 视图层
- 在我们前端开发中，通常就是DOM层，即页面。
- 主要的作用是给用户展示各种信息。

**Model层：**

- 数据层
- 数据可能是我们固定的死数据，更多的是来自我们服务器，从网络上请求下来的数据，即data。
- 在我们计数器的案例中，就是后面抽取出来的obj，当然，里面的数据可能没有这么简单。

**VueModel层：**

- 视图模型层
- 视图模型层是View和Model沟通的桥梁，即Vue实例。
- 一方面它实现了Data Binding，也就是数据绑定，将Model的改变实时的反应到View中
- 另一方面它实现了DOM Listener，也就是DOM监听，当DOM发生一些事件(点击、滚动、touch等)时，可以监听到，并在需要的情况下改变对应的Data。



## 生命周期

### 概念

生命周期钩子的 `this` 上下文指向调用它的 Vue 实例。

所有的生命周期钩子自动绑定 `this` 上下文到实例中，因此你可以访问数据，对 property 和方法进行运算。这意味着**你不能使用箭头函数来定义一个生命周期方法**，例如`created: () => console.log(this.a)`。 因为箭头函数绑定了父上下文，因此 `this`与你期待的 Vue 实例不同。 

不要在选项属性或回调上使用箭头函数，比如 `created: () => console.log(this.a)` 或 `vm.$watch('a', newValue => this.myMethod())` 。因为箭头函数并没有 `this`，`this` 会作为变量一直向上级词法作用域查找，直至找到为止，经常导致 `Uncaught TypeError: Cannot read property of undefined` 或 `Uncaught TypeError: this.myMethod is not a function` 之类的错误。 

![](Vue.assets/222.jpg)

![](Vue.assets/111.jpg)

 ![Vue å®ä¾çå½å¨æ](Vue.assets/lifecycle.png) 

### 各生命周期函数

**beforeCreate**

在实例初始化之后，数据观测 (data observer) 和 event/watcher 事件配置之前被调用。 

创建前，访问不到data当中的属性以及methods当中的属性和方法，可以在当前生命周期创建一个loading，在页面加载完成之后将loading移除。



**created**

在实例创建完成后被立即调用。在这一步，实例已完成以下的配置：数据观测 (data observer)，property 和方法的运算，watch/event 事件回调。然而，挂载阶段还没开始，`$el` property 目前尚不可用。 

创建后，当前生命周期执行的时候会遍历data中所有的属性，给每一个属性都添加一个getter、setter方法,将data中的属性变成一个响应式属性。遍历data和methods，将这些属性和方法代理到vue的实例上。

*可以进行前后端数据的交互(ajax请求)。*



**beforeMount**

在挂载开始之前被调用：相关的 `render` 函数首次被调用。

1. 模板与数据进行结合，但是还没有挂载到页面，因此我们可以在当前生命周期中进行*数据最后的修改*。
2.  获取不到真实的DOM结构。

该钩子在服务器端渲染期间不被调用。



**mounted**

1. 当前生命周期数据和模板进行相结合，并且已经挂载到页面上，因此可以*获取到真实的DOM元素。* 
2. 在这个生命周期中可以做一些new操作 

实例被挂载后调用，这时 `el` 被新创建的 `vm.$el` 替换了。如果根实例挂载到了一个文档内的元素上，当 `mounted` 被调用时 `vm.$el` 也在文档内。

注意 `mounted` 不会保证所有的子组件也都一起被挂载。如果你希望等到整个视图都渲染完毕，可以在 `mounted` 内部使用 vm.$nextTick：

```javascript
mounted: function () {
  this.$nextTick(function () {
    // Code that will run only after the
    // entire view has been rendered
  })
}
```

该钩子在服务器端渲染期间不被调用。



**beforeUpdate**

1. 当data中的数据更新时才会执行，因此可以用来*检测数据的变化*。

2. 当前生命周期执行的时候会将*更新的数据与模板进行相结合，但是并没有挂载到页面上*，因此我们可以在当前生命周期中做*更新数据的最后修改*。

3. *适合在更新之前访问现有的 DOM*，比如手动移除已添加的事件监听器。

4. 发生在虚拟 DOM 打补丁之前，当前生命周期中的数据和模板还没有更新完毕

该钩子在服务器端渲染期间不被调用，因为只有初次渲染会在服务端进行。



**updated**

1. 数据与模板进行相结合，并且将更新后的数据挂载到页面上。因此我们可以在当前生命周期中*获取到更新后的DOM结构*。这表明组件 DOM 已经更新，应*避免在此期间更改状态*。如果要相应状态改变，通常最好使用 计算属性 或 watcher 取而代之。数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子。

2. 会被频繁触发，在当前周期做事件绑定或实例化操作时，*提前做条件判断*，否则会导致内存泄露. 

注意 `updated` **不会**保证所有的*子组件*也都一起被重绘。如果你希望等到*整个视图*都重绘完毕，可以在 `updated` 里使用 [vm.$nextTick](https://cn.vuejs.org/v2/api/#vm-nextTick)：

```javascript
updated: function () {
  this.$nextTick(function () {
    // Code that will run only after the
    // entire view has been re-rendered
  })
}
```

该钩子在服务器端渲染期间不被调用。



**activated**

被 keep-alive 缓存的组件激活时调用。

该钩子在服务器端渲染期间不被调用。



**deactivated**

被 keep-alive 缓存的组件停用时调用。

该钩子在服务器端渲染期间不被调用。



**beforeDestroy**

可以访问真实DOM结构和data中的数据，一般进行*事件的解绑 监听的移除 定时器的清除等操作。*

实例销毁之前调用。在这一步，实例仍然完全可用。

该钩子在服务器端渲染期间不被调用。



**destroyed**

当前生命周期执行完毕后会将vue与页面之间的关联进行断开。

1. 将DOM与数据间的关联进行断开

2. 在当前生命周期函数中访问不到真实的DOM结构　　

实例销毁后调用。该钩子被调用后，对应 Vue 实例的所有指令都被解绑，所有的事件监听器被移除，所有的子实例也都被销毁。

该钩子在服务器端渲染期间不被调用。



**errorCaptured**

当捕获一个来自子孙组件的错误时被调用。此钩子会收到三个参数：错误对象、发生错误的组件实例以及一个包含错误来源信息的字符串。此钩子可以返回 `false` 以阻止该错误继续向上传播。

你可以在此钩子中修改组件的状态。因此在捕获错误时，在模板或渲染函数中有一个条件判断来绕过其它内容就很重要；不然该组件可能会进入一个无限的渲染循环。

**错误传播规则**

- 默认情况下，如果全局的 `config.errorHandler` 被定义，所有的错误仍会发送它，因此这些错误仍然会向单一的分析服务的地方进行汇报。
- 如果一个组件的继承或父级从属链路中存在多个 `errorCaptured` 钩子，则它们将会被相同的错误逐个唤起。
- 如果此 `errorCaptured` 钩子自身抛出了一个错误，则这个新错误和原本被捕获的错误都会发送给全局的 `config.errorHandler`。
- 一个 `errorCaptured` 钩子能够返回 `false` 以阻止错误继续向上传播。本质上是说“这个错误已经被搞定了且应该被忽略”。它会阻止其它任何会被这个错误唤起的 `errorCaptured` 钩子和全局的 `config.errorHandler`。











## 指令

### v-if、v-show

#### v-if 、v-else-if、v-else

```javascript
// 简单使用
<div id="app"> 
  <div v-if="score>=90">优秀</div>
  <div v-else-if="score>=80">良好</div>
  <div v-else-if="score>=60"> 及格</div>
  <div v-else>不及格</div>
</div>
```

#### v-show

 `v-show` 不支持  `<template>` 元素 

```javascript
// 简单使用
<h2 v-show="isShow">Hello</h2>
```

#### 区别

v-if 为false时，删除DOM中的元素。

v-if 如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

v-show 不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。 

v-show 为false时，仅仅是将元素的display属性设置为none而已。

**开发中如何选择呢？**

​		**当需要在显示与隐藏之间切片很频繁时，使用v-show**

​		**当只有一次切换时，通过使用v-if**

#### key

通常会复用已有元素而不是从头开始渲染。

例如，如果在文本框中输入内容后，再进行切换，内容会被保存。

![1587619244745](Vue.assets/1587619244745.png)![1587619259452](Vue.assets/1587619259452.png)

如果需要避免以上情况，可以绑定key值。

```javascript
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username" key="username-input"> // 绑定key值
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address" key="email-input"> // 绑定key值
</template>
// 此时，每次切换时，输入框都将被重新渲染。
// 上面的 <label> 元素仍然会被高效地复用，因为它们没有添加 key 属性。
```

### v-on

简写为`@`，用于绑定事件监听，如果该方法不需要额外参数，那么方法名后面不需要加 () 。

`<button @click="btnClick"> 按钮点击 </button>`

如果方法本身中有一个参数，那么会默认将原生事件event参数传递进去。如果需要同时传入某个参数，同时需要event时，可以通过 `$event` 传入事件。

不同于组件和 prop，事件名不存在任何自动化的大小写转换。并且 `v-on` 事件监听器在 DOM 模板中会被自动转换为全小写 (因为 HTML 是大小写不敏感的)。 所以 `v-on:myEvent` 和 `v-on:myeven` 是不相同的。

推荐**始终使用 kebab-case 的事件名。**

#### 修饰符

- .stop：调用 event.stopPropagation()。停止冒泡。

- .prevent：调用 event.preventDefault()。阻止默认行为。

- .{keyCode | keyAlias}：只当事件是从特定键触发时才触发回调。`<input @keyup.enter="onEnter">`

  具体的 按键修饰符 见官网。

- .native：监听组件根元素的原生事件。但有时组件会被进行重构，此时，.native会调用失败，但并不会报错，此时，见官网（深入了解组件 -> 自定义事件 -> 将原生事件绑定到组件）。

- .once：只触发一次回调。不像其它只能对原生的 DOM 事件起作用的修饰符，`.once` 修饰符还能被用到自定义的[组件事件](https://cn.vuejs.org/v2/guide/components-custom-events.html)上。 

- .capture： 添加事件监听器时使用事件捕获模式，即内部元素触发的事件先在此处理，然后才交由内部元素进行处理。

  `  <div @click.capture="doThis">...</div>  `

- .self：只当在 event.target 是当前元素自身时触发处理函数， 即事件不是从内部元素触发的。

  `  <div v-on:click.self="doThat">...</div>  `

- .passive：对应 `addEventListener` 中的 `passive` 选项。尤其能够提升移动端的性能。 

  ```javascript
  <!-- 滚动事件的默认行为 (即滚动行为) 将会立即触发 -->
  <!-- 而不会等待 `onScroll` 完成  -->
  <!-- 这其中包含 `event.preventDefault()` 的情况 -->
  <div v-on:scroll.passive="onScroll">...</div>
  ```

- .left：只当点击鼠标左键时触发。
- .right：只当点击鼠标右键时触发。
- .middle：只当点击鼠标中键时触发。

使用修饰符时，顺序很重要；相应的代码会以同样的顺序产生。因此，用 `v-on:click.prevent.self` 会阻止**所有的点击**，而 `v-on:click.self.prevent` 只会阻止对元素自身的点击。 

不要把 `.passive` 和 `.prevent` 一起使用，因为 `.prevent` 将会被忽略，同时浏览器可能会向你展示一个警告。请记住，`.passive` 会告诉浏览器你*不* 想阻止事件的默认行为。 

### v-bind

简写为 `:`  ，用于动态绑定，将字符串解析为对应的变量；如果调用的是方法，方法名后面需要加 () 。v-bind也可以绑定其它的属性：对象、数组、style。

**对象：**

```javascript
<h2 class="title" :class="{active: isActive, line: isLine}"> Hello </h2>
```

（如果过于复杂，可以放在一个methods或者computed中）

```javascript
<h2 class="title" :class="classes"> Hello </h2>
```

**数组：**

```javascript
<h2 class="title" :class=“[‘active’, 'line']"> Hello </h2>
```

（如果过于复杂，可以放在一个methods或者computed中）

#### 修饰符

- .prop：作为一个 DOM property 绑定而不是作为 attribute 绑定。([差别在哪里？](https://stackoverflow.com/questions/6003819/properties-and-attributes-in-html#answer-6004028))
- .camel： 将 kebab-case attribute 名转换为 camelCase， 允许在使用 DOM 模板时将 `v-bind` property 名称驼峰化。例如 `  <svg :view-box.camel="viewBox"></svg>  ` 。
- .sync：语法糖，会扩展成一个更新父组件绑定值的 `v-on` 侦听器。

#### class & style

CSS属性名 驼峰标识（fontSize） 和 ` - `（font-size）[短横线分隔用引号括起来] 都可以

如果状态较多，可以抽出为data或计算属性等。

​	对象语法：

```javascript
<div class="test" :class="{ active: isActive }"></div>
```

```javascript
<div :style="{color: currentColor, fontSize: fontSize + 'px'}"></div>
```

​	数组语法：

```javascript
<div :class="[activeClass, errorClass]"></div>
......
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```

```javascript
<div :style="[baseStyles, overridingStyles]"></div>
```

也可以在数组语法中使用对象语法：

```javascript
<div :class="[{ active: isActive }, errorClass]"></div>
```

#### 多重值

可以为 `style` 绑定中的属性提供一个包含多个值的数组，常用于提供多个带前缀的值。

```javascript
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

这样写只会渲染数组中最后一个被浏览器支持的值。在本例中，如果浏览器支持不带浏览器前缀的 flexbox，那么就只会渲染 `display: flex`。 



### v-for

使用 `item in items` 形式的特殊语法，其中 `items` 是源数据数组，而 `item` 则是被迭代的数组元素的**别名**。 

也可以用 `of` 替代 `in` 作为分隔符，因为它更接近 JavaScript 迭代器的语法。

```javascript
<div v-for="item of items"></div>
```

```javascript
// 数组
<ul id="example-2">
  <li v-for="(item, index) in items"> //  支持一个可选的第二个参数，即当前项的索引
    {{ index }} - {{ item.message }}
  </li>
</ul>
/* 结果：
● 0 - Foo
● 1 - Bar
*/
var example2 = new Vue({
  ...
  data: {
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```

```javascript
// 对象
<ul id="v-for-object" class="demo">
  <li v-for="(value, name, index) in object">
    {{ index }}. {{ name }}: {{ value }} // 第二个的参数为键名；第三个参数作为索引
  </li>
</ul>
/* 结果：
0. title: How to do lists in Vue
1. author: Jane Doe
2. publishedAt: 2016-04-10
*/

new Vue({
  ...
  data: {
    object: {
      title: 'How to do lists in Vue',
      author: 'Jane Doe',
      publishedAt: '2016-04-10'
    }
  }
})
// 在遍历对象时，会按 Object.keys() 的结果遍历，不能保证它的结果在不同的 JavaScript 引擎下都一致。
```

 **`v-for` 也可以接受整数。在这种情况下，它会把模板重复对应次数。** 

```javascript
<div>
  <span v-for="n in 10">{{ n }} </span> // 结果：1 2 3 4 5 6 7 8 9 10
</div>
```

**也可以在 `<template>` 中使用 v-for**

```javascript
<ul>
  <template v-for="item in items" :key="item.id">
    <li>{{ item.msg }}</li>
    <li class="divider" role="presentation"></li>
  </template>
</ul>
```



#### key属性

官方推荐我们在使用v-for时，给对应的元素或组件添加上一个 `:key `属性。

不要使用对象或数组之类的非基本类型值作为 `v-for` 的 `key`。请用字符串或数值类型的值。 

```javascript
<div v-for="item in items" v-bind:key="item.id">
  ......
</div>
```

为什么需要这个key属性呢（了解）？

​		这个其实和Vue的虚拟DOM的Diff算法有关系。

​		这里我们借用[React’s](https://link.zhihu.com/?target=https://calendar.perfplanet.com/2013/diff/)[ diff algorithm](https://link.zhihu.com/?target=https://calendar.perfplanet.com/2013/diff/)中的一张图来简单说明一下：

![1587282688142](Vue.assets/1587282688142.png)

当某一层有很多相同的节点时，也就是列表节点时，我们希望插入一个新的节点

![1587282714060](Vue.assets/1587282714060.png)

​		我们希望可以在B和C之间加一个F，Diff算法默认执行起来是这样的。

![1587282729449](Vue.assets/1587282729449.png)

​		即把C更新成F，D更新成C，E更新成D，最后再插入E，是不是很没有效率？

所以我们需要使用key来给每个节点做一个唯一标识

​		Diff算法就可以正确的识别此节点，找到正确的位置区插入新的节点。

![1587282750329](Vue.assets/1587282750329.png)

所以一句话，key的作用主要是为了高效的更新虚拟DOM。

### v-model

用来实现**表单元素 **` <input> `、` <textarea> `、` <select> ` 和数据的**双向绑定**。 

它会*根据控件类型自动选取正确的方法*来更新元素，但 `v-model` 本质上不过是语法糖。 

`v-model` 会忽略所有表单元素的 `value`、`checked`、`selected` attribute 的初始值，而总是将 Vue 实例的数据作为数据来源。你应该通过 JavaScript 在组件的 `data` 选项中声明初始值。 

```javascript
<div id="app">
	<input type="text" v-model="message">
	<h2>{{message}}</h2> 
</div>
<script src="../js/vue. js"></script>
<script>
	let app = new Vue({
		el: '#app',
		data: {
			message: ''
    }
	})
</script>
/*
	当我们在输入框输入内容时,因为input中的v-model绑定了message，所以会实时将输入的内容传递给message，message发生改变。当message发生改变时，因为上面我们使用Mustache语法，将message的值插入到DOM中，所以DOM会发生响应的改变。
	所以，通过v-model实现了双向的绑定。
*/
```

#### 双向数据绑定的原理

见  源码分析 -> 7.双向数据绑定

**radio：**单选框

![](Vue.assets/22.png)

**checkbox：**多选框

复选框分为两种情况：单个勾选框和多个勾选框

单个勾选框：

​		v-model即为布尔值。

​		此时input的value并不影响v-model的值。

多个复选框：

​		当是多个复选框时，因为可以选中多个，所以对应的data中属性是一个数组。

​		当选中某一个时，就会将input的value添加到数组中。

![](Vue.assets/23.png)

**select：**

和checkbox一样，select也分单选和多选两种情况。

单选：只能选中一个值。

​		v-model绑定的是一个值。

​		当我们选中option中的一个时，会将它对应的value赋值到mySelect中

多选：可以选中多个值。

​		v-model绑定的是一个数组。

​		当选中多个值时，就会将选中的option对应的value添加到数组mySelects中

![](Vue.assets/24.png)

#### 修饰符

- .lazy：默认情况下，v-model默认是在input事件中同步输入框的数据的。也就是说，一旦有数据发生改变对应的data中的数据就会自动发生改变。lazy修饰符可以让数据在*失去焦点*或者*回车*时才会更新。


- .number：在输入框中输入的内容自动转成数字类型。默认情况下，无论在输入框中输入的是字母还是数字，都会被当做字符串类型进行处理。


- .trim：过滤内容左右两边的空格。


#### 用于组件

```javascript
<input v-model="searchText">
// 当用在组件上时，v-model 则会这样：
<custom-input
  :value="searchText"
  @input="searchText = $event"
></custom-input>
```

为了让它正常工作，这个组件内的 ` <input> ` 必须：

- 将其 `value` 绑定到一个名叫 `value` 的 prop 上
- 在其 `input` 事件被触发时，将新的值通过自定义的 `input` 事件抛出

```javascript
Vue.component('custom-input', {
  props: ['value'],
  template: `
    <input
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)"
    >
  `
})
// 现在 v-model 就应该可以在这个组件上完美地工作起来了：
<custom-input v-model="searchText"></custom-input>
```



### 其他

- **v-once**：后面不需要跟表达式，元素和组件只渲染一次，之后不会跟着数据的改变而改变。

- **v-html：**如果服务器返回的是HTML代码，v-html=”……”会将string的html解析出来并进行渲染。

- **v-text**：和Mustache的作用比较相似，将数据显示在界面中，但不够灵活，一般直接用Mustache。

- **v-pre：**{{message}}直接显示为文本，而不进行解析。

- **v-cloak**：vue解析之前存在，但解析之后会自动删除v-cloak指令，因此可用于进行临时性的操作，例如添加临时性的样式，vue解析之后v-cloak指令消失从而样式也消失。

- **@load**：监听图片加载完成。

  ```javascript
  <img :src="..." alt="" @load="imageLoad">
  ....
  methods: {
    imageLoad() {
      ...
    }
  }
  ```

- **动态参数**：用方括号括起来的 JavaScript 表达式作为一个指令的参数

  ```javascript
  <a :[attributeName]="url"> ... </a>
  // 这里的 attributeName 会被作为一个 JavaScript 表达式进行动态求值，求得的值将会作为最终的参数来使用。例如，如果你的 Vue 实例有一个 data property attributeName，其值为 "href"，那么这个绑定将等价于 v-bind:href。
  
  // 同样地，你可以使用动态参数为一个动态的事件名绑定处理函数：
  <a @[eventName]="doSomething"> ... </a>
  // 当 eventName 的值为 "focus" 时，v-on:[eventName] 将等价于 v-on:focus。
  ```


### 自定义指令

#### 基本

**注册全局指令：**

```javascript
// 注册一个全局自定义指令 `v-focus`  指令名不包含 v-
Vue.directive('focus', function (el, binding) { 
  // el：指令属性所在的标签对象；binding：包含指令相关信息数据的对象
  console.log(binding)
	......
})
```

**注册局部指令：**

局部指令只在所注册的Vue实例中有效。

```javascript
new Vue({
  ...
  directives: {
    // 如果指令名包含 - 等字符则需要用引号包裹
    // 'fo-cos': ...
    focus: function (el, binding) { 
      // el：指令属性所在的标签对象；binding：包含指令相关信息数据的对象
      ......
    }
  }
})
  
// 使用
<input v-focus>
```

#### 钩子函数(生命周期函数)

一个指令定义对象可以提供如下几个钩子函数(生命周期函数) (均为可选)： 

- `bind`：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
- `inserted`：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
- `update`：所在组件的 VNode 更新时调用，**但是可能发生在其子 VNode 更新之前**。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新 (详细的钩子函数参数见下)。

- `componentUpdated`：指令所在组件的 VNode **及其子 VNode** 全部更新后调用。
- `unbind`：只调用一次，指令与元素解绑时调用。

#### 钩子函数参数

- `el`：指令所绑定的元素，可以用来直接操作 DOM。
- `binding`：一个对象，包含以下 property：
  - `name`：指令名，不包括 `v-` 前缀。
  - `value`：指令的绑定值，例如：`v-my-directive="1 + 1"` 中，绑定值为 `2`。
  - `oldValue`：指令绑定的前一个值，仅在 `update` 和 `componentUpdated` 钩子中可用。无论值是否改变都可用。
  - `expression`：字符串形式的指令表达式。例如 `v-my-directive="1 + 1"` 中，表达式为 `"1 + 1"`。
  - `arg`：传给指令的参数，可选。例如 `v-my-directive:foo` 中，参数为 `"foo"`。
  - `modifiers`：一个包含修饰符的对象。例如：`v-my-directive.foo.bar` 中，修饰符对象为 `{ foo: true, bar: true }`。
- `vnode`：Vue 编译生成的虚拟节点。移步 [VNode API](https://cn.vuejs.org/v2/api/#VNode-接口) 来了解更多详情。
- `oldVnode`：上一个虚拟节点，仅在 `update` 和 `componentUpdated` 钩子中可用。

除了 `el` 之外，其它参数都应该是只读的，切勿进行修改。如果需要在钩子之间共享数据，建议通过元素的 [`dataset`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/dataset) 来进行。 

```javascript
// 这是一个使用了这些 property 的自定义钩子样例： 
<div id="hook-arguments-example" v-demo:foo.a.b="message"></div>

Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify
    el.innerHTML =
      'name: '       + s(binding.name) + '<br>' +
      'value: '      + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: '   + s(binding.arg) + '<br>' +
      'modifiers: '  + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  },
  
})

new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'hello!'
  }
})
```

##### 动态指令参数

指令的参数可以是动态的。例如，在 `v-mydirective:[argument]="value"` 中，`argument` 参数可以根据组件实例数据进行更新！这使得自定义指令可以在应用中被灵活使用。 

```javascript
// 例如你想要创建一个自定义指令，用来通过固定布局将元素固定在页面上。我们可以像这样创建一个通过指令值来更新竖直位置像素值的自定义指令：
<div id="baseexample">
  <p>Scroll down the page</p>
  <p v-pin="200">Stick me 200px from the top of the page</p>
</div>

Vue.directive('pin', {
  // 声明周期函数
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    el.style.top = binding.value + 'px'
  },
  inserted: function () { ... },
  update: function () { ... },
  componentUpdated: function () { ... },
  unbind: function () { ... }
})

new Vue({
  el: '#baseexample'
})

// 这会把该元素固定在距离页面顶部 200 像素的位置。但如果场景是我们需要把元素固定在左侧而不是顶部又该怎么办呢？这时使用动态参数就可以非常方便地根据每个组件实例来进行更新。
<div id="dynamicexample">
  <h3>Scroll down inside this section ↓</h3>
  <p v-pin:[direction]="200">I am pinned onto the page at 200px to the left.</p>
</div>

Vue.directive('pin', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    var s = (binding.arg == 'left' ? 'left' : 'top')
    el.style[s] = binding.value + 'px'
  }
})

new Vue({
  el: '#dynamicexample',
  data: function () {
    return {
      direction: 'left'
    }
  }
})
```

#### 函数简写

在很多时候，你可能想在 `bind` 和 `update` 时触发相同行为，而不关心其它的钩子。比如这样写： 

```javascript
Vue.directive('color-swatch', function (el, binding) {
  el.style.backgroundColor = binding.value
})
```

#### 对象字面量

如果指令需要多个值，可以传入一个 JavaScript 对象字面量。记住，指令函数能够接受所有合法的 JavaScript 表达式。 

```javascript
<div v-demo="{ color: 'white', text: 'hello!' }"></div>

Vue.directive('demo', function (el, binding) {
  console.log(binding.value.color) // => "white"
  console.log(binding.value.text)  // => "hello!"
})
```



## 计算属性 computed

避免在模板中放入太多的逻辑从而让模板过重且难以维护。 

计算"属性"的调用不需要加 () [因为只调用的getter方法]。

计算属性有缓存，如果多次使用时，会立即返回之前的计算结果，而不必再次执行函数。而方法则没有缓存机制，会再次调用。因此多使用计算属性而不是方法。

在某些情况，我们可能需要对数据进行一些转化后再显示，或者需要将多个数据结合起来进行显示

![](Vue.assets/17.png)

每个计算属性都包含一个getter和一个setter

​		在上面的例子中，我们只是使用getter来读取。

​		在某些情况下，你也可以提供一个setter方法（一般setter方法不需要指定）。

​		在需要写setter的时候，代码如下：

![](Vue.assets/18.png)

调用 `fullName` 时，setter会被调用。



**完整写法：**

```javascript
computed: {
	fullName: {
		set: function () { //计算属性一般不设set方法，作为只读属性
			console.1og()
		},
		get: function () {
			return console.1og()
		}
	}
}
```

**增强写法：**函数名 () { … }  

```javascript
computed: {
	showGoods() {
		return this . goods [this . currentType] .list
	}
}
```

**ES5语法：**

```javascript
fullName: function() {
	return this.firstName + ‘ ‘ + this.lastName
}
```

**ES6语法：**省略 `:function` 

```javascript
fullName() {
	return this.firstName + ‘ ‘ + this.lastName
}
```



##  watch

`watch` 提供了一个更通用的方法，来响应数据的变化。当需要在数据变化时执行异步或开销较大的操作时，这个方式是最有用的。 

```javascript
<div id="watch-example">
  <input v-model="question">
</div>
```

```javascript
var watchExampleVM = new Vue({
  ...
  data: {
    question: '',
  },
  watch: {
    // question值 发生改变时这个函数就会运行
    question: function (newValue, oldValue) {
      ......
    }
  },
})
```



## 过滤器

过滤器可以用在两个地方：**双花括号插值和 `v-bind` 表达式** 。过滤器应该被添加在 JavaScript 表达式的尾部，由“管道”符号指示： 

```javascript
<!--  -->
{{ message | showPrice }} // 在双花括号中

<!-- 在 `v-bind` 中 -->
<div v-bind:id="rawId | showPrice"></div> // 

// 在组件的选项中定义本地的过滤器 
data: { … },
filters: { // 定义过滤器
	showPrice (price) {
		return '￥' + price.toFixed(2)
	}
}

// 当全局过滤器和局部过滤器重名时，以局部过滤器优先。

// 创建 Vue 实例之前全局定义过滤器
Vue.filter('capitalize', function (value) {
  ...
})
// 过滤器要在Vue实例之前
new Vue({ ... })
         
// 过滤器可以串联： 
{{ message | filterA | filterB }}
// message完成filterA的过滤后的结果再进行filterB的过滤
        
// 过滤器是 JavaScript 函数，因此可以接收参数：
{{ message | filterA('arg1', arg2) }}
```



## 组件化开发

组件是可复用的 Vue 实例，所以它们与 `new Vue` 接收相同的选项，例如 `data`、`computed`、`watch`、`methods` 以及生命周期钩子等。仅有的例外是像 `el` 这样根实例特有的选项。 

### 注册组件

1. 创建组件构造器对象。调用Vue.extend()方法。

2. 注册组件。调用Vue.component()方法。

3. 使用组件。在Vue实例的作用范围内使用组件。

![](Vue.assets/333-1587213543172.jpg)

**1.Vue.extend()：**

​		调用Vue.extend()创建的是一个组件构造器。 

​		通常在创建组件构造器时，传入template代表我们自定义组件的模板。

​		该模板就是在使用到组件的地方，要显示的HTML代码。

​		事实上，这种写法在Vue2.x的文档中几乎已经看不到了，它会直接使用下面我们会讲到的语法糖，但是在很多资料还是会提到这种方式，而且这种方式是学习后面方式的基础。

**2.Vue.component()：**

​		调用Vue.component()是将刚才的组件构造器注册为一个组件，并且给它起一个组件的标签名称。

​		所以需要传递两个参数：1、注册组件的标签名 2、组件构造器

**3.组件必须挂载在某个Vue实例下，否则它不会生效。**（见下页）

​		我们来看下面我使用了三次`<my-cpn></my-cpn>`

​		而第三次其实并没有生效：

![](Vue.assets/25.png)

#### 全局组件和局部组件

**全局组件：**

当调用`Vue.component()`注册组件时，组件的注册是全局的，表示该组件可以在任意Vue实例和其他的子组件中也可以使用。

在Vue实例中创建的组件是局部组件，局部组件只能在其注册的Vue实例中使用。

**局部组件：**

如果我们注册的组件是挂载在某个实例中, 那么就是一个局部组件，即在某个Vue实例的`components`中注册另一个，但在另一个Vue实例中则需要再次注册，否则无法使用。局部注册的组件在其他的组件中也需要再次注册。

#### 模块系统中局部注册

创建一个文件夹，专门用于存放组件文件，并通过`import`导入并注册。

```javascript
import ComponentA from './ComponentA'
import ComponentC from './ComponentC'

export default {
  components: {
    ComponentA,
    ComponentC
  },
  // ...
}
```

#### 基础组件的自动化全局注册

**基础组件**：相对通用的组件，会在各个组件中被频繁的用到。例如只是包裹了一个输入框或按钮之类的元素。

基础组件的注册会使`components`有一个很长的列表。或在不同的组件页面频繁调用。 

使用 webpack (或在内部使用了 webpack 的 [Vue CLI 3+](https://github.com/vuejs/vue-cli))，就可以使用 `require.context` 只全局注册这些非常通用的基础组件。

一般将这些通用基础组件放置在同一个文件夹下，方便匹配这些组件的路径。

以下为一份示例代码，可以在应用入口文件 (比如 `src/main.js`) 中全局导入基础组件的： 

**全局注册的行为必须在根 Vue 实例 (通过 `new Vue`) 创建之前发生**。 

```javascript
import Vue from 'vue'
import upperFirst from 'lodash/upperFirst'
import camelCase from 'lodash/camelCase'

const requireComponent = require.context(
  './components', // 其组件目录的相对路径
  false, // 是否查询其子目录
  /Base[A-Z]\w+\.(vue|js)$/ // 匹配基础组件文件名的正则表达式
)

requireComponent.keys().forEach(fileName => {
  const componentConfig = requireComponent(fileName) // 获取组件配置
  const componentName = upperFirst( // 获取组件的 PascalCase 命名
    camelCase( // 获取和目录深度无关的文件名
      fileName
        .split('/')
        .pop()
        .replace(/\.\w+$/, '')
    )
  )

  // 全局注册组件
  Vue.component(
    componentName,
    // 如果这个组件选项是通过 `export default` 导出的，
    // 那么就会优先使用 `.default`，
    // 即，通过export default导出的单文件组件就使用componentConfig.default,否则就使用componentConfig。
    // 否则回退到使用模块的根。
    componentConfig.default || componentConfig
  )
})
```

```javascript
// 在相应的组件页面直接使用注册好的全局组件：
<base-xxxx :title="'自动化全局注册基础组件'"></base-xxxx>
```

#### 注册组件的语法糖

主要是省去了调用Vue.extend()的步骤，而是可以直接使用一个对象来代替。

**原来的写法：**

```javascript
// 1.创建组件构造器
const cpn1 = Vue.extend({
	template:
		<div>
			<h2>我是标题1</h2>
			<p>我是内容，哈哈哈哈</p>
		</div>
})
	
// 2.注册组件
Vue. component('cpn1', cpn1)
```

**全局组件语法糖的写法：**

```javascript
//全局语法糖写法
Vue.component('cpn1', { // 第一个参数为组件名
	......
})
```

**局部组件语法糖写法：**

```javascript
//局部组件语法糖写法:
const app = new Vue({
	el: ' #app',
	components: {
		'cpn2': { // 一般都是将内容抽离出去，见下面的模板分离写法
			template:
				<div>
					<h2>我是标题1< /h2>
					<p>我是内容，哈哈哈哈</p>
				</div>
		},
    'component-a': ComponentA,
    ...
	}
})
```

**但是内部还是调用的`.extend()`**

#### 模板的分离写法

语法糖简化Vue组件的注册过程，分离写法则用于简化template模块写法，将JS代码中的HTML部分抽离出去。

**1.使用`<script>`标签：**

![](Vue.assets/666.jpg)

**2.使用`<template>`标签：**

![](Vue.assets/777.jpg)

#### 组件命名规范

**使用 kebab-case：**

W3C规范：自定义组件名字母全小写且包含一个连字符。例如，`my-component-name` ，当使用 kebab-case (短横线分隔命名) 定义一个组件时，你也必须在引用这个自定义元素时使用 kebab-case。

**使用 PascalCase：**

例如，`MyComponentName ` ，当使用 PascalCase (首字母大写命名) 定义一个组件时，你在引用这个自定义元素时两种命名法都可以使用。也就是说 ` <my-component-name> ` 和 ` <MyComponentName> ` 都是可接受的。注意，尽管如此，直接在 DOM (即非字符串的模板) 中使用时只有 kebab-case 是有效的。 



### 组件数据存放

组件是一个单独功能模块的封装，因此应当有属于自己的HTML模板和自己的数据data。

组件中不能直接访问Vue实例中的data。而且即使可以访问，如果将所有的数据都放在Vue实例中，Vue实例就会变的非常臃肿。

组件对象有自己的data属性和methods等其他属性。

#### 为什么组件中的data必须是一个函数

我们希望每个组件实例都有自己独立的唯一数据，而不是所有这些组件都共享相同的数据。

当使用了多个组件时，每个组件都应该有自己的数组。（值传递，而不是引用传递）

data不直接使用对象，而是用函数返回一个对象，对象内部保存着数据。

Vue让每个组件对象都返回一个新的对象，如果是同一个对象的，组件在多次使用后会相互影响。

如果data是对象而不是函数，则每个组件都指向同一个对象，当组件1对数据进行修改时，组件2再次拿到的数据是经过组件1修改后的数据（**引用传递会对组件间的数据会相互干扰**）。

**如果data是对象和不是函数，则如果创建了多个组件实例，则多个组件实例会共用同一个data。**

**data使用函数返回一个新的对象时，每个组件都会有属于自己的对象。**这个函数返回一个对象，对象内部保存着数据。



### 父子组件

#### 概念

将组件1（子组件）在组件2（父组件）中进行注册，组件2在app中进行注册，则在app中能显示嵌套在组件2中的组件1，但是不能直接在app中使用组件1。

![](Vue.assets/555.jpg)

#### 父子组件间的通信

父组件通过 `props` 向子组件传递数据

子组件通过 事件 向父组件发送消息

![1587301541908](Vue.assets/1587301541908.png)

##### 父 --> 子

使用`props`来声明从父级接收到的数据。*任何*类型的值都可以传给一个 prop。 

props的值有两种方式：

​		1.字符串数组，数组中的字符串就是传递时的名称。

​		2.对象，对象可以设置传递时的类型，也可以设置默认值等。

![](Vue.assets/888.jpg)

###### props数据验证

当 prop 验证失败的时候，(开发环境构建版本的) Vue 将会产生一个控制台的警告。

注意那些 prop 会在一个组件实例创建**之前**进行验证，所以实例的属性 (如 `data`、`computed` 等) 在 `default` 或 `validator` 函数中是不可用的。 

```javascript
Vue.component('my-component'，{
	props:{ // 对象形式
    propa: Number, // 基础的类型检查( nu11’ 匹配任何类型)
    propB: [string, Number], // 多个可能的类型
    propC: {
    	type: String,
    	required: true // 必填的字符串
    }，
    propD: {
      type: Number,
      default: 100 // 带有默认值的数字
    },
    propE: {
   		type: object, // 带有默认值的对象
    	default() { // 对象或数组默认值必须从一个工厂函数获取（即必须用函数的写法）
    		return { message: 'hello' }
   		}
		}，
  	propF: {
			validator(value) { //自定义验证函数
				return ['success', 'warning'， ' danger'].index0f(value) !== -1
    	}
  	}
	}
})
```

```javascript
// 也可以是自定义的类型
function Person (firstName, 1astName) {
  this.firstName = firstName
  this.lastName = 1astName
}

Vue.component('blog-post'，{
	props: {
		author: Person
	}
})
```

###### prop的单向数据流

prop是单向数据流，此外，每次父级组件发生变更时，子组件中所有的 prop 都将会刷新为最新的值。这意味着你**不**应该在一个子组件内部改变prop。 总之，不要直接去操作prop，而定义一个data或computed等在接收prop后再操作data或computed。

注意：在 JavaScript 中对象和数组是通过引用传入的，所以对于一个数组或对象类型的 prop 来说，在子组件中改变变更这个对象或数组本身**将会**影响到父组件的状态。 

```javascript
// 1.prop 用来传递一个初始值;子组件将其作为一个本地的 prop 数据来使用。
props: ['initialCounter'],
data: function () {
  return {
    counter: this.initialCounter // 定义一个本地的 data 属性并将这个 prop 用作其初始值
  }
}

// 2.prop 以一种原始的值传入且需要进行转换。
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase() // 使用这个 prop 的值来定义一个计算属性
  }
}
```

###### 非Prop的Attribute

**（不全，看官网）**

非 prop 特性：指它可以直接传入组件，而不需要定义相应的 prop。 

尽管为组件定义明确的 prop 是推荐的传参方式，组件的作者却并不总能预见到组件被使用的场景。所以，组件可以接收任意传入的特性，这些特性都会被添加到组件的根元素上。 

例如，假设我们使用了第三方组件 `bs-date-input`，它包含一个 Bootstrap 插件，该插件需要在 `input` 上添加 `data-3d-date-picker` 这个特性。这时可以把特性直接添加到组件上 (不需要事先定义 `prop`)： 

```javascript
<bs-date-input data-3d-date-picker="true"></bs-date-input>
// 然后这个 data-3d-date-picker="true" attribute 就会自动添加到 <bootstrap-date-input> 的根元素上。
```

```javascript
Vue.component('bs-date-input', {
    ...
    mounted: function () {
      // this.$el就是组件的根元素，getAttribute已经是JS的标准语法了：取得标签上属性的值。
      console.log(this.$el.getAttribute('data-3d-date-picker'); // 使用
    }
});
```

**禁用 Attribute 继承：**

```javascript
Vue.component('my-component', {
  inheritAttrs: false,
  ...
})
```



##### 子 --> 父

在子组件中，通过`$emit()`来触发事件。在父组件中，通过`v-on`来监听子组件事件。

​		第二个参数为子组件传递的值，在父组件中用 ` $event ` 来获取。

​		如果子组件的事件处理函数是一个方法，那么这个值将会作为第一个参数传入这个方法。

![](Vue.assets/14.png)

#### 父子组件的访问方式

##### $parent

不推荐使用。

如果我们想在子组件中直接访问父组件，可以通过 `$parent` 。

注意事项：

​		尽管在Vue开发中，我们允许通过$parent来访问父组件，但是在真实开发中尽量不要这样做。

​		子组件应该尽量避免直接访问父组件的数据，因为这样耦合度太高了。

​		如果我们将子组件放在另外一个组件之内，很可能该父组件没有对应的属性，往往会引起问题。

​		不要通过$parent直接修改父组件的状态，否则父组件中的状态会变得混乱，很不利于调试和维护。

![](Vue.assets/15.png)

##### $children

不推荐使用。

父组件访问子组件：使用$children或$refs reference(引用)

在实际开发中，一般不会用`$children`，而是使用`$refs`，因为`$children`的下标值可能会发生改变。

$children：

​		this.$children是一个数组类型，它包含所有子组件对象。

​		我们这里通过一个遍历，取出所有子组件的message状态。

![](Vue.assets/13.png)

##### $refs

$children的缺陷：

​		通过$children访问子组件时，是一个数组类型，访问其中的子组件必须通过索引值。

​		但是当子组件过多，我们需要拿到其中一个时，往往不能确定它的索引值，甚至还可能会发生变化。

​		有时候，我们想明确获取其中一个特定的组件，这个时候就可以使用$refs。

$refs的使用：

​		$refs和ref指令通常是一起使用的。

​		首先，我们通过ref给某一个子组件绑定一个特定的ID。

​		其次，通过this.$refs.ID就可以访问到该组件了。

```javascript
<chi1d-cpn1 ref="child1"></child-cpnl>
<chi1d-cpn2 ref="child2"></child-cpn2>
<button @click="showRefsCpn">通过refs访问子组件</button>

showRefsCpn() {
	console.1og(this.$refs.child1.message);
	console.1og(this.$refs.child2.message);
}
```

##### $root

用于访问根组件，但在模块化开发中，Vue实例往往很少有东西。

#### 非父子组件通信

1. 采用 **事件总线** 的方式

   ```javascript
   // main.js
   ....
   Vue.prototype.$bus = new Vue() // new 一个 Vue 实例，作为事件总线
   ....
   ```

   ```javascript
   // 在一个组件中emit事件
   methods: {
     imageLoad() {
       this.$bus.$emit('itemImageLoad') // 需要先在 main.js 文件中定义事件总线
     }
   }
   ```

   ```javascript
   // 在另一个组件中接收事件
   created() {
     this.$bus.$on('itemImageLoad', () => {
       this.$refs.scroll.refresh()
     })
   }
   ```

2. 使用 **Vuex**



### 动态组件

在不同组件之间进行动态切换。例如：![1587648558171](Vue.assets/1587648558171.png)

可以通过 is 来实现： `<component :is="xxx"></component>`

`xxx`  可以包括：

- 已注册组件的名字，或
- 一个组件的选项对象

这个 attribute 可以用于常规 HTML 元素，但这些元素将被视为组件，这意味着所有的 attribute **都会作为 DOM attribute 被绑定**。对于像 `value` 这样的 property，若想让其如预期般工作，你需要使用 [`.prop` 修饰器](https://cn.vuejs.org/v2/api/#v-bind)。 

#### 解析DOM模板时的注意事项

有些 HTML 元素，诸如 `<ul>`、`<ol>`、`<table>` 和 `<select>`，对于哪些元素可以出现在其内部是有严格限制的。而有些元素，诸如 `<li>`、`<tr>` 和 `<option>`，只能出现在其它某些特定的元素内部。

这会导致我们使用这些有约束条件的元素时遇到一些问题。例如：

```javascript
<table>
  <blog-post-row></blog-post-row>
</table>
```

这个自定义组件 ` <blog-post-row> ` 会被作为无效的内容提升到外部，并导致最终渲染结果出错。幸好这个特殊的 `is` attribute 给了我们一个变通的办法：

```javascript
<table>
  <tr is="blog-post-row"></tr>
</table>
```

需要注意的是**如果我们从以下来源使用模板的话，这条限制是\*不存在\*的**：

- 字符串 (例如：`template: '...'`)
- 单文件组件（.vue）
- `<script type="text/x-template">`

#### keep-alive

避免组件切换后被销毁并重新创建，而希望其保持切换前的状态。可以用一个 ` <keep-alive> `元素将其动态组件包裹起来：

```javascript
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

注意这个 ` <keep-alive> ` 要求被切换到的组件都有自己的名字，不论是通过组件的 `name`选项还是 局部/全局注册。 

### 异步组件（懒加载）

可能需要将应用分割成小一些的代码块，并且只在需要的时候才从服务器加载一个模块。为了简化，Vue 允许你以一个工厂函数的方式定义你的组件，这个工厂函数会异步解析你的组件定义。Vue 只有在这个组件需要被渲染的时候才会触发该工厂函数，且会把结果缓存起来供未来重渲染。例如： 

```javascript
Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    // 向 `resolve` 回调传递组件定义
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})
```

这个工厂函数会收到一个 `resolve` 回调，这个回调函数会在你从服务器得到组件定义的时候被调用。你也可以调用 `reject(reason)` 来表示加载失败。这里的 `setTimeout` 是为了演示用的，如何获取组件取决于你自己。一个推荐的做法是将异步组件和 [webpack 的 code-splitting 功能](https://webpack.js.org/guides/code-splitting/)一起配合使用： 

```javascript
Vue.component('async-webpack-example', function (resolve) {
  // 这个特殊的 `require` 语法将会告诉 webpack
  // 自动将你的构建代码切割成多个包，这些包
  // 会通过 Ajax 请求加载
  require(['./my-async-component'], resolve)
})
```

也可以在工厂函数中返回一个 `Promise`，所以把 webpack 2 和 ES2015 语法加在一起，我们可以写成这样：

```javascript
Vue.component(
  'async-webpack-example',
  // 这个 `import` 函数会返回一个 `Promise` 对象。
  () => import('./my-async-component')
)
```

当使用局部注册的时候，你也可以直接提供一个返回 `Promise` 的函数： 

```javascript
new Vue({
  // ...
  components: {
    'my-component': () => import('./my-async-component')
  }
})
```

#### 处理加载状态

这里的异步组件工厂函数也可以返回一个如下格式的对象： 

```javascript
const AsyncComponent = () => ({
  // 需要加载的组件 (应该是一个 `Promise` 对象)
  component: import('./MyComponent.vue'),
  // 异步组件加载时使用的组件
  loading: LoadingComponent,
  // 加载失败时使用的组件
  error: ErrorComponent,
  // 展示加载时组件的延时时间。默认值是 200 (毫秒)
  delay: 200,
  // 如果提供了超时时间且组件加载也超时了，
  // 则使用加载失败时使用的组件。默认值是：`Infinity`
  timeout: 3000
})
```

### 处理边界情况

#### 访问元素 & 组件

##### 访问根实例

在大多数情况下，最好不要触达另一个组件实例内部或手动操作 DOM 元素。不过也确实在一些情况下做这些事情是合适的。 

通过 `$root` 可以访问根实例(根组件)。对于小型项目来说这很方便，但对于大型项目，推荐使用Vuex。

```javascript
// Vue 根实例
new Vue({
  data: { foo: 1 },
  computed: { bar: function (){ /* ... */ } },
  methods: { baz: function (){ /* ... */ } }
})
// 访问
this.$root.foo // 获取根组件的数据
this.$root.foo = 2 // 写入根组件的数据
this.$root.bar // 访问根组件的计算属性
this.$root.baz() // 调用根组件的方法
```

##### 访问父组件

同样的，通过 `$parent` 可以从一个子组件访问父组件的实例。可以在后期随时触达父级组件，以替代将数据以 prop 的方式传入子组件的方式。 

##### 访问子组件

通过 `ref` 为子组件赋予一个 ID 引用，通过 `$refs` 来使用。 

`$refs` 只会在组件渲染完成之后生效，并且它们不是响应式的。这仅作为一个用于直接操作子组件的“逃生舱”——你应该避免在模板或计算属性中访问 `$refs`。 

```javascript
<base-input ref="usernameInput"></base-input> // 定义

this.$refs.usernameInput // 使用
```

##### 依赖注入

$parent 无法很好的扩展到更深层级的嵌套组件上，例如有时需要 $parent.$parent，甚至更多。

这也是依赖注入的用武之地，不用关心DOM层级，它用到了两个新的实例选项：`provide` 和 `inject`。

`provide` 和 `inject` 主要为高阶插件/组件库提供用例。并不推荐直接用于应用程序代码中。  

`provide` 选项允许我们指定我们想要**提供**给后代组件的数据/方法。 

```javascript
// 父组件中定义provide
...
data: {...}
provide: function () {
  return {
    getMap: this.getMap
  }
}
```

然后在任何后代组件里，我们都可以使用 `inject` 选项来接收指定的我们想要添加在这个实例上的 property：

```javascript
// 子组件中使用inject
...
data: {...}
inject: ['getMap']
created: function () {
  var vm = this
  vm.getMap(...) // 直接使用
}
```

相比 `$parent` 来说，这个用法可以让我们在*任意*后代组件中访问 `getMap`，而不需要暴露整个 ` <google-map> ` 实例。这允许我们更好的持续研发该组件，而不需要担心我们可能会改变/移除一些子组件依赖的东西。同时这些组件之间的接口是始终明确定义的，就和 `props` 一样。

实际上，你可以把依赖注入看作一部分“大范围有效的 prop”，除了：

- 祖先组件不需要知道哪些后代组件使用它提供的 property
- 后代组件不需要知道被注入的 property 来自哪里

然而，依赖注入还是有负面影响的。它将你应用程序中的组件与它们当前的组织方式耦合起来，使重构变得更加困难。同时所提供的 property 是非响应式的。这是出于设计的考虑，因为使用它们来创建一个中心化规模化的数据跟使用$root可能需要换用一个像 Vuex 这样真正的状态管理方案了。 

#### 程序化的事件侦听器

其实就是比方说创建实例和清除实例放在一起。

某些第三方插件必须在当前组件卸载后清除该实例（比如说百度的富文本框UEditor如果不清除再次在下个组件使用时会有bug，类似于小程序的语音实例，必须离开页面的时候销毁当前语音实例，不然语音会一直播放）

方案一：

```javascript
data() {            
    return {                              
        timer: null  // 定时器名称          
    }        
},
  
// 使用定时器：
this.timer = setIterval (() => {
  ...
}, 1000)

// 最后在beforeDestroy()生命周期内清除定时器：
beforeDestroy() {
    clearInterval(this.timer);        
    this.timer = null;
}
/* 以上这种方案有两点不好的地方：
(1)它需要在这个组件实例中保存这个数据timer，这是多余的。
(2)我们的建立定时器代码独立于我们的清理代码（定时器需要卸载页面的时候卸载），这使得我们比较难于程序化的清理我们建立的所有东西（意思是执行代码和清除代码不在一起）*/
```

方案二：该方法是通过$once这个事件侦听器在定义完定时器之后的位置来清除定时器。 

```javascript
mounted: function () {
  const timer = setInterval(() =>{                    
      // 某些定时器操作                
  }, 500);            
  // 通过$once来监听定时器，在beforeDestroy钩子可以被清除。
  this.$once('hook:beforeDestroy', () => {            
      clearInterval(timer);                                    
  })
}

```

简单来说就是把所有创建实例和需要销毁的实例代码放在一起了，放在一起而已(创建实例和销毁实例)........

甚至可以在一个页面开启多个这种创建实例和销毁实例

```javascript
mounted: function () {
  this.attachDatepicker('startDateInput')
  this.attachDatepicker('endDateInput')
},
methods: {
  attachDatepicker: function (refName) {
    var picker = new Pikaday({
      field: this.$refs[refName],
      format: 'YYYY-MM-DD'
    })

    this.$once('hook:beforeDestroy', function () {
      picker.destroy()
    })
  }
}
```

- 通过 `$on(eventName, eventHandler)` 侦听一个事件
- 通过 `$once(eventName, eventHandler)` 一次性侦听一个事件
- 通过 `$off(eventName, eventHandler)` 停止侦听一个事件

当你需要在一个组件实例上手动侦听事件时，它们是派得上用场的。它们也可以用于代码组织工具。 

#### 循环引用

##### 递归组件

```javascript
// 组件是可以在它们自己的模板中调用自身的。不过它们只能通过 `name` 选项来做这件事：
name: 'unique-name-of-my-component'

// 当你使用 Vue.component 全局注册一个组件时，这个全局的 ID 会自动设置为该组件的 name 选项。
Vue.component('unique-name-of-my-component', {
  // ...
})

// 稍有不慎，递归组件就可能导致无限循环：
name: 'stack-overflow',
template: '<div><stack-overflow></stack-overflow></div>'

// 类似上述的组件将会导致“max stack size exceeded”错误，所以请确保递归调用是条件性的 (例如使用一个最终会得到 false 的 v-if)。
```

##### 组件之间的循环引用

```javascript
// <tree-folder> 组件
<p>
  <span>{{ folder.name }}</span>
  <tree-folder-contents :children="folder.children"/>
</p>

// <tree-folder-contents> 组件
<ul>
  <li v-for="child in children">
    <tree-folder v-if="child.children" :folder="child"/>
    <span v-else>{{ child.name }}</span>
  </li>
</ul>
```

这些组件在渲染树中互为对方的后代*和*祖先——一个悖论！当通过 `Vue.component` 全局注册组件的时候，这个悖论会被自动解开。如果你是这样做的，那么你可以跳过这里。 

然而，如果你使用一个*模块系统*依赖/导入组件，例如通过 webpack 或 Browserify，你会遇到一个错误： 

```
Failed to mount component: template or render function not defined.
```

把两个组件称为 A 和 B。模块系统发现它需要 A，但是首先 A 依赖 B，但是 B 又依赖 A，但是 A 又依赖 B，如此往复。这变成了一个循环，不知道如何不经过其中一个组件而完全解析出另一个组件。为了解决这个问题，我们需要给模块系统一个点，在那里“A *反正*是需要 B 的，但是我们不需要先解析 B。” 

在我们的例子中，把 ` <tree-folder> ` 组件设为了那个点。我们知道那个产生悖论的子组件是 ` <tree-folder-contents> ` 组件，所以我们会等到生命周期钩子 `beforeCreate` 时去注册它： 

```javascript
beforeCreate: function () {
  this.$options.components.TreeFolderContents = 
    										require('./tree-folder-contents.vue').default
}
```

或者，在本地注册组件的时候，你可以使用 webpack 的异步 `import`： 

```javascript
components: {
  TreeFolderContents: () => import('./tree-folder-contents.vue')
}
```

这样问题就解决了！ 

#### 模板定义的替代品

##### 内联模板

当 `inline-template` 出现在一个子组件上时，这个组件将会使用其里面的内容作为模板，而不是将其作为被分发的内容。内联模板需要定义在 Vue 所属的 DOM 元素内。 

```javascript
<my-component inline-template>
  <div>
    <p>These are compiled as the component's own template.</p>
    <p>Not parent's transclusion content.</p>
  </div>
</my-component>
```

`inline-template` 会让模板的作用域变得更加难以理解。所以作为最佳实践，请在组件内优先选择 `template` 选项或 `.vue` 文件里的一个 ` <template> ` 元素来定义模板。 

##### X-Template

另一个定义模板的方式是在一个 `  <script>  ` 元素中，并为其带上 `text/x-template` 的类型，然后通过一个 id 将模板引用过去。 x-template 需要定义在 Vue 所属的 DOM 元素外。 ： 

```javascript
<script type="text/x-template" id="hello-world-template">
  <p>Hello hello hello</p>
</script>

Vue.component('hello-world', {
  template: '#hello-world-template'
})
```

这些可以用于模板特别大的 demo 或极小型的应用，但是其它情况下请避免使用，因为这会将模板和该组件的其它定义分离开。 

#### 控制更新

##### 强制更新

如果你发现你自己需要在 Vue 中做一次强制更新，99.9% 的情况，是你在某个地方做错了事。

然而，如果你已经做到了上述的事项仍然发现在极少数的情况下需要手动强制更新，那么你可以通过 [`$forceUpdate`](https://cn.vuejs.org/v2/api/#vm-forceUpdate) 来做这件事。 

##### v-once 创建低开销的静态组件

渲染普通的 HTML 元素在 Vue 中是非常快速的，但有的时候你可能有一个组件，这个组件包含了**大量**静态内容。在这种情况下，你可以在根元素上添加 `v-once` 使这些内容只计算一次然后缓存起来（元素和组件只渲染一次，之后不会跟着数据的改变而改变）：

```javascript
Vue.component('terms-of-service', {
  template: `
    <div v-once>
      <h1>Terms of Service</h1>
      ... a lot of static content ...
    </div>
  `
})
```

不要过度使用这个模式。当需要渲染大量静态内容时，极少数的情况下它会给你带来便利，除非你非常留意渲染变慢了，不然它完全是没有必要的——再加上它在后期会带来很多困惑。例如，另一个开发者并不熟悉 `v-once` 或漏看了它在模板中，他们可能会花很多个小时去找出模板为什么无法正确更新。 



## 插槽 

插槽内可以包含任何模板代码，包括 HTML 和 其他的组件。

### 基础用法

```javascript
// 开启插槽
<template id="myCpn">
	<div>
   	// 使用<slot>开启插槽，如果没有在该组件中插入任何其他内容，就默认显示<slot>..</slot>中的内容
		<div class="..."> // 不要在 <slot> 上直接加样式，因为 <slot> 最终会被替换
  		<slot>我是一个插槽中的默认内容</slot>
  	</div>
	</div>
</template>
<script src="../js/vue. js"></script>
<script>
	Vue.component('my-cpn', {
		template: '#myCpn'
	})
	let app = new Vue({
		el: '#app'
	})
</script>
<style scoped>
	...
</style>
```

```javascript
// 使用
<div id="app"> 
	<my-cpn></my-cpn>
	<my-cpn>
		<h2>我是替换插槽的内容</h2>
		<p>我也是替换插槽的内容</p>
	<my-cpn> 
</div>
```

结果：

​		我是一个插槽中的默认内容

​		**我是替换插槽的内容**

​		我也是替换插槽的内容

### 具名插槽

有时我们需要多个插槽。

attribute：`name` ，用来定义额外的插槽。 不带 `name` 的 ` <slot> ` 出口会带有隐含的名字“default”。 

```javascript
// 定义具名插槽
<div class="container">
  <header>
    <slot name="header"></slot> // 把页头放这里
  </header>
  <main>
    <slot></slot> // 把主要内容放这里
  </main>
  <footer>
    <slot name="footer"></slot> // 把页脚放这里
  </footer>
</div>
```

在一个 ` template ` 元素上 使用 `v-slot` 指令向具名插槽提供内容，并以 `v-slot` 的参数的形式提供其名称

任何没有被包裹在带有 `v-slot` 中的内容都会被视为默认插槽的内容：

```javascript
// 使用
<base-layout>
  <template v-slot:header> // 需要用<template>包裹，并在其中指定要替换的具体的插槽，而不是具体的内容中 用 v-slot
    <h1>Here might be a page title</h1> // 具体的内容
  </template>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

// 如果你希望更明确一些，仍然可以在一个 <template> 中包裹默认插槽的内容
//	<template v-slot:default>
//    <p>A paragraph for the main content.</p>
//    <p>And another one.</p>
//  </template>

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>
```

#### v-slot 的语法糖

`v-slot:` 替换为字符 `#`。例如 `v-slot:header` 可以被重写为 `#header` 。

和其它指令一样，该缩写只在其有参数的时候才可用。这意味着以下语法是无效的： 

```javascript
<!-- 这样会触发一个警告 -->
<current-user #="{ user }">
  {{ user.firstName }}
</current-user>

// 正确用法：
<current-user #default="{ user }">
  {{ user.firstName }}
</current-user>
```

### 作用域插槽

插槽内容能够访问子组件中的数据。

```javascript
// 编译作用域
<navigation-link url="/profile">
  Clicking here will send you to: {{ url }}
  // 这里的 `url` 会是 undefined，因为 "/profile" 是传递给 <navigation-link> 的而不是在 <navigation-link> 组件 内部 定义的。
</navigation-link>
```

对于**编译作用域**，官方给出了一条准则：

​		**父组件模板的所有东西都会在父级作用域内编译；子组件模板的所有东西都会在子级作用域内编译。**

对于**作用域插槽**，可以用一句话进行总结：

​		**父组件替换插槽的标签，但是内容由子组件来提供。**

使用：简单来说，就是像父组件向子组件定义prop一样，传入数据。

```javascript
// <current-user> 组件：
<span>
  <slot>{{ user.lastName }}</slot>
</span>

// 使用：
<current-user>
  {{ user.firstName }}
</current-user>
// 上述代码不会正常工作，因为只有 <current-user> 组件可以访问到 user 而提供的内容是在父级渲染的。
```

```javascript
// 定义
<template id="myCpn">
<div> 
	<slot :data="pLanguages"></slot> // 传入数据
< /div>
</ template>
<script src="../js/vue. js"></script>
<script>
	Vue.component('my-cpn', {
	template: '#myCpn'
	data() {
		return {
			pLanguages: [
				'JavaScript', 'Python', 'Swift'，'Go', 'C++' // 子组件中定义数据，并且绑定到slot上
			]
    }
	}
})

// 使用
// 绑定在 <slot> 元素上的 attribute 被称为插槽 prop。现在在父级作用域中，我们可以使用带值的 v-slot 来定义我们提供的插槽 prop 的名字：
<div id="app">
	<my-cpn>
		<template v-slot="abc"> // 名称一般为 slotProps
			<span v-for="info in abc.data">{{info}}</span>
		</template>
	</my-cpn>
</div>
```

#### 默认插槽的缩写语法

注意： **默认插槽的缩写语法不能和具名插槽混用，因为它会导致作用域不明确：**

因此，**尽量使用完整的语法**： 

```javascript
<!-- 无效，会导致警告 -->
<current-user v-slot="slotProps">
  {{ slotProps.user.firstName }}
  <template v-slot:other="otherSlotProps">
    slotProps is NOT available here
  </template>
</current-user>

// 完整的语法：
<current-user>
  <template v-slot:default="slotProps">
    {{ slotProps.user.firstName }}
  </template>

  <template v-slot:other="otherSlotProps">
    ...
  </template>
</current-user>
```

#### 解构插槽 Prop

作用域插槽的内部工作原理是将你的插槽内容包括在一个传入单个参数的函数里： 

```javascript
function (slotProps) {
  // 插槽内容
}
```

这意味着 `v-slot` 的值实际上可以是任何能够作为函数定义中的参数的 JavaScript 表达式。 因此，可以用ES5的解构来传入具体的插槽prop：

```javascript
<current-user v-slot="{ user }">
  {{user.firstName}}
</current-user>

<current-user v-slot="{ user: person }"> // prop重命名
  ...
</current-user>

<current-user v-slot="{ user = {firstName: 'Guest'} }"> // 定义默认值
  {{ user.firstName }}
</current-user>
```

### 动态插槽名

动态指令参数也可以用在 `v-slot` 上，来定义动态的插槽名： 

```javascript
<base-layout>
  <template v-slot:[dynamicSlotName]>
    ...
  </template>
</base-layout>
```

### $

通过 [`this.$slots`](https://cn.vuejs.org/v2/api/#vm-slots) 访问静态插槽的内容，每个插槽都是一个 VNode 数组：

```javascript
render: function (createElement) {
  // `<div><slot></slot></div>`
  return createElement('div', this.$slots.default)
}
```

可以通过 [`this.$scopedSlots`](https://cn.vuejs.org/v2/api/#vm-scopedSlots) 访问作用域插槽，每个作用域插槽都是一个返回若干 VNode 的函数： 

```javascript
props: ['message'],
render: function (createElement) {
  // `<div><slot :text="message"></slot></div>`
  return createElement('div', [
    this.$scopedSlots.default({
      text: this.message
    })
  ])
}
```

如果要用渲染函数向子组件中传递作用域插槽，可以利用 VNode 数据对象中的 `scopedSlots` 字段： 

```javascript
render: function (createElement) {
  // `<div><child v-slot="props"><span>{{ props.text }}</span></child></div>`
  return createElement('div', [
    createElement('child', {
      // 在数据对象中传递 `scopedSlots`
      // 格式为 { name: props => VNode | Array<VNode> }
      scopedSlots: {
        default: function (props) {
          return createElement('span', props.text)
        }
      }
    })
  ])
}
```



## 可复用性

### 混入

混入 (mixin) 提供了一种非常灵活的方式，来分发 Vue 组件中的可复用功能。一个混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被“混合”进入该组件本身的选项。 

当混入与Vue实例有同名选项时，会进行合并；当冲突时，以Vue实例数据为优先。

```javascript
// 定义一个混入对象
var myMixin = {
  created: function () { // 同名钩子函数将合并为一个数组，因此都将被调用。另外，混入对象的钩子将在组件自身钩子之前调用。
    ...
  },
  data: function () {
    return {
      message: 'hello',
      foo: 'abc'
    }
  },
  methods: {
    hello: function () {
      console.log('hello from mixin!')
    }
  }
}

// 使用
new Vue({
  mixins: [mixin],
  data: function () {
    return {
      message: 'goodbye', // 当组件和混入对象含有同名选项时，会进行合并，以组件的数据为优先。
      bar: 'def'
    }
  },
  created: function () {
    this.hello() // => "hello from mixin!"
    console.log(this.$data) // => { message: "goodbye", foo: "abc", bar: "def" }
  }
})

```

#### 全局混入

混入也可以进行全局注册。使用时格外小心！一旦使用全局混入，它将影响**每一个**之后创建的 Vue 实例。使用恰当时，这可以用来为自定义选项注入处理逻辑。 

```javascript
// 为自定义的选项 'myOption' 注入一个处理器。
Vue.mixin({
  created: function () {
    var myOption = this.$options.myOption
    if (myOption) {
      console.log(myOption)
    }
  }
})

new Vue({
  myOption: 'hello!'
})
// => "hello!"
```

谨慎使用全局混入，因为它会影响每个单独创建的 Vue 实例 (包括第三方组件)。大多数情况下，只应当用于自定义选项，就像上面示例一样。推荐将其作为插件发布，以避免重复应用混入。 

#### 自定义选项合并策略

如果想让自定义选项以自定义逻辑合并，可以向 `Vue.config.optionMergeStrategies` 添加一个函数： 

```javascript
Vue.config.optionMergeStrategies.myOption = function (toVal, fromVal) {
  // 返回合并后的值
}
```

对于多数值为对象的选项，可以使用与 `methods` 相同的合并策略： 

```javascript
var strategies = Vue.config.optionMergeStrategies
strategies.myOption = strategies.methods
```

可以在 [Vuex](https://github.com/vuejs/vuex) 1.x 的混入策略里找到一个更高级的例子： 

```javascript
const merge = Vue.config.optionMergeStrategies.computed
Vue.config.optionMergeStrategies.vuex = function (toVal, fromVal) {
  if (!toVal) return fromVal
  if (!fromVal) return toVal
  return {
    getters: merge(toVal.getters, fromVal.getters),
    state: merge(toVal.state, fromVal.state),
    actions: merge(toVal.actions, fromVal.actions)
  }
}
```



### 插件

插件通常用来为 Vue 添加全局功能。插件的功能范围没有严格的限制——一般有下面几种：

1. 添加全局方法或者 property。如：[vue-custom-element](https://github.com/karol-f/vue-custom-element)
2. 添加全局资源：指令/过滤器/过渡等。如 [vue-touch](https://github.com/vuejs/vue-touch)
3. 通过全局混入来添加一些组件选项。如 [vue-router](https://github.com/vuejs/vue-router)
4. 添加 Vue 实例方法，通过把它们添加到 `Vue.prototype` 上实现。
5. 一个库，提供自己的 API，同时提供上面提到的一个或多个功能。如 [vue-router](https://github.com/vuejs/vue-router)

#### 使用

插件的引入应放在vue文件引入的下方。

通过全局方法 `Vue.use()` 使用插件。它需要在你调用 `new Vue()` 启动应用之前完成： 

```javascript
Vue.use(MyPlugin) // 调用 `MyPlugin.install(Vue)`
Vue.use(MyPlugin, { someOption: true }) // 也可以传入一个可选的选项对象

new Vue({
  // ...组件选项
})


```

`Vue.use` 会自动阻止多次注册相同插件，届时即使多次调用也只会注册一次该插件。 

#### 定义/开发 插件

Vue.js 的插件应该暴露一个 `install` 方法。这个方法的

```javascript
import Toast from './Toast' // 1.导入相应的组件.vue文件

// 2.定义一个对象
const obj = {}

// 第一个参数是 `Vue` 构造器，第二个参数是一个可选的选项对象
obj.install = function (Vue, options) { 
  // 3.创建组件构造器
  const toastContrustor = Vue.extend(Toast)
  
  // 4.用new的方式，根据组件构造器，创建出一个组件对象
  const toast = new toastContrustor()
  
  // 5.将组件对象手动挂载到某一个元素上
  toast.$mount(document.createElement('div'))
  
  // 6.toast.$el对应的就是div，将Toast.vue中的html添加到div
  document.body.appendChild(toast.$el)
  
  // 7.在原型上添加
  Vue.prototype.$toast = toast // 是toast，不是Toast
  
  // 添加全局方法或 property
  Vue.myGlobalMethod = function () {
    ...
  }

  // 添加全局资源
  Vue.directive('my-directive', {
    bind(el, binding, vnode, oldVnode) {
      ...
    }
    ...
  })

  // 注入组件选项
  Vue.mixin({
    created: function () {
      ...
    }
    ...
  })

  // 添加实例方法
  Vue.prototype.$myMethod = function (methodOptions) {
    ...
  }
}

// 8.导出
export default obj
```

```javascript
// main.js
...

import toast from 'components/common/toast/index' // 导入

Vue.use(toast) // 安装 toast 插件，默认会调用install方法

...

new Vue({
...
}).$mount('#app')
```



## 路由 vue-router

链接：https://router.vuejs.org/zh/installation.html

### 基本使用

前端路由的核心是**改变URL，页面不进行整体的刷新。**  

如果做到改变URL而不刷新页面：

1.URL的hash

​		URL的hash也就是锚点(#), 本质上是改变window.location的href属性

​		我们可以通过直接赋值location.hash来改变href, 但是页面不发生刷新

2.H5的history模式

​		如果不想要很丑的 hash，我们可以用路由的history 模式，充分利用 `history.pushState` API 来完成 URL 跳转而无须重新加载页面。 

​		当你使用 history 模式时，URL 就像正常的 url，例如 `http://yoursite.com/user/id` 

​		这种模式要玩好，还需要后台配置支持。因为我们的应用是个单页客户端应用，如果后台没有正确的配置，当用户在浏览器直接访问 `http://oursite.com/user/id` 就会返回 404，这就不好看了。

​		后端配置见官网：https://router.vuejs.org/zh/guide/essentials/history-mode.html#%E5%90%8E%E7%AB%AF%E9%85%8D%E7%BD%AE%E4%BE%8B%E5%AD%90



```javascript
// H5的history模式
const router = new VueRouter({
  mode: 'history',
  routes: [...]
})
```

安装

```
npm install vue-router --save
```

使用

```javascript
// router/index.js

// 1.导入 路由 和 组件
import Vue from 'vue'
import VueRouter from 'vue-router'

// 3.导入对应的组件
import Home from '...'
const Home = () => ('...') // 懒加载
import ...

// 2.通过Vue.use(插件)，安装插件
Vue.use(VueRouter)

// 4.创建VueRouter对象
const routes = [ // 将路由定义抽出
  {
    path: '',
    redirect: '/home' // 重定向
  },
  {
    path: '/home', // path是URL显示的路径
    component: Home
  },
  {
    ...
  }
]
const router = new VueRouter({
	// 5.配置路由和组件之间的应用关系
	routes,
  mode: 'history' // 可选，改变路径的方式为H5的history模式，默认为URL的hash模式
})

// 6.导出
export default router
```

```javascript
// main.js
import Vue from 'vue'
import App from './App.vue'
import router from "./router"; // 7.导入路由

....

new Vue({
  render: h => h(App),
  router // 8.注册路由
}).$mount('#app')
```

```javascript
// App.vue
<template>
<div id="app">
	<h1>我是网站的标题< /h1>

	// 这里使用<router-link>，但一般都是通过代码 this.$router.replace(...) 进行跳转
	<router-link to="/home">首页</router-link> // 使用<router-link>，用于跳转（渲染为a标签）
	<router-link to="/about">关于</router-link>

	<router-view></router-view> // 使用<router-view>，用于显示组件及组件的位置
</div>
......
```



### `<router-link>` 补充

在前面的`<router-link>`中, 我们只是使用了一个属性：to，用于指定跳转的路径.

`<router-link>`还有一些其他属性：

**tag：**tag可以指定`<router-link>`之后渲染成什么组件, 比如上面的代码会被渲染成一个`<li>`元素，而不是`<a>` ，默认会渲染为 `<a>` 标签。

**replace：**replace不会留下history记录, 所以指定replace的情况下, 后退键返回不能返回到上一个页面中。

**active-class：**当`<router-link>`对应的路由匹配成功时, 会自动给当前元素设置一个router-link-active的class, 设置active-class可以修改默认的名称。

​		在进行高亮显示的导航菜单或者底部tabbar时, 会使用到该类。

​		但是通常不会修改类的属性, router-link-active。

```javascript
<router-link to="/about" tag="button" replace active-class="active">关于</router-link>

<style>
  .active {
    color: red;
  }
</style>
```

对于active-class，如果想对所有的`<router-link>`的`active-class`进行统一的修改，则可以在路由中配置`linkActiveClass`：

```javascript
const router = new VueRouter({
	routes，
	linkActiveClass: 'active' // 统一修改
})
```

### 动态路由

在某些情况下，一个页面的path路径可能是不确定的

```javascript
{
	path: '/user/:userid', // 动态路径参数 以冒号开头
	component: User
}

<router-link :to="'/user/' + userId">用户</router-link>

// 参数值会被设置到 this.$route.params
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
```

#### 响应路由参数的变化

当使用路由参数时，例如从 `/user/foo` 导航到 `/user/bar`，**原来的组件实例会被复用**。因为两个路由都渲染同个组件，比起销毁再创建，复用则显得更加高效。**不过，这也意味着组件的生命周期钩子不会再被调用**。 

复用组件时，想对路由参数的变化作出响应的话，你可以简单地 watch (监测变化) `$route` 对象： 

```javascript
const User = {
  template: '...',
  watch: {
    '$route' (to, from) {
      // 对路由变化作出响应...
    }
  }
}

// 或者使用中 beforeRouteUpdate 导航守卫：
const User = {
  template: '...',
  beforeRouteUpdate (to, from, next) {
    // react to route changes...
    // don't forget to call next()
  }
}
```

#### 捕获所有路由或 404 Not found 路由

常规参数只会匹配被 `/` 分隔的 URL 片段中的字符。如果想匹配**任意路径**，我们可以使用通配符 (`*`)： 

```javascript
{
  path: '*' // 会匹配所有路径
}
{
  path: '/user-*' // 会匹配以 `/user-` 开头的任意路径
}
```

当使用*通配符*路由时，确保路由的顺序是正确的（ 谁先定义的，谁的优先级就最高。），也就是说含有*通配符*的路由应该放在最后。路由 `{ path: '*' }` 通常用于客户端 404 错误。如果使用了*History 模式*，请确保[正确配置你的服务器](https://router.vuejs.org/zh/guide/essentials/history-mode.html)。

当使用一个*通配符*时，`$route.params` 内会自动添加一个名为 `pathMatch` 参数。它包含了 URL **通过通配符被匹配的部分**：

```javascript
// 给出一个路由 { path: '/user-*' }
this.$router.push('/user-admin')
this.$route.params.pathMatch // 'admin'

// 给出一个路由 { path: '*' }
this.$router.push('/non-existing')
this.$route.params.pathMatch // '/non-existing'
```

#### $route

`$route` 是处于活跃状态的那个路由组件

其中，可以通过 `this.$route.params.userid `来获取路径

### 嵌套路由

路径和组件的关系如下：

![1587438710352](Vue.assets/1587438710352.png)

实现嵌套路由有两个步骤:

​		创建对应的子组件, 并且在路由映射中配置对应的子路由.

​		在组件内部使用`<router-view>`标签

```javascript
...
const Home = () => import('../components/Home')
const HomeNews = () => import('../components/HomeNews')
...

// 配置路由
const routes = [
  ...
  {
    // path: '/home/:id', // 可以是动态路径
  	path: '/home', // 路径
    component: Home,
    children: [ // 配置 嵌套路由 子路由
      {
        // 默认路由
        path: '', 
        redirect: 'message'
      },
      {
        path: 'news', // 不需要写 '/'
        components: HomeNews 
        // 当 /home/news 匹配成功，HomeNews会被渲染在 <router-view> 中
      },
      {
        // 没有匹配到路径时，<router-view> 不会渲染任何东西，因为没有匹配到合适的子路由。
        // 如果你想要渲染点什么，可以提供一个 空的 子路由：
        path: '', 
        components: HomeNews 
      },
      ...
    ]
  }
]
```

```javascript
// Home.vue Home路由组件
<template>
	<div>
		...
		<router-link to="/home/news">新闻</router-link> // to需要写子路由的完整路径
		<router-view></router-view> // <router-view> 是最顶层的出口，渲染最高级路由匹配到的组件。
	</div>
</template>
```

### 代码跳转路由(编程式导航)

`router.push(location, onComplete?, onAbort?)`

`router.replace(location, onComplete?, onAbort?)`

在 2.2.0+， 可选的在 `router.push` 或 `router.replace` 中提供 `onComplete` 和 `onAbort` 回调作为第二个和第三个参数，会在导航成功完成 (所有异步钩子被解析后) 或终止 (导航到相同的路由、或在当前导航完成之前导航到另一个不同的路由) 时进行调用。在 3.1.0+，可以省略第二个和第三个参数，此时如果支持 Promise，`router.push` 或 `router.replace` 将返回一个 Promise。 

如果目的地和当前路由相同，只有参数发生了改变 (比如从一个用户资料到另一个 `/users/1` -> `/users/2`)，你需要使用 [`beforeRouteUpdate`](https://router.vuejs.org/zh/guide/essentials/dynamic-matching.html#响应路由参数的变化) 来响应这个变化 (比如抓取用户信息)。 

使用`this.$router.push` 或 `this.$router.replace`，参数可以是一个字符串路径，或者一个描述地址的对象：

```javascript
// 字符串
router.push('home')

// 对象
router.push({ path: 'home' })

// 命名的路由
router.push({ name: 'user', params: { userId: '123' }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})

/* 如果提供了 path，params 会被忽略，上述例子中的 query 并不属于这种情况。取而代之的是下面例子的做法，你需要提供路由的 name 或手写完整的带有参数的 path： */
const userId = '123'
router.push({ name: 'user', params: { userId }}) // -> /user/123
router.push({ path: `/user/${userId}` }) // -> /user/123
// 这里的 params 不生效
router.push({ path: '/user', params: { userId }}) // -> /user
```

```javascript
// $router对象就是下面定义的router1对象
const router1 = new VueRouter({
	routes
})
```

```javascript
<template>
	<div>
  	<button @click="homeClick">首页</button> // 进行监听
		......
  </div>
</template>

<script>
export default {
	name: 'App',
  data() {
    return {
      userId: 'zhangsan' // 参数
    }
  },
  methods: {
    homeClick() {
      // history.pushState() // 不要这样做，这样是绕过了vue-router而去直接修改路径
      this.$router.push('/home')
      this.$router.replace('/home')
      this.$router.push('/home/' + this.userId) // 携带参数
    }
  }
}
</script>
```

`router.go(n)` ：意思是在 history 记录中向前或者后退多少步，参数是一个整数。

### 命名路由

通过一个名称来标识一个路由显得更方便一些，特别是在链接一个路由，或者是执行一些跳转的时候。 

```javascript
const Foo = { template: '<div>This is Foo</div>' }
...

const router = new VueRouter({
  routes: [
    { path: '/foo', 
      name: 'foo', 
      component: Foo 
    },
    {
      path: '/user/:userId',
      name: 'user', // 命名
      component: User
    }
  ]
})

// 不带动态参数的示例：
<router-link :to="{ name: 'home' }">home</router-link>

// 带动态参数：
// 两种方式都会把路由导航到 /user/123 路径：
// 要链接到一个命名路由，可以给 router-link 的 to 属性传一个对象：
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
// 这跟代码调用 router.push() 是一回事：
router.push({ name: 'user', params: { userId: 123 }})
```

### 命名视图

同时 (同级) 展示多个视图，而不是嵌套展示。

在界面中拥有多个单独命名的视图，而不是只有一个单独的出口。

如果 `router-view` 没有设置名字，那么默认为 `default`。 

```javascript
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>

// 多个视图需要多个组件，确保正确使用 components 配置 (带上 s)：
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

#### 嵌套命名视图

```
/settings/emails                                     /settings/profile
+-----------------------------------+                +------------------------------+
| UserSettings                      |                | UserSettings                 |
| +-----+-------------------------+ |                | +-----+--------------------+ |
| | Nav | UserEmailsSubscriptions | |  +---------->  | | Nav | UserProfile        | |
| |     +-------------------------+ |                | |     +--------------------+ |
| |     |                         | |                | |     | UserProfilePreview | |
| +-----+-------------------------+ |                | +-----+--------------------+ |
+-----------------------------------+                +------------------------------+

Nav 只是一个常规组件。
UserSettings 是一个视图组件。
UserEmailsSubscriptions、UserProfile、UserProfilePreview 是嵌套的视图组件。
```

```javascript
// UserSettings.vue 组件的 <template> 部分，嵌套的视图组件在此忽略
<div>
  <NavBar/>
  <router-view/>
  <router-view name="helper"/>
</div>

// 然后用这个路由配置完成该布局：
{
  path: '/settings',
  // 你也可以在顶级路由就配置命名视图
  component: UserSettings,
  children: [
    {
      path: 'emails',
      component: UserEmailsSubscriptions
    }, 
    {
      path: 'profile',
      components: {
        default: UserProfile,
        helper: UserProfilePreview
      }
    }
  ]
}
```

### 重定向和别名

“重定向”的意思是，当用户访问 `/a` 时，URL 将会被替换成 `/b`，然后匹配路由为 `/b` 。

```javascript
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: '/b' } // 从 /a 重定向到 /b
    { path: '/a', redirect: { name: 'foo' }} // 重定向的目标也可以是一个命名的路由
    { path: '/a', redirect: to => { // 可以是一个方法，动态返回重定向目标
      // 方法接收 目标路由 作为参数
      // return 重定向的 字符串路径/路径对象
    }}
  ]
})
```

`/a` 的别名是 `/b`，意味着，当用户访问 `/b` 时，URL 会保持为 `/b`，但是路由匹配则为 `/a`，就像用户访问 `/a` 一样。

“别名”的功能让你可以自由地将 UI 结构映射到任意的 URL，而不是受限于配置的嵌套路由结构。 

```javascript
const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' }
  ]
})
```

更多高级用法，请查看[例子](https://github.com/vuejs/vue-router/blob/dev/examples/route-alias/app.js)。 

### 参数传递

#### $router.params & $router.query

参数主要有两种类型：**params**、**query**

*query用path来引入，params只能用name来传递，不能使用path。*

**params：**

​		配置路由格式：`/router/:id`

​		传递的方式：在path后面跟上对应的值，即`<router-link :to="'/user/' + userId">用户</router-link>`中的`userId`

​		传递后形成的路径：/router/123，/router/abc

**query：**

​		配置路由格式：/router, 也就是普通配置

​		传递的方式：对象中使用query的key作为传递方式

​		传递后形成的路径：/router?id=123，/router?id=abc

**传递参数有两种方式：**

**方式一：`<router-link>`**

```javascript
// 在<router-link>中传递
<router-link
	:to="{
		path: '/profile/' + 123, // 路径
    query: {name: 'why', age: 18} // 参数
	}"
	
	to="{ 
		name: 'user', 
    params: { id: 123 }
	}"
>我的</router-link>
```

**方式二：JS代码** `this.$router.push`

```javascript
// 在代码跳转路由中传递
<script>
	name: 'App',
  methods: {
		toProfile() {
      this.$router.push({
        path: '/profile/' + 123, // 路径
        query: {name: 'why', age: 18} // 参数
      })
    },
    toProfile() {
      this.$router.push({ 
        name: 'user', 
        params: { id: 123 }
      })
    }
  }
</script>
```

通过`$route`取出并使用参数：

```javascript
// 传参是$router，使用是$route
<h2>{{ $route.query.name }}</h2>
<h2>{{ $route.params.id }}</h2>
```

#### 使用props（推荐）

在组件中使用 `$route` 会使之与其对应路由形成高度耦合，从而使组件只能在某些特定的 URL 上使用，限制了其灵活性。 

```javascript
// 定义路由 props
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User, props: true },

    // 对于包含命名视图的路由，你必须分别为每个命名视图添加 `props` 选项：
    {
      path: '/user/:id',
      components: { default: User, sidebar: Sidebar },
      props: { default: true, sidebar: false }
    }
  ]
})

// 同方法1，在a页面跳转的时候传参
<router-link to="{ path:'/user/123'}">

// 在路由user组件中使用props接受参数
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
```

#### 使用Vuex

虽然不太推荐，有点杀鸡用牛刀 ，但是也可以通过vuex实现

```javascript
// 定义store
import Vuex from 'vuex'
import Vue  from 'vuex'
Vue.use(Vuex)
export default new Vuex.Store({
  state:{
    id: ''
  },
  mutations:{
    setId(state, id) {
      state.id = id            
    }
  }
})

// 在a页面设置
 this.$store.commit( 'setId' ，(123) )

// 在b页面取
this.$store.state.id
```

#### 使用本地存储

localstorage 或者 Session Storage。 原理很简单，a页面存，b页面取，不推荐。

### 导航守卫

#### 全局守卫

“导航”表示路由正在发生改变。 

导航守卫主要用来监听路由的进入和离开的。 **参数或查询的改变并不会触发进入/离开的导航守卫**。 

vue-router提供了beforeEach和afterEach的钩子函数，它们会在路由即将改变前和改变后触发。

```javascript
/*
当一个导航触发时，全局前置守卫按照创建顺序调用。守卫是异步解析执行，此时导航在所有守卫 resolve 完之前一直处于 等待中。
每个守卫方法接收三个参数：
	1.to: Route: 即将要进入的目标的路由对象
	2.from: Route: 当前导航即将要离开的路由对象
	3.next: Function: 调用该方法后, 才能进入下一个钩子，一定要调用该方法来 resolve 这个钩子。
		3.1 next(): 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 confirmed (确认的)。
		3.2 next(false): 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 from 路由对应的地址。
		3.3 next('/') 或者 next({ path: '/' }): 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。你可以向 next 传递任意位置对象，且允许设置诸如 replace: true、name: 'home' 之类的选项以及任何用在 router-link 的 to prop 或 router.push 中的选项。
		3.4 next(error): (2.4.0+) 如果传入 next 的参数是一个 Error 实例，则导航会被终止且该错误会被传递给 router.onError() 注册过的回调。
*/

// 全局前置守卫
router.beforeEach((to, from, next) => {
	...
	next() // 必须调用next
})

// 全局解析守卫
// 在 2.5.0+ 你可以用 router.beforeResolve 注册一个全局守卫。这和 router.beforeEach 类似，区别是在导航被确认之前，同时在所有组件内守卫和异步路由组件被解析之后，解析守卫就被调用。
router.beforeResolve((to, from, next) => {
  ...
  next();
});

// 全局后置钩子
router.afterEach((to, from) => {
  ...
  // afterEach 不需要主动调用next()函数
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
      beforeEnter: (to, from, next) => { // 与全局前置守卫的参数一样
        // ...
      }
    }
  ]
})
```

 #### 组件内的守卫

可以在路由组件内直接定义以下路由导航守卫：

`beforeRouteEnter`、`beforeRouteUpdate`、`beforeRouteLeave`：

```javascript
const Foo = {
  template: `...`,
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当守卫执行前，组件实例还没被创建
    ...
    // 可以通过传一个回调给 next来访问组件实例。在导航被确认的时候执行回调，并且把组件实例作为回调方法的参数。
    next(vm => {
    	// 通过 `vm` 访问组件实例
      // 注意 beforeRouteEnter 是支持给 next 传递回调的唯一守卫。对于 beforeRouteUpdate 和 beforeRouteLeave 来说，this 已经可用了，所以不支持传递回调，因为没有必要了。
  	})
    
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
    ...
    this.name = to.params.name
  	next()
  },
  beforeRouteLeave (to, from, next) {
    // 这个离开守卫通常用来禁止用户在还未保存修改前突然离开。该导航可以通过 next(false) 来取消。
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
    ...
    const answer = window.confirm('Do you really want to leave? you have unsaved 		changes!')
  	if (answer) {
   	 next()
 	 	} else {
   	 next(false)
 	 	}
  }
}
```

#### 完整的导航解析流程

1. 导航被触发。
2. 在失活的组件里调用离开守卫。
3. 调用全局的 `beforeEach` 守卫。
4. 在重用的组件里调用 `beforeRouteUpdate` 守卫 (2.2+)。
5. 在路由配置里调用 `beforeEnter`。
6. 解析异步路由组件。
7. 在被激活的组件里调用 `beforeRouteEnter`。
8. 调用全局的 `beforeResolve` 守卫 (2.5+)。
9. 导航被确认。
10. 调用全局的 `afterEach` 钩子。
11. 触发 DOM 更新。
12. 用创建好的实例调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数。

### 路由元信息

每个路由都有一个 meta 元数据字段， 可以设置一些自定义信息，供页面组件或者路由钩子函数中使用。 

说白了就是通过meta对象中定义的一些属性来判断当前路由是否需要进一步处理，如果需要处理，按照自己想要的效果进行处理即可。通过 `$route.meta.xxxx`获取路由元信息中的数据。

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      children: [
        {
          path: 'bar',
          component: Bar,
          // 定义路由的时候可以配置 meta 字段： 
          meta: { // a meta field
            requiresAuth: true 
          } 
        }
      ]
    }
  ]
})
```

我们称呼 `routes` 配置中的每个路由对象为 **路由记录**。路由记录可以是嵌套的，因此，当一个路由匹配成功后，他可能匹配多个路由记录。

例如，根据上面的路由配置，`/foo/bar` 这个 URL 将会匹配父路由记录以及子路由记录。 

一个路由匹配到的所有路由记录会暴露为 `$route` 对象 (还有在导航守卫中的路由对象) 的 `$route.matched` 数组，我们需要遍历 `$route.matched` 来检查路由记录中的 `meta` 字段。

下面例子展示在全局导航守卫中检查元字段：

```javascript
router.beforeEach((to, from, next) => {
  if (to.matched.some(record => record.meta.requiresAuth)) {
    // this route requires auth, check if logged in
    // if not, redirect to login page.
    if (!auth.loggedIn()) {
      next({
        path: '/login',
        query: { redirect: to.fullPath }
      })
    } else {
      next()
    }
  } else {
    next() // 确保一定要调用 next()
  }
})

解析：
1、meta 字段就是路由元信息字段，requiresAuth 是自己起的字段名称，用来标记这个路由信息是否需要检测，true 表示要检测，false 表示不需要检测（这个名称随便起，比如我自己的就起的 requiresId，建议起个有意义的名称）

2、if (to.matched.some(record => record.meta.requiresAuth))，返回遍历的某个路由对象，定义为record，检测这个对象是否拥有meta这个对象，如果有meta这个对象，检测它的meta对象是不是有requiresAuth属性，并且为true，如果满足上述条件，就确定了是这个/foo/bar路由。

3、this route requires auth, check if logged in ，说明auth信息是必须的，检验是否登录，也就是在这个路由下，如果检测到auth这个用户名，那么进行下一步操作。

4、案例下面就有了判断，if (!auth.loggedIn())，通过自己封装的检验方法auth.loggedIn()，检验用户是否登录，从而决定渲染下一步操作。
```

#### 使用示例

```javascript
const router =  new Router({
  routes: [ 
    {
      path: '/', name: 'index', component: index, 
      meta: { title: '首页' } // 定义 meta
    },
    {
      path: '/hello', name: 'hello', component: hello,
      meta: { title: '欢迎页' } // 定义 meta
    }
  ]
})

// 在组件中使用 meta router.beforeEach((to, from, next) => {
  window.document.title = to.meta.title;
  next();
})数据
<template>
  <div>
    <h1>{{ $route.meta.title }}</h1> // 改变当前页面的标题，$route 为当前活跃的路由
  </div>
</template>

// 在路由钩子中使用 meta 数据
router.beforeEach((to, from, next) => {
  window.document.title = to.meta.title;
  next();
})

// 动态改变meta数据
this.$route.meta.title = "还是首页";
```

### 数据获取

有时候，进入某个路由后，需要从服务器获取数据。例如，在渲染用户信息时，你需要从服务器获取用户的数据。我们可以通过两种方式来实现： 

- **导航完成之后获取**：先完成导航，然后在接下来的组件生命周期钩子中获取数据。在数据获取期间显示“加载中”之类的指示。
- **导航完成之前获取**：导航完成前，在路由进入的守卫中获取数据，在数据获取成功后执行导航。

从技术角度讲，两种方式都不错 —— 就看你想要的用户体验是哪种。

#### 导航完成之后获取

当你使用这种方式时，我们会马上导航和渲染组件，然后在组件的 `created` 钩子中获取数据。这让我们有机会在数据获取期间展示一个 loading 状态，还可以在不同视图间展示不同的 loading 状态。 

```javascript
// 假设我们有一个 Post 组件，需要基于 $route.params.id 获取文章数据：
<template>
  <div class="post">
    <div v-if="loading" class="loading">
      Loading...
    </div>

    <div v-if="error" class="error">
      {{ error }}
    </div>

    <div v-if="post" class="content">
      <h2>{{ post.title }}</h2>
      <p>{{ post.body }}</p>
    </div>
  </div>
</template>
/* --------------------------------------------- */
export default {
  data () {
    return {
      loading: false,
      post: null,
      error: null
    }
  },
  created () {
    // 组件创建完后获取数据，
    // 此时 data 已经被 observed 了
    this.fetchData()
  },
  watch: {
    // 如果路由有变化，会再次执行该方法
    '$route': 'fetchData'
  },
  methods: {
    fetchData () {
      this.error = this.post = null
      this.loading = true
      // replace getPost with your data fetching util / API wrapper
      getPost(this.$route.params.id, (err, post) => {
        this.loading = false
        if (err) {
          this.error = err.toString()
        } else {
          this.post = post
        }
      })
    }
  }
}
```

#### 导航完成之前获取

通过这种方式，我们在导航转入新的路由前获取数据。我们可以在接下来的组件的 `beforeRouteEnter` 守卫中获取数据，当数据获取成功后只调用 `next` 方法。 

```javascript
export default {
  data () {
    return {
      post: null,
      error: null
    }
  },
  beforeRouteEnter (to, from, next) {
    getPost(to.params.id, (err, post) => {
      next(vm => vm.setData(err, post))
    })
  },
  // 路由改变前，组件就已经渲染完了
  // 逻辑稍稍不同
  beforeRouteUpdate (to, from, next) {
    this.post = null
    getPost(to.params.id, (err, post) => {
      this.setData(err, post)
      next()
    })
  },
  methods: {
    setData (err, post) {
      if (err) {
        this.error = err.toString()
      } else {
        this.post = post
      }
    }
  }
}
```

### 滚动行为

查看完整例子请[移步这里](https://github.com/vuejs/vue-router/blob/dev/examples/scroll-behavior/app.js)。 

使用前端路由，当切换到新路由时，想要页面滚到顶部，或者是保持原先的滚动位置，就像重新加载页面那样。

**注意: 这个功能只在支持 `history.pushState`（H5 history模式） 的浏览器中可用。**

当创建一个 Router 实例，你可以提供一个 `scrollBehavior` 方法

`scrollBehavior` 方法接收的第一个和第二个参数： `to` 和 `from` 路由对象。

第三个参数 `savedPosition` 当且仅当 `popstate` 导航 (通过浏览器的 前进/后退 按钮触发) 时才可用。 *savePosition：会记录滚动条的坐标，点击前进/后退的时候记录值{x:?,y:?}* 

 ![QQæªå¾20171204104952](Vue.assets/5635328447320522.jpg) 

这个方法返回滚动位置的对象信息，长这样：

- `{ x: number, y: number }`
- `{ selector: string, offset? : { x: number, y: number }}` (offset 只在 2.6.0+ 支持)

如果返回一个 falsy (译者注：falsy 不是 `false`，[参考这里](https://developer.mozilla.org/zh-CN/docs/Glossary/Falsy))的值，或者是一个空对象，那么不会发生滚动。

```javascript
const router = new VueRouter({
  mode:'history',
  routes: [...],
  scrollBehavior (to, from, savedPosition) {
    // return 期望滚动到哪个的位置
    if (savedPosition) {
      return savedPosition
    } else {
      return { x: 0, y: 0 }
    }
  }
})
```

#### 设hash

```javascript
// Document.vue
<template>
    <div>
        我是文档
        <p id="abc">定位到这个元素</p> // abc元素
    </div>
</template>

// App.vue 在路径后面添加 hash 值。
<router-link :to="{path:'/document#abc'}">document</router-link>

let router = new VueRouter({
  mode:'history',
  scrollBehavior(to,from,savePosition){
    if(to.hash && to.hash === '#abc'){ // 先判断目标路由有没有hash值
      return {selector:to.hash}
      ...
    }
  },
  routes:[]
})
```

#### 异步滚动

也可以返回一个 Promise 来得出预期的位置描述：  2.8.0 新增

将其挂载到从页面级别的过渡组件的事件上，令其滚动行为和页面过渡一起良好运行是可能的。但是考虑到用例的多样性和复杂性，我们仅提供这个原始的接口，以支持不同用户场景的具体实现。 

```javascript
scrollBehavior (to, from, savedPosition) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({ x: 0, y: 0 })
    }, 500)
  })
}
```



### 路由懒加载

当打包构建应用时，Javascript 包会变得非常大，影响页面加载。

如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。

路由懒加载的主要作用就是将路由对应的组件打包成一个个的js代码块，只有在这个路由被访问到的时候，才加载对应的组件。

打包文件在使用懒加载前：

![1587437901949](Vue.assets/1587437901949.png)

打包文件在使用懒加载后：

![1587437918233](Vue.assets/1587437918233.png)

**懒加载的方式：**

方式一：结合Vue的异步组件和Webpack的代码分析**（过时）**

```javascript
const Home = resolve => { require.ensure(['../components/Home.vue'], () => { resolve(require('../components/Home.vue')) })};
```

方式二：AMD写法**（过时）**

```javascript
const About = resolve => require(['../components/About.vue'], resolve);
```

方式三：在ES6中, 我们可以有更加简单的写法来组织Vue异步组件和Webpack的代码分割.

```javascript
const Home = () => import('../components/Home.vue')
```

**直接用以下方式：**

![1587438037561](Vue.assets/1587438037561.png)

**或者将加载抽出：**

```javascript
...
const Home = () => import('../components/Home')
......

const routes = [
	{
		path: '/home',
		component: Home
	},
  .......
]
```

###  $route、$router的区别 

$router为VueRouter实例，想要导航到不同URL，则使用$router.push方法。

$route为当前router跳转对象里面可以获取name、path、query、params等。

### keep-alive

keep-alive 是 Vue 内置的一个组件，可以使被包含的组件保留状态，或避免重新渲染。

​		include：字符串或正则表达，只有匹配的组件会被缓存

​		exclude：字符串或正则表达式，任何匹配的组件都不会被缓存

router-view 也是一个组件，如果直接被包在 keep-alive 里面，所有路径匹配到的视图组件都会被缓存

```javascript
<keep-alive>
	<router-view exclude="Profile,User"> // Profile、User为组件的name
		<!-- 所有路径匹配到的视图组件都会被缓存 -->
	</router-view>
</keep-alive>
```

此时，可以使用 `activated` / `deactivated` 两个函数

## API

### 全局API

#### Vue.extend()

使用基础 Vue 构造器（不是实例），创建一个“子类”。参数是一个包含组件选项的对象。 

主要用于需要 **动态渲染** 的组件，或者类似于 `window.alert()` 提示组件。

```javascript
<div id="mount-point"></div>

// 创建构造器
var Profile = Vue.extend({
  template: '<p>{{firstName}} {{lastName}} aka {{alias}}</p>',
  data: function () {
    return {
      firstName: 'Walter',
      lastName: 'White',
      alias: 'Heisenberg'
    }
  }
})
// 创建实例，并挂载到指定元素上。
new Profile().$mount('#mount-point')

// 结果
<p>Walter White aka Heisenberg</p>
```

```javascript
// 第二种写法
let tem = {
  template: '...',
  data: function () {    
    return{    
      ...
    }
  }
}

// 调用
const TemConstructor = Vue.extend(tem) 
const intance = new TemConstructor({el:"#app"}) //生成一个实例，并且挂载在#app上
```

#### Vue.nextTick()

`vm.$nextTick()` 用法与 `Vue.nextTick()` 相同，只是回调的 `this` 自动绑定到调用它的实例上。 

在**下次 DOM 更新循环结束之后**执行延迟回调。在修改数据之后立即使用这个方法，**获取更新后的 DOM 及其数据**。 

放在 `Vue.nextTick()` 回调函数中的应该是会对DOM进行操作的 JS 代码。

将回调函数延迟在下一次dom更新数据后调用，简单的理解是：当数据更新，在dom中渲染完毕后，自动执行该函数。

> **注意：但是此时虽然DOM渲染完成，但是如果有图片，则图片仍然是没有加载完成的(注意缓存的影响)，此时如果进行例如获取offsetTop的值等之类的操作，是不准确的。**

```javascript
// 修改数据
vm.msg = 'Hello'
// DOM 还没有更新
Vue.nextTick(function () {
  // DOM 更新了
})

// 作为一个 Promise 使用
// 如果没有提供回调且在支持 Promise 的环境中，则返回一个 Promise。请注意 Vue 不自带 Promise 的 polyfill，所以如果目标浏览器不原生支持 Promise，得自己提供 polyfill。
Vue.nextTick()
  .then(function () {
    // DOM 更新了
  })
```

**使用时机：**

1. 生命周期的**created()钩子函数进行的DOM操作一定要放在Vue.nextTick()的回调函数中**，原因是在created()钩子函数执行的时候DOM 其实并未进行任何渲染。

```javascript
created(){
  let that = this;
  that.$nextTick(function() {  //不使用this.$nextTick()方法会报错
    that.$refs.aa.innerHTML="created中更改了按钮内容";  //写入到DOM元素
  });
},
```

2. 需要**操作改变后的新的DOM元素**。

```javascript
// 使用示例
methods:{
  testClick: function(){
    let that=this;
    that.testMsg="修改后的值";
    that.$nextTick(function(){
      console.log(that.$refs.aa.innerText);  //输出：修改后的值
    });
  }
}
```

**原理：**

原因是，Vue是异步执行dom更新的，一旦观察到数据变化，Vue就会开启一个队列，然后把在同一个事件循环 (event loop) 当中观察到数据变化的 watcher 推送进这个队列。如果这个watcher被触发多次，只会被推送到队列一次。这种缓冲行为可以有效的去掉重复数据造成的不必要的计算和DOm操作。而在下一个事件循环时，Vue会清空队列，并进行必要的DOM更新。
当你设置 vm.someData = 'new value'，DOM 并不会马上更新，而是在异步队列被清除，也就是下一个事件循环开始时执行更新时才会进行必要的DOM更新。如果此时你想要根据更新的 DOM 状态去做某些事情，就会出现问题。为了在数据变化之后等待 Vue 完成更新 DOM ，可以在数据变化之后立即使用 Vue.nextTick(callback) 。这样回调函数在 DOM 更新完成后就会调用。

#### Vue.set()

当你把一个普通的JS对象传给vue实例的data选项时，vue将遍历此对象的所有属性，并使用 *Object.defineProperty* 把这些属性全部转为 *getter/setter\ 。Object.defineProperty*是ES5中一个无法shim的特性，这也就是为什么vue不支持IE8以及更低版本的浏览器。

受现代JS的限制，**vue不能检测到对象属性的添加或删除**。由于vue会在初始化实例时对属性执行 *getter/setter* 转化过程，所以属性必须在data对象上存在才能让vue转换它，这样它才能是响应的。

**vue不允许在已经创建的实例上动态添加新的根级响应式属性**，不过**可以使用Vue.set()方法将响应式属性添加到嵌套的对象上。** 

```javascript
/* Vue.set(target, key, value)
            target：要更改的数据源(可以是对象或者数组；不能是Vue实例，或者Vue实例的根数据对象。)
            key：要更改的具体数据
            value ：重新赋的值
*/
export default() {
  data() {
    food: {
      name: 'apple' 
    }
  }
}
...
Vue.set(food, 'count', 1);
```

#### Vue.delete

删除对象的 property。如果对象是响应式的，确保删除能触发更新视图。这个方法主要用于避开 Vue 不能检测到 property 被删除的限制，但是你应该很少会使用它。 

```javascript
data:{
  namelist : {
    id: 1, 
    name: '叶落森'
  }       
}

Vue.delete(this.namelist,'name');
```

#### Vue.directive

注册全局指令。详情可以参照 `自定义指令` 。

**参数**：

- `{string} id`
- `{Function | Object} [definition]`

```javascript
// 注册
Vue.directive('my-directive', {
  // 以下5个为自定义指令的生命周期函数
  bind: function () { ... },
  inserted: function () { ... },
  update: function () { ... },
  componentUpdated: function () { ... },
  unbind: function () { ... }
})

// 注册
Vue.directive("hello", function(el, binding, vnode){
  /*
      el: 指令所绑定的元素，可以用来直接操作DOM
      binding: 一个对象，包含指令的很多信息
      vnode: Vue编译生成的虚拟节点
  */
  el.style["color"]= binding.value;
})

// 注册 (指令函数)
Vue.directive('my-directive', function () {
  // 这里将会被 `bind` 和 `update` 调用
})

// getter，返回已注册的指令
var myDirective = Vue.directive('my-directive')
```

#### Vue.filter

注册全局过滤器。详情见 `过滤器` 。

```javascript
// 注册
Vue.filter('my-filter', function (value) {
  // 返回处理后的值
  ......
  return ...
})

// getter，返回已注册的过滤器
var myFilter = Vue.filter('my-filter')
```

#### Vue.component

注册全局组件。注册还会自动使用给定的 `id` 设置组件的名称。详情见 `注册组件` 。

```javascript
// 注册组件，传入一个扩展过的构造器
Vue.component('my-component', Vue.extend({ /* ... */ }))

// 注册组件，传入一个选项对象 (自动调用 Vue.extend)
Vue.component('my-component', { /* ... */ })

// 获取注册的组件 (始终返回构造器)
var MyComponent = Vue.component('my-component')
```

#### Vue.use

安装 Vue.js 插件。如果插件是一个对象，必须提供 `install` 方法。如果插件是一个函数，它会被作为 install 方法。install 方法调用时，会将 Vue 作为参数传入。

该方法需要在调用 `new Vue()` 之前被调用。

当 install 方法被同一个插件多次调用，插件将只会被安装一次。

```javascript
// 调用 `MyPlugin.install(Vue)`
Vue.use(MyPlugin)

new Vue({
  // ...组件选项
})

// 也可以传入一个可选的选项对象
Vue.use(MyPlugin, { someOption: true })
```

#### Vue.mixin

全局注册一个混入，影响注册之后所有创建的每个 Vue 实例。插件作者可以使用混入，向组件注入自定义的行为。**不推荐在应用代码中使用**。详情见 `可复用性 - 混入 - 全局混入` 。

#### Vue.compile

将一个模板字符串编译成 render 函数。**只在完整版时可用**。 

```javascript
var res = Vue.compile('<div><span>{{ msg }}</span></div>')

new Vue({
  data: {
    msg: 'hello'
  },
  render: res.render,
  staticRenderFns: res.staticRenderFns
})
```

#### Vue.observable

让一个对象可响应。Vue 内部会用它来处理 `data` 函数返回的对象。可用于实现简易版的Vuex，或在项目规模不值得使用Vuex时实现组件间数据共享、状态管理。

返回的对象可以直接用于渲染函数和计算属性内，并且会在发生变更时触发相应的更新。也可以作为最小化的跨组件状态存储器，用于简单的场景：

```javascript
const state = Vue.observable({ count: 0 })

const Demo = {
  render(h) {
return h('button', { on: {click: () => {state.count++}} }, `count is: ${state.count}`)
  }
}
```

**简单案例：**

在项目src目录下*建立store.js，在组件里使用提供的 store和 mutation方法，同理其它组件也可以这样使用，从而实现多个组件共享数据状态。* 

```javascript
// src/store.js 首先创建一个 store.js，包含一个 store和一个 mutations，用来指向数据和处理方法。
import Vue from 'vue';

export let store = Vue.observable({count:0,name:'李四'});
export let mutations={
  setCount(count) {
    store.count = count;
  },
  changeName(name) {
    store.name = name;
  }
}
```

```javascript
// Home.vue 然后在组件Home.vue中引用，在组件里使用数据和方法：
<template>  
  <div class="container">  
    <home-header></home-header>  
    <button @click="setCount(count+1)">+1</button>
    <button @click="setCount(count-1)">-1</button>
    <div>store中count：{{count}}</div>
    <button @click="changeName(name1)">父页面修改name</button>
    <div>store中name：{{name}}</div>
    <router-link to="/detail" ><h5>跳转到详情页</h5></router-link>
  </div>  
</template>  
<script>  
  import HomeHeader from '../components/HomeHeader'   
	import {store, mutations} from '@/store'

	export default {  
  	data () {  
    	return {  
      	name1:'主页的name'
    	}  
  	},  
    components: {  
      HomeHeader 
    },  
    computed:{
      count(){
        return store.count
      },
      name(){
        return store.name
      }
    },
    methods:{
      setCount: mutations.setCount,
      changeName: mutations.changeName    
    }
	}  
</script> 
```

```javascript
// Detail.vue 再定义一个子页面观察数据：
<template>  
    <div class="detail">  
        <detail-header></detail-header>  
        <button @click="changeName(name2)">子页面修改name</button>
        <p>store中name：{{name}}</p>
    </div>  
</template>  
<script>  
    import DetailHeader from '../components/DetailHeader'  
    import {store,mutations} from '@/store'
    export default {  
        components: {  
            DetailHeader  
        }  ,
        data(){
            return {
                name2: '子页的name'
            }
        },
        computed:{
            name(){
                return store.name
            }
        },
        methods:{
            changeName: mutations.changeName
        }
    }  
</script>
```

在 Vue 2.x 中，被传入的对象会直接被 `Vue.observable` 变更，所以如[这里展示的](https://cn.vuejs.org/v2/guide/instance.html#数据与方法)，它和被返回的对象是同一个对象。在 Vue 3.x 中，则会返回一个可响应的代理，而对源对象直接进行变更仍然是不可响应的。因此，为了向前兼容，我们推荐始终操作使用 `Vue.observable` 返回的对象，而不是传入源对象。 

#### Vue.version

提供字符串形式的 Vue 安装版本号。这对社区的插件和组件来说非常有用，你可以根据不同的版本号采取不同的策略。 

```javascript
var version = Number(Vue.version.split('.')[0])

if (version === 2) {
  // Vue v2.x.x
} else if (version === 1) {
  // Vue v1.x.x
} else {
  // Unsupported versions of Vue
}
```

### DOM

#### $el

![1590297266244](Vue.assets/1590297266244.png)

即获取 Vue实例挂载到的DOM元素，返回的是一个DOM对象。

```javascript
const vm = new Vue({
	el: ' #app', // 即所挂载的DOM元素
	...
})
```

#### template

![1590300422493](Vue.assets/1590300422493.png)

#### render

`(createElement: () => VNode) => VNode`

字符串模板的代替方案，允许你发挥 JavaScript 最大的编程能力。该渲染函数接收一个 `createElement` 方法作为第一个参数用来创建 `VNode`。

如果组件是一个函数组件，渲染函数还会接收一个额外的 `context` 参数，为没有实例的函数组件提供上下文信息。

Vue 选项中的 `render` 函数若存在，则 Vue 构造函数不会从 `template` 选项或通过 `el`选项指定的挂载元素中提取出的 HTML 模板编译渲染函数。 

#### renderError

![1590302556474](Vue.assets/1590302556474.png)

### 特殊 attribute

#### is

1. 动态切换组件。

   ```javascript
   // 渲染一个“元组件”为动态组件。依 is 的值，来决定哪个组件被渲染。
   
   <!-- 动态组件由 vm 实例的 `componentId` property 控制 -->
   <component :is="componentId"></component>
   
   <!-- 也能够渲染注册过的组件或 prop 传入的组件 -->
   <component :is="$options.components.child"></component>
   ```

   ```javascript
   <template>
     <div>
       <div class="tab-header">
         <button @click="tabChange('Home')">Home</button>
         <button @click="tabChange('Posts')">Posts</button>
         <button @click="tabChange('Archive')">Archive</button>
       </div>
       <div class="tab-body" :is="active"></div>
     </div>
   </template>
   <script>
   import Home from './views/Home.vue';  import Posts from './views/Posts.vue';
   import Archive from './views/Archive.vue';
   export default {
     components: { Home, Posts, Archive},
     data() {
       return {
         active: 'Home'
       };
     },
     methods: {
       tabChange(id){
         this.active = id
       }
     }
   }
   </script>
   ```

2. 解析DOM模板。有些 `HTML` 元素，诸如 `<ul>`、`<ol>`、`<table>` 和 `<select>`，对于哪些元素可以出现在其内部是有严格限制的。而有些元素，诸如 `<li>`、`<tr>` 和 `<option>`，只能出现在其它某些特定的元素内部。

   ```javascript
   <ul>
     <my-component></my-component>
     <my-component></my-component>
   </ul>
   ```

   `my-component`是我们自己写的组件，但是`html`在渲染`dom`的时候，`my-component`对`ul`来说并不是有效的`dom`，会被作为无效的内容提升到外部，并导致最终渲染结果出错。正是因为`html`模板的限制，于是也可以使用 `is` 来做一个变通。

   ```javascript
   <ul>
     <li is='my-component'></li>
   </ul>
   ```









## 网络请求

在网络请求时，在 请求数据的具体组件 和 向服务器请求数据的文件 之间再套一层，例如：

![img](Vue.assets/clip_image002.jpg) ![img](Vue.assets/clip_image004-1587216535850.jpg)

request.js向服务器请求数据，Home.vue面向home.js请求数据而不是直接面向request.js请求数据。

- 如果所有组件都向request.js请求数据的话，request.js会很臃肿。
- 如果request.js中的某一请求数据的方法，例如axios过时的话，在request.js中统一更换方法即可，不需要修改Home.vue向home.js请求数据的过程。

```javascript
// request.js
import axios from 'axios'
（npm install axios --save）

export function request(config) {
  // 1.创建axios的实例

  // 接口：
  // baseURL = "http://123.207.32.32:8000/api/hy"
  // baseURL = "http://106.54.54.237:8000/api/hy"
  const instance = axios.create({
    baseURL: 'http://123.207.32.32:8000/api/hy',
    timeout: 5000
  })

  // 2.axios的拦截器
  // 2.1.请求拦截的作用
  instance.interceptors.request.use(config => {
    return config
  }, err => {
    // console.log(err);
  })

  // 2.2.响应拦截
  instance.interceptors.response.use(res => {
    return res.data
  }, err => {
    console.log(err);
  })

  // 3.发送真正的网络请求
  return instance(config)
}
```

```javascript
// home.js
import {request} from "./request";

export function getHomeMultidata() {
  return request({
    url: '/home/multidata'
  })
}
```



## 响应式的数组方法

数组的各方法看[这里](https://www.jianshu.com/p/9521594710d7)

不是所有的方法都是响应式的（操作后界面会发生改变）

以下的数组方法是响应式的

- **.pop()：将数组末尾的元素删除**
- **.push()：添加数组元素**
- **.shift()：将数组的第一个元素删除**
- **.unshift()：在数组最前面添加元素**
- **.splice()：删除/插入/替换元素**
- **.sort()：排序**
- **.reverse()：反转**
- **Vue.set(要修改的对象, 索引值 , 修改后的值)：Vue内部的函数**

### 替换数组

变更方法，顾名思义，会变更调用了这些方法的原始数组。相比之下，也有非变更方法，例如 `filter()`、`concat()` 和 `slice()`。它们不会变更原始数组，而**总是返回一个新数组**。当使用非变更方法时，可以用新数组替换旧数组： 

```javascript
example1.items = example1.items.filter(function (item) {
  return item.message.match(/Foo/)
})
```

你可能认为这将导致 Vue 丢弃现有 DOM 并重新渲染整个列表。幸运的是，事实并非如此。Vue 为了使得 DOM 元素得到最大范围的重用而实现了一些智能的启发式方法，所以用一个含有相同元素的数组去替换原来的数组是非常高效的操作。 

注意：  由于 JavaScript 的限制，Vue **不能检测**数组和对象的变化。 

## 其他

1. 不让虚拟DOM复用：用不同的key=”..”标识。

2. v-if不显示时是直接从DOM中删除，而v-show是添加display: none。

3. 目前的Vue中，v-bind中不支持驼峰标识，必须用 – 将小写连接 。

4. --save-dev：开发时依赖   --save：运行时依赖

5. $router是创建出来的VueRouter对象；$route是处于活跃的那个路由

6. 打包后的三个文件：

   app…：自己写的代码

   manifest…：用于支撑底层操作的代码

   vendor…：第三方的代码（axios、vue、…）

7. 插槽最终是会被替换的，所以不要在插槽上直接添加属性或样式，而应该在外面定义一个div或其他的，然后将插槽放在里面

8. export 没有用default时，导入需要用 { }

   **import** Swiper **from** **'./Swiper'**

   **import** SwiperItem **from** **'./SwiperItem'** **export** {
   	Swiper, SwiperItem
   }

   此时，

   import Swiper from …

   import SwiperItem from …

   可以变成

   import {Swiper , SwiperItem} from …

9. is属性。有些 `HTML` 元素，诸如 ``、``、`` 和 ``，对于哪些元素可以出现在其内部是有严格限制的。而有些元素，诸如 ``、 和 ``，只能出现在其它某些特定的元素内部。

   ```
   <ul>
     <my-component></my-component>
     <my-component></my-component>
   </ul>
   ```

   `my-component`是我们自己写的组件，但是`html`在渲染`dom`的时候，`my-component`对`ul`来说并不是有效的`dom`，会被作为无效的内容提升到外部，并导致最终渲染结果出错。正是因为`html`模板的限制，于是也可以使用`is`来做一个变通。

   ```
   <ul>
     <li is='my-component'></li>
   </ul>
   ```



# 脚手架 CLI

## Vue CLI

### 什么是Vue CLI

如果你只是简单写几个Vue的Demo程序, 那么你不需要Vue CLI.

如果你在开发大型项目, 那么你需要, 并且必然需要使用Vue CLI

​		使用Vue.js开发大型应用时，我们需要考虑代码目录结构、项目结构和部署、热加载、代码单元测试等事情。

​		如果每个项目都要手动完成这些工作，那无疑效率比较低效，所以通常我们会使用一些脚手架工具来帮助完成这些事情。

**CLI是什么意思**

​		CLI是Command-Line Interface, 翻译为命令行界面, 但是俗称脚手架.

​		Vue CLI是一个官方发布 vue.js 项目脚手架

​		使用 vue-cli 可以快速搭建Vue开发环境以及对应的webpack配置.



### Vue CLI使用前提

#### Node

**安装NodeJS**

​		可以直接在官方网站中下载安装.

​		网址: http://nodejs.cn/download/

![1583985353086](Vue.assets/1583985353086.png)

**检测安装的版本**

​		默认情况下自动安装Node和NPM

​		Node环境要求8.9以上或者更高版本

![1583985368275](Vue.assets/1583985368275.png)

**什么是NPM呢**

​		NPM的全称是Node Package Manager

​		是一个NodeJS包管理和分发工具，已经成为了非官方的发布Node模块（包）的标准。

​		后续我们会经常使用NPM来安装一些开发过程中依赖包.



#### webpack

Vue.js官方脚手架工具就使用了webpack模板

​		对所有的资源会压缩等优化操作

​		它在开发过程中提供了一套完整的功能，能够使得我们开发过程中变得高效。

Webpack的全局安装

​		npm install webpack -g



### Vue CLI的使用

安装Vue脚手架

​		npm install -g @vue/cli

![1583985509595](Vue.assets/1583985509595.png)

注意：上面安装的是Vue CLI3，如果想要按照Vue CLI2的方式初始化项目是不可以的。

![1583985530613](Vue.assets/1583985530613.png)

Vue CLI2初始化项目

​		vue init webpack my-project

Vue CLI3初始化项目

​		vue create my-project



## Vue CLI2

### Vue CLI2详解

![1583985627558](Vue.assets/1583985627558.png)



### 目录结构

![1583985657081](Vue.assets/1583985657081.png)



### Runtime-Compiler 和 Runtime-only 的区别

![1583985693066](Vue.assets/1583985693066.png)

简单总结

​		如果在之后的开发中，你依然使用template，就需要选择Runtime-Compiler

​		如果你之后的开发中，使用的是.vue文件夹开发，那么可以选择Runtime-only

![1583985707049](Vue.assets/1583985707049.png)



### render 和 template

Runtime-Compiler 和 Runtime-only

![1583985753971](Vue.assets/1583985753971.png)

![1583985757459](Vue.assets/1583985757459.png)

为什么存在这样的差异呢？

​		我们需要先理解Vue应用程序是如何运行起来的。

​		Vue中的模板如何最终渲染成真实DOM。



### Vue程序运行过程

![1583985787646](Vue.assets/1583985787646.png)

Runtime-compiler：template -> ast -> render -> vdom -> UI

Runtime-only：render -> vdom -> UI（1.性能更好 2.代码量更少）



### render函数的使用

![1583985829189](Vue.assets/1583985829189.png)

![1583985831637](Vue.assets/1583985831637.png)

![1583985834343](Vue.assets/1583985834343.png)



### npm run build

![1583985882890](Vue.assets/1583985882890.png)



### npm run dev

![1583985897891](Vue.assets/1583985897891.png)



### 修改配置：webpack.base.conf.js起别名

![1583985929413](Vue.assets/1583985929413.png)



## Vue CLI3

### 认识Vue CLI3

于2019.8月发布

vue-cli 3 与 2 版本有很大区别

​		vue-cli 3 是基于 webpack 4 打造，vue-cli 2 还是 webapck 3

​		vue-cli 3 的设计原则是“0配置”，移除的配置文件根目录下的，build和config等目录

​		vue-cli 3 提供了 vue ui 命令，提供了可视化配置，更加人性化

​		移除了static文件夹，新增了public文件夹，并且index.html移动到public中

![1583986013496](Vue.assets/1583986013496.png)



### 目录结构

![1583986504952](Vue.assets/1583986504952.png)



### 配置去哪了

UI方面的配置

​		启动配置服务器：vue ui

![1583986554013](Vue.assets/1583986554013.png)

一大堆配置文件去哪里了？

![1583986576216](Vue.assets/1583986576216.png)



### 自定义配置：起别名

![1583986596508](Vue.assets/1583986596508.png)



# Vuex

## 概念

**单向数据流：**

<img src="Vue.assets/1592747525994.png" alt="1592747525994" style="zoom:67%;" />

 应用遇到**多个组件共享状态**时，单向数据流的简洁性很容易被破坏：

- 多个视图依赖于同一状态。在多层嵌套的组件间传参会非常繁琐，且无法进行兄弟组件间的参数传递。
- 来自不同视图的行为需要变更同一状态。采用父子组件直接引用或者通过事件来变更和同步状态的多份拷贝，这些模式非常脆弱，会导致无法维护的代码。

因此，Vuex将组件的共享状态抽取出来，以一个全局单例模式进行管理。

对于够简单的应用，一个简单的 [store 模式](https://cn.vuejs.org/v2/guide/state-management.html#简单状态管理起步使用) 就足够了。但是，如果需要构建一个中大型单页应用，需要考虑如何更好地在组件外部管理状态，Vuex 将会成为自然而然的选择。 

**状态管理图例：**

![1592745108236](Vue.assets/1592745108236.png)

### 依赖Promise

Vuex 依赖 Promise。如果使用的浏览器并没有实现 Promise (比如 IE)，可以使用一个 polyfill 的库，例如 [es6-promise](https://github.com/stefanpenner/es6-promise)。

你可以通过 CDN 将其引入：

```html
<script src="https://cdn.jsdelivr.net/npm/es6-promise@4/dist/es6-promise.auto.js"></script>
```

然后 `window.Promise` 会自动可用。

如果你喜欢使用诸如 npm 或 yarn 等包管理器，可以按照下列方式执行安装：

```bash
npm install es6-promise --save
yarn add es6-promise
```

或者更进一步，将下列代码添加到你使用 Vuex 之前的一个地方：

```js
import 'es6-promise/auto'
```

## 使用

```javascript
// 安装
<script src="/path/to/vue.js"></script>
<script src="/path/to/vuex.js"></script>

npm install vuex --save

yarn add vuex
```

```javascript
// store文件夹中的index.js
import Vue from 'vue'
import Vuex from 'vuex'

// 1.安装插件
Vue.use(Vuex)

// 2.创建 store（仓库）对象
const store = new Vuex.Store({
  state: {
    title: "hello world",
    ...
  },
  mutations: {
    
  },
  actions: {
    
  },
  getters: {
    
  },
  modules: {
    
  }
})

// 3.导出store，挂载到 Vue 实例中
export default store
```

```javascript
// App.vue
import Vue from 'vue'
import App from '/App'
import store from './store' // 导入

Vue.config.productionTip = false

new Vue({
	el: '#app”,
	store, // 挂载，为 Vue 实例提供创建好的 store，这可以把 store 的实例注入所有的子组件，这样才能使用 this.$store ，ES6写法，相当于（store: store）
	render: h => h(App)
})
```

```javascript
// 使用
在其他组件：
1. 通过 `this.$store.state.xxx` 来获取状态
2. 通过 `this.$store.commit('mutation中方法')` 来修改状态
```



## store

每一个 Vuex 应用的核心就是 **store**（仓库）。store 基本上就是一个容器，包含应用中大部分的**状态 (state)**。Vuex 和单纯的全局对象有以下两点不同：

1. Vuex 的状态存储是响应式的，组件的显示随着数据的改变而改变。
2. 不能直接改变 store 中的状态，改变 store 中的状态的唯一途径就是显式地 **提交（mutation）**。这样可以跟踪每一个状态的变化。

通过 `new Vuex.Store(...)` 来创建一个 store 。

## state

Vuex提出使用单一状态树（单一数据源），即，即使有很多的状态，也只在一个vuex实例中进行操作，而不是创建多个vuex实例进行管理。单状态树和模块化并不冲突，可以将状态和状态变更事件分布到各个子模块中。 

Vuex是响应式的， 从 store 实例中读取状态最简单的方法就是在计算属性中返回某个状态（**不推荐**），这种模式导致组件依赖全局状态单例。在模块化的构建系统中，在每个需要使用 state 的组件中需要频繁地导入，并且在测试组件时需要模拟状态：

```javascript
// 创建一个 Counter 组件
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return store.state.count
    }
  }
}
// 
```

通过在根实例中注册 `store` 选项，该 store 实例会注入到根组件下的所有子组件中，且子组件能通过 `this.$store` 访问到（**推荐**）：

```javascript
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```

### mapState 辅助函数

当一个组件需要获取多个状态的时候，将这些状态都声明为计算属性会有些重复和冗余。可以使用 `mapState` 辅助函数帮助生成计算属性：

```javascript
// 实际开发中使用
computed: {
  /* 使用ES6对象展开运算符将此对象混入到外部对象中 */
  ...mapState({
    title1: "title", // 写法1 第一个title代表定义到当前组件的属性，"title"代表Vuex中的属性名
    title2: state => state.title, // 写法2 
    ...
  }),
    
  localComputed () { /* ... */ },
}
```

原来的写法：

```javascript
import { mapState } from 'vuex' // 在单独构建的版本中辅助函数为 Vuex.mapState

export default {
  // ...
  computed: mapState({
    count: state => state.count, // 箭头函数可使代码更简练
    countAlias: 'count', // 
    countPlusLocalState (state) { // 为了能够使用 `this` 获取局部状态，必须使用常规函数
      return state.count + this.localCount
    }
  })
  
  // 当映射的计算属性的名称与 state 的子节点名称相同时，我们也可以给 mapState 传一个字符串数组。
  computed: mapState([
    'count' // 映射 this.count 为 store.state.count
  ])
}
```

`mapState` 函数返回的是一个对象，如何将它与局部计算属性混合使用呢？通常，需要使用一个工具函数将多个对象合并为一个，以使我们可以将最终对象传给 `computed` 属性。但是自从有了[对象展开运算符](https://github.com/tc39/proposal-object-rest-spread)，可以简化写法。

> 使用 Vuex 并不意味着你需要将**所有的**状态放入 Vuex。虽然将所有的状态放到 Vuex 会使状态变化更显式和易调试，但也会使代码变得冗长和不直观。如果有些状态严格属于单个组件，最好还是作为组件的局部状态。你应该根据你的应用开发需要进行权衡和确定。

## getters

getters用于操作state中的数据，类似于computed操作data中的数据，主要用于有**多个**组件需要用到相应处理后的state数据。与计算属性一样，getter的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。 

```javascript
// 对于只有单个组件中需要对state中的数据进行相应的操作，可以
computed: {
  doneTodosCount () {
    return this.$store.state.todos.filter(todo => todo.done).length
  }
}
```

```javascript
...
getters: {
  doneTodosCount: state => { // 第一个参数为Vuex中的state，用来获取state中的数据
    return state.todos.filter(todo => todo.done).length
  }
	getLength: (state, getters) => { // 第二个参数即为getters本身，用于获取getters中的其他方法
		return getters.abc.length
	},
  // getters默认是不能传递参数的, 如果希望传递参数, 那么只能让getters本身返回另一个函数
  // 注意：getter通过方法访问时，每次都会进行调用，不会缓存结果。
  getTodoById: state => {
    return id => { // 第二个参数的形参表示传入的形参，以下例子中为 2
			return state.todos.find(todo => todo.id === id)
    }
  }
}
```

```javascript
// 使用
<template>
	<div id="app">
		...
		<h2>{{$store.getters.getLength}}</h2>
		<h2>{{$store.getters.getTodoById(2)}}</h2> // 传递参数
	</div>
</template>
...
```

### mapGetters 辅助函数

将 store 中的 getter 映射到局部计算属性

```javascript
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
  // 使用对象展开运算符将 getter 混入 computed 对象中
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
			// 如果想给一个 getter 属性取别名，使用对象形式：
      xxx1: 'xxx' // 将`this.doneCount`映射为`this.$store.getters.xxx`
    ]),
    
    localComputed () { /* ... */ },
  }
}
```

## mutation

### 使用

mutation是store更新状态的**唯一**方式。

每个mutation包括两部分：

​		字符串的**事件类型**（type）

​		一个**回调函数**（handler），该回调函数的第一个参数就是state。

注意：mutation中的每个方法所做的事情尽可能单一，如果有异步操作、复杂的逻辑判断等，放到action中。

mutation的定义方式：

```javascript
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment(state) { // 没有传递参数
    state.count++
    },
    incrementTen(state, n) {
      state.count += n
    },
    incrementCount(state, payload) {
       // 但此时，mutations中的方法接受的Payload是一个对象，参数需要从对象中提取 payload.amount
      state.count += payload.amount
    }
  }
})
```

通过mutation更新，注意，不能直接调用一个mutation，需要以相应的 type 调用。

```javascript
// 在组件中调用
<template>
 <button @click="addCount(5)">5</button> // 监听点击
</template>

methods: {
  addCount(count) {
    // 通过 $store.commit 调用 mutations 中的方法更新state中的状态
    this.$store.commit('increment') // 不带参数
    this.$store.commit('incrementTen', 10) // 额外的参数被称为是mutation的载荷(Payload)
    this.$store.commit('incrementCount', { // 大多数情况下，载荷(Payload)应该是一个对象
  		amount: 10, // 此时，mutations中的方法接受的Payload是一个对象，参数需要从对象中提取
      ...
    })
    
    // 提交mutation的另一种方式是直接使用包含type属性的对象，当使用对象风格的提交方式，整个对象都作为载荷传给mutation函数，因此mutation函数的写法不变。但此时，mutations中的方法接受的Payload是一个对象，参数需要从对象中提取，例如payload.amount。
    const count = { ... } // 如果有多个参数需要传递，可以放在一个对象中，再传递整个对象
    this.$store.commit({
      type: 'incrementCount', // 即mutations中对应的方法
      amount: 10,
      ....
      count, // 可以使用ES6语法，相当于 count: count
    })
  }
}
...
```

### mapMutations 辅助函数

```javascript
import { mapMutations } from 'vuex'
export default {
  ....
  methods: {
    ...mapMutations([
      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`

      // `mapMutations` 也支持载荷：
      'incrementBy' 
  // 将 `this.incrementBy(amount)` 映射为`this.$store.commit('incrementBy', amount)`
    ]),
      
    // 使用别名
    ...mapMutations({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
    })
  }
}
```

### 遵守 Vue 的响应规则

Vuex的store中的state是响应式的，当state中的数据发生改变时，Vue组件会自动更新。

这就要求我们必须遵守一些Vuex对应的规则：

1. 提前在store中初始化所需的属性。

2. 当给state中的对象添加新属性时, 使用下面的方式：

   ​	方式一：使用 `Vue.set(obj, 'newProp', 123)`

   ​	方式二：用新对象给旧对象重新赋值（替换老对象）， 例如，利用对象展开运算符我们可以这样写： 

   ```javascript
   state.obj = { ...state.obj, newProp: 123 }
   ```

在state中定义的数据，都会被加入到响应式系统中，而响应式系统会监听属性的变化，当属性发生变化时，会通知所有界面中用到该属性的地方，让界面发生刷新。

如果新添加并未在store中初始化的数据，则不会响应式更新界面。

如果想要后来添加（并未在store中初始化）的数据也变成响应式，需要调用`Vue.set()`方法。

```javascript
state.info['address'] = '洛杉矶' // 直接添加的数据不是响应式的
Vue.set(state.info, 'address', '洛杉矶')

// 同样
delete state.info.age // 直接删除的数据也不是响应式的
Vue.delete(state.info, 'age') 
```

### 使用type常量

用不用常量取决于你——在需要多人协作的大型项目中，这会很有帮助。但如果你不喜欢，你完全可以不这样做。 

在store文件夹下创建 `mutations-types.js` 文件

```javascript
// mutations-types.js
export const INCREMENT = 'increment'
```

```javascript
// 组件中使用
import {INCREMENT} from '../mutations-types.js'
...
methods: {
  add() {
    this.$store.commit(INCREMENT) // 使用
  }
}
```

```javascript
// 在vuex中使用
import {INCREMENT} from '../mutations-types.js'
...
mutations: {
  // 可以使用ES6的计算属性命名功能来使用一个常量作为函数名
  [INCREMTN](state) {
    ...
  },
  ...
}
```

### 必须是同步函数

mutation 必须是同步方法。

主要的原因是，当我们使用devtools时，可以让devtools可以帮助我们捕捉mutation的快照。

但是如果是异步函数，回调何时执行很难确定，那么devtools将不能追踪这个操作什么时候会被完成——实质上任何在回调函数中进行的状态的改变都是不可追踪的。 

但，如果确实需要使用异步操作，比如网络请求，此时，则**用action代替mutation进行异步操作**。

## action

action 类似于 mutation，不同在于：

- Action 通过 mutation 来更改状态，而不是直接变更状态。
- Action 可以包含任意异步操作及复杂的判断逻辑。

action 函数接受一个与 store 实例具有相同方法和属性的 context 对象（但并不是store实例），因此可以调用 `context.commit`提交一个 mutation，通过 `context.state` 和 `context.getters` 来获取 state 和 getters。 

```javascript
// Vuex
...
mutations: {
  increment(state, payload) {
    state.count += payload.acount 
  }
},
actions: {
  // 在 action 中调用 mutation 进行异步操作
	// context: 上下文 可以理解为 $store对象
  increment (context) { // 没有异步操作
    context.commit('increment')
  }
  // payload为传递的参数
	xxx(context, payload) { // 有异步操作
    return new Promise((resolve) => { // 建议使用 promise
			setTimeout(() => { // 模拟异步操作
				context.commit('increment') // 不传参数
        context.commit('increment', payload) // 将参数继续传递到mutation
				resolve()
			}，1000)
		})
	}
  
  // 实践中，经常用到ES6的 参数解构 来简化代码，特别是需要多次使用commit的时候
  yyy ({ commit }) {
    commit('increment') // 直接用commit，不需要context.commit
  }
  
  // 在另一个action中：
  actionB ({ dispatch, commit }) {
    return dispatch('actionA').then(() => {
      commit('someOtherMutation')
    })
  }
  
  // 结合 async / await
  // 假设 getData() 和 getOtherData() 返回的是 Promise
  async actionA ({ commit }) {
    commit('gotData', await getData())
  },
  async actionB ({ dispatch, commit }) {
    await dispatch('actionA') // 等待 actionA 完成
    commit('gotOtherData', await getOtherData())
  }
}
...
```

```javascript
// 组件
...
methods: {
  add() {
    // 组件中, 使用 $store.dispatch 调用 action 中的方法
    this.$store.dispatch('increment') // 不传递参数
    // 以载荷形式分发
    this.$store.dispatch('increment', {
      amount: 10
    })
    // 以对象形式分发
    this.$store.dispatch({
      type: 'increment',
      amount: 10
    })
    // 用.then接受Promise的返回
    this.$store.dispatch('increment', {acount: 5}).then(res => { // 传递参数
      console.log('完成了更新操作')
    }) 
  }
}
```

###  mapActions 辅助函数

```javascript
import { mapActions } from 'vuex'
export default {
  ....
  methods: {
    ...mapActions([
      'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`
  
      // `mapActions` 也支持载荷：
      'incrementBy' 
  // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
    ]),
    
    // 使用别名
    ...mapActions({
      add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
    })
  }
}
```

###  组合 Action

Action 通常是异步的，那么如何知道 action 什么时候结束呢？更重要的是，我们如何才能组合多个 action，以处理更加复杂的异步流程？

首先，你需要明白 `store.dispatch` 可以处理被触发的 action 的处理函数返回的 Promise，并且 `store.dispatch` 仍旧返回 Promise：

```js
actions: {
  actionA ({ commit }) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  }
}
```

现在你可以：

```javascript
store.dispatch('actionA').then(() => {
  // ...
})
```

在另外一个 action 中也可以：

```js
actions: {
  // ...
  actionB ({ dispatch, commit }) {
    return dispatch('actionA').then(() => {
      commit('someOtherMutation')
    })
  }
}
```

最后，如果我们利用 [async / await](https://tc39.github.io/ecmascript-asyncawait/)，我们可以如下组合 action：

```js
// 假设 getData() 和 getOtherData() 返回的是 Promise

actions: {
  async actionA ({ commit }) {
    commit('gotData', await getData())
  },
  async actionB ({ dispatch, commit }) {
    await dispatch('actionA') // 等待 actionA 完成
    commit('gotOtherData', await getOtherData())
  }
}
```

> 一个 `store.dispatch` 在不同模块中可以触发多个 action 函数。在这种情况下，只有当所有触发函数完成后，返回的 Promise 才会执行。



## modele

Vue使用单一状态树，应用的所有状态都会集中到一个比较大的对象store，当应用变得非常复杂时，store对象就有可能变得相当臃肿。

为了解决这个问题，Vuex允许我们将store分割成模块module，每个module拥有自己的state、mutations、actions、getters、甚至是嵌套子模块等。所有的module最终会合并到store中。

```javascript
const moduleA = {
	state: { 
    name: 'xxx' 
  }，
  // 一般不与store中的mutation、getters等方法命名冲突
	mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
	state: { ... }，
	mutations: { ... }，
	actions: { ... }
}

const store = new Vuex. Store({
	modules: {
		a: moduleA,
		b: moduleB
  }
})
```

```javascript
// 组件中调用
$store.state.a.name
$store.dispatch('aUpdateName')
$store.getters.sumWithRootCount // 根据方法名直接调用即可，不关心定义在哪里
```

### 模块的局部状态

对于模块内部的 mutation 和 getter，接收的第一个参数是**模块的局部状态对象**，而不是store。

对于模块内部的 getter，根节点状态会作为**第三个参数**暴露出来，其余的则是模块自身的局部状态或方法。

对于模块内部的 action，局部状态通过 `context.state` 暴露出来，根节点状态则为 `context.rootState` 。根结点的方法为 `context.rootGetters` 。

```javascript
const moduleA = {
  state: () => ({
    count: 0
  }),
  mutations: {
    increment (state) { // 这里的 `state` 对象是模块的局部状态，而不是store
      state.count++
    }
  },
  getters: {
    doubleCount (state) {
      return state.count * 2
    },
    // state、getters为模块自身的局部状态或方法，rootState为store中的state
    sumWithRootCount (state, getters, rootState) {
      return state.count + rootState.count
    }
  },
  actions: {
    // 局部状态通过 `context.state` 暴露出来，根节点状态则为 `context.rootState` 。
    incrementIfOddOnRootSum ({ state, commit, rootState }) { // ES6解构语法
      if ((state.count + rootState.count) % 2 === 1) {
        commit('increment')
        // 此时，commit 只会调用自己模块(moduleA)中的mutations
        // 如果想要拿到根(store)中的方法，则使用：context.rootGetters 或 解构为rootGetters
      }
    }
  }
}
```

### 命名空间

默认情况下，模块内部的 action、mutation 和 getter 是注册在**全局命名空间**的，这样使得多个模块能够对同一 mutation 或 action 作出响应。

如果希望你的模块具有更高的封装度和复用性，可以通过添加 `namespaced: true` 的方式使其成为带命名空间的模块。当模块被注册后，它的所有 getter、action 及 mutation 都会自动根据模块注册的路径调整命名。

启用了命名空间的 getter 和 action 会收到局部化的 `getter`，`dispatch` 和 `commit`。换言之，你在使用模块内容（module assets）时不需要在同一模块内额外添加空间名前缀。更改 `namespaced` 属性后不需要修改模块内的代码。

```js
const store = new Vuex.Store({
  modules: {
    account: {
      namespaced: true, // 添加namespaced

      // 模块内容（module assets）
      state: () => ({ ... }), // 模块内的状态已经是嵌套的了，使用namespaced属性不会产生影响
      getters: {
        isAdmin () { ... } // -> getters['account/isAdmin']
      },
      actions: {
        login () { ... } // -> dispatch('account/login')
      },
      mutations: {
        login () { ... } // -> commit('account/login')
      },

      // 嵌套模块
      modules: {
        // 继承父模块的命名空间
        myPage: {
          state: () => ({ ... }),
          getters: {
            profile () { ... } // -> getters['account/profile']
          }
        },

        // 进一步嵌套命名空间
        posts: {
          namespaced: true, // 添加namespaced

          state: () => ({ ... }),
          getters: {
            popular () { ... } // -> getters['account/posts/popular']
          }
        }
      }
    }
  }
})
```

#### 在带命名空间的模块内访问全局内容（Global Assets）

如果你希望使用全局 state 和 getter，`rootState` 和 `rootGetters` 会作为**第三**和**第四**个参数传入 getter，也会通过 `context` 对象的属性传入 action。

若需要在全局命名空间内分发 action 或提交 mutation，将 `{ root: true }` 作为第三参数传给 `dispatch`或 `commit` 即可。

```js
modules: {
  foo: {
    namespaced: true,

    getters: {
      // 在这个模块的 getter 中，`getters` 被局部化了
      // 你可以使用 getter 的第四个参数来调用 `rootGetters`
      someGetter (state, getters, rootState, rootGetters) {
        getters.someOtherGetter // -> 'foo/someOtherGetter'
        rootGetters.someOtherGetter // -> 'someOtherGetter'
      },
      someOtherGetter: state => { ... }
    },

    actions: {
      // 在这个模块中， dispatch 和 commit 也被局部化了
      // 他们可以接受 `root` 属性以访问根 dispatch 或 commit
      someAction ({ dispatch, commit, getters, rootGetters }) {
        getters.someGetter // -> 'foo/someGetter'
        rootGetters.someGetter // -> 'someGetter'

        dispatch('someOtherAction') // -> 'foo/someOtherAction'
        dispatch('someOtherAction', null, { root: true }) // -> 'someOtherAction'

        commit('someMutation') // -> 'foo/someMutation'
        commit('someMutation', null, { root: true }) // -> 'someMutation'
      },
      someOtherAction (ctx, payload) { ... }
    }
  }
}
```

#### 在带命名空间的模块注册全局 action

若需要在带命名空间的模块注册全局 action，你可添加 `root: true`，并将这个 action 的定义放在函数 `handler` 中。例如：

```js
actions: {
  someOtherAction ({dispatch}) {
    dispatch('someAction')
  }
},
modules: {
  foo: {
    namespaced: true,

    actions: {
      someAction: {
        root: true,
        handler (namespacedContext, payload) { ... } // -> 'someAction'
      }
    }
  }
}
```

#### 带命名空间的绑定函数

当使用 `mapState`, `mapGetters`, `mapActions` 和 `mapMutations` 这些函数来绑定带命名空间的模块时，写起来可能比较繁琐：

```js
computed: {
  ...mapState({
    a: state => state.some.nested.module.a,
    b: state => state.some.nested.module.b
  })
},
methods: {
  ...mapActions([
    'some/nested/module/foo', // -> this['some/nested/module/foo']()
    'some/nested/module/bar' // -> this['some/nested/module/bar']()
  ])
}
```

对于这种情况，可以将模块的空间名称字符串作为第一个参数传递给上述函数，这样所有绑定都会自动将该模块作为上下文。于是上面的例子可以简化为：

```js
computed: {
  ...mapState('some/nested/module', {
    a: state => state.a,
    b: state => state.b
  })
},
methods: {
  ...mapActions('some/nested/module', [
    'foo', // -> this.foo()
    'bar' // -> this.bar()
  ])
}
```

而且，你可以通过使用 `createNamespacedHelpers` 创建基于某个命名空间辅助函数。它返回一个对象，对象里有新的绑定在给定命名空间值上的组件绑定辅助函数：

```js
import { createNamespacedHelpers } from 'vuex'

const { mapState, mapActions } = createNamespacedHelpers('some/nested/module')

export default {
  computed: {
    // 在 `some/nested/module` 中查找
    ...mapState({
      a: state => state.a,
      b: state => state.b
    })
  },
  methods: {
    // 在 `some/nested/module` 中查找
    ...mapActions([
      'foo',
      'bar'
    ])
  }
}
```

#### 给插件开发者的注意事项

如果你开发的插件（Plugin）提供了模块并允许用户将其添加到 Vuex store，可能需要考虑模块的空间名称问题。对于这种情况，你可以通过插件的参数对象来允许用户指定空间名称：

```js
// 通过插件的参数对象得到空间名称，然后返回 Vuex 插件函数
export function createPlugin (options = {}) {
  return function (store) {
    // 把空间名字添加到插件模块的类型（type）中去
    const namespace = options.namespace || ''
    store.dispatch(namespace + 'pluginAction')
  }
}
```

### 模块动态注册

在 store 创建**之后**，可以使用 `store.registerModule` 方法注册模块：

```js
import Vuex from 'vuex'

const store = new Vuex.Store({ /* 选项 */ })

// 注册模块 `myModule`
store.registerModule('myModule', {
  ...
})
// 注册嵌套模块 `nested/myModule`
store.registerModule(['nested', 'myModule'], {
  ...
})
```

之后就可以通过 `store.state.myModule` 和 `store.state.nested.myModule` 访问模块的状态。

模块动态注册功能使得其他 Vue 插件可以通过在 store 中附加新模块的方式来使用 Vuex 管理状态。例如，[`vuex-router-sync`](https://github.com/vuejs/vuex-router-sync) 插件就是通过动态注册模块将 vue-router 和 vuex 结合在一起，实现应用的路由状态管理。

你也可以使用 `store.unregisterModule(moduleName)` 来动态卸载模块。注意，你不能使用此方法卸载静态模块（即创建 store 时声明的模块）。

注意，你可以通过 `store.hasModule(moduleName)` 方法检查该模块是否已经被注册到 store。

#### 保留 state

在注册一个新 module 时，你很有可能想保留过去的 state，例如从一个服务端渲染的应用保留 state。你可以通过 `preserveState` 选项将其归档：`store.registerModule('a', module, { preserveState: true })`。

当你设置 `preserveState: true` 时，该模块会被注册，action、mutation 和 getter 会被添加到 store 中，但是 state 不会。这里假设 store 的 state 已经包含了这个 module 的 state 并且你不希望将其覆写。

### 模块重用

有时我们可能需要创建一个模块的多个实例，例如：

- 创建多个 store，他们公用同一个模块 (例如当 `runInNewContext` 选项是 `false` 或 `'once'` 时，为了[在服务端渲染中避免有状态的单例](https://ssr.vuejs.org/en/structure.html#avoid-stateful-singletons))
- 在一个 store 中多次注册同一个模块

如果我们使用一个纯对象来声明模块的状态，那么这个状态对象会通过引用被共享，导致状态对象被修改时 store 或模块间数据互相污染的问题。

实际上这和 Vue 组件内的 `data` 是同样的问题。因此解决办法也是相同的——使用一个函数来声明模块状态：

```js
const MyReusableModule = {
  state: () => ({
    foo: 'bar'
  }),
  // mutation, action 和 getter 等等...
}
```

## 目录结构

Vuex 并不限制你的代码结构。但是，它规定了一些需要遵守的规则：

1. 应用层级的状态应该集中到单个 store 对象中。
2. 提交 **mutation** 是更改状态的唯一方法，并且这个过程是同步的。
3. 异步逻辑都应该封装到 **action** 里面。

不管是怎样的结构，只要遵守以上规则即可。如果你的 store 文件太大，只需将 action、mutation 和 getter 分割到单独的文件。

对于大型应用，我们会希望把 Vuex 相关代码分割到模块中。下面是项目结构示例：

```javascript
...
 └─ store
 			├─ index.js					 # 我们组装模块并导出 store 的地方
 			├─ actions.js				 # 根级别的 action
 			├─ mutations.js			 # 根级别的 mutation
 			└─ modules
 						├─ cart.js		 # 购物车模块
 						└─ products.js # 产品模块
```

## 插件

Vuex 的 store 接受 `plugins` 选项，这个选项暴露出每次 mutation 的钩子。Vuex 插件就是一个函数，它接收 store 作为唯一参数：

```js
const myPlugin = store => {
  // 当 store 初始化后调用
  store.subscribe((mutation, state) => {
    // 每次 mutation 之后调用
    // mutation 的格式为 { type, payload }
  })
}
```

然后像这样使用：

```js
const store = new Vuex.Store({
  // ...
  plugins: [myPlugin]
})
```

### 在插件内提交 Mutation

在插件中也只能通过提交 mutation 来改变state。

下面是个大概例子，实际上 `createPlugin` 方法可以有更多选项来完成复杂任务：

```js
export default function createWebSocketPlugin (socket) {
  return store => {
    socket.on('data', data => {
      store.commit('receiveData', data)
    })
    store.subscribe(mutation => {
      if (mutation.type === 'UPDATE_DATA') {
        socket.emit('update', mutation.payload)
      }
    })
  }
}
const plugin = createWebSocketPlugin(socket)

const store = new Vuex.Store({
  state,
  mutations,
  plugins: [plugin]
})
```

### 生成 State 快照

有时候插件需要获得状态的“快照”，比较改变的前后状态。想要实现这项功能，你需要对状态对象进行深拷贝：

```js
const myPluginWithSnapshot = store => {
  let prevState = _.cloneDeep(store.state)
  store.subscribe((mutation, state) => {
    let nextState = _.cloneDeep(state)

    // 比较 prevState 和 nextState
    ....

    // 保存状态，用于下一次 mutation
    prevState = nextState
  })
}
```

**生成状态快照的插件应该只在开发阶段使用**，使用 webpack 或 Browserify，让构建工具帮我们处理：

```js
const store = new Vuex.Store({
  // ...
  plugins: process.env.NODE_ENV !== 'production'
    ? [myPluginWithSnapshot]
    : []
})
```

上面插件会默认启用。在发布阶段，你需要使用 webpack 的 [DefinePlugin](https://webpack.js.org/plugins/define-plugin/) 或者是 Browserify 的 [envify](https://github.com/hughsk/envify) 使 `process.env.NODE_ENV !== 'production'` 为 `false`。

### 内置 Logger 插件

> 如果正在使用 [vue-devtools](https://github.com/vuejs/vue-devtools)，你可能不需要此插件。

Vuex 自带一个日志插件用于一般的调试:

```js
import createLogger from 'vuex/dist/logger'

const store = new Vuex.Store({
  plugins: [createLogger()]
})
```

`createLogger` 函数有几个配置项：

```js
const logger = createLogger({
  collapsed: false, // 自动展开记录的 mutation
  filter (mutation, stateBefore, stateAfter) {
    // 若 mutation 需要被记录，就让它返回 true 即可
    // 顺便，`mutation` 是个 { type, payload } 对象
    return mutation.type !== "aBlocklistedMutation"
  },
  actionFilter (action, state) {
    // 和 `filter` 一样，但是是针对 action 的
    // `action` 的格式是 `{ type, payload }`
    return action.type !== "aBlocklistedAction"
  },
  transformer (state) {
    // 在开始记录之前转换状态
    // 例如，只返回指定的子树
    return state.subTree
  },
  mutationTransformer (mutation) {
    // mutation 按照 { type, payload } 格式记录
    // 我们可以按任意方式格式化
    return mutation.type
  },
  actionTransformer (action) {
    // 和 `mutationTransformer` 一样，但是是针对 action 的
    return action.type
  },
  logActions: true, // 记录 action 日志
  logMutations: true, // 记录 mutation 日志
  logger: console, // 自定义 console 实现，默认为 `console`
})
```

日志插件还可以直接通过 `  <script>  ` 标签引入，它会提供全局方法 `createVuexLogger`。

要注意，logger 插件会生成状态快照，所以仅在开发环境使用。

## 严格模式

开启严格模式，仅需在创建 store 的时候传入 `strict: true`：

```js
const store = new Vuex.Store({
  // ...
  strict: true
})
```

在严格模式下，无论何时发生了状态变更且不是由 mutation 函数引起的，将会抛出错误。这能保证所有的状态变更都能被调试工具跟踪到。

### 开发环境与发布环境

**不要在发布环境下启用严格模式**！严格模式会深度监测状态树来检测不合规的状态变更——请确保在发布环境下关闭严格模式，以避免性能损失。

类似于插件，我们可以让构建工具来处理这种情况：

```js
const store = new Vuex.Store({
  // ...
  strict: process.env.NODE_ENV !== 'production'
})
```

## 表单处理

当在严格模式中使用 Vuex 时，在属于 Vuex 的 state 上使用 `v-model` 会比较棘手：

```html
<input v-model="obj.message">
```

假设这里的 `obj` 是在计算属性中返回的一个属于 Vuex store 的对象，在用户输入时，`v-model` 会试图直接修改 `obj.message`。在严格模式中，由于这个修改不是在 mutation 函数中执行的, 这里会抛出一个错误。

用“Vuex 的思维”去解决这个问题的方法是：给 ` <input> ` 中绑定 value，然后侦听 `input` 或者 `change`事件，在事件回调中调用一个方法:

```javascript
<input :value="message" @input="updateMessage">
// ...
computed: {
  ...mapState({
    message: state => state.obj.message
  })
},
methods: {
  updateMessage (e) {
    this.$store.commit('updateMessage', e.target.value)
  }
}
```

下面是 mutation 函数：

```js
// ...
mutations: {
  updateMessage (state, message) {
    state.obj.message = message
  }
}
```

### 双向绑定的计算属性

必须承认，这样做比简单地使用“`v-model` + 局部状态”要啰嗦得多，并且也损失了一些 `v-model` 中很有用的特性。另一个方法是使用带有 setter 的双向绑定计算属性：

```js
<input v-model="message">
// ...
computed: {
  message: {
    get () {
      return this.$store.state.obj.message
    },
    set (value) {
      this.$store.commit('updateMessage', value)
    }
  }
}
```

## 测试

官网：https://vuex.vuejs.org/zh/guide/testing.html

## 热重载

使用 webpack 的 [Hot Module Replacement API](https://webpack.js.org/guides/hot-module-replacement/)，Vuex 支持在开发过程中热重载 mutation、module、action 和 getter。你也可以在 Browserify 中使用 [browserify-hmr](https://github.com/AgentME/browserify-hmr/) 插件。

对于 mutation 和模块，你需要使用 `store.hotUpdate()` 方法：

```js
// store.js
import Vue from 'vue'
import Vuex from 'vuex'
import mutations from './mutations'
import moduleA from './modules/a'

Vue.use(Vuex)

const state = { ... }

const store = new Vuex.Store({
  state,
  mutations,
  modules: {
    a: moduleA
  }
})

if (module.hot) {
  // 使 action 和 mutation 成为可热重载模块
  module.hot.accept(['./mutations', './modules/a'], () => {
    // 获取更新后的模块
    // 因为 babel 6 的模块编译格式问题，这里需要加上 `.default`
    const newMutations = require('./mutations').default
    const newModuleA = require('./modules/a').default
    // 加载新模块
    store.hotUpdate({
      mutations: newMutations,
      modules: {
        a: newModuleA
      }
    })
  })
}
```

参考热重载示例 [counter-hot](https://github.com/vuejs/vuex/tree/dev/examples/counter-hot)。

### 动态模块热重载

如果你仅使用模块，你可以使用 `require.context` 来动态地加载或热重载所有的模块。

```js
// store.js
import Vue from 'vue'
import Vuex from 'vuex'

// 加载所有模块
function loadModules() {
  const context = require.context("./modules", false, /([a-z_]+)\.js$/i)

  const modules = context
    .keys()
    .map((key) => ({ key, name: key.match(/([a-z_]+)\.js$/i)[1] }))
    .reduce(
      (modules, { key, name }) => ({
        ...modules,
        [name]: context(key).default
      }),
      {}
    )

  return { context, modules }
}

const { context, modules } = loadModules()

Vue.use(Vuex)

const store = new Vuex.Store({
  modules
})

if (module.hot) {
  // 在任何模块发生改变时进行热重载。
  module.hot.accept(context.id, () => {
    const { modules } = loadModules()

    store.hotUpdate({
      modules
    })
  })
}
```

























# 风格指南

## 必要的

1. 组件名字应为 多个单词，例如 ` todo-item `，Vue的内置组件除外。这样可以避免跟HTML元素冲突。

2. 组件的data必须是函数，使每个组件实例都拥有自己的数据容器。当data的值是对象时，它会在这个组件的所有实例之间共享。

3. 尽量详细定义Prop，例如类型检查、是否必要等。

4. `v-for` 设置 `key` 键值。例如 ` :key="todo.id" `

5. 不要 `v-if` 和 `v-for` 在同一元素一起使用。 比如 `v-for="user in users" v-if="user.isActive"` 。在这种情形下，请将 `users` 替换为一个计算属性 (比如 `activeUsers`)，让其返回过滤后的列表；或用div等元素包裹后将 `v-if` 用在div上。
   `v-for` 比 `v-if` 具有更高的优先级。如果每一次都需要遍历整个数组，将会影响速度，尤其是当之需要渲染很小一部分的时候。 

   ```javascript
   // 错误用法
   <ul>
     <li
       v-for="user in users"
       v-if="user.isActive"
       :key="user.id">
       	{{ user.name }}
     </li>
   </ul>
   // 即使100个user中之需要使用一个数据，也会循环整个数组。
   ```

   ```javascript
   // 正确用法
   computed: {
   	activeUsers: function () {
   		return this.users.filter(function (user) {
     		return user.isActive
   		})
   	}
   }
   <ul> // 或将 v-if 用在 <ul> 上
   	<li
     v-for="user in activeUsers"
     :key="user.id">
   		{{ user.name }}
   	</li>
   </ul>
   ```

6. 对于应用来说，顶级 `App` 组件和布局组件中的样式可以是全局的，但是其它所有组件的样式都应该是有作用域的。

7. Vue使用模块作用域保持不允许外部访问的函数的私有性。如果无法做到这一点，就始终为插件、混入等不考虑作为对外公共 API 的自定义私有 property 使用 `$_` 前缀。并附带一个命名空间以回避和其它作者的冲突 (比如 `$_yourPluginName_`)。
   Vue 使用 `_` 前缀来定义其自身的私有 property，所以使用相同的前缀 (比如 `_update`)  有覆写实例 property 的风险。 
    对于 `$` 前缀来说，其在 Vue 生态系统中的目的是暴露给用户的一个特殊的实例 property，所以把它用于*私有* property 并不合适。 
   我们推荐把这两个前缀结合为 `$_`，作为一个用户定义的私有 property 的约定，以确保不会和 Vue 自身相冲突。 

## 强烈推荐

1. 尽量把每个组件单独分成文件。

2. 单文件组件的命名 始终是单词大写开头 或 横线连接：` MyComponent.vue ` 或 ` my-component.vue `

3. 基础组件名。应用特定样式和约定的基础组件 (也就是展示类的、无逻辑的或无状态的组件【只包括HTML元素、其他基础组件、第三方UI组件库】绝不会包括全局状态) 应该全部以一个特定的前缀开头，比如 `Base`、`App` 或 `V` ，再包含所包裹元素的名字， 比如 `BaseButton`、`BaseTable` 。
   好处：

   * 以字母顺序排序时，你的应用的基础组件会全部列在一起 

   * 组件名应该始终是多个单词，所以这样做可以避免你在包裹简单组件时随意选择前缀 (比如 `MyButton`、`VueButton`)。 

   * 这些组件会被频繁使用， 你可能想把它们放到全局而不是在各处分别导入它们。使用相同的前缀可以让 webpack 这样工作： 

     ```javascript
     var requireComponent = require.context("./src", true, /Base[A-Z]\w+\.(vue|js)$/)
     requireComponent.keys().forEach(function (fileName) {
       var baseComponentConfig = requireComponent(fileName)
       baseComponentConfig = baseComponentConfig.default || baseComponentConfig
       var baseComponentName = baseComponentConfig.name || (
         fileName
           .replace(/^.+\//, '')
           .replace(/\.\w+$/, '')
       )
       Vue.component(baseComponentName, baseComponentConfig)
     })
     ```

4. 单例组件名。在每个页面中只使用一次。只应该拥有单个活跃实例的组件应该以 `The` 前缀命名，以示其唯一性。例如` TheHeading.vue `。

5. 和父组件紧密耦合的子组件应该以父组件名作为前缀命名。父组件命名的目录中嵌套子组件这种方案并不是很好，因为会导致1.许多文件的名字相同 2.过多嵌套的子目录 

   ```javascript
   // 例如
   components/
   |- TodoList.vue
   |- TodoListItem.vue
   |- TodoListItemButton.vue
   ```

6. 组件名应该以高级别的 (通常是一般化描述的) 单词开头，以描述性的修饰词结尾。

   ```javascript
   // 例如
   components/
   |- SearchButtonClear.vue
   |- SearchButtonRun.vue
   |- SearchInputQuery.vue
   |- SearchInputExcludeGlob.vue
   |- SettingsCheckboxTerms.vue
   |- SettingsCheckboxLaunchOnStartup.vue
   ```

7. 在单文件组件、字符串模板和JSX中没有内容的组件应该是自闭合的——但在DOM模板里永远不要这么做。

   ```javascript
   <!-- 在单文件组件、字符串模板和 JSX 中 -->
   <MyComponent/>
   
   <!-- 在 DOM 模板中 -->
   <my-component></my-component>
   ```

8. 对于绝大多数项目来说，在[单文件组件](https://cn.vuejs.org/v2/guide/single-file-components.html)和字符串模板中组件名应该总是 PascalCase 的——但是在 DOM 模板中总是 kebab-case 的。

   ```javascript
   <!-- 在单文件组件和字符串模板中 -->
   <MyComponent/>
     
   <!-- 在 DOM 模板中 -->
   <my-component></my-component>
   
   <!-- 在所有地方 -->
   <my-component></my-component>
   ```

9. JS/[JSX](https://cn.vuejs.org/v2/guide/render-function.html#JSX) 中的组件名应该始终是 PascalCase 的，尽管在较为简单的应用中只使用 `Vue.component` 进行全局组件注册时，可以使用 kebab-case 字符串。
   在 JavaScript 中，PascalCase 是类和构造函数 (本质上任何可以产生多份不同实例的东西) 的命名约定。 
   对于**只**通过 `Vue.component` 定义的推荐 kebab-case。 

   ```javascript
   Vue.component('MyComponent', {
     // ...
   })
   
   Vue.component('my-component', {
     // ...
   })
   
   import MyComponent from './MyComponent.vue'
   
   export default {
     name: 'MyComponent',
     // ...
   }
   ```

10. 组件名应该倾向于完整单词而不是缩写。编辑器中的自动补全已经让书写长命名的代价非常之低了，而其带来的明确性却是非常宝贵的。 例如` StudentDashboardSettings.vue `

11. 在声明 prop 的时候，其命名应该始终使用 camelCase，而在模板和 [JSX](https://cn.vuejs.org/v2/guide/render-function.html#JSX) 中应该始终使用 kebab-case。

12. 多个 attribute 的元素应该分多行撰写，每个 attribute 一行。

    ```javascript
    <MyComponent
      foo="a"
      bar="b"
      baz="c"
    />
    ```

13. 组件模板应该只包含简单的表达式，复杂的表达式则应该重构为计算属性或方法。

14. 复杂计算属性分割为尽可能多的更简单的 property。

    ```javascript
    // 不好
    computed: {
      price: function () {
        var basePrice = this.manufactureCost / (1 - this.profitMargin)
        return (
          basePrice -
          basePrice * (this.discountPercent || 0)
        )
      }
    }
    
    // 好
    computed: {
      basePrice: function () {
        return this.manufactureCost / (1 - this.profitMargin)
      },
      discount: function () {
        return this.basePrice * (this.discountPercent || 0)
      },
      finalPrice: function () {
        return this.basePrice - this.discount
      }
    }
    ```

15. 非空 HTML attribute 值应该始终带引号。

    ```javascript
    <input type="text">
    
    <AppSidebar :style="{ width: sidebarWidth + 'px' }">
    ```

16. 指令缩写：要么都用，要么一个都不用。

## 推荐



## 谨慎使用

1. 元素选择器应该避免在 `scoped` 中出现，尽量选类选择器。
2. 优先通过 prop 和事件进行父子组件之间的通信，而不是 `this.$parent` 或变更 prop，特殊情况除外。
3. 优先通过 [Vuex](https://github.com/vuejs/vuex) 管理全局状态，而不是通过 `this.$root` 或一个全局事件总线。



# 源码分析

非Vue源码，而是模仿Vue的demo。

## 1. 准备

1.[].slice.call(lis): 将伪数组转换为真数组

2.node.nodeType: 得到节点类型

3.Object.defineProperty(obj, propertyName, {}): 给对象添加/修改属性(指定描述符)
	configurable: true/false  是否可以重新define
	enumerable: true/false 是否可以枚举(for..in / keys())
	value: 指定初始值
	writable: true/false value是否可以修改存取(访问)描述符
	get: 函数, 用来得到当前属性值
	set: 函数, 用来监视当前属性值的变化

4.Object.keys(obj): 得到对象自身可枚举的属性名的数组

5.DocumentFragment: 文档碎片(高效批量更新多个节点)

6.obj.hasOwnProperty(prop): 判断prop是否是obj自身的属性

## 2. 数据代理(MVVM.js)

1.通过一个对象代理对另一个对象中属性的操作(读/写)

2.通过vm对象来代理data对象中所有属性的操作

3.好处: 更方便的操作data中的数据

4.基本实现流程
	1). 通过Object.defineProperty()给vm添加与data对象的属性对应的属性描述符
	2). 所有添加的属性都包含getter/setter
	3). 在getter/setter内部去操作data中对应的属性数据。

```javascript

```

## 3. 模板解析(compile.js)

1.模板解析的关键对象: compile对象

2.模板解析的基本流程:

​		1). 将el的所有子节点取出, 添加到一个新建的文档fragment对象中

​		2). 对fragment中的所有层次子节点递归进行编译解析处理

​				* 对大括号表达式文本节点进行解析

​				* 对元素节点的指令属性进行解析

​				* 事件指令解析

​				* 一般指令解析

​		3). 将解析后的fragment添加到el中显示

3.解析**大括号表达式**文本节点: textNode.textContent = value

​		1). 根据正则对象得到匹配出的表达式字符串: 子匹配/RegExp.$1

​		2). 从data中取出表达式对应的属性值

​		3). 将属性值设置为文本节点的textContent

​			**例如，{{name}}**

​				**一、取出大括号中的值name**

​				**二、根据name匹配相应的值**

​				**三、设置到节点的textContent中**

4.事件指令解析: elementNode.addEventListener(事件名, 回调函数.bind(vm))

​				v-on:click="test"

​		**1). 从指令名中取出事件名**

​		**2). 根据指令的值(表达式)从methods中得到对应的事件处理函数对象**

​		**3). 给当前元素节点绑定指定事件名和回调函数的dom事件监听**

​		**4). 指令解析完后，移除此指令属性**

5.一般指令解析: elementNode.xxx = value

​		**1). 得到指令名和指令值(表达式)**   text/html/class   msg/myClass

​		**2). 从data中根据表达式得到对应的值**

​		**3). 根据指令名确定需要操作元素节点的什么属性**

​				*** v-text ---> textContent属性**

​				*** v-html ---> innerHTML属性**

​				*** v-class ---> className属性**

​		**4). 将得到的表达式的值设置到对应的属性上**

​		**5). 移除元素的指令属性**



```javascript
/*
	解析 大括号表达式 和 指令
*/
function Compile(el, vm) {
  this.$vm = vm; // 保存vm到compile对象
  this.$el = this.isElementNode(el) ? el : document.querySelector(el); // 将el对应的元素对象保存到compile对象中

  if (this.$el) { // 如果el元素存在
    /* 编译模板的整体三步 */
    this.$fragment = this.node2Fragment(this.$el); // 1.取出el元素中所有子节点, 封装在一个framgment对象中
    this.init(); // 2.编译fragment中所有层次子节点（这一步是最重要的）
    this.$el.appendChild(this.$fragment); // 3.将编译好的fragment添加到页面的el元素中
  }
}

Compile.prototype = {
  node2Fragment: function (el) {
    // 创建空的fragment
    var fragment = document.createDocumentFragment(),
        child;

    while (child = el.firstChild) { // 将el元素中所有子节点拷贝到fragment
      fragment.appendChild(child);
    }

    return fragment; // 将包含所有el元素中的子节点的fragment返回
  },

  init: function () {
    this.compileElement(this.$fragment); // 编译指定元素（所有层次的子节点）
  },

  compileElement: function (el) {
    var childNodes = el.childNodes, // 得到最外层所有子节点
        me = this; // 保存compile对象

    /* 遍历所有子节点（文本节点 / 元素节点） */
    [].slice.call(childNodes).forEach(function (node) {
      var text = node.textContent; // 得到节点的文本内容
      var reg = /\{\{(.*)\}\}/; // 正则对象(匹配大括号表达式)
     
      if (me.isElementNode(node)) { // 判断节点是否是一个元素节点
        me.compile(node); // 编译元素节点的指令（解析指令）
      } else if (me.isTextNode(node) && reg.test(text)) { // 判断节点是否是大括号表达式格式的文本节点
        me.compileText(node, RegExp.$1); // 编译大括号表达式格式的文本节点
      }
     
      if (node.childNodes && node.childNodes.length) { // 如果子节点还有子节点，递归调用实现所有层次节点的编译
        me.compileElement(node); // 递归调用，编译节点
      }
    });
  },

  compile: function (node) {
    var nodeAttrs = node.attributes, // 得到所有标签属性节点
        me = this;
    /* 遍历所有属性 */
    [].slice.call(nodeAttrs).forEach(function (attr) {
      var attrName = attr.name; // 得到属性名  v-on:click
     
      if (me.isDirective(attrName)) { // 判断是否是指令属性
        var exp = attr.value; // 得到表达式(属性值): test
        var dir = attrName.substring(2); // 从属性名中得到指令名: on:click
       
        if (me.isEventDirective(dir)) { // 如果是事件指令  on: 开头
          compileUtil.eventHandler(node, me.$vm, exp, dir); // 解析事件指令
        } else { // 如果是普通指令
          compileUtil[dir] && compileUtil[dir](node, me.$vm, exp); // 解析普通指令
        }

        node.removeAttribute(attrName); // 完成后将指令属性移除
      }
    });
  },

  compileText: function (node, exp) {
    // 调用编译工具对象解析
    compileUtil.text(node, this.$vm, exp);
  },

  isDirective: function (attr) {
    return attr.indexOf('v-') == 0;
  },

  isEventDirective: function (dir) {
    return dir.indexOf('on') === 0;
  },

  isElementNode: function (node) {
    return node.nodeType == 1;
  },

  isTextNode: function (node) {
    return node.nodeType == 3;
  }
};

/* 包含多个解析指令的方法的工具对象 */
var compileUtil = {
  // 解析: v-text/{{}}
  text: function (node, vm, exp) {
    this.bind(node, vm, exp, 'text');
  },

  // 解析: v-html
  html: function (node, vm, exp) {
    this.bind(node, vm, exp, 'html');
  },

  // 解析: v-model
  model: function (node, vm, exp) {
    this.bind(node, vm, exp, 'model');

    var me = this,
        val = this._getVMVal(vm, exp);

    node.addEventListener('input', function (e) {
      var newValue = e.target.value;

      if (val === newValue) {
        return;
      }

      me._setVMVal(vm, exp, newValue);
      val = newValue;
    });
  },

  // 解析: v-class，原生中没有这个语法，这里是为了方便
  class: function (node, vm, exp) {
    this.bind(node, vm, exp, 'class');
  },

  // 真正用于解析指令的bind方法
  bind: function (node, vm, exp, dir) {
    /* 实现初始化显示 */
    // 得到更新节点的函数
    var updaterFn = updater[dir + 'Updater']; // 根据指令名（dir）(text)得到对应的更新节点函数
    updaterFn && updaterFn(node, this._getVMVal(vm, exp)); // 调用函数更新节点

    // 创建表达式对应的watcher对象，实现节点的更新显示
    new Watcher(vm, exp, function (value, oldValue) { // 当表达式对应的属性值变化时回调
      updaterFn && updaterFn(node, value, oldValue); // 更新界面中的指定节点
    });

  },

  // 事件处理
  eventHandler: function (node, vm, exp, dir) {
    var eventType = dir.split(':')[1], // 得到事件名/类型: click
        fn = vm.$options.methods && vm.$options.methods[exp]; // 从methods中得到表达式所对应的函数（事件回调函数） test(){}

    if (eventType && fn) { // 如果都存在
      // 给节点绑定指定事件名和回调函数(强制绑定this为vm)的DOM事件监听
      node.addEventListener(eventType, fn.bind(vm), false);
    }
  },

  // 从vm得到表达式对应的值
  _getVMVal: function (vm, exp) {
    // 对表达式进行拆分，例如 a.b.c
    var val = vm._data;
    exp = exp.split('.');
    exp.forEach(function (k) {
      val = val[k];
    });
    return val;
  },

  _setVMVal: function (vm, exp, value) {
    var val = vm._data;
    exp = exp.split('.');
    exp.forEach(function (k, i) {
      // 非最后一个key，更新val的值
      if (i < exp.length - 1) {
        val = val[k];
      } else {
        val[k] = value;
      }
    });
  }
};

/* 包含多个用于更新节点的方法的工具对象 */
var updater = {
  // 更新节点的textContent属性值
  textUpdater: function (node, value) {
    node.textContent = typeof value == 'undefined' ? '' : value; // 显示文本内容
  },

  // 更新节点的innerHTML属性值
  htmlUpdater: function (node, value) {
    node.innerHTML = typeof value == 'undefined' ? '' : value;
  },

  // 更新节点的className属性值
  classUpdater: function (node, value, oldValue) {
    var className = node.className; // 静态class属性的值
    // space?' ':'' 表示，如果有静态的class，则将其用空格与动态的class连接
    node.className = className + (space?' ':'') + value;
  },

  // 更新节点的value属性值
  modelUpdater: function (node, value, oldValue) {
    node.value = typeof value == 'undefined' ? '' : value;
  }
};
```



## 4. 数据劫持 --> 数据绑定

1.数据绑定(model==>View)

​		1). 一旦更新了data中的某个属性数据，所有界面上直接使用或间接使用了此属性的节点都会更新

​				操作元素 ==> 操作数据

​				* 初始化显示：页面(表达式 / 指令)能从data读取数据显示(模板 编译/解析)

​				* 更新显示：更新data中的属性数据 ==> 页面更新

2.数据劫持

​				Vue用 数据劫持 的技巧 来实现 数据绑定 的效果

​		1). 数据劫持是vue中用来实现数据绑定的一种技术

​		2). 基本思想: 通过defineProperty()来监视data中所有属性(任意层次)数据的变化，一旦变化就去更新界面

3.四个重要对象

​		1). Observer

​				* 用来对data所有属性数据进行劫持的构造函数

​				* 给data中所有属性重新定义属性描述(get/set)

​				* data中的每个属性创建对应的dep对象

​		2). Dep(Depend)

​				* data中的每个属性(所有层次)都对应一个dep对象

​					例如：以下会创建4个Dep

```javascript
data: {
	name: '123',
	abc: {
		name: '456',
		age: 18
	}							
}
```

​				* 创建的时机

​						~ 在初始化define data各个属性进行数据劫持时创建对应的dep对象

​						~ 在 data 中的某个属性值被设置为新的对象时

​				* Dep的结构

```javascript
{
	id,  // 每个dep都有一个唯一的id
	subs // 包含n个对应watcher的数组(subscribes的简写)
}
```

​				* subs属性说明

​						~ 当一个watcher被创建时, 内部会将当前watcher对象添加到对应的dep对象的subs中

​                 		~ 当此data属性的值发生改变时，所有subs中的watcher都会收到更新的通知，更新对应的界面

​		3). Compile

​				* 用来解析模板页面的对象的构造函数(一个实例)

​				* 利用compile对象解析模板页面

​				* 每解析一个表达式(非事件指令)都会创建一个对应的watcher对象, 并建立watcher与dep的关系

​				* complie与watcher关系: 一对多的关系

​		4). Watcher

​				* 模板中每个表达式(不包含事件指令)都对应一个watcher对象

​						例如，以下包含3个watcher对象

```javascript
<div id="test">
	<p>{{name}}</p> // 1个
	<p v-text="name"</p> // 2个
	<p v-text="name"</p> // 3个
	<button v-on:click="updata">更新</button> // 不包含事件指令
</div>
```

​				* 监视当前表达式数据的变化

​				* 创建的时机: 在初始化编译模板(初始化解析大括号表达式/一般指令)时创建

​				* 对象的组成

```javascript
{
  this.vm = vm,   // vm对象
  this.exp = exp, // 对应指令的表达式
  this.cb = cb,   // 当表达式所对应的数据发生改变的回调函数
  this.value = this.get(),  // 表达式当前的值
  this.depIds = {} // 表达式中各级属性所对应的dep对象的集合对象
  			 					// 属性名为dep的id, 属性值为dep
}
```

​		5). 总结: dep与watcher的关系: 多对多

​				* 一个data中的属性对应对应一个dep，一个dep中可能包含多个watcher（属性在模板中被多次使用【{{a}} / v-text="a"   2个watcher】  data属性 --> Dep --> n个watcher）

​				* 模板中一个非事件表达式对应一个watcher，一个watcher中可能包含多个dep（表达式中包含了多个data属性即多层表达式【a.b  2个Dep；a.b.c  3个Dep】  表达式 --> watcher --> n个Dep）

​				* 数据绑定使用到2个核心技术：

​						~ defineProperty()

​						~ 消息订阅与发布

## 5. 代码

```javascript
/* compile.js
相关于Vue的构造函数
 */
function Compile(el, vm) {
  this.$vm = vm; // 保存vm到compile对象
  this.$el = this.isElementNode(el) ? el : document.querySelector(el); // 将el对应的元素对象保存到compile对象中

  if (this.$el) { // 如果el元素存在
    /* 编译模板的整体三步 */
    this.$fragment = this.node2Fragment(this.$el); // 1.取出el元素中所有子节点, 封装在一个framgment对象中
    this.init(); // 2.编译fragment中所有层次子节点（这一步是最重要的）
    this.$el.appendChild(this.$fragment); // 3.将编译好的fragment添加到页面的el元素中
  }
}

Compile.prototype = {
  node2Fragment: function (el) {
    // 创建空的fragment
    var fragment = document.createDocumentFragment(),
        child;

    while (child = el.firstChild) { // 将el元素中所有子节点拷贝到fragment
      fragment.appendChild(child);
    }

    return fragment; // 将包含所有el元素中的子节点的fragment返回
  },

  init: function () {
    this.compileElement(this.$fragment); // 编译指定元素（所有层次的子节点）
  },

  compileElement: function (el) {
    var childNodes = el.childNodes, // 得到最外层所有子节点
        me = this; // 保存compile对象

    /* 遍历所有子节点（文本节点 / 元素节点） */
    [].slice.call(childNodes).forEach(function (node) {
      var text = node.textContent; // 得到节点的文本内容
      var reg = /\{\{(.*)\}\}/; // 正则对象(匹配大括号表达式)
     
      if (me.isElementNode(node)) { // 判断节点是否是一个元素节点
        me.compile(node); // 编译元素节点的指令（解析指令）
      } else if (me.isTextNode(node) && reg.test(text)) { // 判断节点是否是大括号表达式格式的文本节点
        me.compileText(node, RegExp.$1); // 编译大括号表达式格式的文本节点
      }
     
      if (node.childNodes && node.childNodes.length) { // 如果子节点还有子节点，递归调用实现所有层次节点的编译
        me.compileElement(node); // 递归调用，编译节点
      }
    });
  },

  compile: function (node) {
    var nodeAttrs = node.attributes, // 得到所有标签属性节点
        me = this;
    /* 遍历所有属性 */
    [].slice.call(nodeAttrs).forEach(function (attr) {
      var attrName = attr.name; // 得到属性名  v-on:click
     
      if (me.isDirective(attrName)) { // 判断是否是指令属性
        var exp = attr.value; // 得到表达式(属性值): test
        var dir = attrName.substring(2); // 从属性名中得到指令名: on:click
       
        if (me.isEventDirective(dir)) { // 如果是事件指令  on: 开头
          compileUtil.eventHandler(node, me.$vm, exp, dir); // 解析事件指令
        } else { // 如果是普通指令
          compileUtil[dir] && compileUtil[dir](node, me.$vm, exp); // 解析普通指令
        }

        node.removeAttribute(attrName); // 完成后将指令属性移除
      }
    });
  },

  compileText: function (node, exp) {
    // 调用编译工具对象解析
    compileUtil.text(node, this.$vm, exp);
  },

  isDirective: function (attr) {
    return attr.indexOf('v-') == 0;
  },

  isEventDirective: function (dir) {
    return dir.indexOf('on') === 0;
  },

  isElementNode: function (node) {
    return node.nodeType == 1;
  },

  isTextNode: function (node) {
    return node.nodeType == 3;
  }
};

/* 包含多个解析指令的方法的工具对象 */
var compileUtil = {
  // 解析: v-text/{{}}
  text: function (node, vm, exp) {
    this.bind(node, vm, exp, 'text');
  },

  // 解析: v-html
  html: function (node, vm, exp) {
    this.bind(node, vm, exp, 'html');
  },

  // 解析: v-model
  model: function (node, vm, exp) {
    this.bind(node, vm, exp, 'model'); // 实现数据的初始化显示和创建对应watcher

    var me = this,
        val = this._getVMVal(vm, exp); // 得到表达式的值

    node.addEventListener('input', function (e) { // 给节点绑定input事件监听(输入改变时)
      var newValue = e.target.value; // 得到输入的最新值
      if (val === newValue) { return; } // 如果没有变化，直接结束
      me._setVMVal(vm, exp, newValue); // 将最新value保存给表达式所对应的属性
      val = newValue; // 保存最新的值
    });
  },

  // 解析: v-class，原生中没有这个语法，这里是为了方便
  class: function (node, vm, exp) {
    this.bind(node, vm, exp, 'class');
  },

  // 真正用于解析指令的bind方法
  bind: function (node, vm, exp, dir) {
    /* 实现初始化显示 */
    // 得到更新节点的函数
    var updaterFn = updater[dir + 'Updater']; // 根据指令名（dir）(text)得到对应的更新节点函数
    updaterFn && updaterFn(node, this._getVMVal(vm, exp)); // 调用函数更新节点

    // 创建表达式对应的watcher对象
    new Watcher(vm, exp, function (value, oldValue) { 
      /* 更新界面 */
      updaterFn && updaterFn(node, value, oldValue); // 当对应的属性值发生了变化时, 自动调用, 更新对应的节点
    });

  },

  // 事件处理
  eventHandler: function (node, vm, exp, dir) {
    var eventType = dir.split(':')[1], // 得到事件名/类型: click
        fn = vm.$options.methods && vm.$options.methods[exp]; // 从methods中得到表达式所对应的函数（事件回调函数） test(){}

    if (eventType && fn) { // 如果都存在
      // 给节点绑定指定事件名和回调函数(强制绑定this为vm)的DOM事件监听
      node.addEventListener(eventType, fn.bind(vm), false);
    }
  },

  // 从vm得到表达式对应的值
  _getVMVal: function (vm, exp) {
    // 对表达式进行拆分，例如 a.b.c
    var val = vm._data;
    exp = exp.split('.');
    exp.forEach(function (k) {
      val = val[k];
    });
    return val;
  },

  _setVMVal: function (vm, exp, value) {
    var val = vm._data;
    exp = exp.split('.');
    exp.forEach(function (k, i) {
      // 非最后一个key，更新val的值
      if (i < exp.length - 1) {
        val = val[k];
      } else {
        val[k] = value;
      }
    });
  }
};

/* 包含多个用于更新节点的方法的工具对象 */
var updater = {
  // 更新节点的textContent属性值
  textUpdater: function (node, value) {
    node.textContent = typeof value == 'undefined' ? '' : value; // 显示文本内容
  },

  // 更新节点的innerHTML属性值
  htmlUpdater: function (node, value) {
    node.innerHTML = typeof value == 'undefined' ? '' : value;
  },

  // 更新节点的className属性值
  classUpdater: function (node, value, oldValue) {
    var className = node.className; // 静态class属性的值
    // space?' ':'' 表示，如果有静态的class，则将其用空格与动态的class连接
    node.className = className + (space?' ':'') + value;
  },

  // 更新节点的value属性值
  modelUpdater: function (node, value, oldValue) {
    node.value = typeof value == 'undefined' ? '' : value;
  }
};
```

```javascript
/* mvvm.js
相关于Vue的构造函数
 */
function MVVM(options) {
  this.$options = options; // 将选项对象保存到vm
  var data = this._data = this.$options.data; // 将data对象保存到vm和datq变量中
  var me = this; //将vm保存在me变量中
  
  Object.keys(data).forEach(function (key) { // 遍历data中所有属性，key是data的某个属性名，这里只有name
    me._proxy(key); // 对指定属性实现代理
  });

  observe(data, this); // 对data进行监视

  this.$compile = new Compile(options.el || document.body, this) // 创建一个用来编译模板的compile对象
}

MVVM.prototype = {
  $watch: function (key, cb, options) {
    new Watcher(this, key, cb);
  },

  // 对指定属性实现数据代理的方法
  _proxy: function (key) {
    var me = this; // 保存vm
    // 给vm添加指定属性名的属性(使用的是属性描述符)
    Object.defineProperty(me, key, { // defineProperty方法
      configurable: false, // 不能再重新定义，防止别人修改
      enumerable: true, // 可以枚举遍历
      // 当通过vm.name读取属性值时调用，从data中获取对应的属性值返回
      get: function proxyGetter() {
        return me._data[key]; // 读取data中对应属性值返回(实现代理 读 操作)
      },
      // 当通过vm.name = 'xxx'时调用，value被保存到data中对应的属性上
      set: function proxySetter(newVal) {
        me._data[key] = newVal; // 将最新的值保存到data中对应的属性上(实现代理 写 操作)
      }
    });
  }
};
```

```javascript
/* observer.js

*/
function Observer(data) {
    this.data = data; // 保存data对象
    this.walk(data); // 开始对data的监视
}

Observer.prototype = {
    walk: function(data) {
        var me = this; // 保存observer对象
        Object.keys(data).forEach(function(key) { // 遍历data中所有属性
            me.defineReactive(me.data, key, data[key]); // 针对指定属性进行响应式的数据绑定
        });
    },

    defineReactive: function(data, key, val) {
        var dep = new Dep(); // 创建与当前属性对应的dep对象 dep:dependency
        var childObj = observe(val); // 间接递归调用实现对data中所有层次属性的数据劫持
       
        Object.defineProperty(data, key, { // 给data重新定义属性，添加set/get
            enumerable: true, // 可枚举
            configurable: false, // 不能重新定义
            get: function() { // get方法 返回值，建立dep与watcher之间的关系 
                if (Dep.target) { // 只有指定了watcher才建立关系
                    dep.depend(); // 建立dep与watcher的关系
                }
                return val; // 返回属性值
            },
            set: function(newVal) { // set方法 监视key属性的变化，更新界面
                if (newVal === val) {
                    return;
                }
                val = newVal;
                childObj = observe(newVal); // 新的值是object的话，进行监听
                dep.notify(); // 通过所有相关的订阅者(watcher)
            }
        });
    }
};

function observe(value, vm) {
    if (!value || typeof value !== 'object') { // 被观察的必须是一个对象, 因为监视的是对象内部的属性
        return;
    }
    return new Observer(value); // 创建一个对应的观察都对象
};

var uid = 0;

function Dep() {
    this.id = uid++; // 标识属性
    this.subs = []; // 相关的所有watcher的数组
}

Dep.prototype = {
    // 添加watcher到dep中
    addSub: function(sub) {
        this.subs.push(sub);
    },
    
    // 建立dep与watcher之间的关系
    depend: function() {
        Dep.target.addDep(this);
    },

    // 移出
    removeSub: function(sub) {
        var index = this.subs.indexOf(sub);
        if (index != -1) {
            this.subs.splice(index, 1);
        }
    },

    notify: function() {
        // 通知所有相关的watcher(一个订阅者)进行更新
        this.subs.forEach(function(sub) {
            sub.update();
        });
    }
};

Dep.target = null;
```

```javascript
/* watcher.js

*/
function Watcher(vm, exp, cb) {
  this.cb = cb;  // 更新界面的回调
  this.vm = vm;
  this.exp = exp; // 表达式
  this.depIds = {};  // 包含所有相关的dep的容器对象
  this.value = this.get(); // 得到表达式的初始值
}

Watcher.prototype = {
  update: function () {
    this.run();
  },
  run: function () {
    var value = this.get(); // 得到最新的值
    var oldVal = this.value; // 得到旧值
   
    if (value !== oldVal) { // 如果不相同
      this.value = value;
      this.cb.call(this.vm, value, oldVal); // 调用回调函数更新对应的界面
    }
  },
  addDep: function (dep) {
    if (!this.depIds.hasOwnProperty(dep.id)) { // 判断dep与watcher的关系是否已经建立
      dep.addSub(this); // 将watcher添加到dep  用于更新
      this.depIds[dep.id] = dep; // 将dep添加到watcher  用于防止重复建立关系
    }
  },

  // 得到表达式的值，建立dep与watcher的关系
  get: function () {
    Dep.target = this; // 给dep指定当前的watcher
    var value = this.getVMVal(); // 获取当前表达式的值, 内部调用get()建立dep与watcher的关系
    Dep.target = null; // 将dep中的watcher去除，建立关系后就没有必要再在dep中保存watcher
    return value;
  },

  // 得到表达式对应的值
  getVMVal: function () {
    var exp = this.exp.split('.');
    var val = this.vm._data;
    exp.forEach(function (k) {
      val = val[k];
    });
    return val;
  }
};
```



## 6. MVVM 原理图分析

 ![data](Vue.assets/data.png) 

![1587820815205](Vue.assets/1587820815205.png)

![1587820930445](Vue.assets/1587820930445.png)

![1592538087394](Vue.assets/1592538087394.png)

![1592541166175](Vue.assets/1592541166175.png)

## 7. 双向数据绑定

双向数据绑定是建立在单向数据绑定（model ---> View）的基础之上

双向数据绑定的实现流程：
		1.在解析v-model指令时，给当前元素添加input监听
		2.当input的value发生改变时，取出最新的值赋给对应的data属性，而data属性通过单向数据绑定更新界面。

​		3.当界面的值发生变化时，将data中对应的值赋给input的value属性，改变input的值。

 							a.b=3 -> set -> dep -> watcher 

v-model 其实是一个语法糖，它的背后本质上是包含两个操作：

​		1.v-bind绑定一个value属性

​		2.v-on指令给当前元素绑定事件

```javascript
<input v-model="searchText">
// 相当于
<input
  :value="searchText"
  @input="searchText = $event.target.value">
```

# 框架对比

## [React](https://cn.vuejs.org/v2/guide/comparison.html#React)

React 和 Vue 有许多相似之处，它们都有：

- 使用虚拟DOM
- 提供了响应式 (Reactive) 和组件化 (Composable) 的视图组件。
- 将注意力集中保持在核心库，而将其他功能如路由和全局状态管理交给相关的库。

### [运行时性能](https://cn.vuejs.org/v2/guide/comparison.html#运行时性能)

React 和 Vue 都是非常快的，所以速度并不是在它们之中做选择的决定性因素。对于具体的数据表现，可以移步这个[第三方 benchmark](https://stefankrause.net/js-frameworks-benchmark8/table.html)，它专注于渲染/更新非常简单的组件树的真实性能。

#### 优化

在 React 中，当某个组件的状态发生变化时，它会以该组件为根，重新渲染**整个**组件子树。

如要避免不必要的子组件的重渲染，你需要在所有可能的地方使用 `PureComponent`，或是手动实现 `shouldComponentUpdate` 方法。同时你可能会需要使用不可变的数据结构来使得你的组件更容易被优化。

然而，使用 `PureComponent` 和 `shouldComponentUpdate` 时，需要保证该组件的整个子树的渲染输出都是由该组件的 props 所决定的。如果不符合这个情况，那么此类优化就会导致难以察觉的渲染结果不一致。这使得 **React 中的组件优化伴随着相当的心智负担**。

在 Vue 中，组件的依赖是在渲染过程中自动追踪的，所以系统能精确知晓哪个组件确实需要被重渲染。你可以理解为每一个组件都已经自动获得了 `shouldComponentUpdate`，并且没有上述的子树问题限制。

**Vue 的这个特点使得开发者不再需要考虑此类优化，从而能够更好地专注于应用本身。**

### [HTML & CSS](https://cn.vuejs.org/v2/guide/comparison.html#HTML-amp-CSS)

在 React 中，一切都是 JavaScript。不仅仅是 HTML 可以用 JSX 来表达，现在的潮流也越来越多地将 CSS 也纳入到 JavaScript 中来处理。这类方案有其优点，但也存在一些不是每个开发者都能接受的取舍。

Vue 的整体思想是拥抱经典的 Web 技术，并在其上进行扩展。我们下面会详细分析一下。

#### JSX vs Templates

**在 React 中，所有的组件的渲染功能都依靠 JSX。JSX 是使用 XML 语法编写 JavaScript 的一种语法糖**。

使用 JSX 的渲染函数有下面这些优势：

- 你可以使用完整的编程语言 JavaScript 功能来构建你的视图页面。比如你可以使用临时变量、JS 自带的流程控制、以及直接引用当前 JS 作用域中的值等等。
- 开发工具对 JSX 的支持相比于现有可用的其他 Vue 模板还是比较先进的 (比如，linting、类型检查、编辑器的自动完成)。

事实上 Vue 也提供了[渲染函数](https://cn.vuejs.org/v2/guide/render-function.html)，甚至[支持 JSX](https://cn.vuejs.org/v2/guide/render-function.html#JSX)。然而，我们默认推荐的还是模板。任何合乎规范的 HTML 都是合法的 Vue 模板，这也带来了一些特有的优势：

- 对于很多习惯了 HTML 的开发者来说，模板比起 JSX 读写起来更自然。这里当然有主观偏好的成分，但如果这种区别会导致开发效率的提升，那么它就有客观的价值存在。
- 基于 HTML 的模板使得将已有的应用逐步迁移到 Vue 更为容易。
- 这也使得设计师和新人开发者更容易理解和参与到项目中。
- 你甚至可以使用其他模板预处理器，比如 Pug 来书写 Vue 的模板。

有些开发者认为模板意味着需要学习额外的 DSL (Domain-Specific Language 领域特定语言) 才能进行开发——我们认为这种区别是比较肤浅的。首先，JSX 并不是没有学习成本的——它是基于 JS 之上的一套额外语法。同时，正如同熟悉 JS 的人学习 JSX 会很容易一样，熟悉 HTML 的人学习 Vue 的模板语法也是很容易的。最后，DSL 的存在使得我们可以让开发者用更少的代码做更多的事，比如 `v-on` 的各种修饰符，在 JSX 中实现对应的功能会需要多得多的代码。

更抽象一点来看，我们可以把组件区分为两类：一类是偏视图表现的 (presentational)，一类则是偏逻辑的 (logical)。我们推荐在前者中使用模板，在后者中使用 JSX 或渲染函数。这两类组件的比例会根据应用类型的不同有所变化，但整体来说我们发现表现类的组件远远多于逻辑类组件。

#### 组件作用域内的 CSS

除非你把组件分布在多个文件上 (例如 [CSS Modules](https://github.com/gajus/react-css-modules))，CSS 作用域在 React 中是通过 CSS-in-JS 的方案实现的 (比如 [styled-components](https://github.com/styled-components/styled-components) 和 [emotion](https://github.com/emotion-js/emotion))。这引入了一个新的面向组件的样式范例，它和普通的 CSS 撰写过程是有区别的。另外，虽然在构建时将 CSS 提取到一个单独的样式表是支持的，但 bundle 里通常还是需要一个运行时程序来让这些样式生效。当你能够利用 JavaScript 灵活处理样式的同时，也需要权衡 bundle 的尺寸和运行时的开销。

如果你是一个 CSS-in-JS 的爱好者，许多主流的 CSS-in-JS 库也都支持 Vue (比如 [styled-components-vue](https://github.com/styled-components/vue-styled-components) 和 [vue-emotion](https://github.com/egoist/vue-emotion))。这里 React 和 Vue 主要的区别是，Vue 设置样式的默认方法是[单文件组件](https://cn.vuejs.org/v2/guide/single-file-components.html)里类似 `style` 的标签。

[单文件组件](https://cn.vuejs.org/v2/guide/single-file-components.html)让你可以在同一个文件里完全控制 CSS，将其作为组件代码的一部分。

```
<style scoped>
  @media (min-width: 250px) {
    .list-container:hover {
      background: orange;
    }
  }
</style>
```

这个可选 `scoped` attribute 会自动添加一个唯一的 attribute (比如 `data-v-21e5b78`) 为组件内 CSS 指定作用域，编译的时候 `.list-container:hover` 会被编译成类似 `.list-container[data-v-21e5b78]:hover`。

最后，Vue 的单文件组件里的样式设置是非常灵活的。通过 [vue-loader](https://github.com/vuejs/vue-loader)，你可以使用任意预处理器、后处理器，甚至深度集成 [CSS Modules](https://vue-loader.vuejs.org/en/features/css-modules.html)——全部都在 `` 标签内。

### [规模](https://cn.vuejs.org/v2/guide/comparison.html#规模)

#### 向上扩展

Vue 和 React 都提供了强大的路由来应对大型应用。React 社区在状态管理方面非常有创新精神 (比如 Flux、Redux)，而这些状态管理模式甚至 [Redux 本身](https://yarnpkg.com/en/packages?q=redux vue&p=1)也可以非常容易的集成在 Vue 应用中。实际上，Vue 更进一步地采用了这种模式 ([Vuex](https://github.com/vuejs/vuex))，更加深入集成 Vue 的状态管理解决方案 Vuex 相信能为你带来更好的开发体验。

两者另一个重要差异是，Vue 的路由库和状态管理库都是由官方维护支持且与核心库同步更新的。React 则是选择把这些问题交给社区维护，因此创建了一个更分散的生态系统。但相对的，React 的生态系统相比 Vue 更加繁荣。

最后，Vue 提供了 [CLI 脚手架](https://github.com/vuejs/vue-cli)，能让你通过交互式的脚手架引导非常容易地构建项目。你甚至可以使用它[快速开发组件的原型](https://cli.vuejs.org/zh/guide/prototyping.html#快速原型开发)。React 在这方面也提供了 [create-react-app](https://github.com/facebookincubator/create-react-app)，但是现在还存在一些局限性：

- 它不允许在项目生成时进行任何配置，而 Vue CLI 运行于可升级的运行时依赖之上，该运行时可以通过[插件](https://cli.vuejs.org/zh/guide/plugins-and-presets.html#插件)进行扩展。
- 它只提供一个构建单页面应用的默认选项，而 Vue 提供了各种用途的模板。
- 它不能用用户自建的[预设配置](https://cli.vuejs.org/zh/guide/plugins-and-presets.html#预设配置)构建项目，这对企业环境下预先建立约定是特别有用的。

而要注意的是这些限制是故意设计的，这有它的优势。例如，如果你的项目需求非常简单，你就不需要自定义生成过程。你能把它作为一个依赖来更新。如果阅读更多关于[不同的设计理念](https://github.com/facebookincubator/create-react-app#philosophy)。

#### 向下扩展

React 学习曲线陡峭，在你开始学 React 前，你需要知道 JSX 和 ES2015，因为许多示例用的是这些语法。你需要学习构建系统，虽然你在技术上可以用 Babel 来实时编译代码，但是这并不推荐用于生产环境。

就像 Vue 向上扩展好比 React 一样，Vue 向下扩展后就类似于 jQuery。你只要把如下标签放到页面就可以运行：

<script src="https://cdn.jsdelivr.net/npm/vue"></script>
然后你就可以编写 Vue 代码并应用到生产中，你只要用 min 版 Vue 文件替换掉就不用担心其他的性能问题。

由于起步阶段不需学 JSX，ES2015 以及构建系统，所以开发者只需不到一天的时间阅读[指南](https://cn.vuejs.org/v2/guide/)就可以建立简单的应用程序。

### [原生渲染](https://cn.vuejs.org/v2/guide/comparison.html#原生渲染)

React Native 能使你用相同的组件模型编写有本地渲染能力的 APP (iOS 和 Android)。能同时跨多平台开发，对开发者是非常棒的。相应地，Vue 和 [Weex](https://weex.apache.org/) 会进行官方合作，Weex 是阿里巴巴发起的跨平台用户界面开发框架，同时也正在 Apache 基金会进行项目孵化，Weex 允许你使用 Vue 语法开发不仅仅可以运行在浏览器端，还能被用于开发 iOS 和 Android 上的原生应用的组件。

在现在，Weex 还在积极发展，成熟度也不能和 React Native 相抗衡。但是，Weex 的发展是由世界上最大的电子商务企业的需求在驱动，Vue 团队也会和 Weex 团队积极合作确保为开发者带来良好的开发体验。

另一个选择是 [NativeScript-Vue](https://nativescript-vue.org/)，一个用 Vue.js 构建完全原生应用的 [NativeScript](https://www.nativescript.org/) 插件。

### [MobX](https://cn.vuejs.org/v2/guide/comparison.html#MobX)

Mobx 在 React 社区很流行，实际上在 Vue 也采用了几乎相同的反应系统。在有限程度上，React + Mobx 也可以被认为是更繁琐的 Vue，所以如果你习惯组合使用它们，那么选择 Vue 会更合理。

### [Preact 和其它类 React 库](https://cn.vuejs.org/v2/guide/comparison.html#Preact-和其它类-React-库)

类 React 的库们往往尽可能地与 React 共享 API 和生态。因此上述比较对它们来说也同样适用。它们和 React 的不同往往在于更小的生态。因为这些库无法 100% 兼容 React 生态中的全部，部分工具和辅助库也可能无法使用。或者即使看上去能工作，但也有可能随时发生不兼容，除非你用的这个类 React 库官方与 React 保持严格一致。



-------------------------------------------------------------



## [Angular (原本的 Angular 2)](https://cn.vuejs.org/v2/guide/comparison.html#Angular-原本的-Angular-2)

我们将新的 Angular 独立开来讨论，因为它是一个和 AngularJS 完全不同的框架。例如：它具有优秀的组件系统，并且许多实现已经完全重写，API 也完全改变了。

### [TypeScript](https://cn.vuejs.org/v2/guide/comparison.html#TypeScript)

Angular 事实上必须用 TypeScript 来开发，因为它的文档和学习资源几乎全部是面向 TS 的。TS 有很多好处——静态类型检查在大规模的应用中非常有用，同时对于 Java 和 C# 背景的开发者也是非常提升开发效率的。

然而，并不是所有人都想用 TS——在中小型规模的项目中，引入 TS 可能并不会带来太多明显的优势。在这些情况下，用 Vue 会是更好的选择，因为在不用 TS 的情况下使用 Angular 会很有挑战性。

最后，虽然 Vue 和 TS 的整合可能不如 Angular 那么深入，我们也提供了官方的[类型声明](https://github.com/vuejs/vue/tree/dev/types)和[组件装饰器](https://github.com/vuejs/vue-class-component)，并且知道有大量用户在生产环境中使用 Vue + TS 的组合。我们也和微软的 TS / VSCode 团队进行着积极的合作，目标是为 Vue + TS 用户提供更好的类型检查和 IDE 开发体验。

### [运行时性能](https://cn.vuejs.org/v2/guide/comparison.html#运行时性能-2)

这两个框架都很快，有非常类似的 benchmark 数据。你可以[浏览具体的数据](https://stefankrause.net/js-frameworks-benchmark8/table.html)做更细粒度的对比，不过速度应该不是决定性的因素。

### [体积](https://cn.vuejs.org/v2/guide/comparison.html#体积)

在体积方面，最近的 Angular 版本中在使用了 [AOT](https://en.wikipedia.org/wiki/Ahead-of-time_compilation) 和 [tree-shaking](https://en.wikipedia.org/wiki/Tree_shaking) 技术后使得最终的代码体积减小了许多。但即使如此，一个包含了 Vuex + Vue Router 的 Vue 项目 (gzip 之后 30kB) 相比使用了这些优化的 `angular-cli` 生成的默认项目尺寸 (~65KB) 还是要小得多。

### [灵活性](https://cn.vuejs.org/v2/guide/comparison.html#灵活性)

Vue 相比于 Angular 更加灵活，Vue 官方提供了构建工具来协助你构建项目，但它并不限制你去如何组织你的应用代码。有人可能喜欢有严格的代码组织规范，但也有开发者喜欢更灵活自由的方式。

### [学习曲线](https://cn.vuejs.org/v2/guide/comparison.html#学习曲线)

要学习 Vue，你只需要有良好的 HTML 和 JavaScript 基础。有了这些基本的技能，你就可以非常快速地通过阅读[指南](https://cn.vuejs.org/v2/guide/)投入开发。

Angular 的学习曲线是非常陡峭的——作为一个框架，它的 API 面积比起 Vue 要大得多，你也因此需要理解更多的概念才能开始有效率地工作。当然，Angular 本身的复杂度是因为它的设计目标就是只针对大型的复杂应用；但不可否认的是，这也使得它对于经验不甚丰富的开发者相当的不友好。

# Vue 3.0

## 1. 全新文档`RFCs`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a717a643832346a3331693730753067706d2e6a7067.jpg)](https://camo.githubusercontent.com/247a4865c79fbebdb88edd1d88575e8e2635025a/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a717a643832346a3331693730753067706d2e6a7067)

`Vue.js 3.0 Beta`发布后的工作重点是保证稳定性和推进各类库集成

所有的进度和文档都将在全新`RFCs`文档可以看到。

## 2. 六大亮点

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6435706769696a333168383075303077792e6a7067.jpg)](https://camo.githubusercontent.com/ef00e0de329318ac463aa8c1ec481b02b730b639/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6435706769696a333168383075303077792e6a7067)

- `Performance`：性能更比`Vue 2.0`强。
- `Tree shaking support`：可以将无用模块“剪辑”，仅打包需要的。
- `Composition API`：组合`API`
- `Fragment, Teleport, Suspense`：“碎片”，`Teleport`即`Protal传送门`，“悬念”
- `Better TypeScript support`：更优秀的 Ts 支持
- `Custom Renderer API`：暴露了自定义渲染`API`

下面将按顺序分别描述。

## 3.`Performance`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6436337071346a333135613069676770312e6a7067.jpg)](https://camo.githubusercontent.com/306dba7ff97d0957424357dda076a1d0c9f33d98/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6436337071346a333135613069676770312e6a7067)

1. 重写了虚拟`Dom`的实现（且保证了兼容性，脱离模版的渲染需求旺盛）。
2. 编译模板的优化。
3. 更高效的组件初始化。
4. `update`性能提高 1.3~2 倍。
5. `SSR`速度提高了 2~3 倍。

下面是各项性能对比

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a646170797a776a3331686b30753034647a2e6a7067.jpg)](https://camo.githubusercontent.com/56d7f024b5419b45243b86fa9021201b7532b533/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a646170797a776a3331686b30753034647a2e6a7067)

### 要点 1：编译模板的优化

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c677931676532326276333764686a33316b36306c61646b732e6a7067.jpg)](https://camo.githubusercontent.com/0fcaa76824c79261a14abb1f1d39efaf5fa20e78/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c677931676532326276333764686a33316b36306c61646b732e6a7067)

假设要编译以下代码

```
<div>
  <span/>
  <span>{{ msg }}</span>
</div>
```

将会被编译成以下模样：

```
import { createVNode as _createVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("span", null, "static"),
    _createVNode("span", null, _toDisplayString(_ctx.msg), 1 /* TEXT */)
  ]))
}

// Check the console for the AST
```

注意看第二个`_createVNode`结尾的“1”：

Vue 在运行时会生成`number`（大于 0）值的`PatchFlag`，用作标记。

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765323262797a6377616a33316b383073303131712e6a7067.jpg)](https://camo.githubusercontent.com/850ca2fad6963d540a95105a4b4ce4cfbd5f46bc/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765323262797a6377616a33316b383073303131712e6a7067)仅带有`PatchFlag`标记的节点会被真正追踪，且**无论层级嵌套多深，它的动态节点都直接与`Block`根节点绑定，无需再去遍历静态节点**

再看以下例子：

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c67793167653232633261626b326a33316b67306f346e33302e6a7067.jpg)](https://camo.githubusercontent.com/3d7bbadeb85f634c1787b980682ab8715337a550/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c67793167653232633261626b326a33316b67306f346e33302e6a7067)

```
<div>
  <span>static</span>
  <span :id="hello" class="bar">{{ msg }}   </span>
</div>
```

会被编译成：

```
import { createVNode as _createVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("span", null, "static"),
    _createVNode("span", {
      id: _ctx.hello,
      class: "bar"
    }, _toDisplayString(_ctx.msg), 9 /* TEXT, PROPS */, ["id"])
  ]))
}
PatchFlag` 变成了`9 /* TEXT, PROPS */, ["id"]
```

它会告知我们不光有`TEXT`变化，还有`PROPS`变化（id）

这样既跳出了`virtual dom`性能的瓶颈，又保留了可以手写`render`的灵活性。 等于是：既有`react`的灵活性，又有基于模板的性能保证。

### 要点 2: 事件监听缓存：`cacheHandlers`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c677931676532326336617969636a33316b67306b387137772e6a7067.jpg)](https://camo.githubusercontent.com/81bbb32dc5d9f4e3a484a01e8a794d4cb09c2252/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c677931676532326336617969636a33316b67306b387137772e6a7067)假设我们要绑定一个事件：

```
<div>
  <span @click="onClick">
    {{msg}}
  </span>
</div>
```

关闭`cacheHandlers`后：

```
import { toDisplayString as _toDisplayString, createVNode as _createVNode, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("span", { onClick: _ctx.onClick }, _toDisplayString(_ctx.msg), 9 /* TEXT, PROPS */, ["onClick"])
  ]))
}
```

`onClick`会被视为`PROPS`动态绑定，后续替换点击事件时需要进行更新。

开启`cacheHandlers`后：

```
import { toDisplayString as _toDisplayString, createVNode as _createVNode, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("span", {
      onClick: _cache[1] || (_cache[1] = $event => (_ctx.onClick($event)))
    }, _toDisplayString(_ctx.msg), 1 /* TEXT */)
  ]))
}
```

`cache[1]`，会自动生成并缓存一个内联函数，“神奇”的变为一个静态节点。 Ps：相当于`React`中`useCallback`自动化。

并且支持手写内联函数：

```
<div>
  <span @click="()=>foo()">
    {{msg}}
  </span>
</div>
```

### 补充：`PatchFlags`枚举定义

而通过查询`Ts`枚举定义，我们可以看到分别定义了以下的追踪标记： [![img](Vue.assets/68747470733a2f2f696d676b722e636e2d626a2e7566696c656f732e636f6d2f62663937313434612d656637362d346133362d623735362d3338386166353437396663652e706e67.png)](https://camo.githubusercontent.com/d7f68a97aa291c833cd372f3f3a91553dfe38805/68747470733a2f2f696d676b722e636e2d626a2e7566696c656f732e636f6d2f62663937313434612d656637362d346133362d623735362d3338386166353437396663652e706e67)感兴趣的可以看源码：`packages/shared/src/patchFlags.ts`

## 4. `Tree shaking support`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a7365766767766a3331686530753061666b2e6a7067.jpg)](https://camo.githubusercontent.com/fdae8fdac3104ba492d980c15c13f41e649cfdfd/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a7365766767766a3331686530753061666b2e6a7067)

- 可以将无用模块“剪辑”，仅打包需要的（比如`v-model,`，用不到就不会打包）。

- 一个简单“

  ```
  HelloWorld
  ```

  ”大小仅为：13.5kb

  - 11.75kb，仅`Composition API`。

- 包含运行时完整功能：22.5kb

  - 拥有更多的功能，却比`Vue 2`更迷你。

很多时候，我们并不需要 `vue`提供的所有功能，在 `vue 2` 并没有方式排除掉，但是 3.0 都可能做成了按需引入。

## 5. `Composition API`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6464713365346a333168383075306771732e6a7067.jpg)](https://camo.githubusercontent.com/7a98e8d599d186c472e1a15fcb199992361ac464/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6464713365346a333168383075306771732e6a7067)与`React Hooks` 类似的东西，实现方式不同。

- 可与现有的 `Options API`一起使用
- 灵活的逻辑组合与复用
- `vue 3`的响应式模块可以和其他框架搭配使用

混入(`mixin`) 将不再作为推荐使用， `Composition API`可以实现更灵活且无副作用的复用代码。

感兴趣的可以查看：https://composition-api.vuejs.org/#summary [![img](https://camo.githubusercontent.com/adf3fafcf4023131ac0a6aaa1d5359a41c9e10b8/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a7433733573716a3331706f3075306538312e6a7067)](https://camo.githubusercontent.com/adf3fafcf4023131ac0a6aaa1d5359a41c9e10b8/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a7433733573716a3331706f3075306538312e6a7067)

```
Composition API`包含了六个主要`API
```

可以到这里查看：https://composition-api.vuejs.org/api.html#setup

Ps：其它的均为常见的工具函数，可先忽略不看。

## 6. `Fragment`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6468753132336a333168653075307768782e6a7067.jpg)](https://camo.githubusercontent.com/c9548fa74d7ecf39476b0a554321c6cdb92abde9/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6468753132336a333168653075307768782e6a7067)

`Fragment`翻译为：“碎片”

- 不再限于模板中的单个根节点
- `render` 函数也可以返回数组了，类似实现了 `React.Fragments` 的功能 。
- `Just works`

### 6.1 `<Teleport>`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a646c74316e686a333168623075306770362e6a7067.jpg)](https://camo.githubusercontent.com/7acae66ac7099ff2c5e1bbd33ad2f445092c6f4f/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a646c74316e686a333168623075306770362e6a7067)

- 以前称为` <Portal> `，译作传送门。
- 更多细节将由@Linusborg 分享

` <Teleport> `原先是对标 `React Portal`（增加多个新功能，更强）

**但因为`Chrome`有个提案，会增加一个名为`Portal`的原生`element`，为避免命名冲突，改为`Teleport`**

### 6.2 `<Suspense>`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6470666b6b746a3331686530753030786a2e6a7067.jpg)](https://camo.githubusercontent.com/01041e2674f74694a93e6ce61aab0e6a4de50844/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6470666b6b746a3331686530753030786a2e6a7067)

`Suspense`翻译为：“悬念”

- 可在嵌套层级中等待嵌套的异步依赖项
- 支持`async setup()`
- 支持异步组件

虽然`React 16`引入了`Suspense`，但直至现在都不太能用。如何将其与异步数据结合，还没完整设计出来。

Vue 3 的``更加轻量：

**仅 5%应用能感知运行时的调度差异，综合考虑下，Vue3 的`` 没和 React 一样做运行调度处理**

## 7. 更好的`TypeScript`支持

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6474356533736a3331686530753034366b2e6a7067.jpg)](https://camo.githubusercontent.com/dd8b88d3937fcfaa63303b7cb5ab28320c693667/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6474356533736a3331686530753034366b2e6a7067)

- `Vue 3`是用`TypeScript`编写的库，可以享受到自动的类型定义提示

- ```
  JavaScript
  ```

  和

  ```
  TypeScript
  ```

  中的 API 是相同的。

  - 事实上，代码也基本相同

- 支持`TSX`

- `class`组件还会继续支持，但是需要引入`vue-class-component@next`，该模块目前还处在 alpha 阶段。

还有`Vue 3 + TypeScript` 插件正在开发，有类型检查，自动补全等功能。目前进展可喜。 [![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a64787a7937346a333168633075303130702e6a7067.jpg)](https://camo.githubusercontent.com/c0f1460a6b77264a0426e9eb6c0eba067698fd22/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a64787a7937346a333168633075303130702e6a7067)

## 8. `Custom Renderer API`：自定义渲染器 API

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a647a387236686a333168653075307139662e6a7067.jpg)](https://camo.githubusercontent.com/707176001444a99551ffc689d91e80a3e75d10e5/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a647a387236686a333168653075307139662e6a7067)

- 正在进行`NativeScript Vue`集成
- 用户可以尝试`WebGL`自定义渲染器，与普通 Vue 应用程序一起使用（`Vugel`）。

意味着以后可以通过 `vue`， `Dom` 编程的方式来进行 `webgl` 编程 。感兴趣可以看这里：[Getting started vugel](https://vugel.planning.nl/#application)

## 9. 剩余工作

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6538683366676a333168613075306770762e6a7067.jpg)](https://camo.githubusercontent.com/569617c7e9a7ac4a0e656e1a560dc7c0e08dc065/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6538683366676a333168613075306770762e6a7067)

### 9.1 `Docs & Migration Guides`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6561623071646a3331686d3075303435782e6a7067.jpg)](https://camo.githubusercontent.com/4fe02b4ad47fbadecbd191eb14c8358171c4bdbf/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6561623071646a3331686d3075303435782e6a7067)

- 新的文档编写交由`@NataliaTepluhina, @sdras, @bencodezen & @phanan` 负责
- `@sdras` 正在做自动升级迁移工具
- `@sodatea` 已经开始研究`CodeMods`

### 9.2 `Router`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a65656a6833736a3331686c3075307768762e6a7067.jpg)](https://camo.githubusercontent.com/795607141f166f7c1ab0a1e4cf75604a4c7345e1/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a65656a6833736a3331686c3075307768762e6a7067)

- 下一代 `Router`：`vue-router@next`已在`alpha`阶段，感谢`@posva`

有部分的`API`变动，可到`RFC`上看。

### 9.3 `Vuex`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a656a393164326a333168673075306e33312e6a7067.jpg)](https://camo.githubusercontent.com/e9dda5473d90bdeeac6d6194cfa665af863d9f0c/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a656a393164326a333168673075306e33312e6a7067)`

- 下一代`Vuex`：，`vuex@next`（与`Vue 3 compat`相同的 API），已在`alpha`阶段，感谢`@KiaKing`。
- 团队正在为下一次迭代试验`Vuex API`的简化

目前以兼容`Vue 3`为主，基本上没有`API`变动，莫慌。

### 9.4 `CLI`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a656d6e747a746a33316838307530776a612e6a7067.jpg)](https://camo.githubusercontent.com/7f4aa8ceb614195df52a7f71609f96b7494b3ae8/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a656d6e747a746a33316838307530776a612e6a7067)

- `CLI`插件：`vue-cli-plugin-vue-next`by `@sodatea`
- （wip）`CodeMods`支持升级`Vue 2`应用

### 9.5 新工具：`vite`（法语 “快”）

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6571786476306a333168323075306e31322e6a7067.jpg)](https://camo.githubusercontent.com/0e2b81bac56b84108bfb170c6fd004270bdaeb47/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6571786476306a333168323075306e31322e6a7067)地址：https://github.com/vuejs/vite

一个简易的`http`服务器，无需`webpack`编译打包，根据请求的`Vue`文件，直接发回渲染，且支持热更新（非常快）

### 9.6 `vue-test-utils`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6575376371636a333168673075303078632e6a7067.jpg)](https://camo.githubusercontent.com/de3d254b37cab4281db89b76c841775b3a1d133a/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6575376371636a333168673075303078632e6a7067)

- 下一代  `test-utils`:`test-utils@next` 

  - by `@lmiller1990, @dobromir-hristov, @afontcu & @JessicaSachs`

### 9.7 `DevTools`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a65787a7234786a333168673075306770782e6a7067.jpg)](https://camo.githubusercontent.com/4cec62c7f11f715de08274385455d919d6c1c61c/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a65787a7234786a333168673075306770782e6a7067)

- 早期的原型已经由@Akryum 完成，当我们到`beta`时，将完全集成。

目前需要花更多精力去完善。

### 9.8 `IDE Support (Vetur)`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a66316f776f716a3331686d307530776a7a2e6a7067.jpg)](https://camo.githubusercontent.com/a8a9cca7be8132cb14fd977f5b148097e028b177/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a66316f776f716a3331686d307530776a7a2e6a7067)

- `@znck`目前正在试验模板的类型检查
- `@octref`将在 5 月为`Vue 3`进行`Vetur`集成

### 9.9 `Nuxt`

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6633303273686a33316861307530646b632e6a7067.jpg)](https://camo.githubusercontent.com/aacc0d82436fb2ab50d9e82b93c067ddf1f6e3df/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6633303273686a33316861307530646b632e6a7067)

目前`Nuxt`的整合工作也正在进行中，内部团队已经跑起来了。还需要时间磨合

## 10 `Vue 2.x`还有 2.7 版本

[![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6636756a326f6a3331686f3075306167632e6a7067.jpg)](https://camo.githubusercontent.com/1e3f8d7fdb1e8783de89b7f52faa6c5e01994aba/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a6636756a326f6a3331686f3075306167632e6a7067)

- 将有最后一个小版本（2.7）
- 从`Vue 3`向后移植兼容的改进(不损坏兼容性前提下)
- 加上在`Vue 3`中删除的功能的弃用警告
- `LTS1` 18 个月。

最后建议：`Vue 3`虽好，如果你的项目很稳定，且对新功能无过多的要求或者迁移成本过高，则不建议升级。

## 结束

**直播中用到的渲染模板查看工具地址**：https://vue-next-template-explorer.netlify.app/ [![img](Vue.assets/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a696c696268756a33316b67306f633078692e6a7067.jpg)](https://camo.githubusercontent.com/e471c2d01fa16da05adf4c41eede27471146e372/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f30303753385a496c6779316765317a696c696268756a33316b67306f633078692e6a7067)



# 知识点

1. Vue的核心：Object.defineProperty 方法

2. 虚拟DOM只是一个对象，它有tag表明是什么标签，data表明一个数据对象有什么属性，children列表表示还有子节点。虚拟DOM本质上是轻量的JS数据，格式来表示真实DOM在特定时间的外观，每次更新时生成虚拟DOM的副本。虚拟DOM并不会让框架更快，只是解决原始DOM的局限性的一个比较好的方法，以声明方式构成你想要的DOM的结构。另一个好处是把渲染逻辑从真实DOM中分离出来，先计算差异，然后将差异应用到DOM上，除了最后的渲染，不需要接触DOM。

   实际上，如果抽象出这些最终的连接点，操作DOM的API，然后把它们应用到其他地方，就可以应用到任何支持虚拟环境的JS应用，但不一定是DOM，可以是原生渲染引擎，例如iOS或Android，或者在服务器端，可以将虚拟DOM转换为字符串或字符串查找器，让虚拟DOM进行原生渲染，使得React Native，Weeks，Native script这样的项目成为可能，他们的架构都是类似的JS应用实际上是嵌入到JS引擎中运行，它只发送必要的消息，例如差异更改，或任何实际的节点操作发送到渲染器。

   渲染函数只是返回虚拟DOM的一个函数

   template在Vue是通过渲染函数编译的

3. 每个组件都有一个渲染函数返回虚拟DOM，每个组件都有一个负责监视的观察者收集、清理依赖项并通知所有内容。只要依赖的渲染属性发生变化，就会调用渲染。每个组件都只负责自己的依赖在这个组件树中的渲染。

4. JSX和模板 都是声明DOM的和状态的关系。模板只是一种更静态、更受约束的表达形式，模板语法相近，可以快速进行迁移，进行编译优化；而JSX或渲染函数更加动态，更具动态性可以更自由地使用编程语言。

5. 普通组件和函数组件，普通组件支持实例化，而函数组件不支持。函数组件接受参数然后返回虚拟DOM，它不拥有任何状态，函数组件的应用场景是你需要优化一个有很多节点的组件。

6. 异步代码最好和mutation代码分开

7. 本质上，在VueX中，action和mutation的概念实际上只是把异步代码和状态更改代码分开。mutation只关注状态的变更。