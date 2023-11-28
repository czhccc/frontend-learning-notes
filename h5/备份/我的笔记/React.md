

# react

## 核心概念

### JSX

全称 `JavaScript XML` ，react定义的一种类似于XML的JS扩展语法: XML+JS，用来创建react虚拟DOM(元素)对象，不是字符串, 也不是HTML/XML标签，最终产生的就是一个JS对象。

```javascript
<script type="text/babel"> // 必须使用 babel
	const ele1 = ( // 可以包含很多子元素
    <div>
      <h1>Hello!</h1>
      <h2>Good to see you here.</h2>
    </div>
  ) // 不需要加上引号，否则就是字符串了
	const ele2 = <img src={user.avatarUrl} />; // 假如标签里面没有内容，可以使用 /> 来闭合标签
	const ele3 = <div tabIndex="0"></div> // 使用引号，来将属性值指定为字符串字面量
	const ele4 = <img src={user.avatarUrl}></img> // 使用大括号，插入一个 JS 表达式
  ...
</script>
```

遇到 `<` 开头的代码，以标签的语法解析: html同名标签转换为html同名元素，其它标签需要特别解析。因此，必须要写 `</..>` 结束标签。对于像img这种自结束标签，也必须<img />加上 /

遇到以 `{` 开头的代码，以JS语法解析: 标签中的js代码必须用 `{ }` 包含

JSX 语法上更接近 JavaScript 而不是 HTML，所以 React DOM 使用 `camelCase`（驼峰命名）来定义属性的名称，而不使用 HTML 属性名称的命名约定。 

浏览器不能直接解析JSX代码，需要babel转译为纯JS的代码才能运行，只要用了JSX，都要加上 `type="text/babel"` ，声明需要babel来处理。Babel 会把 JSX 转译成一个名为 `React.createElement()` 函数调用。 

```javascript
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
// 等价于
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);

// React.createElement() 会预先执行一些检查，以帮助你编写无错代码，实际上它创建了一个这样的对象：
// 注意：这是简化过的结构
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```

#### render() 渲染虚拟DOM

语法：`ReactDOM.render(virtualDOM, containerDOM)`，将虚拟DOM元素渲染到真实容器DOM中显示。 React DOM 会将元素和它的子元素与它们之前的状态进行比较，并只会进行必要的更新来使 DOM 达到预期的状态。 

​	参数一：纯js或jsx创建的虚拟dom对象

​	参数二：用来包含虚拟DOM元素的真实dom元素对象(一般是一个div)

**render()函数渲染组件标签**

1. React内部创建组件实例对象
2. 得到虚拟DOM，并解析为真实DOM
3. 插入到指定的元素内部

```javascript
<script type="text/javascript">
  const msg = 'I Like You'
	const myId = 'Atguigu'
  
  /* 方式一：纯JS */
  // 1.创建虚拟DOM方式一：纯JS（本质）
  const vDom1 = React.createElement('h2',{id: myId.toLowerCase()}},msg.tuUpperCase())
  // 2.渲染虚拟DOM
  ReactDOM.render(vDom1, document.getElementById('test1'))
</script>

<script type="text/babel"> // 用babel
  /* 方式二：JSX */
  // 1.创建虚拟DOM方式二：JSX（方式一的语法糖）
  const vDom2 = <h1 id={myId.toUpperCase()}>{msg.toLowerCase()}</h1> 
  // 2.渲染虚拟DOM
  ReactDOM.render(vDom2, document.getElementById('test2'))
</script>
```

为什么要采用虚拟DOM，而不是直接修改真实的DOM呢？（最主要的3个原因）

1. 很难跟踪状态发生的改变：原有的开发模式，我们很难跟踪到状态发生的改变，不方便针对我们应用程序进行调试。
2. 操作真实DOM性能较低：传统的开发模式会进行频繁的DOM操作，而这一的做法的性能非常低。真实DOM是一个非常重（复杂）的对象，而虚拟DOM对象则比较轻（简单）。其次，真实DOM操作会引起浏览器的回流和重绘，非常消耗浏览器的性能。
3. 从命令式编程变为声明式编程。

##### render函数的返回值

当 render 被调用时，它会检查 this.props 和 this.state 的变化并（只能）返回以下类型之一： 

- **React 元素**：
  - 通常通过 JSX 创建。
  - 例如，<div /> 会被 React 渲染为 DOM 节点，<MyComponent /> 会被 React 渲染为自定义组件。
  - 无论是 <div /> 还是 <MyComponent /> 均为 React 元素。
- **数组或 fragments**：使得 render 方法可以返回多个元素。
- **Portals**：可以渲染子节点到不同的 DOM 子树中。
- **字符串或数值类型**：它们在 DOM 中会被渲染为文本节点
- **布尔类型或 null**：什么都不渲染

注意，`render()` 渲染函数在渲染时，在 `{ }` 中会不显示/忽略一些：

```javascript
// 会忽略 null、undefined、Boolean、对象 ，原因很简单，用于控制元素的渲染，例如使用三目运算符，结果为false时不要渲染相应的元素
  	{
      ...
      this.state{
        test1: null, //null
        test2: undefined, // undefined
        test3: true, // Boolean
        test4: { 
          name: 'xxx',
          age: 'yyy'
        }
      }
  	}
    render() { 
      return 
        <div>
          <h2>{this.state.test1}</h2> // 不显示
          <h2>{this.state.test2}</h2> // 不显示
          <h2>{this.state.test3}</h2> // 不显示
      		<h2>{this.state.test4}</h2> // 报错，不能直接展示对象
        </div>
}

// 如果我们确实需要其显示，可以用如下方法：
// 1. toString()
<h2>{this.state.tuest3.toString}</h2>
      
// 2. 如果没有toString()方法，可以使用 String()
test1: String(null)

// 3. 加上空字符串即可
<h2>{this.state.test2 + ""}</h2>
```

#### 深入JSX

##### JSX的本质

实际上，JSX 仅仅只是 `React.createElement(component, props, ...children)` 函数的语法糖，所以JSX才需要babel进行转换，将JSX转换为React.createElement()。

```javascript
<MyButton color="blue" shadowSize={2}>Click Me</MyButton>
// 会编译为：
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
```

如果你想测试一些特定的 JSX 会转换成什么样的 JavaScript，你可以尝试使用 [在线的 Babel 编译器](https://babeljs.io/repl/#?presets=react&code_lz=GYVwdgxgLglg9mABACwKYBt1wBQEpEDeAUIogE6pQhlIA8AJjAG4B8AEhlogO5xnr0AhLQD0jVgG4iAXyJA)。

1. 使用JSX语法糖编写代码。
2. 通过babel将JSX转换为`React.createElement()`函数。
3. `React.createElement()`函数本质上是创建一个`ReactElement`对象（虚拟DOM对象树）。
4. `ReactElement`对象（虚拟DOM对象树）通过`render`函数渲染/映射到某一元素上，变为真实DOM元素【这一过程称为“协调”】。

##### 指定 React 元素类型

JSX 标签的第一部分指定了 React 元素的类型。

大写字母开头的 JSX 标签意味着它们是 React 组件。这些标签会被编译为对命名变量的直接引用，所以，当你使用 JSX ` <Foo /> ` 表达式时，`Foo` 必须包含在作用域内。

**React 必须在作用域内**

由于 JSX 会编译为 `React.createElement` 调用形式，所以 `React` 库也必须包含在 JSX 代码作用域内。例如，在如下代码中，虽然 `React` 和 `CustomButton` 并没有被直接使用，但还是需要导入：

```javascript
import React from 'react';
import CustomButton from './CustomButton';
function WarningButton() {
  // return React.createElement(CustomButton, {color: 'red'}, null);  
  return <CustomButton color="red" />;
}
```

如果你不使用 JavaScript 打包工具而是直接通过 `  <script>  ` 标签加载 React，则必须将 `React` 挂载到全局变量中。

**在 JSX 类型中使用点语法**

```javascript
const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>{props.color}</div>;
  }
}

function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue" /> // 点语法
}
```

**用户定义的组件必须以大写字母开头**

以小写字母开头的元素代表一个 HTML 内置组件，比如 `` 或者 `` 会生成相应的字符串 `'div'` 或者 `'span'` 传递给 `React.createElement`（作为参数）。大写字母开头的元素则对应着在 JavaScript 引入或自定义的组件，如 ` <Foo /> ` 会编译为 `React.createElement(Foo)`。

使用大写字母开头命名自定义组件。如果确实需要一个以小写字母开头的组件，则在 JSX 中使用它之前，必须将它赋值给一个大写字母开头的变量。否则，会将小写的组件名解析为并不存在的HTML标签，从而导致错误。

**在运行时选择类型**

不能将通用表达式作为 React 元素类型。如果你想通过通用表达式来（动态）决定元素类型，你需要首先将它赋值给大写字母开头的变量。 

```javascript
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // 错误！JSX 类型不能是一个表达式。
  return <components[props.storyType] story={props.story} />;
  
  // 正确！JSX 类型可以是大写字母开头的变量。
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
```

##### JSX 中的 Props

有多种方式可以在 JSX 中指定 props。

**JS 表达式作为 Props**

可以把包裹在 `{}` 中的 JavaScript 表达式作为一个 prop 传递给 JSX 元素。`if` 语句及 `for` 循环不是 JS 表达式，所以不能在 JSX 中直接使用，但是，你可以用在 JSX 以外的代码中，并定义一个变量，再作为 `props` 传入。 

```javascript
<MyComponent foo={1 + 2 + 3 + 4} />
```

**字符串字面量**

可以将字符串字面量赋值给 prop。如下两个 JSX 表达式是等价的：

```javascript
<MyComponent message="hello world" />
<MyComponent message={'hello world'} />
```

当你将字符串字面量赋值给 prop 时，它的值是未转义的。所以，以下两个 JSX 表达式是等价的：

```javascript
<MyComponent message="&lt;3" />
<MyComponent message={'<3'} />
```

这种行为通常是不重要的，这里只是提醒有这个用法。

**Props 默认值为 “True”**

如果你没给 prop 赋值，它的默认值是 `true`。以下两个 JSX 表达式是等价的：

```javascript
<MyTextBox autocomplete />
<MyTextBox autocomplete={true} />
```

通常，我们不建议不传递 value 给 prop，因为这可能与 ES6对象简写 混淆。

**属性展开**

 可以使用展开运算符 `...` 来在 JSX 中传递整个 props 对象。以下两个组件是等价的： 

```javascript
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
```

还可以选择只保留当前组件需要接收的 props，并使用 `...` 将其他 props 传递下去。

```javascript
const Button = props => {
  // kind 会被保留，不会被传递给 <button> 元素。所有其他的 props 会通过 ...other 对象传递
  const { kind, ...other } = props;  
  const className = kind === "primary" ? "PrimaryButton" : "SecondaryButton";
  return <button className={className} {...other} />;
};

const App = () => {
  return (
    <div>
      <Button kind="primary" onClick={() => console.log("clicked!")}>
        Hello World!
      </Button>
    </div>
  );
};
```

##### JSX 中的 子元素

包含在开始和结束标签之间的 JSX 表达式内容将作为特定属性 `props.children` 传递给外层组件。有几种不同的方法来传递子元素：

**字符串字面量**

```javascript
<MyComponent>Hello world!</MyComponent> // 此时 props.children 就只是 "Hello world!"
```

**JSX 子元素**

React 16 开始，可以返回数组：

```javascript
render() {
  // 不需要用额外的元素包裹列表元素！
  return [
    // 不要忘记设置 key
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
  ];
}
```

**JS表达式作为子元素**

JavaScript 表达式可以被包裹在 `{}` 中作为子元素。

```javascript
// 以下表达式是等价的：
<MyComponent>foo</MyComponent>
<MyComponent>{'foo'}</MyComponent>

// 这对于展示任意长度的列表非常有用。例如，渲染 HTML 列表：
function Item(props) {
  return <li>{props.message}</li>;
}

function TodoList() {
  const todos = ['finish doc', 'submit pr', 'nag dan to review'];
  return (
    <ul>
      {todos.map(message => <Item key={message} message={message} />)}
		</ul>
  );
}

// JS表达式也可以和其他类型的子元素组合。这种做法可以方便地替代模板字符串：
function Hello(props) {
  return <div>Hello {props.addressee}!</div>;
}
```

**函数作为子元素**

`props.children` 和其他 prop 一样，它可以传递任意类型的数据，而不仅仅是 React 已知的可渲染类型。例如，如果你有一个自定义组件，你可以把回调函数作为 `props.children` 进行传递： 

```javascript
// 调用子元素回调 numTimes 次，来重复生成组件
function Repeat(props) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {    
    items.push(props.children(i));
  }
  return <div>{items}</div>;
}

function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {index => <div key={index}>This is item {index} in the list</div>}    				     </Repeat>
  );
}
```

你可以将任何东西作为子元素传递给自定义组件，只要确保在该组件渲染之前能够被转换成 React 理解的对象。这种用法并不常见，但可以用于扩展 JSX。

**布尔类型、Null 以及 Undefined 将会忽略**

`false`, `null`, `undefined`, `true` 是合法的子元素,但它们并不会被渲染。以下的 JSX 表达式渲染结果相同：

```javascript
<div />
<div></div>
<div>{false}</div>
<div>{null}</div>
<div>{undefined}</div>
<div>{true}</div>
```

这有助于依据特定条件来渲染其他的 React 元素。例如，在以下 JSX 中，仅当 `showHeader` 为 `true` 时，才会渲染 ` <Header /> ` 组件：

```javascript
<div>
  {showHeader && <Header />}  
	<Content />
</div>
```

值得注意的是有一些 [“falsy” 值](https://developer.mozilla.org/en-US/docs/Glossary/Falsy)，如数字 `0`，仍然会被 React 渲染。例如，以下代码并不会像你预期那样工作，因为当 `props.messages` 是空数组时，`0` 仍然会被渲染：

```javascript
<div>
  {props.messages.length && <MessageList messages={props.messages} />}
</div>

// 要解决这个问题，确保 `&&` 之前的表达式总是布尔值：
<div>
  {props.messages.length > 0 && <MessageList messages={props.messages} />}
</div>

// 反之，如果你想渲染 `false`、`true`、`null`、`undefined` 等值，你需要先将它们转换为字符串
<div>
  My JavaScript variable is {String(myVariable)}
</div>
```

### 组件化

1. 组件名必须首字母大写。React 会将以小写字母开头的组件视为原生 DOM 标签。 
2. 虚拟DOM元素只能有一个根元素
3. 虚拟DOM元素必须有结束标签，像img标签也必须写为<img />

#### 3类组件

##### 函数组件

- 组件名必须大写开头
- 没有this
- 没有生命周期，也会被更新并挂载，但是没有生命周期函数
- 没有内部的状态state（函数中定义的变量是临时变量）

```javascript
// 接收唯一带有数据的 “props” 并返回一个 React 元素。本质上就是JS函数。
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

##### 类组件

- 组件名必须大写开头
- 需要继承自React.Component
- 必须实现render函数
- constructor是可选的
- this.state中维护的是组件内部的数据

```javascript
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

##### 组合（嵌套）组件

组件可以在其输出中引用其他组件。 

```javascript
// 例如，我们可以创建一个可以多次渲染 Welcome 组件的 App 组件：
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(<App />, document.getElementById('root'));
```

#### 3大属性

##### props

当 React 元素为用户自定义组件时，它会将 JSX 所接收的属性（attributes）以及子组件（children）转换为单个对象传递给组件，这个对象被称之为 `props` 。 

1. 每个组件对象都会有props（properties的简写）属性，任何东西都可以通过props进行传递。
2. 组件标签的所有属性都保存在props中，通过 `props.xxx` 来获取。
3. 通过标签属性从组件外向组件内传递变化的数据。
4. 只读性：组件内部不要修改props数据。
5. 向下流动（单向数据流）：组件会在其 props 中接收参数，但是组件本身无法知道它是来自于props、还是state，或是手动输入的。

```javascript
// 传递属性
const ele = <Welcome name="Sara" />;

// 获取传入的属性
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

// 对props中的属性值进行类型限制和必要性限制
/* 详细见 高级指引 - PropTypes 类型检查 */
Person.propTypes = {
	name: React.PropTypes.string.isRequired,
	age: React.PropTypes.number.isRequired
}

// 扩展属性: 将对象的所有属性通过props传递
<Person {...person}/>

// 默认属性值
Person.defaultProps = {
	name: 'Mary'
}

// 组件类的构造函数
constructor (props) {
	super(props)
	console.log(props) // 查看所有属性
}
```

##### state & setState()

```javascript
// 初始化状态
constructor (props) {
  super(props)
  this.state = {
    stateProp1 : value1,
    stateProp2 : value2
  }
}

// 读取某个状态值
this.state.stateProp1

// 更新状态 ----> 组件界面更新
this.setState({
  stateProp1 : value1,
  stateProp2 : value2
})
```

1. 通过问自己以下三个问题，你可以逐个检查相应数据是否属于 state：

   - 该数据是否是由父组件通过 props 传递而来的？如果是，那它应该不是 state。
   - 该数据是否随时间的推移而保持不变？如果是，那它应该也不是 state。
   - 你能否根据其他 state 或 props 计算出该数据的值？如果是，那它也不是 state。

2. state的存放位置：

   - 找到根据这个 state 进行渲染的所有组件。
   - 找到他们的共同所有者（common owner）组件（在组件层级上高于所有需要该 state 的组件）。
   - 该共同所有者组件或者比它层级更高的组件应该拥有该 state。
   - 如果你找不到一个合适的位置来存放该 state，就可以直接创建一个新的组件来存放该 state，并将这一新组件置于高于共同所有者组件层级的位置。

3. state是组件对象最重要的属性，值是对象(可以包含多个数据)，state 是私有的，并且完全受控于当前组件。 

4. 组件被称为"状态机"，通过更新组件的state来更新对应的页面显示(重新渲染组件)。

5. 不要直接修改state，而是用 `setState()` 。

   ```javascript
   this.state.comment = 'Hello'; // 错误
   this.setState({comment: 'Hello'}); // 正确，用setState()
   ```

   `setState`方法与包含在其中的执行是一个很复杂的过程，这段程式码从React最初的版本到现在，也有无数次的修改。它的工作除了要更动`this.state`之外，还要负责触发重新渲染(render)，这里面要经过React核心中diff演算法，最终才能决定是否要进行重渲染，以及如何渲染。而且为了批次与效能的理由，多个`setState`呼叫有可能在执行过程中还需要被合并，所以它被设计以异步的或延时的来进行执行是相当合理的。 

6. `setState()` 可能会引发不必要的渲染。

   `state`本身的设计是无法直接更改，`setState`的设计是用来更动`state`值，也会触发重新渲染(re-render)，按照逻辑就是反正不管如何，只要开发者呼叫`setState`，React就去作整个视图的重新渲染就是。所以`setState`必定会作重新渲染的执行，只是要如何渲染是由React来决定。

   重新渲染(re-render)指的主要是页面上视图(View)的重新再呈现，这是React原本的核心设计，但这个设计是有一些问题的。最主要的是state(状态)并不一定单纯只用来记录与视图(View)有关的状态，也有可能是某个内部控制用的属性值，或是只套用在内部使用的资料。当你改变了这些与视图无关的state(状态)值，以现在的React设计来说，照样要触发重新渲染的执行过程，这在某些复杂的应用时，由于造成不必要的渲染，也有可能造成效能上的问题。

7. `setState()` 无法完全掌控应用中所有组件的状态。

   `state`(状态)是独立于每个组件内部的，而且它是个不能直接更动的对象，这个设计当然是为了要保持组件的封装与独立性，但所以如果当要开发一个复杂的应用时，必定需要使用那些能掌控所有组件资料，以及能提供各组件间资料互动的函式库，例如Flux, Redux或MobX等等。

   React组件目前只能透过各种生命周期的方法，与外部资源、计时器或DOM事件来进行挂勾(Hook)，这些都无法直接使用`setState`方法来进行管理，因此`setState`并没有办法完全掌控一个应用中所有组件的状态，它比较像是每个组件中的都有的一种接口方法，单纯要依靠`setState`方法来管控整个React应用，完全是不足够的。

8. `setState()`的合并（「Batch Update」批量更新）。 不管setState()被调用多少次，都会在函数执行结束以后，被合并为一次并重新渲染，以优化性能,。当你调用 `setState()` 的时候，React 会把你提供的对象合并到当前的 state。所以当State是一个多键值的结构时，可以单独更新其中的一个，此时会进行“差分”更新，不会影响其他的属性值。 

   ```javascript
   // 例如，你的 state 包含几个独立的变量：
   constructor(props) {
     super(props);
     this.state = {
       posts: [],
       comments: []
     };
   }
   //然后你可以分别调用 setState() 来单独地更新它们：
   componentDidMount() {
     fetchPosts().then(response => {
       this.setState({ posts: response.posts }); 
       // 这里的合并是浅合并，完整保留 this.state.posts，但是完全替换了 this.state.comments。
     });
     fetchComments().then(response => {
       this.setState({ comments: response.comments }); 
       // 这里的合并是浅合并，完整保留 this.state.comments，但是完全替换了 this.state.posts。
     });
   }
   ```

   所以，类似于下面的代码，可能不会有我们想要的效果：

   ```javascript
   //假设现在this.state.value = 0
   function eventHandler(){
       this.setState({value:this.state.value + 1});
       this.setState({value:this.state.value + 1});
       this.setState({value:this.state.value + 1});
   }
   //最后this.state.value仍然会是1，不是3
   
   // 上面的操作会导致三次组件渲染，而实际运行上面的代码可以发现组件只渲染了一次。
   ```

   所以不能依赖this.state来计算未来状态。如果想实现这样的效果，应该传一个函数给setState。这个函数有两个参数，第一个为`previous state`，第二个为`props` ：

   `this.setState((state, props) => ({ ... }));`

   这里的例子和props无关，只需要第一个参数，所以要达到效果，代码是这样：

   ```javascript
   // 假设 this.state = { value: 0 }
   function eventHandler(){
     
       this.setState((state) => ({ value: state.value + 1}));
       this.setState((state) => ({ value: state.value + 1}));
       this.setState((state) => ({ value: state.value + 1}));
   }
   // 现在this.state.value是3
   ```

9. `setState()` 可能是异步的（准确地说：不是保证同步的）。脱离了React的控制，React不知道如何进行Batch Update，setState()就会是同步的，例如 `setTimeout` 的回调函数。 

   不保证`this.state`会立即更新，所以在调用这个方法后存取`this.state`可能会回传旧的值。

   不保证呼叫`setState`就会同步地执行，而它们也可能最终被被批量调用(多次呼叫的情况下)。你可以提供额外的回调(callback)，回调(callback)将会在`setState`实际被完成时被执行。

   将setState的改变的效果进入队列，并在某个特定的时间节点合并，一起刷新state是一个很好的机制。

   因为 `this.props` 和 `this.state` 可能会异步更新，所以你不要依赖他们的值来更新下一个状态。

   ```javascript
   /* 
   	setState()方法同步re-render在很多情况下是很差的选	择，例如，如果我们给父节点和子节点两个组件，都绑定了点击事件，当他们要触发setState的时候，我们并不希望事件的冒泡，导致父节点被re-render两次。
   	因此，setState()方法并不是同步re-render的，无法同步re-render就无法同步刷新this.props，这就导致当setState同步改变父节点state的时候，父节点传递给子节点的props是不能同步刷新的，就会导致有时候我们获取到的props并不是我们想要的值，例如下面的例子，此代码可能会无法更新计数器：
   */
   this.setState({
     counter: this.state.counter + this.props.increment,
   });
   ```

   要解决这个问题，可以让 `setState()` 接收一个函数而不是一个对象。这个函数用上一个 `state` 作为第一个参数，将更新后的 `props` 作为第二个参数：

   ```javascript
   this.setState((state, props) => ({
     counter: state.counter + props.increment
   }));
   
   // 当更新state，然后想打印state的时候，应该使用回调
   this.setState({key: val}, () = console.log(this.state));    
   ```

    ![img](React.assets/1705225-e1af7dba79b23bd2.webp) 

##### refs 与 事件处理

**refs**

1. 组件内的标签都可以定义ref属性来标识自己

   ​	<input type="text" ref={input => this.msgInput = input}/>

​			回调函数在组件初始化渲染完或卸载时自动调用

2. 在组件中可以通过this.msgInput来得到对应的真实DOM元素

3. 作用: 通过ref获取组件内容特定标签对象, 进行读取其相关数据

**事件处理**

1. 通过onXxx属性指定组件的事件处理函数(注意大小写)

   ​	React使用的是自定义(合成)事件，而不是使用的原生DOM事件

   ​	React中的事件是通过事件委托方式处理的(委托给组件最外层的元素)

2. 通过event.target得到发生事件的DOM元素对象

```javascript
<input onFocus={this.handleClick}/>
handleFocus(event) {
	event.target // 返回input对象
}
```

**注意**

1. 组件内置的方法中的 this 为 组件对象

2. 在组件类中自定义的方法中 this 为 null，解决方法：

   ​	a. 强制绑定this：通过函数对象的bind()

   ​	b. 使用箭头函数

### 生命周期

#### v16.4+

在新的生命周期中去掉了三个过时的：

- `componentWillMount()`
- `componentWillReceiveProps()`
- `componentWillUpdate()`

而添加了两个新的：

- `getDerivedStateFromProps()`
- `getSnapshotBeforeUpdate()`

图片来源：https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/

![1590482241082](React.assets/1590482241082.png)

![1590487378678](React.assets/1590487378678.png)

#### 挂载时的生命周期

##### constructor()

constructor中一般只做两件事情：

	1. 通过给 this.state 赋值对象来初始化内部的state
 2. 为事件绑定this

##### getDerivedStateFromProps()

强制开发者在render之前只做无副作用的操作，能做的操作局限在根据props和state决定新的state。被废弃的所有生命周期函数才能实现的功能，都可以通过 `getDerivedStateFromProps` 来实现。 

无论是 Mounting 还是 Updating，也无论是因为什么引起的 Updating，全部都会被调用。

返回一个对象来更新状态，或者返回 null 来不更新任何内容。 

其是一个静态函数（ 简单说，就是应该一个纯函数），所以函数体内不能访问 this。

```javascript
static getDerivedStateFromProps(nextProps, prevState) { // 一般需要 static
  // 根据 nextProps 和 prevState 计算出预期的状态改变，返回结果会被送给setState
}
```

注意：

① getDerivedStateFromProps前面要加上`static`保留字，声明为静态方法，不然会被react忽略掉。

![img](React.assets/5287253-fc5cf32ae64fbefe.webp) 

② getDerivedStateFromProps里面的 `this `为 `undefined`。

static静态方法只能Class(构造函数)来调用(App.staticMethod✅)，而实例是不能的( (new App()).staticMethod ❌ )。当调用React Class组件时，该组件会实例化；

所以，React Class组件中，静态方法getDerivedStateFromProps无权访问Class实例的this，即this为undefined。

使用时机：

​	派生状态会导致代码冗余，并使组件难以维护。 [确保你已熟悉这些简单的替代方案](https://links.jianshu.com/go?to=https%3A%2F%2Fzh-hans.reactjs.org%2Fblog%2F2018%2F06%2F07%2Fyou-probably-dont-need-derived-state.html)

- 如果你需要执行副作用（例如，数据提取或动画）以响应 props 中的更改，请改用 componentDidUpdate。
- 如果只想在 prop 更改时重新计算某些数据，请使用 memoization helper 代替。
- 如果你想在 prop 更改时“重置”某些 state，请考虑使组件完全受控或使用 key 使组件完全不受控 代替。

##### render()

根据组件的`props`和`state`（两者的重传递、重赋值，无论值是否有变化，都可以引起组件重新render） ，return 一个React元素（描述组件，即UI），不负责组件实际渲染工作，之后由React自身根据此元素去渲染出页面DOM。render是纯函数，不能在里面执行`this.setState`，会有改变组件状态的副作用。

##### componentDidMount()

组件挂载到DOM后调用，且只会被调用一次 

可以进行的操作：

- 依赖于DOM的操作（一般在React中不会直接操作DOM）
- 发送网络请求合适的地方（官方推荐）
- 添加一些订阅（会在componentWillUnmount取消订阅）

#### 更新时的生命周期

##### 引起组件更新的原因

造成组件更新有两类（三种）情况：`new props`、`setState`、`forceupdate`

1. 父组件重新render，父组件重新render引起子组件重新render的情况有两种：

   1. 1  直接使用，每当父组件重新render导致的重传props，子组件将直接跟着重新渲染，无论props是否有变 化。可通过shouldComponentUpdate方法优化。 

   ```javascript
   class Child extends Component {
      shouldComponentUpdate(nextProps) { 
        // 应该使用这个方法，否则无论props是否有变化都将会导致组件跟着重新渲染
        if(nextProps.someThings === this.props.someThings){
        	 return false
        }
      }
      render() {
        return <div>{this.props.someThings}</div>
      }
   }
   ```

    1.2  在componentWillReceiveProps方法中，将props转换成自己的state。

   ```javascript
   class Child extends Component {
       constructor(props) {
           super(props);
           this.state = {
               someThings: props.someThings
           };
       }
       componentWillReceiveProps(nextProps) { // 父组件重传props时就会调用这个方法
           this.setState({someThings: nextProps.someThings});
       }
       render() {
           return <div>{this.state.someThings}</div>
       }
   }
   ```

    在该函数(componentWillReceiveProps)中调用 this.setState() 将不会引起第二次渲染。因为componentWillReceiveProps中判断props是否变化了，若变化了，this.setState将引起state变化，从而引起render，此时就没必要再做第二次因重传props引起的render了，不然重复做一样的渲染了。

2. 组件本身调用setState，无论state有没有变化。可通过shouldComponentUpdate方法优化。 

   ```javascript
   class Child extends Component {
      constructor(props) {
           super(props);
           this.state = {
             someThings:1
           }
      }
      shouldComponentUpdate(nextStates){ // 应该使用这个方法，否则无论state是否有变化都将会导致组件重新渲染
           if(nextStates.someThings === this.state.someThings){
             return false
           }
       }
   
      handleClick = () => { // 虽然调用了setState ，但state并无变化
           const preSomeThings = this.state.someThings
            this.setState({
               someThings: preSomeThings
            })
      }
   
       render() {
           return <div onClick = {this.handleClick}>{this.state.someThings}</div>
       }
   }
   ```

##### getDerivedStateFromProps()

见上。

##### shouldComponentUpdate()

##### componentWillUpdate()

##### render()

见上。

##### getSnapshotBeforeUpdate()

这个函数应该大部分开发者都用不上（或者说，不要用）。

可用于在更新之前保存一些数据，一般return一个对象，

DOM元素还没有被更新， 可以读取DOM但无法操作DOM，只能去获取DOM信息（例如滚动位置），此生命周期返回的任何值都将作为参数 `snapshot`，这个 `snapshot` 会作为 `componentDidUpdate `的第三个参数传入。 

`snapshot `可以是任何值，到底怎么用完全看开发者自己，`getSnapshotBeforeUpdate` 把 `snapshot` 返回，然后DOM改变，`snapshot `传递给 `componentDidUpdate`。 

```javascript
// 官方例子
class ScrollingList extends React.Component {
  constructor(props) {
    super(props);
    this.listRef = React.createRef();
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // 我们是否要添加新的 items 到列表?
    // 捕捉滚动位置，以便我们可以稍后调整滚动.
    if (prevProps.list.length < this.props.list.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // 如果我们有snapshot值, 我们已经添加了新的items.
    // 调整滚动以至于这些新的items 不会将旧items推出视图。
    // (这边的snapshot是 getSnapshotBeforeUpdate方法的返回值)
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>{/* ...contents... */}</div>
    );
  }
}
```

##### componentDidUpdate()

会在更新后被立即调用，首次渲染不会执行该方法。

可以进行的操作：

- 当组件更新后，可以在此处对 DOM 进行操作
- 如果对更新前后的props进行了比较，也可以在此处进行网络请求（例如，props变化时，执行网络请求）。

注意：该生命周期函数是有3个参数的：

```javascript
componentDidUpdate(preProps, preState, snapshot) { 
	// 参数1：更新前的props
  // 参数2：更新前的state
  // 参数3：用于getSnapshotBeforeUpdate()生命周期函数
}
```



#### 卸载时

##### componentWillUnmouent()

会在组件卸载及销毁之前直接调用。

可以进行的操作：

清理操作，例如，清除 timer，取消网络请求或清除在componentDidMount()中创建的订阅等。

### 事件处理

- React 事件的命名采用小驼峰式（camelCase），而不是纯小写。

- 使用 JSX 语法时你需要传入一个函数作为事件处理函数，而不是一个字符串。

  ```javascript
  <button onclick="activateLasers()">...</button> // 传统HTML
  <button onClick={activateLasers}>...</button> // React
  ```

-  你不能通过返回 `false` 的方式阻止默认行为。你必须显式的使用 `preventDefault` 。 

  ```javascript
  function handleClick(e) {
    e.preventDefault(); // 显式的使用 `preventDefault`
    console.log('The link was clicked.');
  }
  ```

-  使用 React 时，你一般不需要使用 `addEventListener` 为已创建的 DOM 元素添加监听器，需要在该元素初始渲染的时候添加监听器。 

  ```javascript
  <button onClick={this.xxx}>...</button>
  ```

- class 的方法默认不会绑定 `this`。如果你忘记绑定 `this.handleClick` 并把它传入了 `onClick`，当你调用这个函数的时候 `this `的值为 `undefined`。

  ```javascript
  /* 推荐 */
  /* 方式一 传入一个箭头函数，在箭头函数中调用要执行的函数 */
  render() {
    return (
    	<div>
      	<button onClick={ () => {this.xxx(abc)} }>按钮</button>  
  			<button onClick={ e => {this.xxx(abc, e)} }>按钮</button> // e是event对象
      </div>
    )
  }
  xxx(abc) {
    ...
  }
  
  /* 方式二：行间定义事件后面使用bind绑定this */
  // 一般不使用这种方式，因为代码重复太多，例如有多个按钮调用xxx方法时，每个按钮都需要进行bind
  render() {
    return (
      <div>
      	<button onClick={this.xxx.bind(this, yyy)}>按钮</button> // bind括号内第一个参数是修改this的，后面可以设置其他参数进行传值。
  		<div>
    )
  }
  btnClick() {
    ...
  }
  
  /* 方式三：在构造函数内部声明this指向 */
  class xxx extends React.Component {
    constructor(props) {
      ...
      this.handleClick = this.btnClick.bind(this);
    }
    render() {
      return (
        <div>
        	<button onClick={this.btnClick}>按钮</button>
        <div>
      )
    }
    btnClick() {
      ...
    }
  }
  
  /* 方式四：使用箭头函数声明事件 */
  render() {
    return (
    	<div>
      	<button onClick={this.xxx}>按钮</button>
      </div>
    )
  }
  xxx = () => { ... }
  ```

- 向事件处理程序传递参数。在循环中，通常我们会为事件处理函数传递额外的参数。例如，若 `id` 是你要删除那一行的 ID，以下两种方式都可以向事件处理函数传递参数： 

  ```javascript
  <button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
  <button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
  ```

  上述两种方式是等价的。在这两种情况下，React 的事件对象 `e` 会被作为第二个参数传递。如果通过箭头函数的方式，事件对象必须显式的进行传递，而通过 `bind` 的方式，事件对象以及更多的参数将会被隐式的进行传递。 

### 条件渲染

- 声明一个变量并使用 `if` 语句进行条件渲染。适合逻辑代码非常多的情况

  ```javascript
  function Greeting(props) {
    const isLoggedIn = props.isLoggedIn;
    if (isLoggedIn) {
      return <UserGreeting />;
    } else {
      return <GuestGreeting />;
    }
  }
  
  ReactDOM.render(
    // Try changing to isLoggedIn={true}:
    <Greeting isLoggedIn={false} />,
    document.getElementById('root')
  );
  ```

- 与运算符 && 。

  ```javascript
  function Mailbox(props) {
    const xxx = props.xxx;
    return (
      <div>
      	// 通过花括号包裹代码，你可以在 JSX 中嵌入任何表达式。
      	<h2>{xxx.length && "你好啊"}</h2> // 条件不满足时<h2>还是存在的
        {xxx .length > 0 && <h2>你好啊</h2>} // 条件不满足时<h2>也不存在
      </div>
    );
  }
  
  const messages = ['React', 'Re: React', 'Re:Re: React'];
  ReactDOM.render(
    <Mailbox xxx={messages} />,
    document.getElementById('root')
  );
  ```
  
- 三目运算符。

  ```javascript
  render() {
    const isLoggedIn = this.state.isLoggedIn;
    return (
      <div>
        {isLoggedIn
          ? <LogoutButton onClick={this.handleLogoutClick} />
          : <LoginButton onClick={this.handleLoginClick} />
        }
      </div>
    );
  }
  ```

- 阻止条件渲染。在极少数情况下，你可能希望能隐藏组件，即使它已经被其他组件渲染。若要完成此操作，你可以让 `render` 方法直接返回 `null`，而不进行任何渲染。

  在组件的 `render` 方法中返回 `null` 并不会影响组件的生命周期。

  ```javascript
  function WarningBanner(props) {
    if (!props.warn) { // 如果 warn 的值是 false，那么组件则不会渲染
      return null;
    }
    return (
      <div className="warning">
        Warning!
      </div>
    );
  }
  ```

- 

### 列表 & key

#### 使用 `map()` 渲染组件

```javascript
//  map()方法来遍历数组。将数组中的每个元素变成 <li> 标签，最后将得到的数组赋值给 listItems
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) => <li>{number}</li>);
// 把整个 listItems 插入到 <ul> 元素中，然后渲染进 DOM
ReactDOM.render(<ul>{listItems}</ul>, document.getElementById('root'));
```

```javascript
// 可以在JSX中嵌入map()。JSX 允许在大括号中嵌入任何表达式，所以我们可以内联 map() 返回的结果
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {numbers.map(number => <ListItem key={number.toString()} value={number} />)}
    </ul>
  );
}
```

```javascript
// 也可以将其抽成组件
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) => <li>{number}</li>); // 没有添加key
  return (<ul>{listItems}</ul>);
}
const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(<NumberList numbers={numbers} />, document.getElementById('root'));
```

#### key

上述代码没有指定 `key`，会提示警告：`a key should be provided for list items`，意思是当你创建一个元素时，必须包括一个特殊的 `key` 属性，帮助 React 识别哪些元素改变了，比如被添加或删除。 

key 最好是这个元素在列表中拥有的一个独一无二的字符串。通常，我们使用数据中的 id 来作为元素的 key ； 当元素没有确定 id 的时候，万不得已你可以使用元素索引 index 作为 key，但是如果列表项目的顺序可能会变化，我们不建议使用索引来用作 key 值。

如果你选择不指定显式的 key 值，那么 React 将默认使用索引用作为列表项目的 key 值。 

```javascript
const todoItems = todos.map((todo) =>
  <li key={todo.id}> {todo.text} </li>
);

const todoItems = todos.map((todo, index) =>
  <li key={index}> {todo.text} </li>
);
```

**用key提取组件**

元素的 key 只有放在就近的数组上下文中才有意义。

比方说，如果你提取出一个 `ListItem` 组件，你应该把 key 保留在数组中的这个 ` <ListItem /> ` 元素上，而不是放在 `ListItem` 组件中的 ` <li> ` 元素上。 

一个好的经验法则是：在 `map()` 方法中的元素需要设置 key 属性。

```javascript
function ListItem(props) {
  return <li>{props.value}</li>; // 这里不需要指定 key
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // 正确！key 应该在数组的上下文中被指定
  	<ListItem key={number.toString()} value={number} /> 
  );
  return (<ul>{listItems}</ul>);
}
  
const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(<NumberList numbers={numbers} />, document.getElementById('root'));
```

**key只是在兄弟节点之间必须唯一**

数组元素中使用的 key 在其兄弟节点之间应该是唯一的，但它们不需要全局唯一。 

```javascript
const sidebar = (
  <ul>
    {props.posts.map(post => <li key={post.id}>...</li>)}
  </ul>
);
const content = props.posts.map(post => <div key={post.id}>...</div>);
```

### 表单

在 React 里，HTML 表单元素的工作方式和其他的 DOM 元素有些不同，这是因为表单元素通常会保持一些内部的 state。

#### 受控组件

在 HTML 中，表单元素（如` <input> `、 ` <textarea> ` 和 ` <select> `）之类的表单元素通常自己维护 state，并根据用户输入进行更新。而在 React 中，可变状态（mutable state）通常保存在组件的 state 属性中，并且只能通过使用 `setState()`来更新。

我们可以把两者结合起来，使 React 的 state 成为“唯一数据源”。渲染表单的 React 组件还控制着用户输入过程中表单发生的操作。被 React 以这种方式控制取值的表单输入元素就叫做“受控组件”。

```javascript
// 例如，input元素的value从state中获取，使得state成为唯一数据源，onChange事件改变state中的value值，从而改变value值，输入的值始终由React的state驱动。
this.state = {value: ''};
handleChange(event) { this.setState({value: event.target.value}) }
<input type="text" value={this.state.value} onChange={this.handleChange} />
```

##### textarea 标签

在 HTML 中, ` <textarea> ` 元素通过其子元素定义其文本:

```javascript
<textarea>
  你好， 这是在 text area 里的文本
</textarea>
```

而在 React 中，` <textarea> ` 使用 `value` 属性代替。这样，可以使得使用 ` <textarea> ` 的表单和使用单行 input 的表单非常类似：

```javascript
this.state = { value: '请撰写一篇关于你喜欢的 DOM 元素的文章.' };
handleChange(event) => { this.setState({value: event.target.value}); }
<textarea value={this.state.value} onChange={this.handleChange} />        
```

##### select 标签

React 不使用 `selected` 进行默认选项，而是在根 `select` 标签上使用 `value` 属性，这在受控组件中更便捷，因为您只需要在根标签中更新它。 

```javascript
handleChange(event) {
  this.setState({value: event.target.value});
}
<select value={this.state.value} onChange={this.handleChange}>
	<option value="grapefruit">葡萄柚</option>
	<option value="lime">酸橙</option>
	<option value="coconut">椰子</option>
	<option value="mango">芒果</option>
</select>

// 可以将数组传递到 value 属性中，以支持在 select 标签中选择多个选项：
<select multiple={true} value={['B', 'C']}>
```

##### 文件 input 标签

在 HTML 中，` <input type="file"> ` 允许用户从存储设备中选择一个或多个文件，将其上传到服务器，或通过使用 JavaScript 的 [File API](https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications) 进行控制。

```
<input type="file" />
```

因为它的 value 只读，所以它是 React 中的一个**非受控**组件。

##### 处理多个输入

当需要处理多个 `input` 元素时，可以给每个元素添加 `name` 属性，并让处理函数根据 `event.target.name` 的值选择要执行的操作。 

```javascript
class Reservation extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      xxx: true,
      yyy: 2
    };
  }

  handleInputChange = event => {
    const target = event.target;
    const value = target.name === 'xxx' ? target.checked : target.value;
    const name = target.name;
    this.setState({ [name]: value });
  }

  render() {
    return (
      <form>
        <label>
    参与:	<input name="xxx" type="checkbox" checked={this.state.isGoing}
            onChange={this.handleInputChange} />
        </label>
        <label>
 来宾人数: <input name="yyy" type="number" value={this.state.numberOfGuests}
            onChange={this.handleInputChange} />
        </label>
      </form>
    );
  }
}
```

##### 受控输入空值

在受控组件上指定 value 的 prop 会阻止用户更改输入。如果你指定了 `value`，但输入仍可编辑，则可能是你意外地将 `value` 设置为 `undefined` 或 `null`。 

```javascript
// 输入最初被锁定，但在短时间延迟后变为可编辑
ReactDOM.render(<input value="hi" />, mountNode);

setTimeout(function() {
  ReactDOM.render(<input value={null} />, mountNode);
}, 1000);
```

##### 成熟的解决方案

如果你想寻找包含验证、追踪访问字段以及处理表单提交的完整解决方案，使用 [Formik](https://jaredpalmer.com/formik) 是不错的选择。然而，它也是建立在受控组件和管理 state 的基础之上 —— 所以不要忽视学习它们。

#### 非受控组件

使用非受控组件时，表单数据将交由 DOM 节点来处理。 

要编写一个非受控组件，可以 [使用 ref](https://reactjs.bootcss.com/docs/refs-and-the-dom.html) 来从 DOM 节点中获取表单数据。 

 例如，下面的代码使用非受控组件接受一个表单的值：

```javascript
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.input = React.createRef();
  }

  handleSubmit = event => {
    alert('A name was submitted: ' + this.input.current.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name: <input type="text" ref={this.input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

非受控组件将真实数据储存在 DOM 节点中，所以在使用非受控组件时，有时候反而更容易同时集成 React 和非 React 代码。如果你不介意代码美观性，并且希望快速编写代码，使用非受控组件往往可以减少你的代码量。否则，你应该使用受控组件。 

##### 默认值

在 React 渲染生命周期时，表单元素上的 `value` 将会覆盖 DOM 节点中的值，在非受控组件中，你经常希望 React 能赋予组件一个初始值，但是不去控制后续的更新。 在这种情况下, 你可以指定一个 `defaultValue` 属性，而不是 `value`。

```javascript
render() {
  return (
    <form onSubmit={this.handleSubmit}>
      <label>
        Name: <input defaultValue="Bob" type="text" ref={this.input} />
      </label>
      <input type="submit" value="Submit" />
    </form>
  );
}
```

同样，` <input type="checkbox"> ` 和 ` <input type="radio"> ` 支持 `defaultChecked`，` <select> ` 和 ` <textarea> ` 支 持 `defaultValue`。

##### 文件输入

在 HTML 中，` <input type="file"> ` 可以让用户选择一个或多个文件上传到服务器，或者通过使用 [File API](https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications) 进行操作。

```javascript
<input type="file" />
```

在 React 中，` <input type="file" /> ` 始终是一个非受控组件，因为它的值只能由用户设置，而不能通过代码控制。

您应该使用 File API 与文件进行交互。下面的例子显示了如何创建一个 [DOM 节点的 ref](https://reactjs.bootcss.com/docs/refs-and-the-dom.html) 从而在提交表单时获取文件的信息。

```javascript
class FileInput extends React.Component {
  constructor(props) {
    super(props);
    this.fileInput = React.createRef()
  }
  handleSubmit = event => {
    event.preventDefault();
    alert( `Selected file - ${this.fileInput.current.files[0].name}` );
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Upload file: <input type="file" ref={this.fileInput} />        
        </label>
        <button type="submit">Submit</button>
      </form>
    );
  }
}
ReactDOM.render(<FileInput />, document.getElementById('root'))
```









### 状态提升

将多个组件中需要共享的 `state` 向上移动到它们的最近共同父组件中，父组件相当于数据源，通过 `props` 传递到各个组件，便可实现共享 state。这就是所谓的“状态提升”。 

需要注意的是，此时，所需要的 `state` 是通过 `props.xxx` 传入的，子组件不能对其进行修改（props的只读性）， 在 React 中，这个问题通常是通过使用“受控组件”来解决的。 

```javascript
<input value = {this.state.value} onChange = {this.handleChange}/>
```

在父组件中定义需要传入的 `props` 和 相关的处理函数，一并传入子组件。

### 组合 vs 继承

推荐使用组合而非继承来实现组件间的代码重用。 

#### 包含关系（插槽）

有些组件无法提前知晓子组件的具体内容。

建议这些组件使用一个特殊的 `children` prop 来将他们的子组件传递到渲染结果中（类似于Vue的插槽，但React中没有“槽”这一概念的限制，任何东西都可以作为props传递）： 

```javascript
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
```

这使得别的组件可以通过 JSX 嵌套，将任意组件作为子组件传递给它们。

```javascript
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title"> Welcome </h1>      
    	<p className="Dialog-message">        
    		Thank you for visiting our spacecraft!      
    	</p>    
    </FancyBorder>
  );
}
```

` <FancyBorder> ` JSX 标签中的所有内容都会作为一个 `children` prop 传递给 `FancyBorder` 组件。因为 `FancyBorder` 将 `{props.children}` 渲染在一个 `  <div>  ` 中，被传递的这些子组件最终都会出现在输出结果中。 

有时需要在一个组件中预留出几个“洞”。这种情况下，我们可以不使用 `children`，而是自行约定：将所需内容传入 props，并使用相应的 prop。 

```javascript
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">
        {props.left}
      </div>
      <div className="SplitPane-right">
        {props.right}
      </div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={ <Contacts /> }
      right={ <Chat /> } 
		/>
  );
}
```

#### 特例关系（组合）

有些时候，我们会把一些组件看作是其他组件的特殊实例（实例通过props传入真正的数据使组件具体化）。

在 React 中，我们也可以通过组合来实现这一点。“特殊”组件可以通过 props 定制并渲染“一般”组件：

```javascript
// WelcomeDialog 可以说是 Dialog 的特殊实例
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
    </FancyBorder>
  );
}

function WelcomeDialog() {
  return (
    <Dialog
      title="Welcome"
      message="Thank you for visiting our spacecraft!" />
  );
}
```

```javascript
// class 形式的组合：
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title"> {props.title} </h1>
      <p className="Dialog-message"> {props.message} </p>
      															 {props.children}
    </FancyBorder>
  );
}

class SignUpDialog extends React.Component {
  constructor(props) {
    super(props);
    this.state = {login: ''};
  }

  render() {
    return (
      <Dialog title="Mars Exploration Program"
              message="How should we refer to you?">
        <input value={this.state.login} onChange={this.handleChange} />
        <button onClick={this.handleSignUp}> Sign Me Up! </button>
      </Dialog>
    );
  }

  handleChange = e => { this.setState({login: e.target.value}); }
  handleSignUp = () => { alert(`Welcome aboard, ${this.state.login}!`); }
}
```

#### 继承【不需要】

在 Facebook，我们在成百上千个组件中使用 React。我们并没有发现需要使用继承来构建组件层次的情况。

Props 和组合为你提供了清晰而安全地定制组件外观和行为的灵活方式。注意：组件可以接受任意 props，包括基本数据类型，React 元素以及函数。

如果你想要在组件间复用非 UI 的功能，我们建议将其提取为一个单独的 JavaScript 模块，如函数、对象或者类。组件可以直接引入（import）而无需通过 extend 继承它们。



## 高级指引



### 代码分割

#### React.lazy

```javascript
import OtherComponent from './OtherComponent' // 不使用 React.lazy
```

`React.lazy` 函数能让你像渲染常规组件一样处理动态引入（的组件）。

1. `React.lazy` 接受一个函数，这个函数需要动态调用 `import()`。它必须返回一个 `Promise`，该 Promise 需要 resolve 一个 `default` export 的 React 组件。
2.  `Suspense` 组件中渲染 lazy 组件，如此使得我们可以使用在等待加载 lazy 组件时做优雅降级（如 loading 指示器等）。`fallback` 属性接受任何在组件加载过程中你想展示的 React 元素。你可以将 `Suspense` 组件置于懒加载组件之上的任何位置。你甚至可以用一个 `Suspense` 组件包裹多个懒加载组件。 

```javascript
// 1.引入
import React, { Suspense } from 'react'; 

// 2.调用React.lazy()函数
const OtherComponent = React.lazy(() => import('./OtherComponent')); 
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

function MyComponent() {
  return (
    <div>
    	// 3.在Suspense中使用组件
      <Suspense fallback={<div>Loading...</div>}> // 4.设置fallback属性
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </div>
  );
}

```

#### 异常捕获边界

具体见 **错误边界** 。

如果模块加载失败（如网络问题），它会触发一个错误。你可以通过[异常捕获边界（Error boundaries）](https://reactjs.bootcss.com/docs/error-boundaries.html)技术来处理这些情况，以显示良好的用户体验并管理恢复事宜。

```javascript
import React, { Suspense } from 'react';
import MyErrorBoundary from './MyErrorBoundary';

const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

const MyComponent = () => (
  <div>
  	// 使用 <MyErrorBoundary> 包裹
    <MyErrorBoundary> 
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </MyErrorBoundary>
  </div>
);
```

#### 基于路由的代码分割

决定在哪引入代码分割需要一些技巧，一个不错的选择是从路由开始。大多数网络用户习惯于页面之间能有个加载切换过程。你也可以选择重新渲染整个页面，这样用户就不必在渲染的同时再和页面上的其他元素进行交互。 

```javascript
// 这里是一个例子，展示如何在你的应用中使用 React.lazy 和 React Router 这类的第三方库，来配置基于路由的代码分割。
import React, { Suspense, lazy } from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';

const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Switch>
        <Route exact path="/" component={Home}/>
        <Route path="/about" component={About}/>
      </Switch>
    </Suspense>
  </Router>
);
```

#### 命名导出

`React.lazy` 目前只支持默认导出（default exports）。如果你想被引入的模块使用命名导出（named exports），你可以创建一个中间模块，来重新导出为默认模块。这能保证 tree shaking 不会出错，并且不必引入不需要的组件。

即，将其他模块用命名导出汇集到中间模块后，再将中间模块用默认导出，再用lazy引入中间模块。

- 默认导出：对于导出内容的命名无关紧要，只要给定名称即可，但默认导出只有一个。
- 命名导出：导入时，名称必须与导出的名称一致，可以导出多个。

```javascript
// ManyComponents.js
export const MyComponent = /* ... */;
export const MyUnusedComponent = /* ... */;

// MyComponent.js
export { MyComponent as default } from "./ManyComponents.js";

// MyApp.js
import React, { lazy } from 'react';
const MyComponent = lazy(() => import("./MyComponent.js"));
```

 ### Context

#### 使用时机

Context 主要应用场景在于很多不同层级的组件需要访问同样一些的数据。请谨慎使用，因为这会使得组件的复用性变差。如果你只是想避免层层传递一些属性，[组件组合](https://reactjs.bootcss.com/docs/composition-vs-inheritance.html)有时候是一个比 context 更好的解决方案。

Context 提供了一个无需为每层组件手动添加 props，就能在组件树间进行数据传递的方法。 

Context 设计目的是为了共享那些对于一个组件树而言是“全局”的数据，例如当前认证的用户、主题或首选语言。

```javascript
// 1.为当前的 theme 创建一个 context（“light”为默认值）。
const xxx = React.createContext('light');

class App extends React.Component {
  render() {
    // 使用一个 Provider 来将当前的 theme 传递给以下的组件树。无论多深，任何组件都能读取这个值。
    // 在这个例子中，我们将 “dark” 作为当前的值传递下去。
    return (
      // 2.使用创建的 context 包裹需要作用到的组件，并指定当前值 value
      <xxx.Provider value="dark">
        <Toolbar />
      </xxx.Provider>
    );
  }
}

// 中间的组件再也不必指明往下传递 theme 了。
function Toolbar() { // 中间组件
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // 3.方式一：指定 contextType 读取当前的 theme context。React 会往上找到最近的 theme Provider，然后使用它的值。在这个例子中，当前的 theme 值为 “dark”。
  static contextType = ThemeContext; 
  render() {
    // 4.方式一：通过this.context使用值。如果需要传递多个可能同名的context，只能用Consumer来写
    return <Button theme={this.context} />; 
    
    return (
      // 4.方式二：使用Consumer来接收值，Consumer里面不能直接渲染其他组件，而是要声明一个函数，参数是组件树中层这个 context 最接近的 Provider 的对应属性。
      <xxx.Consumer> 
      	{
          yyy => <h1>title: {yyy}</h1>
        }
      </xxx.Consumer>
    )
  }
}
// 3.方式二：除了static contextType = ThemeContext 也可以这样写：
ThemedButton.contextType = ThemeContext;
```

#### API

**React.createContext**

```javascript
const MyContext = React.createContext(defaultValue);
```

创建一个 Context 对象。当 React 渲染一个订阅了这个 Context 对象的组件，这个组件会从组件树中离自身最近的那个匹配的 `Provider` 中读取到当前的 context 值。

**只有**当组件所处的树中没有匹配到 Provider 时，其 `defaultValue` 参数才会生效。这有助于在不使用 Provider 包装组件的情况下对组件进行测试。注意：将 `undefined` 传递给 Provider 的 value 时，消费组件的 `defaultValue` 不会生效。

**Context.Provider**

```javascript
<MyContext.Provider value={/* 某个值 */}>
```

每个 Context 对象都会返回一个 Provider React 组件，它允许消费组件订阅 context 的变化。

Provider 接收一个 `value` 属性，传递给消费组件。一个 Provider 可以和多个消费组件有对应关系。多个 Provider 也可以嵌套使用，里层的会覆盖外层的数据。

当 Provider 的 `value` 值发生变化时，它内部的所有消费组件都会重新渲染。Provider 及其内部 consumer 组件都不受制于 `shouldComponentUpdate` 函数，因此当 consumer 组件在其祖先组件退出更新的情况下也能更新。

通过新旧值检测来确定变化，使用了与 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description) 相同的算法。

**Class.contextType**

```javascript
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* 在组件挂载完成后，使用 MyContext 组件的值来执行一些有副作用的操作 */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* 基于 MyContext 组件的值进行渲染 */
  }
}
MyClass.contextType = MyContext;
```

挂载在 class 上的 `contextType` 属性会被重赋值为一个由 `React.createContext()` 创建的 Context 对象。这能让你使用 `this.context` 来消费最近 Context 上的那个值。你可以在任何生命周期中访问到它，包括 render 函数中。

**Context.Consumer**

```javascript
<MyContext.Consumer>
  {value => /* 基于 context 值进行渲染*/}
</MyContext.Consumer>
```

这里，React 组件也可以订阅到 context 变更。这能让你在函数式组件中完成订阅 context。

这需要[函数作为子元素（function as a child）](https://reactjs.bootcss.com/docs/render-props.html#using-props-other-than-render)这种做法。这个函数接收当前的 context 值，返回一个 React 节点。传递给函数的 `value` 值等同于往上组件树离这个 context 最近的 Provider 提供的 `value` 值。如果没有对应的 Provider，`value` 参数等同于传递给 `createContext()` 的 `defaultValue`。

**Context.displayName**

context 对象接受一个名为 `displayName` 的 property，类型为字符串。React DevTools 使用该字符串来确定 context 要显示的内容。

示例，下述组件在 DevTools 中将显示为 MyDisplayName：

```javascript
const MyContext = React.createContext(/* some value */);
MyContext.displayName = 'MyDisplayName';
<MyContext.Provider> // "MyDisplayName.Provider" 在 DevTools 中
<MyContext.Consumer> // "MyDisplayName.Consumer" 在 DevTools 中
```

#### 嵌套组件更新中更新 Context

可以通过 context 传递一个函数，使得 consumers 组件更新 context。

```javascript
// theme-context.js
export const ThemeContext = React.createContext({
  // 确保传递给 createContext 的默认值数据结构是调用的组件（consumers）所能匹配的
  theme: themes.dark,  
  toggleTheme: () => {} // 传递函数      
});
```

```javascript
// theme-toggler-button.js
import {ThemeContext} from './theme-context';

function ThemeTogglerButton() {
  return (
    <ThemeContext.Consumer> // 使用Consumer来接收值
    	// 不仅仅只获取 theme 值，它也从 context 中获取到一个 toggleTheme 函数
      {({theme, toggleTheme}) => (
        <button onClick={toggleTheme} style={{backgroundColor: theme.background}}>
          Toggle Theme
        </button>
      )}
    </ThemeContext.Consumer>
  );
}

export default ThemeTogglerButton;
```

```javascript
// app.js
import {ThemeContext, themes} from './theme-context';
import ThemeTogglerButton from './theme-toggler-button';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.toggleTheme = () => {
      this.setState(
        state => ({theme: state.theme === themes.dark ? themes.light : themes.dark})
      );
    };

       
    this.state = {      
      theme: themes.light,
      toggleTheme: this.toggleTheme // State包含了更新函数，它会被传递进context provider。
    };  
  }

  render() {
    return (
      <ThemeContext.Provider value={this.state}> // 整个 state 都被传递进 provider
      	<Content />
      </ThemeContext.Provider>
    );
  }
}

function Content() {
  return (
    <div>
      <ThemeTogglerButton />
    </div>
  );
}

ReactDOM.render(<App />, document.root);
```

#### 消费多个 Context

为了确保 context 快速进行重渲染，需要使每一个 consumers 组件的 context 在组件树中成为一个单独的节点。 如果两个或者更多的 context 值经常被一起使用，那你可能要考虑一下另外创建你自己的渲染组件，以提供这些值。 

```javascript
const ThemeContext = React.createContext('light');
const UserContext = React.createContext({ name: 'Guest' });

class App extends React.Component {
  render() {
    const {signedInUser, theme} = this.props;
    return (
      <ThemeContext.Provider value={theme}>
        <UserContext.Provider value={signedInUser}>
          <Layout />
        </UserContext.Provider>
      </ThemeContext.Provider>
    );
  }
}

function Layout() {
  return (
    <div>
      <Sidebar />
      <Content />
    </div>
  );
}

// 一个组件可能会消费多个 context，只需要嵌套使用即可
function Content() {
  return (
    <ThemeContext.Consumer>
      {theme => (
        <UserContext.Consumer>
          {user => ( <ProfilePage user={user} theme={theme} /> )}
        </UserContext.Consumer>
      )}
    </ThemeContext.Consumer>
  );
}
```

#### 注意事项

1. 减少不必要使用`context`，`react`重视函数式编程，讲究复用，而使用了`context`的组件，使复用性降低。

2. 传统版本的`react`，尤其要注意`context`在自己的可控范围内，其实最大的问题也就是前面说的`SUC`的问题。

3.  `react-router`其实也用了`Context`的原理。 

4.  **一旦 Provider(提供者) 的 value 属性发生变化时，所有作为 Provider(提供者) 后代的 consumer(使用者) 组件都将重新渲染。 从 Provider 到其后代使用者的传播不受 shouldComponentUpdate 方法的约束，因此即使祖先组件退出更新，也会更新 consumer(使用者**)。 context 会使用参考标识（reference identity）来决定何时进行渲染。这可能使得当 provider 的父组件进行重渲染时，可能会在 consumers 组件中触发意外的渲染。 所以通常的方式是：将`context`的数据保存在`Provide`的`state`属性中，每次通过`setState`更新对应的属性。例如：

   ```javascript
   // 当每一次 Provider 重渲染时，以下的代码会重渲染所有下面的 consumers 组件，因为每次render时{something: 'something'}都指向一个新对象
   class App extends React.Component {
     render() {
       return (
         <MyContext.Provider value={{something: 'something'}}>
           <Toolbar />
         </MyContext.Provider>
       );
     }
   }
   
   // 为了防止这种情况，将 value 状态提升到父节点的 state 里
   class App extends React.Component {
     constructor(props) {
       super(props);
       this.state = {
         value: {something: 'something'} // 将值保存在state中
       };
     }
   
     render() {
       return (
         <Provider value={this.state.value}>
           <Toolbar />
         </Provider>
       );
     }
   }
   ```

#### useContext Hook

接收一个 context 对象（`React.createContext` 的返回值）并返回该 context 的当前值。当前的 context 值由上层组件中距离当前组件最近的 ` <MyContext.Provider> ` 的 `value` prop 决定。

当组件上层最近的 ` <MyContext.Provider> ` 更新时，该 Hook 会触发重渲染，并使用最新传递给 `MyContext` provider 的 context `value` 值。即使祖先使用 [`React.memo`](https://reactjs.bootcss.com/docs/react-api.html#reactmemo) 或 [`shouldComponentUpdate`](https://reactjs.bootcss.com/docs/react-component.html#shouldcomponentupdate)，也会在组件本身使用 `useContext` 时重新渲染。

`useContext` 的参数必须是 context 对象本身：

- **正确：** `useContext(MyContext)`
- **错误：** `useContext(MyContext.Consumer)`
- **错误：** `useContext(MyContext.Provider)`

调用了 `useContext` 的组件总会在 context 值变化时重新渲染。如果重渲染组件的开销较大，你可以 [通过使用 memoization 来优化](https://github.com/facebook/react/issues/15156#issuecomment-474590693)。

> 如果你在接触 Hook 前已经对 context API 比较熟悉，那应该可以理解，`useContext(MyContext)` 相当于 class 组件中的 `static contextType = MyContext` 或者 ` <MyContext.Consumer> `。
>
> `useContext(MyContext)` 只是让你能够读取 context 的值以及订阅 context 的变化，仍然需要在上层组件树中使用 ` <MyContext.Provider> ` 来为下层组件提供 context。

在 `Hooks` 环境中，依旧可以使用 `Consumer`，但是 `ContextType` 作为类静态成员肯定是用不了。Hooks 提供了 `useContext`,不但解决了 `Consumer` 难用的问题同时也解决了　`contextType` 只能使用一个 `context` 的问题。 

```javascript
import React from "react";
import ReactDOM from "react-dom";

// 创建 Context，它返回一个具有两个值的对象：{ Provider, Consumer }
const NumberContext = React.createContext(); 

function App() {
  return (
    <NumberContext.Provider value={42}> // 使用 Provider 为所有子孙代提供 value 值 
      <div>
        <Display />
      </div>
    </NumberContext.Provider>
  );
}

function Display() {
  return (
    <NumberContext.Consumer> // 使用 Consumer 从上下文中获取 value
      { value => <div>The answer is {value}</div> }
    </NumberContext.Consumer>
  );
}
// 使用 useContext hook
function Display() {
  const value = useContext(NumberContext); // 使用 useContext() 获取 value
  return (
    <div> The answer is {value} </div>;
	);
}

ReactDOM.render(<App />, document.querySelector("#root"));
```

```javascript
// 多重嵌套的情况，组件需要从多个父上下文中接收数据
function HeaderBar() {
  return (
    <CurrentUser.Consumer>
      {user =>
        <Notifications.Consumer>
          {notifications =>
            <header>
              {user.name}! {notifications.length}!
            </header>
          }
				</Notifications.Consumer>
      }
    </CurrentUser.Consumer>
  );
}

// 使用 useContext hook
function HeaderBar() {
  const user = useContext(CurrentUser);
  const notifications = useContext(Notifications);
  return (
    <header>
      {user.name}! {notifications.length}!
    </header>
  );
}
```

### 错误边界

部分 UI 的 JavaScript 错误不应该导致整个应用崩溃，为了解决这个问题，React 16 引入了一个新的概念 —— 错误边界，一种 React 组件，**可以捕获并打印发生在其子组件树任何位置的 JavaScript 错误，并且，它会渲染出备用 UI，而不是渲染那些崩溃了的子组件树**。错误边界在渲染期间、生命周期方法和整个组件树的构造函数中捕获错误。 

> **注意**
>
> 错误边界**无法**捕获以下场景中产生的错误：
>
> - 事件处理
> - 异步代码（例如 `setTimeout` 或 `requestAnimationFrame` 回调函数）
> - 服务端渲染
> - 它自身抛出来的错误（并非它的子组件）

如果一个 class 组件中定义了 [`static getDerivedStateFromError()`](https://reactjs.bootcss.com/docs/react-component.html#static-getderivedstatefromerror) 或 [`componentDidCatch()`](https://reactjs.bootcss.com/docs/react-component.html#componentdidcatch)这两个生命周期方法中的任意一个（或两个）时，那么它就**变成**一个错误边界。当抛出错误后，请使用 `static getDerivedStateFromError()` 渲染备用 UI ，使用 `componentDidCatch()` 打印错误信息。 

```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 更新 state 使下一次渲染能够显示降级后的 UI
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 你同样可以将错误日志上报给服务器
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // 你可以自定义降级后的 UI 并渲染
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}
```

```javascript
// 然后你可以将它作为一个常规组件去使用
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```

错误边界的工作方式类似于 JS 的 `catch {}`，不同的地方在于错误边界只针对 React 组件。只有 class 组件才可以成为错误边界组件。大多数情况下, 你只需要声明一次错误边界组件, 并在整个应用中使用它。

注意**错误边界仅可以捕获其子组件的错误**，它无法捕获其自身的错误。如果一个错误边界无法渲染错误信息，则错误会冒泡至最近的上层错误边界，这也类似于 JavaScript 中 catch {} 的工作机制。

**放置位置**

错误边界的粒度由你来决定，可以将其包装在最顶层的路由组件并为用户展示一个 “Something went wrong” 的错误信息，就像服务端框架经常处理崩溃一样。你也可以将单独的部件包装在错误边界以保护应用其他部分不崩溃。 

**未捕获错误（Uncaught Errors）的新行为**

*自 React 16 起，任何未被错误边界捕获的错误将会导致整个 React 组件树被卸载。*

此变化意味着当你迁移到 React 16 时，你可能会发现一些已存在你应用中但未曾注意到的崩溃。增加错误边界能够让你在应用发生异常时提供更好的用户体验。

例如，Facebook Messenger 将侧边栏、信息面板、聊天记录以及信息输入框包装在单独的错误边界中。如果其中的某些 UI 组件崩溃，其余部分仍然能够交互。

我们也鼓励使用 JS 错误报告服务（或自行构建），这样你能了解关于生产环境中出现的未捕获异常，并将其修复。

**组件栈追踪**

在开发环境下，React 16 会把渲染期间发生的所有错误打印到控制台，即使该应用意外的将这些错误掩盖。除了错误信息和 JavaScript 栈外，React 16 还提供了组件栈追踪。现在你可以准确地查看发生在组件树内的错误信息：

[![Error caught by Error Boundary component](React.assets/error-boundaries-stack-trace.png)](https://reactjs.bootcss.com/static/f1276837b03821b43358d44c14072945/c3a47/error-boundaries-stack-trace.png)

你也可以在组件栈追踪中查看文件名和行号，这一功能在 [Create React App](https://github.com/facebookincubator/create-react-app) 项目中默认开启：

[![Error caught by Error Boundary component with line numbers](React.assets/error-boundaries-stack-trace-line-numbers.png)](https://reactjs.bootcss.com/static/45611d4fdbd152829b28ae2348d6dcba/6dd26/error-boundaries-stack-trace-line-numbers.png)

如果你没有使用 Create React App，可以手动将[该插件](https://www.npmjs.com/package/babel-plugin-transform-react-jsx-source)添加到你的 Babel 配置中。注意它仅用于开发环境，*在生产环境必须将其禁用。*

> 注意
>
> 组件名称在栈追踪中的显示依赖于 [`Function.name`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/name) 属性。如果你想要支持尚未提供该功能的旧版浏览器和设备（例如 IE 11），考虑在你的打包（bundled）应用程序中包含一个 `Function.name` 的 polyfill，如 [`function.name-polyfill`](https://github.com/JamesMGreene/Function.name) 。或者，你可以在所有组件上显式设置 [`displayName`](https://reactjs.bootcss.com/docs/react-component.html#displayname) 属性。

**关于 try / catch**

`try` / `catch` 很棒但它仅能用于命令式代码：

```javascript
try {
  showButton();
} catch (error) {
  // ...
}
```

然而，React 组件是声明式的并且具体指出 *什么* 需要被渲染：

```javascript
<Button />
```

错误边界保留了 React 的声明性质，其行为符合你的预期。例如，即使一个错误发生在 `componentDidUpdate` 方法中，并且由某一个深层组件树的 `setState` 引起，其仍然能够冒泡到最近的错误边界。

**关于事件处理器**

错误边界**无法**捕获事件处理器内部的错误。

React 不需要错误边界来捕获事件处理器中的错误。与 render 方法和生命周期方法不同，事件处理器不会在渲染期间触发。因此，如果它们抛出异常，React 仍然能够知道需要在屏幕上显示什么。

如果你需要在事件处理器内部捕获错误，使用普通的 JavaScript `try` / `catch` 语句：

```javascript
// 下述例子只是演示了普通的 JavaScript 行为，并没有使用错误边界
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { error: null };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    try {      
      // 执行操作，如有错误则会抛出    
    } catch (error) {      
      this.setState({ error });    
    }  
  }

  render() {
    if (this.state.error) {      
      return <h1>Caught an error.</h1>    
    }    
    return <button onClick={this.handleClick}>Click Me</button>  
  }
}
```

### Refs and the DOM

#### 基本使用

Refs 提供了一种方式，允许我们访问 DOM 节点或在 render 方法中创建的 React 元素。 

> **使用时机**
>
> - 管理焦点，文本选择或媒体播放。
> - 触发强制动画。
> - 集成第三方 DOM 库。
>
> 避免使用 refs 来做任何可以通过声明式实现来完成的事情。
>
> 例如，避免在 `Dialog` 组件里暴露 `open()` 和 `close()` 方法，最好传递 `isOpen` 属性。
>
> ### 勿过度使用 Refs

```javascript
// 注意：React.createRef() 仅能用在Class组件；useRef() 仅能用在Function组件
class MyComponent extends React.Component {
  constructor(props) {
    super(props)
    this.myRef = React.createRef() // 1.使用 React.createRef() 创建 Refs
  }
  
  focusTextInput = () => {
    this.myRef.current.focus() // 3.1 通过 current 属性进行访问
  }
  
  componentDidMount() {
    this.myRef123.current.xxx() // 3.2 通过 current 属性进行访问
  }
  
  render() {
    return <input type="text" ref={this.myRef} /> // 2.1 通过 ref 将创建的 Refs 绑定到相关元素上
    <CustomTextInput ref={this.myRef123} /> // 2.2 也可以绑定为Class组件
   // 2.3 不能在函数组件本身（内部实例可以）上使用ref属性，因为函数组件没有实例（详情见下方第3点）
  }
}
```

ref 的值根据节点的类型而有所不同：

- 当 `ref` 属性用于 HTML 元素时，构造函数中使用 `React.createRef()` 创建的 `ref` 接收底层 DOM 元素作为其 `current` 属性。

- 当 `ref` 属性用于自定义 class 组件时，`ref` 对象接收组件的挂载实例作为其 `current` 属性。

- **你不能在函数组件上使用 `ref` 属性**，因为他们没有实例；但可以在函数组件内部使用 `ref` 属性，只要它指向一个 DOM 元素或 class 组件 。如果要在函数组件中使用 `ref`，你可以使用 [`forwardRef`](https://reactjs.bootcss.com/docs/forwarding-refs.html)（可与 [`useImperativeHandle`](https://reactjs.bootcss.com/docs/hooks-reference.html#useimperativehandle) 结合使用），或者可以将该组件转化为 class 组件。 

  ```javascript
  function CustomTextInput(props) {
    // useRef 仅能用在Function组件
    const textInput = useRef(null) // 使用 React.useRef() 创建 Refs
  
    function handleClick() {
      textInput.current.focus() // 通过 current 属性进行访问
    }
  
    return (
      <div>
        <input type="text" ref={textInput} />
        <input type="button" value="Focus the text input" onClick={handleClick}/>
      </div>
    );
  }
  ```

React 会在组件挂载时给 `current` 属性传入 DOM 元素，并在组件卸载时传入 `null` 值。`ref`会在 `componentDidMount` 或 `componentDidUpdate` 生命周期钩子触发前更新。

#### 将 DOM Refs 暴露给父组件

在极少数情况下，你可能希望在父组件中引用子节点的 DOM 节点。通常不建议这样做，因为它会打破组件的封装，但它偶尔可用于触发焦点或测量子 DOM 节点的大小或位置。 

虽然你可以[向子组件添加 ref](https://reactjs.bootcss.com/docs/refs-and-the-dom.html#adding-a-ref-to-a-class-component)，但这不是一个理想的解决方案，因为你只能获取组件实例而不是 DOM 节点。并且，它还在函数组件上无效。

如果你使用 16.3 或更高版本的 React, 这种情况下我们推荐使用 [ref 转发](https://reactjs.bootcss.com/docs/forwarding-refs.html)。**Ref 转发使组件可以像暴露自己的 ref 一样暴露子组件的 ref**。关于怎样对父组件暴露子组件的 DOM 节点，在 [ref 转发文档](https://reactjs.bootcss.com/docs/forwarding-refs.html#forwarding-refs-to-dom-components)中有一个详细的例子。

如果你使用 16.2 或更低版本的 React，或者你需要比 ref 转发更高的灵活性，你可以使用[这个替代方案](https://gist.github.com/gaearon/1a018a023347fe1c2476073330cc5509)将 ref 作为特殊名字的 prop 直接传递。

可能的话，我们不建议暴露 DOM 节点，但有时候它会成为救命稻草。注意这个方案需要你在子组件中增加一些代码。如果你对子组件的实现没有控制权的话，你剩下的选择是使用 [`findDOMNode()`](https://reactjs.bootcss.com/docs/react-dom.html#finddomnode)，但在 [`严格模式`](https://reactjs.bootcss.com/docs/strict-mode.html#warning-about-deprecated-finddomnode-usage) 下已被废弃且不推荐使用。

#### 回调 Refs

另一种设置 refs 的方式，称为“回调 refs”。它能助你更精细地控制何时 refs 被设置和解除。 

你需要传递一个函数，这个函数中接受 组件实例 或 HTML DOM 元素作为参数，以使它们能在其他地方被存储和访问。 

```javascript
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    
    this.textInput = null // 1.定义变量，用于保存回调函数接收的 组件实例 或 DOM元素
    // 传递一个函数，这个函数中接受 组件实例 或 HTML DOM 元素作为参数
    this.setTextInputRef = element => { this.textInput = element } // 2.定义回调函数，接收 组件实例 或 DOM元素
    this.focusTextInput = () => { // 4.定义使用函数
      if (this.textInput) { // 判断定义的变量是否已经接收 组件实例 或 DOM元素
        this.textInput.focus() // 相应的操作
      }
    }
  }

  componentDidMount() { // 调用定义的使用函数
    this.focusTextInput() // 组件挂载后，让文本框自动获得焦点
  }

  render() {
    return (
      <div>
        <input type="text" ref={this.setTextInputRef} /> // 3.绑定 ref
        <input type="button" value="Focus the text input" 
							 onClick={this.focusTextInput} />
      </div>
    );
  }
}
```

React 将在组件挂载时，会调用 `ref` 回调函数并传入 DOM 元素，当卸载时调用它并传入 `null`。在 `componentDidMount` 或 `componentDidUpdate` 触发前，React 会保证 refs 一定是最新的。

你可以在组件间传递回调形式的 refs，就像你可以传递通过 `React.createRef()` 创建的对象 refs 一样。

```javascript
function CustomTextInput(props) {
  return (
    <div>
    	// 2.CustomTextInput 将传递的函数作为特殊的 ref 属性传递给了 <input>
    	// 结果是，在 Parent 中的 this.inputElement 会被设置为与 CustomTextInput 中的 input 元素相对应的 DOM 节点
      <input ref={props.inputRef} />
		</div>
  );
}

class Parent extends React.Component {
  render() {
    return (
      // 1.将 refs回调函数 当作 props 传递给 CustomTextInput 组件
      <CustomTextInput inputRef={ ele => this.xxx = ele } />
    );
  }
}
```

#### 关于回调 refs 的说明

如果 `ref` 回调函数是以内联函数的方式定义的，在更新过程中它会被执行两次，第一次传入参数 `null`，然后第二次会传入参数 DOM 元素。这是因为在每次渲染时会创建一个新的函数实例，所以 React 清空旧的 ref 并且设置新的。通过将 ref 的回调函数定义成 class 的绑定函数的方式可以避免上述问题，但是大多数情况下它是无关紧要的。

### Refs 转发

#### 基本使用

Ref 转发是一项将 [ref](https://reactjs.bootcss.com/docs/refs-and-the-dom.html) 自动地通过组件传递到其一子组件的技巧。对于大多数应用中的组件来说，这通常不是必需的。但其对某些组件，尤其是可重用的组件库是很有用的。 

Ref 转发是一个可选特性，其允许某些组件接收 `ref`，并将其向下传递（换句话说，“转发”它）给子组件。

```javascript
// 3.ref 作为 forwardRef() 函数的第二个参数
const FancyButton = React.forwardRef((props, ref) => (  
  // 4.向下转发 ref 到 <button ref={ref}> ，将其指定为 JSX 属性
  <button ref={ref} className="FancyButton"> xxx </button>
	// 5.当 ref 挂载完成，ref.current 将指向 <button> DOM 节点
));

// 1.通过调用 React.createRef 创建了一个 React ref 并将其赋值给 ref 变量
const ref = React.createRef(); 
// 2.通过指定 ref 为 JSX 属性，将其向下传递给 <FancyButton ref={ref}>
<FancyButton ref={ref}>Click me!</FancyButton>;
```

这样，使用 `FancyButton` 的组件可以获取底层 DOM 节点 `button` 的 ref ，并在必要时访问，就像其直接使用 DOM `button` 一样。

> 注意
>
> 第二个参数 `ref` 只在使用 `React.forwardRef` 定义组件时存在。常规函数和 class 组件不接收 `ref` 参数，且 props 中也不存在 `ref`。
>
> Ref 转发不仅限于 DOM 组件，你也可以转发 refs 到 class 组件实例中。

#### 在高阶组件中转发 refs

让我们从一个输出组件 props 到控制台的 HOC 示例开始：

```javascript
function logProps(WrappedComponent) {  
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }
    render() {
      return <WrappedComponent {...this.props} />
    }
  }
  return LogProps;
}
```

高阶组件logProps 透传（pass through）所有 `props` 到其包裹的组件，所以渲染结果将是相同的。例如：我们可以使用该 HOC 记录所有传递到 FancyButton组件 的 props：

```javascript
class FancyButton extends React.Component {
  focus() { ... }
	...
}

// 我们导出 LogProps，而不是 FancyButton。
// 虽然它也会渲染一个 FancyButton。
export default logProps(FancyButton);
```

上面的示例有一点需要注意：refs 将不会透传下去。这是因为 `ref` 不是 prop 属性。就像 `key`一样，其被 React 进行了特殊处理。如果你对 HOC 添加 ref，该 ref 将引用最外层的容器组件，而不是被包裹的组件。

这意味着用于我们 FancyButton组件 的 refs 实际上将被挂载到 LogProps组件：

```javascript
import FancyButton from './FancyButton';
const ref = React.createRef();
// 我们导入的 FancyButton 组件是高阶组件（HOC）LogProps。
// 尽管渲染结果将是一样的，但我们的 ref 将指向 LogProps 而不是内部的 FancyButton 组件
// 这意味着我们不能调用例如 ref.current.focus() 这样的方法
<FancyButton label="Click Me" handleClick={handleClick} ref={ref}/>;
```

幸运的是，我们可以使用 `React.forwardRef` API 明确地将 refs 转发到内部的 `FancyButton` 组件。`React.forwardRef` 接受一个渲染函数，其接收 `props` 和 `ref` 参数并返回一个 React 节点。例如：

```javascript
function logProps(Component) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      const {forwardedRef, ...rest} = this.props;
      // 将自定义的 prop 属性 “forwardedRef” 定义为 ref
      return <Component ref={forwardedRef} {...rest} />
    }
  }

  // 注意 React.forwardRef 回调的第二个参数 “ref”。
  // 我们可以将其作为常规 prop 属性传递给 LogProps，例如 “forwardedRef”。然后它就可以被挂载到被 LogProps 包裹的子组件上。
  return React.forwardRef((props, ref) => {    
    return <LogProps {...props} forwardedRef={ref} />; 
  })
}
```

#### 在 DevTools 中显示自定义名称

`React.forwardRef` 接受一个渲染函数，来决定为 ref 转发组件显示的内容。 

```javascript
// 1.以下组件将在 DevTools 中显示为 “ForwardRef”
const WrappedComponent = React.forwardRef((props, ref) => {
  return <LogProps {...props} forwardedRef={ref} />;
})

// 2.如果你命名了渲染函数，DevTools 也将包含其名称（例如 “ForwardRef(myFunction)”）
const WrappedComponent = React.forwardRef(
  function myFunction(props, ref) {
    return <LogProps {...props} forwardedRef={ref} />;
  }
)

// 3.你甚至可以设置函数的 displayName 属性来包含被包裹组件的名称
function logProps(Component) {
  class LogProps extends React.Component {
    ....
  }

  function forwardRef(props, ref) {
    return <LogProps {...props} forwardedRef={ref} />;
  }

  // 在 DevTools 中为该组件提供一个更有用的显示名。
  // 例如 “ForwardRef(logProps(MyComponent))”
  const name = Component.displayName || Component.name;
  forwardRef.displayName = `logProps(${name})`;

  return React.forwardRef(forwardRef);
}
```

### Fragments

`render` 函数的返回值需要有一个跟元素，例如 `div` 进行包裹，但这可能导致组件间的嵌套出现问题，例如，嵌套后的组件的真实HTML结构可能会如下所示，在 `<tr>` 标签中出现 `div`。

```javascript
<table>
  <tr>
    <div>
      <td>Hello</td>
      <td>World</td>
    </div>
  </tr>
</table>
```

其中一个解决方法就是：react 16开始， render支持返回数组，这一特性已经可以减少不必要节点嵌套。

```javascript
class Demo extends React.Component {
  render() {
     return [
        <td>Hello</td>, // 需要逗号
      	<td>World</td>
     ]
  }
}
```

另一个解决方法就是使用 `Fragments`。`Fragments `与Vue的 ` <template> ` 功能类似，可做不可见的包裹元素。 

```javascript
function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // 没有`key`，React 会发出一个关键警告
        // key 是唯一可以传递给 Fragment 的属性。未来我们可能会添加对其他属性的支持，例如事件。
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    </dl>
  );
}

/* 简写形式 */
class Columns extends React.Component {
  render() {
    return (
      <> // 你可以像使用任何其他元素一样使用 <> </>，除了它不支持 key 或属性。
        <td>Hello</td>
        <td>World</td>
      </>
    );
  }
}
```











### 高阶组件

#### 基本使用

高阶组件（HOC）是 React 中用于复用组件逻辑的一种高级技巧。HOC 自身不是 React API 的一部分，它是一种基于 React 的组合特性而形成的设计模式。HOC 在 React 的第三方库中很常见，例如 Redux 的 `connect` 。

具体而言，**高阶组件将组件作为参数，并返回新的组件的函数。**组件是将 props 转换为 UI，而高阶组件是将组件转换为另一个组件，用于抽出多个相似的组件的逻辑重复部分进行复用。

HOC 不会修改传入的组件，也不会使用继承来复制其行为。相反，HOC 通过将组件*包装*在容器组件中来*组成*新组件。HOC 是纯函数，没有副作用。 被包装组件接收来自容器组件的所有 prop，同时也接收一个新的用于 render 的 `data` prop。HOC 不需要关心数据的使用方式或原因，而被包装组件也不需要关心数据是怎么来的。 

```javascript
// 此函数接收一个组件 WrappedComponent
function withSubscription(WrappedComponent, selectData) {
  // ...并返回另一个组件...
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.state = { data: selectData(DataSource, props) };
    }

    componentDidMount() { DataSource.addChangeListener(this.handleChange) }
    componentWillUnmount() { DataSource.removeChangeListener(this.handleChange) }

    handleChange= ()=>{ this.setState({ data: selectData(DataSource, this.props) }) }

    render() {
      // ... 并使用新数据渲染被包装的组件!
      // 请注意，我们可能还会传递其他属性
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  }
}
```

**注意：不要改变原始组件。使用组合。**

不要试图在 HOC 中修改组件原型（或以其他方式改变它）。

这样做会产生一些不良后果：

1. 其一是输入组件再也无法像 HOC 增强之前那样使用了。
2. 如果你再用另一个同样会修改 `componentDidUpdate` 的 HOC 增强它，那么前面的 HOC 就会失效
3. 同时，这个 HOC 也无法应用于没有生命周期的函数组件。 

修改传入组件的 HOC 是一种糟糕的，使用者必须知道内部是如何实现的，才能避免与其他 HOC 发生冲突。

```javascript
function logProps(InputComponent) {
  InputComponent.prototype.componentDidUpdate = function(prevProps) {
    console.log('Current props: ', this.props);
    console.log('Previous props: ', prevProps);
  }
  return InputComponent // 返回原始的 input 组件，暗示它已经被修改。
}

const EnhancedComponent = logProps(InputComponent) // 每次调用logProps时，都会有打印。

/* 正确做法：使用组合的方式，通过将组件包装在容器组件中实现功能 */
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('Current props: ', this.props);
      console.log('Previous props: ', prevProps);
    }
    render() {
      // 将 input 组件包装在容器中，而不对其进行修改。
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

#### 约定

##### 将不相关的 props 传递给被包裹的组件

HOC 为组件添加特性。自身不应该大幅改变约定。HOC 返回的组件与原组件应保持类似的接口。

HOC 应该透传与自身无关的 props。大多数 HOC 都应该包含一个类似于下面的 render 方法，以保证 HOC 的灵活性以及可复用性： 

```javascript
render() {
  // 过滤掉非此 HOC 额外的 props，且不要进行透传
  const { extraProp, ...passThroughProps } = this.props;

  // 将 props 注入到被包装的组件中。
  // 通常为 state 的值或者实例方法。
  const injectedProp = someStateOrInstanceMethod;

  // 将 props 传递给被包装组件
  return (
    <WrappedComponent injectedProp={injectedProp} {...passThroughProps} />
  )
}
```

##### 最大化可组合性

并不是所有的 HOC 都一样。有时候它仅接受一个参数，也就是被包裹的组件：

```javascript
const NavbarWithRouter = withRouter(Navbar);
```

HOC 通常可以接收多个参数。比如在 Relay 中，HOC 额外接收了一个配置对象用于指定组件的数据依赖：

```javascript
const CommentWithRelay = Relay.createContainer(Comment, config);
```

最常见的 HOC 签名如下：

```javascript
// React Redux 的 `connect` 函数
const ConnectedComment = connect(commentSelector, commentActions)(CommentList);
```

*刚刚发生了什么？！*如果你把它分开，就会更容易看出发生了什么。

```javascript
// connect 是一个函数，它的返回值为另外一个函数。
const enhance = connect(commentListSelector, commentListActions);
// 返回值为 HOC，它会返回已经连接 Redux store 的组件
const ConnectedComment = enhance(CommentList);
```

换句话说，`connect` 是一个返回高阶组件的高阶函数！

这种形式可能看起来令人困惑或不必要，但它有一个有用的属性。 像 `connect` 函数返回的单参数 HOC 具有签名 `Component => Component`。 输出类型与输入类型相同的函数很容易组合在一起。

```javascript
// 而不是这样...
const EnhancedComponent = withRouter(connect(commentSelector)(WrappedComponent))

// ... 你可以编写组合工具函数
// compose(f, g, h) 等同于 (...args) => f(g(h(...args)))
const enhance = compose(
  // 这些都是单参数的 HOC
  withRouter,
  connect(commentSelector)
)
const EnhancedComponent = enhance(WrappedComponent)
```

（同样的属性也允许 `connect` 和其他 HOC 承担装饰器的角色，装饰器是一个实验性的 JavaScript 提案。）

许多第三方库都提供了 `compose` 工具函数，包括 lodash （比如 [`lodash.flowRight`](https://lodash.com/docs/#flowRight)）， [Redux](https://redux.js.org/api/compose)和 [Ramda](https://ramdajs.com/docs/#compose)。

##### 包装显示名称以便轻松调试

HOC 创建的容器组件会与任何其他组件一样，会显示在 [React Developer Tools](https://github.com/facebook/react-devtools) 中。为了方便调试，请选择一个显示名称，以表明它是 HOC 的产物。 

最常见的方式是用 HOC 包住被包装组件的显示名称。比如高阶组件名为 `withSubscription`，并且被包装组件的显示名称为 `CommentList`，显示名称应该为 `WithSubscription(CommentList)`： 

```javascript
function withSubscription(WrappedComponent) {
  class WithSubscription extends React.Component {/* ... */}
  WithSubscription.displayName = 			`WithSubscription(${getDisplayName(WrappedComponent)})`;
  return WithSubscription;
}

function getDisplayName(WrappedComponent) {
  return WrappedComponent.displayName || WrappedComponent.name || 'Component';
}
```

#### 注意

##### 不要在 render 方法中使用 HOC

React 的 diff 算法（称为协调）使用组件标识来确定它是应该更新现有子树还是将其丢弃并挂载新子树。 如果从 `render` 返回的组件与前一个渲染中的组件相同（`===`），则 React 通过将子树与新子树进行区分来递归更新子树。 如果它们不相等，则完全卸载前一个子树。

通常，你不需要考虑这点。但对 HOC 来说这一点很重要，因为这代表着你不应在组件的 render 方法中对一个组件应用 HOC：

```javascript
render() {
  // 每次调用 render 函数都会创建一个新的 EnhancedComponent
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // 这将导致子树每次渲染都会进行卸载，和重新挂载的操作！
  return <EnhancedComponent />;
}
```

这不仅仅是性能问题，重新挂载组件会导致该组件及其所有子组件的状态丢失。

如果在组件之外创建 HOC，这样一来组件只会创建一次。因此，每次 render 时都会是同一个组件。一般来说，这跟你的预期表现是一致的。

在极少数情况下，你需要动态调用 HOC。你可以在组件的生命周期方法或其构造函数中进行调用。

##### 务必复制静态方法

有时在 React 组件上定义静态方法很有用。例如，Relay 容器暴露了一个静态方法 `getFragment` 以方便组合 GraphQL 片段。

但是，当你将 HOC 应用于组件时，原始组件将使用容器组件进行包装。这意味着新组件没有原始组件的任何静态方法。

```javascript
// 定义静态函数
WrappedComponent.staticMethod = function() {/*...*/}
// 现在使用 HOC
const EnhancedComponent = enhance(WrappedComponent);

// 增强组件没有 staticMethod
typeof EnhancedComponent.staticMethod === 'undefined' // true
```

为了解决这个问题，你可以在返回之前把这些方法拷贝到容器组件上：

```javascript
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  // 必须准确知道应该拷贝哪些方法
  Enhance.staticMethod = WrappedComponent.staticMethod;
  return Enhance;
}
```

但要这样做，你需要知道哪些方法应该被拷贝。你可以使用 [hoist-non-react-statics](https://github.com/mridgway/hoist-non-react-statics) 自动拷贝所有非 React 静态方法:

```javascript
import hoistNonReactStatic from 'hoist-non-react-statics';
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  hoistNonReactStatic(Enhance, WrappedComponent);
  return Enhance;
}
```

除了导出组件，另一个可行的方案是再额外导出这个静态方法。

```javascript
// 使用这种方式代替...
MyComponent.someFunction = someFunction;
export default MyComponent;

// ...单独导出该方法...
export { someFunction };

// ...并在要使用的组件中，import 它们
import MyComponent, { someFunction } from './MyComponent.js';
```

##### Refs 不会被传递

虽然高阶组件的约定是将所有 props 传递给被包装组件，但这对于 refs 并不适用。那是因为 `ref` 实际上并不是一个 prop，就像 `key` 一样，它是由 React 专门处理的。如果将 ref 添加到 HOC 的返回组件中，则 ref 引用指向容器组件，而不是被包装组件。

这个问题的解决方案是通过使用 `React.forwardRef` API（React 16.3 中引入）。[前往 ref 转发章节了解更多](https://reactjs.bootcss.com/docs/forwarding-refs.html)。

### 深入JSX

见 核心概念 - JSX - 深入JSX

### Portals

Portal 提供了一种将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案。一个 portal 的典型用例是当父组件有 `overflow: hidden` 或 `z-index` 样式时，但你需要子组件能够在视觉上“跳出”其容器。例如，对话框、悬浮卡以及提示框。

**createPortal()** 函数的第一个参数 child 是任何可渲染的 React 子元素，例如一个元素，字符串或 fragment。第二个参数 container 是一个 DOM 元素。

```javascript
render() {
  return ReactDOM.createPortal(
    this.props.children,
    container
  )
}
```

> 注意:
>
> 当在使用 portal 时, 记住[管理键盘焦点](https://reactjs.bootcss.com/docs/accessibility.html#programmatically-managing-focus)就变得尤为重要。
>
> 对于模态对话框，通过遵循 [WAI-ARIA 模态开发实践](https://www.w3.org/TR/wai-aria-practices-1.1/#dialog_modal)，来确保每个人都能够运用它。

#### 通过 Portal 进行事件冒泡

尽管 portal 可以被放置在 DOM 树中的任何地方，但在任何其他方面，其行为和普通的 React 子节点行为一致。由于 portal 仍存在于 React 树， 且与 DOM 树 中的位置无关，那么无论其子节点是否是 portal，像 context 这样的功能特性都是不变的。

这包含事件冒泡。一个从 portal 内部触发的事件会一直冒泡至包含 React 树的祖先，即便这些元素并不是 DOM 树 中的祖先。假设存在如下 HTML 结构：

```javascript
<html>
  <body>
    <div id="app-root"></div>
    <div id="modal-root"></div>
  </body>
</html>
```

在 `#app-root` 里的 `Parent` 组件能够捕获到未被捕获的从兄弟节点 `#modal-root` 冒泡上来的事件。

```javascript
// 在父组件里捕获一个来自 portal 冒泡上来的事件，使之能够在开发时具有不完全依赖于 portal 的更为灵活的抽象。例如，如果你在渲染一个 <Modal /> 组件，无论其是否采用 portal 实现，父组件都能够捕获其事件。

// 在 DOM 中有两个容器是兄弟级 （siblings）
const appRoot = document.getElementById('app-root');
const modalRoot = document.getElementById('modal-root');

class Modal extends React.Component {
  constructor(props) {
    super(props);
    this.el = document.createElement('div');
  }

  componentDidMount() {
    // 在 Modal 的所有子元素被挂载后，这个 portal 元素会被嵌入到 DOM 树中，意味着子元素将被挂载到一个分离的 DOM 节点中。如果要求子组件在挂载时可以立刻接入 DOM 树（例如衡量一个 DOM 节点，或在后代节点中使用 ‘autoFocus’），则需添加 state 到 Modal 中，仅当 Modal 被插入 DOM 树中才能渲染子元素。
    modalRoot.appendChild(this.el);
  }

  componentWillUnmount() {
    modalRoot.removeChild(this.el);
  }

  render() {
    return ReactDOM.createPortal(      
      this.props.children,
      this.el    
    ) 
  }
}

class Parent extends React.Component {
  constructor(props) {
    super(props)
    this.state = { clicks: 0 }
  }

  handleClick = () => {    
    // 当子元素里的按钮被点击时，这个将会被触发更新父元素的 state，即使这个按钮在 DOM 中不是直接关联的后代    
    this.setState( state => ({ clicks: state.clicks + 1 }))
  }
  render() {
    return (
      <div onClick={this.handleClick}>        
      	<p>Number of clicks: {this.state.clicks}</p>
        <p>
          Open up the browser DevTools to observe that the button is not a child of the div with the onClick handler.
        </p>
        <Modal>          
        	<Child />        
        </Modal>
			</div>
    )
  }
}

function Child() {
  // 这个按钮的点击事件会冒泡到父元素  // 因为这里没有定义 'onClick' 属性  return (
    <div className="modal">
      <button>Click</button>    
  	</div>
  )
}

ReactDOM.render(<Parent />, appRoot);
```

### Profiler

`Profiler` 测量渲染一个 React 应用多久渲染一次以及渲染一次的“代价”。 它的目的是识别出应用中渲染较慢的部分，或是可以使用[类似 memoization 优化](https://reactjs.bootcss.com/docs/hooks-faq.html#how-to-memoize-calculations)的部分，并从相关优化中获益。 

> 注意：
>
> Profiling 增加了额外的开支，所以**它在[生产构建](https://reactjs.bootcss.com/docs/optimizing-performance.html#use-the-production-build)中会被禁用**，仅在需要时使用它。
>
> 为了将 profiling 功能加入生产环境中，React 提供了使 profiling 可用的特殊的生产构建环境。 从 [fb.me/react-profiling](https://fb.me/react-profiling)了解更多关于如何使用这个构建环境的信息。

`Profiler` 能添加在 React 树中的任何地方来测量树中这部分渲染所带来的开销。 

它需要两个prop：一个是 `id`(string)；另一个是当组件树中的组件“提交”更新的时候被React调用的回调函数 `onRender`(function)。

多个 `Profiler` 组件能测量应用中的不同部分；嵌套使用 `Profiler` 组件来测量相同一个子树下的不同组件。

```javascript
render(
  <App>
    <Profiler id="Panel" onRender={callback}>
      <Panel {...props}>
        <Profiler id="Content" onRender={callback}>
          <Content {...props} />
        </Profiler>
        <Profiler id="PreviewPane" onRender={callback}>
          <PreviewPane {...props} />
        </Profiler>
      </Panel>
    </Profiler>
  </App>
)
```

`Profiler` 需要一个 `onRender` 函数作为参数。 React 会在 profile 包含的组件树中任何组件 “提交” 一个更新的时候调用这个函数。 它的参数描述了渲染了什么和花费了多久。

```javascript
function onRenderCallback(
  id, // 发生提交的 Profiler 树的 “id”
  phase, // "mount" （如果组件树刚加载） 或者 "update" （如果它重渲染了）之一
  actualDuration, // 本次更新 committed 花费的渲染时间
  baseDuration, // 估计不使用 memoization 的情况下渲染整颗子树需要的时间
  startTime, // 本次更新中 React 开始渲染的时间
  commitTime, // 本次更新中 React committed 的时间
  interactions // 属于本次更新的 interactions 的集合
) {
  // 合计或记录渲染时间。。。
}
```

让我们来仔细研究一下各个 prop:

- **`id: string`** - 发生提交的 `Profiler` 树的 `id`。 如果有多个 profiler，它能用来分辨树的哪一部分发生了“提交”。
- **`phase: "mount" | "update"`** - 判断是组件树的第一次装载引起的重渲染，还是由 props、state 或是 hooks 改变引起的重渲染。
- **`actualDuration: number`** - 本次更新在渲染 `Profiler` 和它的子代上花费的时间。 这个数值表明使用 memoization 之后能表现得多好。（例如 [`React.memo`](https://reactjs.bootcss.com/docs/react-api.html#reactmemo)，[`useMemo`](https://reactjs.bootcss.com/docs/hooks-reference.html#usememo)，[`shouldComponentUpdate`](https://reactjs.bootcss.com/docs/hooks-faq.html#how-do-i-implement-shouldcomponentupdate)）。 理想情况下，由于子代只会因特定的 prop 改变而重渲染，因此这个值应该在第一次装载之后显著下降。
- **`baseDuration: number`** - 在 `Profiler` 树中最近一次每一个组件 `render` 的持续时间。 这个值估计了最差的渲染时间。（例如当它是第一次加载或者组件树没有使用 memoization）。
- **`startTime: number`** - 本次更新中 React 开始渲染的时间戳。
- **`commitTime: number`** - 本次更新中 React commit 阶段结束的时间戳。 在一次 commit 中这个值在所有的 profiler 之间是共享的，可以将它们按需分组。
- **`interactions: Set`** - 当更新被制定时，[“interactions”](https://fb.me/react-interaction-tracing) 的集合会被追踪。（例如当 `render` 或者 `setState` 被调用时）。

### Render Props

是指一种在 React 组件之间使用一个值为函数的 prop 共享代码的简单技术。简单来说，就是给组件添加一个值为函数的属性，这个函数可以在组件渲染（render）的时候调用，给原有组件“注入”其它组件的代码（ 为 render 的属性传入组件，来控制渲染）。本质上是将一个方法传递给组件。传递一个渲染逻辑给子组件，带有 render props 的组件，并不是自己的实现的渲染逻辑，而是通过 props 传递实现的渲染逻辑，而这个渲染逻辑是由 render props 方法完成的，例如：

```javascript
<DataProvider render={data => ( <h1>Hello {data.target}</h1> )}/>
```

具体例子：

```javascript
class MouseTracker extends React.Component {
  constructor(props) {
    super(props);
  }
  state = { x: 0, y: 0 }
  handleMouse = event => { this.setState({ x: event.clientX, y: event.clientY }) }
  
  render() {
    return (
      <div style={/* 一些样式 */} onMouseMove={this.handleMouse}> 
        <div> {this.state.x} & {this.state.y} </div>
        {this.props.render(this.state)} // 重点，通过 props 传递过来的 render 方法然后传递一个  this.state 。具体见下面
      </div>
    )
  }
}

/* 使用 */
class App3 extends Component {
  render() {
    return (
      <MouseTracker render={data => <Cat {...data} />} /> // 不一定要叫 render ，这里只是为了表明含义。
    )
  }
}
```

```javascript
// 如果我现在需要一个组件，其中需要拥有控制显示鼠标的逻辑，只不过我只需要获取 clientX 和 clientY 显示的逻辑可能是不同的，比如一个组件 <Car>
class Cat extends Component {
  constructor(props) {
    super(props);
  }
  render() {
    const {x,y} = this.props;
    return <div style={/* 一些样式 */}/>
  }
}
```

```javascript
// 我们可以将 <Cat> 组件放在 <MouseTracker> 组件中使用：
class MouseTracker extends React.Component {
  ....
  render() {
    return (
      <div style={/* 一些样式 */} onMouseMove={this.handleMouse}> 
        <div> {this.state.x} & {this.state.y} </div>
        <Cat {...this.state} /> // 使用，但这样子就写死了，没有复用性
				// {this.props.render(this.state)} // 这样就是动态的，可复用的
      </div>
    )
  }
}
```

### PropTypes 类型检查

#### 基本使用

当传入的 `prop` 值类型不正确时，JavaScript 控制台将会显示警告。出于性能方面的考虑，`propTypes` 仅在开发模式下进行检查。

```javascript
import PropTypes from 'prop-types' // 导入

// 设置 PropTypes
Greeting.propTypes = {
  name: PropTypes.string
}

// 指定 props 的默认值
Greeting.defaultProps = {
  name: 'Stranger'
}

class Greeting extends React.Component {
  render() { return ( <h1>{this.props.name}</h1> ) }
}
```

```javascript
// 在类组件中，可以有另一种写法
class Abc extends Component {
  static propTypes = {
    ...
  },
	
	static defaultProps = {
    ...
  }
}
```

#### 不同类型

```javascript
import PropTypes from 'prop-types';

MyComponent.propTypes = {
  // 你可以将属性声明为 JS 原生类型，默认情况下
  // 这些属性都是可选的。
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // 任何可被渲染的元素（包括数字、字符串、元素或数组）
  // (或 Fragment) 也包含这些类型。
  optionalNode: PropTypes.node,

  // 一个 React 元素。
  optionalElement: PropTypes.element,

  // 一个 React 元素类型（即，MyComponent）。
  optionalElementType: PropTypes.elementType,

  // 你也可以声明 prop 为类的实例，这里使用
  // JS 的 instanceof 操作符。
  optionalMessage: PropTypes.instanceOf(Message),

  // 你可以让你的 prop 只能是特定的值，指定它为
  // 枚举类型。
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),

  // 一个对象可以是几种类型中的任意一个类型
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ]),

  // 可以指定一个数组由某一类型的元素组成
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // 可以指定一个对象由某一类型的值组成
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // 可以指定一个对象由特定的类型值组成
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  }),
  
  // An object with warnings on extra properties
  optionalObjectWithStrictShape: PropTypes.exact({
    name: PropTypes.string,
    quantity: PropTypes.number
  }),   

  // 你可以在任何 PropTypes 属性后面加上 `isRequired` ，确保
  // 这个 prop 没有被提供时，会打印警告信息。
  requiredFunc: PropTypes.func.isRequired,

  // 任意类型的数据
  requiredAny: PropTypes.any.isRequired,

  // 你可以指定一个自定义验证器。它在验证失败时应返回一个 Error 对象。
  // 请不要使用 `console.warn` 或抛出异常，因为这在 `onOfType` 中不会起作用。
  customProp: function(props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' + propName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  },

  // 你也可以提供一个自定义的 `arrayOf` 或 `objectOf` 验证器。
  // 它应该在验证失败时返回一个 Error 对象。
  // 验证器将验证数组或对象中的每个值。验证器的前两个参数
  // 第一个是数组或对象本身
  // 第二个是他们当前的键。
  customArrayProp: PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' + propFullName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  })
};
```

#### 限制单个元素

你可以通过 `PropTypes.element` 来确保传递给组件的 children 中只包含一个元素。

```javascript
import PropTypes from 'prop-types';

class MyComponent extends React.Component {
  render() {
    // 这必须只有一个元素，否则控制台会打印警告。
    const children = this.props.children;
    return ( <div> {children} </div> );
  }
}

MyComponent.propTypes = {
  children: PropTypes.element.isRequired
};
```

## API

















## Hook

Hook 是 React 16.8 的新增特性，可以让你使用 state 以及其他的 React 特性，但不必须将其转换为 class 组件。

Hook 是一些可以让你在函数组件里“钩入” React state 及生命周期等特性的函数。Hook 不能在 class 组件中使用 —— 这使得你不使用 class 也能使用 React。React 内置了一些像 `useState` 这样的 Hook。你也可以创建你自己的 Hook 来复用不同组件之间的状态逻辑。 

###  useState

通过在函数组件里调用它来给组件添加一些内部 state，React 会在重复渲染时记住它当前的值，并且提供最新的值给我们的函数，我们可以通过调用 `setCount` 来更新当前的 `count`。 

```javascript
import React, { useState } from 'react'; // 引入 React 中的 useState Hook。它让我们在函数组件中存储内部 state。

function Example() {
  const [count, setCount] = useState(0); // 在 Example 组件内部，我们通过调用 useState Hook 声明了一个新的 state 变量。它返回一对值给到我们命名的变量上。我们把变量命名为 count，因为它存储的是点击次数。我们通过传 0 作为 useState 唯一的参数来将其初始化为 0。第二个返回的值本身就是一个函数。它让我们可以更新 count 的值，所以我们叫它 setCount。
  // 可以多次调用，以及初始化不同的值
	const [fruit, setFruit] = useState('banana')
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }])
  
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}> // 当用户点击按钮后，我们传递一个新的值给 setCount。React 会重新渲染 Example 组件，并把最新的 count 传给它。
        Click me
      </button>
    </div>
  );
}
```

`useState`  允许你在 React 函数组件中添加 state 的 Hook，返回一对值：1.**当前**状态state；2.一个更新state的函数。这就是我们写 `const [count, setCount] = useState()` 的原因，这与 class 里面 `this.state.count` 和 `this.setState` 类似，唯一区别就是你需要成对的获取它们。

①当前状态state。调用 `useState` 时，它定义一个 “state 变量”，上述例子这个变量叫 `count`， 也可以取其他任何名字，比如 `banana`。这是一种在函数调用时保存变量的方式，它与 class 里面的 `this.state` 提供的功能完全相同。一般来说，在函数退出后变量就会”消失”，而 state 中的变量会被 React 保留。

```javascript
// 如果初始 state 需要通过复杂计算获得，则可以传入一个函数，在函数中计算并返回初始的 state，此函数只在初始渲染时被调用，后续渲染时会被忽略：
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props); // 计算获得需要的值
  return initialState; // 将获得的值作为 state 返回
});
```

②更新state的函数。可以在事件处理函数中或其他一些地方调用这个函数，类似 class 组件的 `this.setState`，但是不会把新的 state 和旧的 state 进行合并。 

```javascript
// 1.你可以用函数式的 setState 结合展开运算符来达到合并更新对象的效果。
// 2.也可以使用 Object.assign
setState(prevState => { return {...prevState, ...updatedValues}; });
// 3.使用 useReducer，适用于管理包含多个子值的 state 对象。
```

如果新的 state 需要通过使用先前的 state 计算得出，那么可以将函数传递给 setState。该函数将接收先前的 state，并返回一个更新后的值。

```javascript
function Counter({initialCount}) {
  const [count, setCount] = useState(initialCount);
  return (
    <>
      ...
      <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
      <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
    </>
  );
}
```

跳过 state 更新：如果你的更新函数返回值与当前 state 完全相同，则随后的重渲染会被完全跳过。因此，也可以传入与当前相同的 state 值，以主动跳过这一次对于子组件的渲染及 effect 的执行。（React 使用 [`Object.is` 比较算法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description) 来比较 state） 。这是内部默认的机制，以提高性能。

需要注意的是，React 可能仍需要在跳过渲染前渲染该组件。不过由于 React 不会对组件树的“深层”节点进行不必要的渲染，所以大可不必担心。如果你在渲染期间执行了高开销的计算，则可以使用 `useMemo` 来进行优化。

③参数。`useState` 唯一的参数就是初始 state，但可以直接传值，也可以传递一个函数（见①）。不同于 `this.state`，这里的 state 不一定要是一个对象，并且这个初始 state 参数只有在第一次渲染时会被用到。 如果我们想要在 state 中存储两个不同的变量，需调用 `useState()` 两次并传入不同的值。

### useEffect

你之前可能已经在 React 组件中执行过数据获取、订阅或者手动修改过 DOM。我们统一把这些操作称为“副作用”，或者简称为“作用”。在函数组件主体内（这里指在 React 渲染阶段）改变 DOM、添加订阅、设置定时器、记录日志以及执行其他包含副作用的操作都是不被允许的，因为这可能会产生莫名其妙的 bug 并破坏 UI 的一致性。  使用 `useEffect` 完成副作用操作。赋值给 `useEffect` 的函数会在组件渲染到屏幕之后执行。 

#### 无需清除的 effect

`useEffect`  接收一个包含命令式、且可能有副作用代码的函数 ，用于给函数组件增加操作副作用的能力，它跟 class 组件中的 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 具有相同的用途，只不过被合并成了一个 API。

```javascript
// 基本使用
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);
  // useEffect() 相当于 componentDidMount 和 componentDidUpdate，因此，例如需要在两个周期函数中都执行一次的方法，可以只在 useEffect() 中只执行一次即可。
  useEffect(() => {
    document.title = `You clicked ${count} times`; // 直接取用定义的 count
  });
  useEffect(() => { ... }); // 可以多次使用 useEffect()

  return (
    <div>
      <p>You clicked {count} times</p> // count 不需要用 this，已经保存在函数的作用域中
      <button onClick={() => setCount(count + 1)}> // setCount 不需要 this，已经保存在函数的作用域中
    		Click me </button>
    </div>
  );
}
```

当你调用 `useEffect` 时，就是在告诉 React 在完成对 DOM 的更改后运行你的“副作用”函数，React 会保存你传递的函数（即 “effect”）， 并且在执行 DOM 更新之后调用它 ，在上述 effect 中，我们设置了 document 的 title 属性 。

将 `useEffect` 放在组件内部，使其是在组件内声明的，所以可以直接访问到组件的 props 和 state。我们不需要特殊的 API 来读取它 —— 它已经保存在函数作用域中。Hook 使用了 JavaScript 的闭包机制，而不用在 JavaScript 已经提供了解决方案的情况下，还引入特定的 React API。 

默认情况下，React 会在**每次**渲染结束后（包括第一次）调用副作用函数， 但你可以选择让它在某个值改变时才执行。 

> 与 `componentDidMount` 或 `componentDidUpdate` 不同， 在浏览器完成布局与绘制**之后**，传给 `useEffect` 的函数会**延迟**调用（但会保证在任何新的渲染前执行，将在组件更新前刷新上一轮渲染的 effect ）。这使得它不会阻塞浏览器更新屏幕，所以适用于许多常见的副作用场景，比如设置订阅和事件处理等情况，因此不应在函数中执行阻塞浏览器更新屏幕的操作。大多数情况下，effect 不需要同步地执行。但在个别情况下（例如测量布局），如果需要其同步执行，可以使用 [`useLayoutEffect`](https://reactjs.bootcss.com/docs/hooks-reference.html#uselayouteffect) Hook。

```javascript
// 可以传递第二个参数，仅在第二个参数发生变化时，才调用方法
useEffect(() => {
  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
}, [props.friend.id]); // 仅在 props.friend.id 发生变化时，重新订阅
```

#### 需要清除的 effect

副作用函数还可以通过返回一个函数来指定如何“清除”副作用。有一些副作用是需要清除的。例如订阅外部数据源。这种情况下，清除工作是非常重要的，可以防止引起内存泄露。

```javascript
// 对于一般的Class组件，可能需要在不同的生命周期函数中进行 创建订阅 和 取消订阅
componentDidMount() {
  ChatAPI.subscribeToFriendStatus( ... ) // 创建订阅
}
componentWillUnmount() {
  ChatAPI.unsubscribeFromFriendStatus( ... ) // 取消订阅
}
```

通过使用 Hook，你可以把组件内相关的副作用组织在一起（例如创建订阅及取消订阅），而不要把它们拆分到不同的生命周期函数里。 

```javascript
import React, { useState, useEffect } from 'react';

// React 会在组件销毁时取消对 ChatAPI 的订阅，然后在后续渲染时重新执行副作用函数。
function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
    	setIsOnline(status.isOnline);
  	}
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => { // 将需要进行清除操作的函数 return 即可，会在组件卸载前执行（详细见下面）。
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
	
  // 可以声明多个 useEffect()。effect 将按照声明的顺序依次调用。
	useEffect( ... )
  
  if (isOnline === null) { return 'Loading...'; }
  return isOnline ? 'Online' : 'Offline';
}
```

每个 effect 都可以 return 一个函数，这是 effect 可选的清除机制，用来将例如添加订阅和删除订阅的逻辑放在一起，如果不需要清除操作，则不必 return 一个函数。React 会在组件卸载前执行清除操作。正如之前学到的，effect 在每次渲染的时候都会执行，因此，如果组件多次渲染，则清除函数也会多次执行，**在执行下一个 effect 之前，上一个 effect 就已被清除**，而不是只在卸载组件的时候执行一次，也意味着组件的每一次更新都会创建新的订阅。

这保证了数据的一致性，避免了在 class 组件中因为没有处理更新逻辑而导致常见的 bug，例如，当组件的旧数据已经显示在页面上，但此时数据更新，页面上展示的数据并没有更新，我们需要手动取消订阅旧的数据，并订阅新的数据。

但是，频繁执行 effect 会造成性能问题，或有时我们并不需要每次组件更新时都创建新的订阅，在 class 组件中，我们可以通过在 `componentDidUpdate` 中添加对 `prevProps` 或 `prevState` 的比较逻辑解决：

```javascript
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) { // 如果相同，则跳过
    document.title = `You clicked ${this.state.count} times`;
  }
}
```

这是很常见的需求，所以它被内置到了 `useEffect` 的 Hook API 中。如果某些特定值在两次重渲染之间没有发生变化，可以通知 React **跳过**对 effect 的调用，只要传递数组作为 `useEffect`的**第二个可选参数**即可：

```javascript
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // 仅在 count 更改时更新，如果前后两次 count 的值相同，则跳过这个 effect

// 对于有清除操作的 effect 同样适用
useEffect(() => {
  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
}, [props.friend.id]); // 仅在 props.friend.id 发生变化时，重新创建订阅
```

> 如果要使用此优化方式，请确保数组中包含了**所有外部作用域中会随时间变化并且在 effect 中使用的变量**，否则你的代码会引用到先前渲染中的旧变量。参阅文档，了解更多关于[如何处理函数](https://reactjs.bootcss.com/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)以及[数组频繁变化时的措施](https://reactjs.bootcss.com/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often)内容。
>
> 如果想执行只运行一次的 effect（仅在组件挂载和卸载时执行），可以传递一个空数组（`[]`）作为第二个参数。这就告诉 React 你的 effect 不依赖于 props 或 state 中的任何值，所以它永远都不需要重复执行。这并不属于特殊情况 —— 它依然遵循依赖数组的工作方式。
>
> 如果传入了一个空数组（`[]`），effect 内部的 props 和 state 就会一直拥有其初始值。尽管传入 `[]` 作为第二个参数更接近大家更熟悉的 `componentDidMount` 和 `componentWillUnmount` 思维模式，但我们有[更好的](https://reactjs.bootcss.com/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)[方式](https://reactjs.bootcss.com/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often)来避免过于频繁的重复调用 effect。除此之外，请记得 React 会等待浏览器完成画面渲染之后才会延迟调用 `useEffect`，因此会使得额外操作很方便。
>
> 推荐启用 [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation) 中的 [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) 规则，能在添加错误依赖时发出警告并给出修复建议。

### 其他 Hook

#### useContext

见 高级指引 - Context 。

**把如下代码与 Context.Provider 放在一起**

```javascript
const themes = {
  light: { foreground: "#000000", background: "#eeeeee" },
  dark: { foreground: "#ffffff", background: "#222222" }
};

const ThemeContext = React.createContext(themes.light);
function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}
function Toolbar(props) {
  return ( <div> <ThemedButton /> </div> );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);  
  return (    
    <button style={{ background: theme.background, color: theme.foreground }}>      
    	I am styled by theme context!    
    </button>  
	);
}
```

#### useReducer

```javascript
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

`useState` 的替代方案。它接收一个形如 `(state, action) => newState` 的 reducer，并返回当前的 state 以及与其配套的 `dispatch` 方法。

在某些场景下，`useReducer` 会比 `useState` 更适用，例如 state包含多个子值，或者下一个 state 依赖于之前的 state 等。并且，使用 `useReducer` 还能给那些会触发深更新的组件做性能优化，因为[你可以向子组件传递 `dispatch` 而不是回调函数](https://reactjs.bootcss.com/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down) 。

```javascript
const initialState = {count: 0};
function reducer(state, action) {
  switch (action.type) {
    case 'increment': return {count: state.count + 1};
    case 'decrement': return {count: state.count - 1};
    default: throw new Error();
  }
}
function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState); // 初始化 useReducer
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement'})}> - </button>
      <button onClick={() => dispatch({type: 'increment'})}> + </button>
    </>
  );
}
```

> React 会确保 `dispatch` 函数的标识是稳定的，并且不会在组件重新渲染时改变。这就是为什么可以安全地从 `useEffect` 或 `useCallback` 的依赖列表中省略 `dispatch`。

**指定初始 state：**

有两种不同初始化 `useReducer` state 的方式，可以根据使用场景选择其中一种。

方式一：将初始 state 作为第二个参数传入 `useReducer` 是最简单的方法：

```javascript
const [state, dispatch] = useReducer(
	reducer,
	{count: initialCount}
);
```

> React 不使用 `state = initialState` 这一由 Redux 推广开来的参数约定。有时候初始值依赖于 props，因此需要在调用 Hook 时指定。如果你特别喜欢上述的参数约定，可以通过调用 `useReducer(reducer, undefined, reducer)` 来模拟 Redux 的行为，但我们不鼓励你这么做。

方式二：惰性地创建初始 state。需要将 `init` 函数作为 `useReducer` 的第三个参数传入，这样初始 state 将被设置为 `init(initialArg)`。这么做可以将用于计算 state 的逻辑提取到 reducer 外部，这也为将来对重置 state 的 action 做处理提供了便利：

```javascript
function reducer(state, action) {
  switch (action.type) {
    case 'increment': return {count: state.count + 1};
    case 'decrement': return {count: state.count - 1};
    case 'reset': return init(action.payload);    
    default: throw new Error();
  }
}
function init(initialCount) { // init函数
  return {count: initialCount}; 
}
function Counter({initialCount}) {
  const [state, dispatch] = useReducer(reducer, initialCount, init); // 传入init
  return (
    <>
      Count: {state.count}
<button onClick={()=>dispatch({type: 'reset', payload: initialCount})}>Reset</button>
      <button onClick={() => dispatch({type: 'decrement'})}> - </button>
      <button onClick={() => dispatch({type: 'increment'})}> + </button>
    </>
  );
}
```

如果 Reducer Hook 的返回值与当前 state 相同，React 将跳过子组件的渲染及副作用的执行。（React 使用 [`Object.is` 比较算法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description) 来比较 state。）

需要注意的是，React 可能仍需要在跳过渲染前再次渲染该组件。不过由于 React 不会对组件树的“深层”节点进行不必要的渲染，所以大可不必担心。如果你在渲染期间执行了高开销的计算，则可以使用 `useMemo` 来进行优化。

#### useCallback

```javascript
const memoizedCallback = useCallback(
  () => { doSomething(a, b); }, // 内联回调函数
  [a, b], // 依赖项数组
);
```

返回一个 [memoized](https://en.wikipedia.org/wiki/Memoization) 回调函数。

把内联回调函数及依赖项数组作为参数传入 `useCallback`，返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如 `shouldComponentUpdate`）的子组件时，它将非常有用。

`useCallback(fn, deps)` 相当于 `useMemo(() => fn, deps)`。

> 依赖项数组不会作为参数传给回调函数。虽然从概念上来说它表现为：所有回调函数中引用的值都应该出现在依赖项数组中。未来编译器会更加智能，届时自动创建数组将成为可能。
>
> 我们推荐启用 [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation) 中的 [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) 规则。此规则会在添加错误依赖时发出警告并给出修复建议。

#### useMemo

```javascript
const memoizedValue = useMemo(
  () => computeExpensiveValue(a, b), 
  [a, b]
);
```

返回一个 [memoized](https://en.wikipedia.org/wiki/Memoization) 值。

把“创建”函数和依赖项数组作为参数传入 `useMemo`，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进行高开销的计算。

记住，传入 `useMemo` 的函数会在渲染期间执行。请不要在这个函数内部执行与渲染无关的操作，诸如副作用这类的操作属于 `useEffect` 的适用范畴，而不是 `useMemo`。

如果没有提供依赖项数组，`useMemo` 在每次渲染时都会计算新的值。

**你可以把 `useMemo` 作为性能优化的手段，但不要把它当成语义上的保证。**将来，React 可能会选择“遗忘”以前的一些 memoized 值，并在下次渲染时重新计算它们，比如为离屏组件释放内存。先编写在没有 `useMemo` 的情况下也可以执行的代码 —— 之后再在你的代码中添加 `useMemo`，以达到优化性能的目的。

> 依赖项数组不会作为参数传给“创建”函数。虽然从概念上来说它表现为：所有“创建”函数中引用的值都应该出现在依赖项数组中。未来编译器会更加智能，届时自动创建数组将成为可能。
>
> 我们推荐启用 [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation) 中的 [`exhaustive-deps`](https://github.com/facebook/react/issues/14920) 规则。此规则会在添加错误依赖时发出警告并给出修复建议。

#### useRef

```javascript
const refContainer = useRef(initialValue);
```

`useRef` 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数（`initialValue`）。返回的 ref 对象在组件的整个生命周期内保持不变。

一个常见的用例便是命令式地访问子组件：

```javascript
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    inputEl.current.focus(); // `current` 指向已挂载到 DOM 上的文本输入元素
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}> Focus the input </button>
    </>
  );
}
```

本质上，`useRef` 就像是可以在其 `.current` 属性中保存一个可变值的“盒子”。

你应该熟悉 ref 这一种[访问 DOM](https://reactjs.bootcss.com/docs/refs-and-the-dom.html) 的主要方式。如果你将 ref 对象以 `  <div ref={myRef} />  ` 形式传入组件，则无论该节点如何改变，React 都会将 ref 对象的 `.current` 属性设置为相应的 DOM 节点。

然而，`useRef()` 比 `ref` 属性更有用。它可以[很方便地保存任何可变值](https://reactjs.bootcss.com/docs/hooks-faq.html#is-there-something-like-instance-variables)，其类似于在 class 中使用实例字段的方式。

这是因为它创建的是一个普通 Javascript 对象。而 `useRef()` 和自建一个 `{current: ...}` 对象的唯一区别是，`useRef` 会在每次渲染时返回同一个 ref 对象。

请记住，当 ref 对象内容发生变化时，`useRef` 并*不会* 通知你。变更 `.current` 属性不会引发组件重新渲染。如果想要在 React 绑定或解绑 DOM 节点的 ref 时运行某些代码，则需要使用[回调 ref](https://reactjs.bootcss.com/docs/hooks-faq.html#how-can-i-measure-a-dom-node) 来实现。

#### useImperativeHandle

```javascript
useImperativeHandle(ref, createHandle, [deps])
```

`useImperativeHandle` 可以让你在使用 `ref` 时自定义暴露给父组件的实例值。在大多数情况下，应当避免使用 ref 这样的命令式代码。`useImperativeHandle` 应当与 [`forwardRef`](https://reactjs.bootcss.com/docs/react-api.html#reactforwardref) 一起使用：

```javascript
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({ focus: ()=>{inputRef.current.focus()} }) );
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```

在本例中，渲染 ` <FancyInput ref={inputRef} /> ` 的父组件可以调用 `inputRef.current.focus()`。

#### useLayoutEffect

其函数签名与 `useEffect` 相同，但它会在所有的 DOM 变更之后同步调用 effect。可以使用它来读取 DOM 布局并同步触发重渲染。在浏览器执行绘制之前，`useLayoutEffect` 内部的更新计划将被同步刷新。

尽可能使用标准的 `useEffect` 以避免阻塞视觉更新。

> 如果你正在将代码从 class 组件迁移到使用 Hook 的函数组件，则需要注意 `useLayoutEffect` 与 `componentDidMount`、`componentDidUpdate` 的调用阶段是一样的。但是，我们推荐你**一开始先用 `useEffect`**，只有当它出问题的时候再尝试使用 `useLayoutEffect`。
>
> 如果你使用服务端渲染，请记住，*无论* `useLayoutEffect` *还是* `useEffect` 都无法在 Javascript 代码加载完成之前执行。这就是为什么在服务端渲染组件中引入 `useLayoutEffect` 代码时会触发 React 告警。解决这个问题，需要将代码逻辑移至 `useEffect` 中（如果首次渲染不需要这段逻辑的情况下），或是将该组件延迟到客户端渲染完成后再显示（如果直到 `useLayoutEffect`执行之前 HTML 都显示错乱的情况下）。
>
> 若要从服务端渲染的 HTML 中排除依赖布局 effect 的组件，可以通过使用 `showChild && ` 进行条件渲染，并使用 `useEffect(() => { setShowChild(true); }, [])` 延迟展示组件。这样，在客户端渲染完成之前，UI 就不会像之前那样显示错乱了。

#### useDebugValue

```javascript
useDebugValue(value)
```

`useDebugValue` 可用于在 React 开发者工具中显示自定义 hook 的标签。

例如，[“自定义 Hook”](https://reactjs.bootcss.com/docs/hooks-custom.html) 章节中描述的名为 `useFriendStatus` 的自定义 Hook：

```javascript
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  // ...
  
  // 在开发者工具中的这个 Hook 旁边显示标签
  // e.g. "FriendStatus: Online"
  useDebugValue(isOnline ? 'Online' : 'Offline');
  return isOnline;
}
```

> 我们不推荐你向每个自定义 Hook 添加 debug 值。当它作为共享库的一部分时才最有价值。

**延迟格式化 debug 值**

在某些情况下，格式化值的显示可能是一项开销很大的操作。除非需要检查 Hook，否则没有必要这么做。

因此，`useDebugValue` 接受一个格式化函数作为可选的第二个参数。该函数只有在 Hook 被检查时才会被调用。它接受 debug 值作为参数，并且会返回一个格式化的显示值。

例如，一个返回 `Date` 值的自定义 Hook 可以通过格式化函数来避免不必要的 `toDateString` 函数调用：

```javascript
useDebugValue(date, date => date.toDateString());
```

### 使用规则

Hook 就是 JavaScript 函数，但是使用它们会有两个额外的规则：

- 只能在**函数最外层**调用 Hook。不要在循环、条件判断或者子函数中调用。确保总是在 React 函数的最顶层调用，就能确保 Hook 在每一次渲染中都按照同样的顺序被调用，这让 React 能够在多次的 `useState` 和 `useEffect` 调用之间保持 hook 状态的正确。 
- 只能在**函数组件 或 自定义的Hook**中调用 Hook。不要在普通的 JavaScript 函数中调用。遵循此规则，确保组件的状态逻辑在代码中清晰可见。 

可以使用 [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks) 的 ESLint 插件来强制执行这两条规则：

```javascript
npm install eslint-plugin-react-hooks --save-dev

//  ESLint 配置
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error", // 检查 Hook 的规则
    "react-hooks/exhaustive-deps": "warn" // 检查 effect 的依赖
  }
}
```

如果在单个组件中使用多个 State Hook 或 Effect Hook，例如：

```javascript
function Form() {
  // 1. Use the name state variable
  const [name, setName] = useState('Mary');

  // 2. Use an effect for persisting the form
  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });

  // 3. Use the surname state variable
  const [surname, setSurname] = useState('Poppins');

  // 4. Use an effect for updating the title
  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });

  // ...
}
```

则是**靠  Hook 调用的顺序 来确定 哪个 state 对应哪个 `useState`** 。只要 Hook 的调用顺序在多次渲染之间保持一致，React 就能正确地将内部 state 和对应的 Hook 进行关联。 

示例中，Hook 的调用顺序在每次渲染中都是相同的，所以它能够正常工作： 

```javascript
// 首次渲染
useState('Mary')           // 1. 使用 'Mary' 初始化变量名为 name 的 state
useEffect(persistForm)     // 2. 添加 effect 以保存 form 操作
useState('Poppins')        // 3. 使用 'Poppins' 初始化变量名为 surname 的 state
useEffect(updateTitle)     // 4. 添加 effect 以更新标题

// 二次渲染
useState('Mary')           // 1. 读取变量名为 name 的 state（参数被忽略）
useEffect(persistForm)     // 2. 替换保存 form 的 effect
useState('Poppins')        // 3. 读取变量名为 surname 的 state（参数被忽略）
useEffect(updateTitle)     // 4. 替换更新标题的 effect

// ...
```

```javascript
// 但如果将一个 Hook (例如 persistForm effect) 调用放到一个条件语句中：
// 🔴 在条件语句中使用 Hook 违反第一条规则
if (name !== '') {
  useEffect(function persistForm() { localStorage.setItem('formData', name); });
}

// 在第一次渲染中 name !== '' 这个条件值为 true，所以我们会执行这个 Hook。但是下一次渲染时我们可能清空了表单，表达式值变为 false。此时的渲染会跳过该 Hook，Hook 的调用顺序发生了改变：
useState('Mary')           // 1. 读取变量名为 name 的 state（参数被忽略）
// useEffect(persistForm)  // 🔴 此 Hook 被忽略！
useState('Poppins')        // 🔴 2 （之前为 3）。读取变量名为 surname 的 state 失败
useEffect(updateTitle)     // 🔴 3 （之前为 4）。替换更新标题的 effect 失败
// React 不知道第二个 useState 的 Hook 应该返回什么。React 会以为在该组件中第二个 Hook 的调用像上次的渲染一样，对应的是 persistForm 的 effect，但并非如此。从这里开始，后面的 Hook 调用都被提前执行，导致 bug 的产生。
// 这就是为什么 Hook 需要在我们组件的最顶层调用。如果我们想要有条件地执行一个 effect，可以将判断放到 Hook 的内部：
useEffect(function persistForm() {
  // 👍 将条件判断放置在 effect 中
  if (name !== '') { localStorage.setItem('formData', name); }
});
```

### 自定义 Hook

通过自定义 Hook，可以将组件逻辑提取到可重用的函数中。

**自定义 Hook 是一个函数，其名称以 “`use`” 开头，函数内部可以调用其他的 Hook（例如 useState() 及 useEffect()）以完成某些功能。** 

```javascript
// 自定义 Hook
import React, { useState, useEffect } from 'react'
const useMousePosition = () => {
    const [position, setPosition] = useState({x: 0, y: 0 })
    useEffect(() => {
        const updateMouse = e => { setPosition({ x: e.clientX, y: e.clientY }) }
        document.addEventListener('mousemove', updateMouse)
        return () => { document.removeEventListener('mousemove', updateMouse) }
    })
    return position
}
export default useMousePosition

// 使用
import React from 'react'
import useMousePosition from './useMousePosition'
function App() {
    const position = useMousePosition()
    return (
        <div> x: {position.x} & y: {position.y} </div>
    )
}
```

与组件中一致，请确保只在自定义 Hook 的顶层无条件地调用其他 Hook。

与组件不同的是，自定义 Hook 不需要具有特殊的标识，可以自由决定参数是什么，以及返回什么。换句话说，它就像一个正常的函数。

-  **自定义 Hook 必须以 “`use`” 开头**。否则React无法判断某个函数是否包含对其内部 Hook 的调用，将无法自动检查 Hook 是否违反了 Hook 的规则。 
-  **在两个组件中使用相同的 Hook 不会共享 state **。自定义 Hook 是一种重用*状态逻辑*的机制(例如设置为订阅并存储当前值)，所以每次使用自定义 Hook 时，其中的所有 state 和副作用都是完全隔离的。 
- **自定义 Hook 如何获取独立的 state？**每次*调用* Hook，它都会获取独立的 state。正如我们可以在一个组件中多次调用 `useState`和 `useEffect`，它们是完全独立的。

由于 Hook 本身就是函数，因此我们可以在它们之间传递信息。

```javascript
const [xxx, yyy] = useState(1);
const zzz = useFriendStatus(xxx);
```



















## 虚拟DOM 与 Diff 算法

![1590487466485](React.assets/1590487466485.png)

## 路由（未完）

















# redux & react-redux

## redux

https://www.redux.org.cn/

### 概念

#### 核心概念

state用来存放数据，但是并没有setter方法，因此不能修改它。想要更新state，就必须要用action。最终，为了把action和state串起来，开发一些函数，即reducer，来接收action返回新的state。

#### 三大原则

**1.单一数据源**

整个应用的 state 被存储在一颗 object tree 中，并且这个 object tree 只存在于唯一一个 store 中。

**2.State 是只读的**

唯一改变 state 的方法就是 action，action 是一个用于描述已发生事件的普通对象。

**3.使用纯函数来执行修改**

为了描述 action 如何改变 state tree，需要编写 reducers。

#### 适用场景

Redux 是一个独立专门用于做状态管理的JS库（不是react插件库，意味着可以用在其他的框架中，但和 React 是最配套的）。

作用：**集中式管理 React 应用中多个组件共享的状态**

以下这些情况，都不需要使用 Redux：

- 用户的使用方式非常简单
- 用户之间没有协作
- 不需要与服务器大量交互，也没有使用 WebSocket
- 视图层（View）只从单一来源获取数据

以下这些情况，才是需要 Redux的（多交互、多数据源）：

- 用户的使用方式复杂
- 不同身份的用户有不同的使用方式（比如普通用户和管理员）
- 多个用户之间可以协作
- 与服务器大量交互，或者使用了WebSocket
- View要从多个来源获取数据

从组件角度看，如果你的应用有以下场景，可以考虑使用 Redux：

- 某个组件的状态，需要共享
- 某个状态需要在任何地方都可以拿到
- 一个组件需要改变全局状态
- 一个组件需要改变另一个组件的状态

#### 工作流程

![1589082521962](React.assets/1589082521962.png)

首先，用户发出 Action。

```javascript
store.dispatch(action);
```

然后，Store 自动调用 Reducer，并且传入两个参数：当前 State 和收到的 Action。Reducer 会返回新的 State 

```javascript
let nextState = todoApp(previousState, action);
```

State 一旦有变化，Store 就会调用监听函数。

```javascript
store.subscribe(listener); // 设置监听函数
```

`listener`可以通过`store.getState()`得到当前状态。如果使用的是 React，这时可以触发重新渲染 View。

```javascript
function listerner() {
  let newState = store.getState();
  component.setState(newState);   
}
```

#### 单向数据流

https://www.redux.org.cn/docs/basics/DataFlow.html

**严格的单向数据流**是 Redux 架构的设计核心。 

这意味着应用中所有的数据都遵循相同的生命周期，这样可以让应用变得更加可预测且容易理解。同时也鼓励做数据范式化，这样可以避免使用多个且独立的无法相互引用的重复数据。

如果这些理由还不足以令你信服，读一下 [动机](https://www.redux.org.cn/docs/introduction/Motivation.html) 和 [Flux 案例](https://medium.com/@dan_abramov/the-case-for-flux-379b7d1982c6)，这里面有更加详细的单向数据流优势分析。虽然 Redux 并不是严格意义上的 [Flux](https://www.redux.org.cn/docs/introduction/Relation to Other Libraries.md)，但它们有共同的设计思想。

Redux 应用中数据的生命周期遵循下面 4 个步骤： 

**1.调用 `store.dispatch(action)` **

**2.Redux store 调用传入的 reducer 函数**

**3.根 reducer 应该把多个子 reducer 输出合并成一个单一的 state 树。** 

**4.Redux store 保存了根 reducer 返回的完整 state 树。** 

#### 异步数据流

默认情况下，[`createStore()`](https://www.redux.org.cn/docs/api/createStore.html) 所创建的 Redux store 没有使用 [middleware](https://www.redux.org.cn/docs/advanced/Middleware.html)，所以只支持 [同步数据流](https://www.redux.org.cn/docs/basics/DataFlow.html)。

你可以使用 [`applyMiddleware()`](https://www.redux.org.cn/docs/api/applyMiddleware.html) 来增强 [`createStore()`](https://www.redux.org.cn/docs/api/createStore.html)。虽然这不是必须的，但是它可以帮助你[用简便的方式来描述异步的 action](https://www.redux.org.cn/docs/advanced/AsyncActions.html)。

像 [redux-thunk](https://github.com/gaearon/redux-thunk) 或 [redux-promise](https://github.com/acdlite/redux-promise) 这样支持异步的 middleware 都包装了 store 的 [`dispatch()`](https://www.redux.org.cn/docs/api/Store.html#dispatch) 方法，以此来让你 dispatch 一些除了 action 以外的其他内容，例如：函数或者 Promise。你所使用的任何 middleware 都可以以自己的方式解析你 dispatch 的任何内容，并继续传递 actions 给下一个 middleware。比如，支持 Promise 的 middleware 能够拦截 Promise，然后为每个 Promise 异步地 dispatch 一对 begin/end actions。

当 middleware 链中的最后一个 middleware 开始 dispatch action 时，这个 action 必须是一个普通对象。这是 [同步式的 Redux 数据流](https://www.redux.org.cn/docs/basics/DataFlow.html) 开始的地方（译注：这里应该是指，你可以使用任意多异步的 middleware 去做你想做的事情，但是需要使用普通对象作为最后一个被 dispatch 的 action ，来将处理流程带回同步方式）。

接着可以查看 [异步示例的完整源码](https://www.redux.org.cn/docs/advanced/ExampleRedditAPI.html)。

### 核心 API 

#### store

redux库最核心的管理对象，将 state、action 与 reducer 联系在一起的对象。

```javascript
import {createStore} from 'redux'
import reducer from './reducers'

const store = createStore(reducer)
```

核心方法：	store.getState()					 获取状态state

​         			  store.dispatch(action)		  分发事件action，触发 reducers 的调用，更新state

​         			  store.subscribe(listener)	 订阅监听，根据新产生的状态state时 自动调用 重新渲染组件 更新界面

保存数据的地方，可以把它看成一个容器，整个应用只能有一个Store。再次强调一下 **Redux 应用只有一个单一的 store**。当需要拆分数据处理逻辑时，你应该使用 reducer 组合 而不是创建多个 store。 

Redux 提供 `createStore` 函数，用来生成 Store。  

```javascript
import { createStore } from 'redux';

// createStore函数接受另一个函数作为参数，返回新生成的 Store 对象。
const store = createStore(fn);
```

##### store.getState()

`Store` 对象包含所有数据。如果想得到某个时点的数据，就要对 Store 生成快照。这种时点的数据集合，就叫做 State。 

```javascript
// 当前时刻的 State，可以通过store.getState()拿到。
import { createStore } from 'redux';
const store = createStore(fn);

const state = store.getState();
```

Redux 规定， 一个 State 对应一个 View。只要 State 相同，View 就相同。你知道 State，就知道 View 是什么样，反之亦然。 

##### store.dispatch()

`store.dispatch()` 是 View 发出 Action 的唯一方法。 

```javascript
import { createStore } from 'redux';
const store = createStore(fn);

// store.dispatch接受一个 Action 对象作为参数，将它发送出去
store.dispatch({
  type: 'ADD_TODO',
  payload: 'Learn Redux'
});

// 结合 Action Creator，这段代码可以改写如下
store.dispatch(addTodo('Learn Redux'));
```

##### store.subscribe()

使用`store.subscribe`方法设置监听函数，一旦 State 发生变化，就自动执行这个函数。 

```javascript
import { createStore } from 'redux';
const store = createStore(reducer);

store.subscribe(listener);
```

只要把 View 的更新函数（对于 React 项目，就是组件的`render`方法或`setState`方法）放入`listen`，就会实现 View 的自动渲染。

`store.subscribe`方法返回一个函数，调用这个函数就可以解除监听。

```javascript
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
);

unsubscribe();
```

##### createStore()

作用：创建包含指定reducer的store对象

```javascript
import {createStore} from 'redux'
import counter from './reducers/counter'
const store = createStore(counter)
```

还可以接受第二个参数（可选），设置 State 初始状态。这通常是服务器给出的。这对开发同构应用时非常有用，服务器端 redux 应用的 state 结构可以与客户端保持一致, 那么客户端可以将从网络接收到的服务端 state 直接用于本地数据初始化。 

```javascript
let store = createStore(todoApp, window.STATE_FROM_SERVER)
// window.STATE_FROM_SERVER就是整个应用的状态初始值。注意，如果提供了这个参数，它会覆盖 Reducer 函数的默认初始值。
```

下面是`createStore`方法的一个简单实现，可以了解一下 Store 是怎么生成的。 

```javascript
const createStore = (reducer) => {
  let state;
  let listeners = [];

  const getState = () => state;

  const dispatch = (action) => {
    state = reducer(state, action);
    listeners.forEach(listener => listener());
  };

  const subscribe = (listener) => {
    listeners.push(listener);
    return () => {
      listeners = listeners.filter(l => l !== listener);
    }
  };

  dispatch({});

  return { getState, dispatch, subscribe };
};
```

#### action

action 是 store 数据的唯一来源。一般来说，通过 [`store.dispatch()`](https://www.redux.org.cn/docs/api/Store.html#dispatch) 将 action 传到 store。

Action 本质上是一个描述“**发生了什么**”的  JavaScript 对象。其中的 `type` 属性是必须的，帮助 reducer 识别不同的 action，表示将要执行的动作。 建议使用 string 而不是 [符号（Symbols）](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol) 作为 action type ，因为 string 是可序列化的，并且使用符号会使记录和重演变得困难。 

注意：actions 只是描述了*有事情发生了*这一事实，并没有描述应用如何更新 state。  

State 的变化，会导致 View 的变化。但是，用户接触不到 State，只能接触到 View。所以，State 的变化必须是 View 导致的。Action 就是 View 发出的通知，表示 State 应该要发生变化了。 

```javascript
包含2个方面的属性：	type	标识属性，值为字符串，必要属性，唯一
									xxx	  数据属性，值类型任意，可选属性

const action = {
  // Action 的名称是ADD_TODO，它携带的信息是字符串Learn Redux。
  type: 'ADD_TODO', // 多数情况下，type 会被定义成字符串常量
  payload: 'Learn Redux' // 可选属性
};

/* 除了 type 字段外，action 对象的结构完全由你自己决定。但应该尽量减少在 action 中传递的数据
	 参照 Flux 标准 Action：https://github.com/redux-utilities/flux-standard-action
	 获取关于如何构造 action 的建议。*/	 
// 例如：
{
  type: TOGGLE_TODO,
  index: 5 // 可选属性
}
```

##### 定义常量

可以这样理解，Action 描述当前发生的事情。改变 State 的唯一办法，就是使用 Action。它会运送数据到 Store。 

当应用规模较大时，建议使用单独的模块或文件来存放 action。

```javascript
import { ADD_TODO, REMOVE_TODO } from '../action-types'
```

使用单独的模块或文件来定义 action type 常量并不是必须的，甚至根本不需要定义。对于小应用来说，使用字符串做 action type 更方便些。不过，在大型应用中把它们显式地定义成常量还是利大于弊的，参照 [减少样板代码](https://www.redux.org.cn/docs/recipes/ReducingBoilerplate.html) 获取更多保持代码简洁的实践经验，简单来说，使用常量具有以下好处：

- 帮助维护命名一致性，因为所有的 action type 汇总在同一位置。
- 有时，在开发一个新功能之前你想看到所有现存的 actions 。而你的团队里可能已经有人添加了你所需要的action，而你并不知道。
- Action types 列表在 Pull Request 中能查到所有添加，删除，修改的记录。这能帮助团队中的所有人及时追踪新功能的范围与实现。
- 如果你在 import 一个 Action 常量的时候拼写错了，你会得到 `undefined` 。在 dispatch 这个 action 的时候，Redux 会立即抛出这个错误，你也会马上发现错误。

##### 创建函数 Action Creator

创建 action 的工厂函数，就是生成 action 的方法。 

View 要发送多少种消息，就会有多少种 Action。如果都手写，会很麻烦。可以定义一个函数来生成 Action，这个函数就叫 Action Creator。 

在 Redux 中的 action 创建函数只是简单的返回一个 action， 这样做将使 action 创建函数更容易被移植和测试：

```javascript
function addTodo(text) {
  return {
    type: ADD_TODO,
    text: ...
  }
}
```

```javascript
// Redux 中只需把 action 创建函数的结果传给 dispatch() 方法即可发起一次 dispatch 过程。
dispatch(addTodo(text))
dispatch(completeTodo(index))

// 或者创建一个 被绑定的 action 创建函数 来自动 dispatch：
const boundAddTodo = text => dispatch(addTodo(text))
const boundCompleteTodo = index => dispatch(completeTodo(index))

// 然后直接调用它们：
boundAddTodo(text);
boundCompleteTodo(index);
```

store 里能直接通过 [`store.dispatch()`](https://www.redux.org.cn/docs/api/Store.html#dispatch) 调用 `dispatch()` 方法，但是多数情况下你会使用 [react-redux](http://github.com/gaearon/react-redux) 提供的 `connect()` 帮助器来调用。[`bindActionCreators()`](https://www.redux.org.cn/docs/api/bindActionCreators.html) 可以自动把多个 action 创建函数 绑定到 `dispatch()`方法上。 

##### 异步 action

当调用异步 API 时，有两个非常关键的时刻：**发起请求的时刻**，**接收到响应的时刻**（也可能是超时）。 

这两个时刻都可能会更改应用的 state。为此，你需要 dispatch 普通的同步 action。一般情况下，每个 API 请求都需要 dispatch 至少三种 action： 

- **一种通知 reducer 请求开始的 action。**

  对于这种 action，reducer 可能会切换一下 state 中的 `isFetching` 标记。以此来告诉 UI 来显示加载界面。

- **一种通知 reducer 请求成功的 action。**

  对于这种 action，reducer 可能会把接收到的新数据合并到 state 中，并重置 `isFetching`。UI 则会隐藏加载界面，并显示接收到的数据。

- **一种通知 reducer 请求失败的 action。**

  对于这种 action，reducer 可能会重置 `isFetching`。另外，有些 reducer 会保存这些失败信息，并在 UI 里显示出来。

为了区分这三种 action，可能在 action 里添加一个专门的 `status` 字段作为标记位：

```javascript
{ type: 'FETCH_POSTS', status: 'error', error: 'Oops' }
{ type: 'FETCH_POSTS', status: 'success', response: { ... } }
```

或者为它们定义不同的 type： 

```javascript{ type: &#39;FETCH_POSTS_FAILURE&#39;, error: &#39;Oops&#39; }
{ type: 'FETCH_POSTS_FAILURE', error: 'Oops' }
{ type: 'FETCH_POSTS_SUCCESS', response: { ... } }
```

采用哪一种方式完全取决于你或你的团队。使用多个 type 会降低犯错误的机率，但是如果你使用像 [redux-actions](https://github.com/acdlite/redux-actions) 这类的辅助库来生成 action 创建函数和 reducer 的话，这就完全不是问题了。无论使用哪种约定，一定要在整个应用中保持统一。 

**实现异步：见 redux 异步（异步中间件）**





 

#### reducer

reducer 是一个纯函数（纯函数见下方），指定了应用状态的变化如何响应 action 并发送到 store，接收旧的 state 和 action，返回新的 state。 

注意：

1.**不要修改原来的 state**，新的DOM树需要和旧的DOM树进行对比，改变原来的状态会改变旧的DOM树。 

2.**在 `default` 情况下返回旧的 `state`。**遇到未知的 action 时，一定要返回旧的 `state`。 

3.保持 reducer 纯净非常重要。 **永远不要**在 reducer 里做这些操作：

- 修改传入参数；
- 执行有副作用的操作，如 API 请求和路由跳转；
- 调用非纯函数，如 `Date.now()` 或 `Math.random()`。

**reducer 是纯函数。它仅仅用于计算下一个 state。它应该是完全可预测的：只要传入参数相同，返回计算得到的下一个 state 就一定相同。没有特殊情况、没有副作用，没有 API 请求、没有变量修改，单纯执行计算。** 

Store 收到 Action 以后，必须给出一个新的 State，这样 View 才会发生变化。这种 State 的计算过程就叫做 Reducer。 

```javascript
const reducer = function (state, action) {
  // ...
  return new_state;
};
```

整个应用的初始状态，可以作为 State 的默认值。下面是一个实际的例子。 

```javascript
export default function counter(state = 0, action) {
  switch (action.type) {
    // 返回的都是新的状态
    case 'INCREMENT':
      return state + action.data
    case 'DECREMENT':
      return state - action.data
    default:
      return state
  }
}

const state = reducer(1, {
  type: 'INCREMENT',
  payload: 2
});
```

实际应用中，Reducer 函数不用像上面这样手动调用，`store.dispatch`方法会触发 Reducer 的自动执行。为此，Store 需要知道 Reducer 函数，做法就是在生成 Store 的时候，将 Reducer 传入`createStore`方法。 

```javascript
import { createStore } from 'redux';
const store = createStore(reducer);

// createStore 接受 Reducer 作为参数，生成一个新的 Store。以后每当`store.dispatch`发送过来一个新的 Action，就会自动调用 Reducer，得到新的 State。 
```

##### reducer的拆分

我们可以使用 Redux 提供的 `combineReducers()` 方法将多个小的 reducer 组合成一个 rootReducer，而每个小的 reducer 只关心自己负责的 `action.type`，因为在整个应用中，有些 state 之间是互不关联的，因此我们可以拆分 reducer，不同的 reducer 处理不同的 `action.type`。 

```javascript
// 假设现在有两个逻辑：一个计数器和一个 TODO。很明显这两个逻辑之间没有任何的联系，因此我们可以拆分成两个 reducer，然后再组合起来传给 createStore()：

// 计数器 reducer
function counter(state = 0, action) {
  switch(action.type) {
    case 'INCREMENT':
      return state + 1;
    case 'DECREMENT':
      return state -1;
    default:
      return state;
  }
}

// TODO reducer
function todos(state = [], action) {
  switch(action.type) {
    case 'ADD_TODO':
      return state.concat([action.text]);
    default :
      return state;
  }
}

// 组合成一个 rootReducer
const rootReducer = combineReducers({
  counter,
  todos
  // ES6写法，相当于 counter: counter, todos: todos
});

// 根据 rootReducer 创建 store
const store = createStore(rootReducer)

console.log(store.getState()); // 打印输出
```

结果：

 ![https://user-images.githubusercontent.com/22176164/39660627-c5f88b60-5075-11e8-93bc-b7ecc37fd49b.png](React.assets/4173916937-5b0bea2787a53_articlex.png) 

可以看到，`store.getState()` 中的 *key* 变成了子 reducer 的函数名，达到了拆分之间没有联系的 state 的目的。而 *key* 变成了子 reducer 的函数名的原因是，在组合 reducer 时的写法：

```
// 组合成一个 rootReducer
const rootReducer = combineReducers({
  counter,
  todos
});
```

这里传入给 `combineReducers()` 方法的是一个对象，这里使用了 ES6 的语法，即当对象的 *key* 和 *value*同名时，可以省略 *key*，上面的写法相当于：

```
const rootReducer = combineReducers({
-  counter,
+  counter: counter,
-  todos
+  todos: todos
});
```

当然，*key* 是可以随意修改的，只是修改后，对于 state 的 *key* 也会有所不同罢了。需要记住的是 `combineReducers()` 方法的参数是一个对象，对象的 *value* 是需要组合的 reducer，对象的*key* 对应的是 state 中的 *key*，返回的是一个组合后的 reducer 函数，可以像普通 reducer 一样传给 `createStore()` 方法创建 store。

######  combineReducer 的简单实现

总之，`combineReducers()`做的就是产生一个整体的 Reducer 函数。该函数根据 State 的 key 去执行相应的子 Reducer，并将返回结果合并成一个大的 State 对象。

以下为 ` combineReducer ` 的简单实现：

```
const combineReducers = reducers => {
  return (state = {}, action) => {
    return Object.keys(reducers).reduce(
      (nextState, key) => {
        nextState[key] = reducers[key](state[key], action);
        return nextState;
      },
      {} 
    );
  };
};
```

##### 纯函数

Reducer 函数最重要的特征是，它是一个纯函数。也就是说，只要是同样的输入，必定得到同样的输出。

纯函数是函数式编程的概念，必须遵守以下一些约束：

- 不得改写参数
- 不能调用系统 I/O 的API
- 不能调用`Date.now()`或者`Math.random()`等不纯的方法，因为每次会得到不一样的结果

由于 Reducer 是纯函数，就可以保证同样的State，必定得到同样的 View。但也正因为这一点，Reducer 函数里面不能改变 State，必须返回一个全新的对象，请参考下面的写法：

```javascript
// State 是一个对象
function reducer(state, action) {
  return Object.assign({}, state, { thingToChange });
  // 或者
  return { ...state, ...newState };
}

// State 是一个数组
function reducer(state, action) {
  return [...state, newItem];
}
```

最好把 State 对象设成只读。你没法改变它，要得到新的 State，唯一办法就是生成一个新对象。这样的好处是，任何时候，与某个 View 对应的 State 总是一个不变的对象。 

### redux 异步（异步中间件）

异步 action 见 redux/核心API/action/异步action

redux默认是不能进行异步处理的。Action 发出以后，Reducer 立即算出 State，这叫做同步；Action 发出以后，过一段时间再执行 Reducer，这就是异步。 

中间件就是一个函数，对`store.dispatch`方法进行了改造，在发出 Action 和执行 Reducer 这两步之间，添加了其他功能。 使得 action创建函数除了能返回对象外，还可以返回函数来实现异步操作。

```
npm install --save redux-thunk
```

```javascript
// 在 store.js 文件中启用中间件
import {createStore, applyMiddleware} from 'redux'
import thunk from 'redux-thunk'  // redux异步中间件

export default createStore(
  reducers,
  applyMiddleware(thunk) // 启用异步中间件，否则action中默认只能return一个对象
)
```

```javascript
// actions.js 中添加异步的函数
...
export const incrementAsync = number => {
  // 这里把 dispatch 方法通过参数的形式传给函数，以此来让它自己也能 dispatch action。
  return dispatch => { // 返回的是一个函数
    // 异步代码
    setTimeout(() => {
      dispatch(increment(number)) // 分发action
    }, 1000)
  }
}
```

```javascript
// 在 connect 中添加异步方法
export default connect(
  state => ({count: state.counter}),
  {increment, decrement, incrementAsync}
)(Counter)
```

```javascript
// 在业务逻辑的jsx中使用异步方法
```

详情见 *使用示例*

#### applyMiddlewares()

https://www.redux.org.cn/docs/advanced/Middleware.html

`applyMiddlewares`  是 Redux 的原生方法，作用是将所有中间件组成一个数组，依次执行。下面是它的源码。

```javascript
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    var store = createStore(reducer, preloadedState, enhancer);
    var dispatch = store.dispatch;
    var chain = [];

    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    };
    chain = middlewares.map(middleware => middleware(middlewareAPI));
    dispatch = compose(...chain)(store.dispatch);

    return {...store, dispatch}
  }
}
```

上面代码中，所有中间件被放进了一个数组`chain`，然后嵌套执行，最后执行`store.dispatch`。可以看到，中间件内部（`middlewareAPI`）可以拿到`getState`和`dispatch`这两个方法。 

#### 其他的方法

[Thunk middleware](https://github.com/gaearon/redux-thunk) 并不是 Redux 处理异步 action 的唯一方式：

- 你可以使用 [redux-promise](https://github.com/acdlite/redux-promise) 或者 [redux-promise-middleware](https://github.com/pburtchaell/redux-promise-middleware) 来 dispatch Promise 来替代函数。
- 你可以使用 [redux-observable](https://github.com/redux-observable/redux-observable) 来 dispatch Observable。
- 你可以使用 [redux-saga](https://github.com/yelouafi/redux-saga/) 中间件来创建更加复杂的异步 action。
- 你可以使用 [redux-pack](https://github.com/lelandrichardson/redux-pack) 中间件 dispatch 基于 Promise 的异步 Action。
- 你甚至可以写一个自定义的 middleware 来描述 API 请求，就像这个 [真实场景的案例](https://www.redux.org.cn/docs/introduction/Examples.html#real-world) 中的做法一样。



### redux 调试工具

安装 浏览器插件 redux-devtools

下载工具依赖包：

```javascript
npm install --save-dev redux-devtools-extension
```

使用：

```js
// store.js

import { composeWithDevTools } from 'redux-devtools-extension'

const store = createStore(
  counter,
  composeWithDevTools(applyMiddleware(thunk)) // 进行包裹
)
```

## react-redux

```javascript
npm install --save react-redux
```

作用：一个react插件库，简化在 react 中编写 redux 的代码。

react-redux 将所有组件分为两大类：UI 组件负责 UI 的呈现，容器组件负责管理数据和逻辑。 

**UI组件**

- 只负责 UI 的呈现，不带有任何业务逻辑

- 没有状态（即不使用`this.state`这个变量）

- 所有数据都由参数（`this.props`）提供

- 不使用任何 Redux 的 API

- 一般保存在 components 文件夹下

- ```javascript
  // 例如 
  const Title = value => <h1>{value}</h1>;
  ```

- 因为不含有状态，UI 组件又称为"纯组件"，即它纯函数一样，纯粹由参数决定它的值。 

**容器组件**

- 负责管理数据和业务逻辑，不负责UI的呈现
- 带有内部状态
- 使用 Redux 的 API
- 一般保存在 containers 文件夹下

如果一个组件既有 UI 又有业务逻辑，将它拆分成下面的结构：外面是一个容器组件，里面包了一个UI 组件。前者负责与外部的通信，将数据传给后者，由后者渲染出视图。

React-Redux 规定，所有的 UI 组件都由用户提供，容器组件则是由 React-Redux 自动生成。也就是说，用户负责视觉层，状态管理则是全部交给它。

### 相关 API

#### `<Provider>`

`connect`方法生成容器组件以后，需要让容器组件拿到`state`对象，才能生成 UI 组件的参数。

一种解决方法是将`state`对象作为参数，传入容器组件。但是，这样做比较麻烦，尤其是容器组件可能在很深的层级，一级级将`state`传下去就很麻烦。

React-Redux 提供`Provider`组件，可以让容器组件拿到`state`。

```javascript
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import todoApp from './reducers'
import App from './components/App'

let store = createStore(todoApp);

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

#### connect()

用于包装 UI 组件生成容器组件

（1）输入逻辑：外部的数据（即`state`对象）如何转换为 UI 组件的参数

（2）输出逻辑：用户发出的动作如何变为 Action 对象，从 UI 组件传出去。

```javascript
import { connect } from 'react-redux'

connect(
	mapStateToprops, // 负责输入逻辑，即将state映射到 UI 组件的参数（props）
	mapDispatchToProps // 负责输出逻辑，即将用户对 UI 组件的操作映射成 Action
)(Counter)
```

#### mapStateToprops()

建立一个从（外部的）`state`对象到（UI 组件的）`props`对象的映射关系。

作为函数，`mapStateToProps`执行后应该返回一个对象，里面的每一个键值对就是一个映射。 

```javascript
const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}
```

`mapStateToProps`是一个函数，它接受`state`作为参数，返回一个对象。这个对象有一个`todos`属性，代表 UI 组件的同名参数，后面的`getVisibleTodos`也是一个函数，可以从`state`算出 `todos` 的值。 

```javascript
// 下面就是getVisibleTodos的一个例子，用来算出todos
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
    default:
      throw new Error('Unknown filter: ' + filter)
  }
}
```

`mapStateToProps`会订阅 Store，每当`state`更新的时候，就会自动执行，重新计算 UI 组件的参数，从而触发 UI 组件的重新渲染。

`mapStateToProps`的第一个参数总是`state`对象，还可以使用第二个参数，代表容器组件的`props`对象。

```javascript
// 容器组件的代码
//    <FilterLink filter="SHOW_ALL">
//      All
//    </FilterLink>

const mapStateToProps = (state, ownProps) => {
  return {
    active: ownProps.filter === state.visibilityFilter
  }
}
```

使用`ownProps`作为参数后，如果容器组件的参数发生变化，也会引发 UI 组件重新渲染。

`connect`方法可以省略`mapStateToProps`参数，那样的话，UI 组件就不会订阅Store，就是说 Store 的更新不会引起 UI 组件的更新。

#### mapDispatchToProps()

`mapDispatchToProps`是`connect`函数的第二个参数，用来建立 UI 组件的参数到`store.dispatch`方法的映射。也就是说，它定义了哪些用户的操作应该当作 Action，传给 Store。它可以是一个函数，也可以是一个对象。

如果`mapDispatchToProps`是一个函数，会得到`dispatch`和`ownProps`（容器组件的`props`对象）两个参数。

```javascript
const mapDispatchToProps = (
  dispatch,
  ownProps
) => {
  return {
    onClick: () => {
      dispatch({
        type: 'SET_VISIBILITY_FILTER',
        filter: ownProps.filter
      });
    }
  };
}
```

从上面代码可以看到，`mapDispatchToProps`作为函数，应该返回一个对象，该对象的每个键值对都是一个映射，定义了 UI 组件的参数怎样发出 Action。

如果`mapDispatchToProps`是一个对象，它的每个键名也是对应 UI 组件的同名参数，键值应该是一个函数，会被当作 Action creator ，返回的 Action 会由 Redux 自动发出。举例来说，上面的`mapDispatchToProps`写成对象就是下面这样。

```javascript
const mapDispatchToProps = {
  onClick: (filter) => {
    type: 'SET_VISIBILITY_FILTER',
    filter: filter
  };
}
```

## 使用示例（逐渐完善）

### 不用 redux

```javascript
// index.js
import React from 'react'
import ReactDOM from 'react-dom'
import App from './components/app'

ReactDOM.render(<App/>, document.getElementById('root'))
```

```jsx
// components/app.jsx
import React, {Component} from 'react'

export default class App extends Component {
  state = {
    count: 0
  }

	// 4个方法
  increment = () => {
    const num = this.refs.numSelect.value*1
    const count = this.state.count + num
    this.setState({count})
  }
  decrement = () => {
    const num = this.refs.numSelect.value*1
    const count = this.state.count - num
    this.setState({count})
  }
  incrementIfOdd = () => {
    let count = this.state.count
    if(count%2==1) {
      const num = this.refs.numSelect.value*1
      count += num
      this.setState({count})
    }
  }
  incrementAsync = () => {
    setTimeout(() => {
      const num = this.refs.numSelect.value*1
      const count = this.state.count + num
      this.setState({count})
    }, 1000)
  }

  render () {
    return (
      <div>
        <p>
          click {this.state} times {' '}
        </p>
        <select ref="numSelect">
          <option value="1">1</option>
          <option value="2">2</option>
          <option value="3">3</option>
        </select>{' '}
        <button onClick={this.increment}>+</button>{' '}
        <button onClick={this.decrement}>-</button>{' '}
        <button onClick={this.incrementIfOdd}>increment if odd</button>{' '}
        <button onClick={this.incrementAsync}>increment async</button>
      </div>
    )
  }
}
```

### 使用 redux

```javascript
文件目录结构：*表示文件夹
src
	*components
  	app.jsx
	*redux
		action-types.js
		actions.js
		reducers.js
		store.js
	index.js
```

```js
// index.js
import React from 'react'
import ReactDOM from 'react-dom'

import App from './components/app'
import store from './redux/store'

// 定义渲染根组件标签的函数，抽成函数，方便下面的复用
const render = () => {
  ReactDOM.render(<App store={store}/>, document.getElementById('root')) // 传入store
}
// 初始化渲染
render()

// 注册(订阅)监听, 一旦状态发生改变, 自动重新渲染
store.subscribe(render)
```

```jsx
// components/app.jsx
/*
应用组件
 */
import React, {Component} from 'react'
import PropTypes from 'prop-types'
import * as actions from '../redux/actions'

export default class App extends Component {
  static propTypes = {
    store: PropTypes.object.isRequired,
  }
	
	/* 4个方法 */
  increment = () => {
    const number = this.refs.numSelect.value * 1
    this.props.store.dispatch(actions.increment(number)) // 调用store的方法更新状态
  }
  decrement = () => {
    const number = this.refs.numSelect.value * 1
    this.props.store.dispatch(actions.decrement(number)) // 调用store的方法更新状态
  }
  incrementIfOdd = () => {
    const number = this.refs.numSelect.value * 1
    let count = this.props.store.getState()
    if (count % 2 === 1) {
      this.props.store.dispatch(actions.increment(number)) // 调用store的方法更新状态
    }
  }
  incrementAsync = () => {
    const number = this.refs.numSelect.value * 1
    setTimeout(() => {
      this.props.store.dispatch(actions.increment(number)) // 调用store的方法更新状态
    }, 1000)
  }

  render() {
    const count = this.props.store.getState() // <App store={store}/> 传入了 store
    
    return (
      <div>
        <p>
          click {this.props.store.getState()} times {' '}
        </p>
        <select ref="numSelect">
          <option value="1">1</option>
          <option value="2">2</option>
          <option value="3">3</option>
        </select>{' '}
        <button onClick={this.increment}>+</button>
        {' '}
        <button onClick={this.decrement}>-</button>
        {' '}
        <button onClick={this.incrementIfOdd}>increment if odd</button>
        {' '}
        <button onClick={this.incrementAsync}>increment async</button>
      </div>
    )
  }
}
```

```javascript
// redux/action-type.js
/*
Action对象的type常量名称模块
 */
export const INCREMENT = 'increment'
export const DECREMENT = 'decrement'
```

```javascript
// redux/actions.js
/*
action creator模块
 */
import {INCREMENT, DECREMENT} from './action-types'

export const increment = number => ({type: INCREMENT, number})
export const decrement = number => ({type: DECREMENT, number})
```

```javascript
// redux/reducers.js
/*
根据老的state和指定action, 处理返回一个新的state
 */
import {INCREMENT, DECREMENT} from './action-types'

export function counter(state = 0, action) {
  switch (action.type) {
    case INCREMENT:
      return state + action.number
    case DECREMENT:
      return state - action.number
    default:
      return state
  }
}
```

```javascript
// redux/store.js
import {createStore} from 'redux'
import {counter} from './reducers'

// 生成一个store对象
const store = createStore(counter) // 内部会第一次调用reducer函数得到初始state=0

export default store
```

### 使用 react-redux

```javascript
文件目录结构：*表示文件夹
src
	*components
  	counter.jsx
	*containers
		app,jsx
	*redux
		action-types.js
		actions.js
		reducers.js
		store.js
	index.js
```

上述 redux 的版本中，redux 与 react 组件的代码耦合度太高，编码不够简洁。

redux/action-types.js  不变

redux/actions.js  不变

redux/reducers.js  不变

redux/store.js 不变

```javascript
// index.js
import React from 'react'
import ReactDOM from 'react-dom'

import {Provider} from 'react-redux' // 导入 Provider

import App from './components/app'
import store from './redux/store'

ReactDOM.render(( 
  // 不再渲染 <App>，使用 <Provider> 包裹
  <Provider store={store}> // 参数也放在 <Provider> 中传递
    <App/>
  </Provider>
), document.getElementById('root'))

/* 不需要再订阅监听
// 注册(订阅)监听, 一旦状态发生改变, 自动重新渲染
store.subscribe(render)
*/
```

```jsx
// containers/app.jsx
/*
包含counter组件的容器组件
 */
import React from 'react'
import {connect} from 'react-redux' // 引入连接函数
import {increment, decrement} from '../redux/actions' // 引入action函数

import Counter from '../components/counter' // 将逻辑代码抽出

// 向外暴露连接App组件的包装组件
export default connect(
  state => ({count: state}), 
  {increment, decrement} 
)(Counter)
// count 与 counter.jsx中声明的属性一致
// {increment, decrement} 相当于 {increment: increment, decrement: decrement} 第一个 increment 与 counter.jsx 中声明的属性一致，第二个 increment 与 action 中的方法名一致
```

```jsx
// components/counter.jsx
/* 
UI组件: 不包含任何redux API
 */
import React from 'react'
import PropTypes from 'prop-types'

export default class Counter extends React.Component {
	
  // 声明需要的属性
  static propTypes = {
    count: PropTypes.number.isRequired,
    increment: PropTypes.func.isRequired,
    decrement: PropTypes.func.isRequired
  }

	/* 4个方法 */
  increment = () => {
    const number = this.refs.numSelect.value * 1
    this.props.increment(number)
  }
  decrement = () => {
    const number = this.refs.numSelect.value * 1
    this.props.decrement(number)
  }
  incrementIfOdd = () => {
    const number = this.refs.numSelect.value * 1
    let count = this.props.count
    if (count % 2 === 1) {
      this.props.increment(number)
    }
  }
  incrementAsync = () => {
    const number = this.refs.numSelect.value * 1
    setTimeout(() => {
      this.props.increment(number)
    }, 1000)
  }

  render() {
    return (
      <div>
        <p>
          click {this.props.count} times {' '}
        </p>
        <select ref="numSelect">
          <option value="1">1</option>
          <option value="2">2</option>
          <option value="3">3</option>
        </select>{' '}
        <button onClick={this.increment}>+</button>
        {' '}
        <button onClick={this.decrement}>-</button>
        {' '}
        <button onClick={this.incrementIfOdd}>increment if odd</button>
        {' '}
        <button onClick={this.incrementAsync}>increment async</button>
      </div>
    )
  }
}
```

### 使用 redux 异步（异步中间件）

其他的不变

```jsx
// redux/store.js
import ...

// 根据counter函数创建store对象
export default createStore(
  reducers,
  composeWithDevTools(applyMiddleware(thunk)) // 启用异步中间件，否则action中默认只能return一个对象
)
```

```javascript
// redux/actions.js
import {INCREMENT, DECREMENT} from './action-types'

// 同步的action返回的是一个对象
// 异步的action返回的是一个函数

export const increment = number => ({type: INCREMENT, number})
export const decrement = number => ({type: DECREMENT, number})

// 异步action creator(返回一个函数)
export const incrementAsync = number => {
  return dispatch => { // 返回的是一个函数
    // 异步代码
    setTimeout(() => {
      dispatch(increment(number)) // 分发action
    }, 1000)
  }
}
```

```jsx
// containers/app.jsx （就增加了incrementAsync的引入和使用）
import ...

// 向外暴露连接App组件的包装组件
export default connect(
  state => ({count: state.counter}),
  {increment, decrement, incrementAsync}
)(Counter)
```

```javascript
// components/counter.jsx
...就增加了incrementAsync的使用
```

## 总：使用 redux & react-redux 的模板

```
目录结构：以下只是其中一种目录结构
src
	*components（UI组件）
		counter.jsx
		....
	*containers（容器组件）
		app.jsx
		....
	*redux
		action-types.js
		actions.js
		reducers.js
		store.js
	index.js
```

```jsx
// components/counter.jsx  纯react代码，不包含redux
import React from 'react'
import PropTypes from 'prop-types'

export default class Counter extends React.Component {
  
  static propTypes = {
    count: PropTypes.number.isRequired,
    increment: PropTypes.func.isRequired,
    ...
  }

	// 方法
  increment = () => {
    const number = this.refs.numSelect.value*1
    this.props.increment(number)
  }
	....

  render () {
    return (
      <div>
        ... HTML结构
      </div>
    )
  }
}
```

```jsx
// containers/app.jsx
import React from 'react'
import {connect} from 'react-redux'
import {increment, decrement, incrementAsync} from '../redux/actions'

import Counter from '../components/counter'

// 向外暴露连接App组件的包装组件
export default connect(
  state => ({count: state.counter}),
  {increment, decrement, incrementAsync}
)(Counter)
```

```javascript
// redux/action-types.js
export const ADD_COMMENT = 'ADD_COMMENT'
export const DELETE_COMMENT = 'DELETE_COMMENT'
...
```

```javascript
// redux/actions.js
import {
  ADD_COMMENT,
  DELETE_COMMENT,
  ...
} from './action-types'

// 同步action
export const addComment = (comment) => ({type: ADD_COMMENT, data: comment})
export const deleteComment = (index) => ({type: DELETE_COMMENT, data: index})

const receiveComments = (comments) => ({type: RECEIVE_COMMENTS, data: comments}) // 当前文件内使用，不需要导出
export const getComments = () => { // 异步action
  return dispatch => { // 返回的是一个函数
    // 异步代码
    setTimeout(() => { 
      const comments = [...]
      dispatch(receiveComments(comments)) // 分发action
    }, 1000)
  }
}
```

```javascript
// redux/reducers.js
import {combineReducers} from 'redux'
import { ADD_COMMENT, ... } from './action-types'

const initComments = []
function comments(state = initComments, action) {
  switch (action.type) {
    case ADD_COMMENT:
      return [...state, action.data]
    case ...:
      return ...
    default:
      return state
  }
}

const xxx = []
function abc(state = xxx, action) {
  switch (action.type) {
    case ...:
      return [...state, action.data]
    default:
      return state
  }
}

export default combineReducers({
  comments,
  abc
})
```

```javascript
// redux/store.js
import React from 'react'
import {
  createStore, // 创建store对象
  applyMiddleware // 异步中间件1
} from 'redux'
import {composeWithDevTools} from 'redux-devtools-extension' // redux调试工具
import reducers from './reducers'
import thunk from 'redux-thunk' // 异步中间件2

// 创建store对象
export default createStore(
  reducers,
  composeWithDevTools(applyMiddleware(thunk))
)
```

```javascript
// index.js
import React from 'react'
import ReactDOM from 'react-dom'
import {Provider} from 'react-redux' // 导入 Provider
import App from './components/app'
import store from './redux/store'

// 渲染根组件标签的函数
ReactDOM.render(( 
  <Provider abc={abc}（传递参数）>
    <App/>
  </Provider>
), document.getElementById('root'))
```

## 自定义 redux & react-redux

### redux

```javascript
/*
redux模块：对象
1). redux模块整体是一个对象模块
2). 内部包含几个函数
    createStore(reducer)  接收一个reducer函数，返回一个store对象 使用：createStore(reducer)
    combineReducers()    接收一个包含多个reducer函数的对象，返回一个新的reducer函数 使用：combineReducers({count, msgs})
    applyMiddleware()  // 太难暂不实现
3). store对象的功能
    getState()  得到内部管理state对象
    dispatch(action)  分发action，会触发reducer调用，返回一个新的state，调用所有绑定的listener
    subscibe(listener)  订阅一个state的监听器
 */

/*
创建store对象的函数
 */
export function createStore(reducer) { // 接收的是整合以后的reducer
  let state // 内部管理的state
  const listeners = [] // 用来缓存监听的数组容器，内部保存n个listener的数组
  
  state = reducer(state, {type: '@@mini-redux/INIT'}) // 第一次调用reducer得到初始state值并保存

  /*获取当前状态
  得到内部管理state对象
   */
  function getState() {
    return state
  }

  /*分发消息
  分发action，会触发reducer调用，返回一个新的state，调用所有绑定的listener
   */
  function dispatch(action) {
   
    state = reducer(state, action) // 调用reducer, 得到一个新的state
    listeners.forEach(listener => listener()) // 调用listeners中所有的监视回调函数, 通知状态变化
  }

  /*订阅监听
  订阅一个state的监听器
   */
  function subscribe(listener) {
    listeners.push(listener) // 将新的监听添加到监听缓存容器中
  }

  return {getState, dispatch, subscribe} // 向外暴露store对象
}

/*合并多个reducer的函数
接收一个包含多个reducer函数的对象，返回一个新的reducer函数
 */
export const combineReducers = (reducers) => {
  // 返回一个reducer声明函数
  return (state = {}, action) => { // 这是一个reducer，会传给createStore()
    // 依次调用所有reducer函数，得到n个新的子状态，封装成对象并返回
   
    const newState = {} // 准备一个用来保存所有新状态的容器对象
    // 包含所有reducer函数名的数组: ['count', 'msgs']
    const keys = Object.keys(reducers)
   
    keys.forEach(key => { // 遍历
      const childReducer = reducer[key] // 得到对应的子reduicer函数
      const childState = state[key] // 得到对应的子state
      const newChildState = childReducer(childState, action) // 执行reducer
     
      newState[key] = newChildState // 整合,保存到新的总state中
    })
    return newState // 返回总的新state
  }
}

/* 更好的写法 */
export const combineReducers = (reducers) => {
  // 返回一个reducer声明函数
  return (state = {}, action) => { // 这是一个reducer，会传给createStore()
    // 依次调用所有reducer函数，得到n个新的子状态，封装成对象并返回
    // 返回包含所有reducer状态的总state对象
    return Object.keys(reducers).reduce((newState, reducerName) => {
      // 调用对应的reducer函数得到对应的新state, 并保存到总state中
      newState[reducerName] = reducers[reducerName](state[reducerName], action)
      return newState
    }, {})
  }
}

// 调用
createStore(combineReducers({
  count,
  msgs
}))
```

### react-redux

```javascript
/*
1). react-redux模块整体是一个对象模块
2). 包含2个重要属性: Provider和connect
3). Provider
		值: 组件类
		作用: 向所有容器子组件提供全局store对象
		使用: <Provider store={store}><Xxx/></Provider>
4). connect
		值: 高阶函数
		作用: 包装组件生成容器组件, 让被包装组件能与redux进行通信
		使用: connect(mapStateToProps, mapDispatchToProps)(Xxx)

1). 理解
		当你觉得多层传递props麻烦, 可以选择使用context
		context是组件对象的一个属性, 它的值是一个对象
		一个组件指定的context内数据, 所有层次子组件都可以读取到
		如果可以尽量不用context, 你可以选择使用react-redux, react-redux内部就利用了context
2). 使用
	父组件:
			static childContextTypes = {
        color: PropTypes.string
      }
			getChildContext() {
        return {color: 'red'};
      }
	后代组件:
			static contextTypes = {
        color: PropTypes.string
      }
          
			render () {
        this.context.color
      }
*/

import React, {Component} from 'react'
import PropTypes from 'prop-types'

/*
react-redux模块: 对象, 包含以下属性：
1.Provider 组件类
2.connect 方法
*/

/*
1. Provider  组件类  为所有的容器子组件提供store(context)
<Provider store={store}>
  <App /> 
</Provider>
 */
export class Provider extends Component {
  static propTypes = {
    store: PropTypes.object.isRequired // 声明当前组件接收store
  }

  // 声明向子组件提供哪些context数据
  // 必须声明向子节点指定全局数据store
  static childContextTypes = {
    store: PropTypes.object.isRequired
  }

  // 指定向子组件指定全局数据store
  // 为子组件提供包含store的context
  getChildContext() {
    return {store: this.props.store}; // 返回的就是context对象
  }

  render() {
    /* 返回所有子节点(如果没有子节点返回undefined, 如果只有一个子节点它是对象, 如果有多个它是数组)
     		this.props.children 代表 Provider内部的所有子标签
     		如果没有子标签：undefined
     		如果只有一个子标签：对象
     		如果有多个子标签：数组 */
    return this.props.children
  }
}

/*
2. connect方法
用法：export default connect((state) => {}, {})(Xxx)
mapStateToProps  函数，用来确定一般属性
mapDispatchToProps  对象，用来确定函数属性，内部会使用dispatch
 */
export function connect(mapStateToProps = () => null, mapDispatchToProps = {}) {
  // 返回函数(接收被包装组件类作为参数)
  return (WrapComponent) => {
    // 返回一个新的组件类（容器组件）
    return class ConnectComponent extends Component {
      // 声明接收全局store
      // 声明获取context数据
      static contextTypes = {
        store: PropTypes.object.isRequired
      }

      // 构造函数的第2个参数为context对象
      constructor(props, context) {
        super(props)
        console.log('constructor', this.context) // 此时组件对象中还没有context
       
        const store = context.store // 得到store，从context中取出store
        // 包含一般属性的对象
        // 一般属性: 调用mapStateToProps函数得到包含所有需要传递一般属性的集合对象
        const stateProps = mapStateToProps(store.getState())
        // 包含函数属性的对象
        // 分发action的函数属性: 调用自定义的整合函数生成包含多个分发action的函数的对象
        const dispatchProps = this.bindActionCreators(mapDispatchToProps)
        // 初始化状态, 包含所有需要传递给WrapComponent组件的一般属性
        // 将所有的一般属性都保存到state中
        this.state = {...stateProps}
        // 将包含dispatch函数的对象保存在组件对象上(不用放到state中)
        // 将所有函数属性的对象保存组件对象
        this.dispatchProps = dispatchProps
      }

      /*
      根据包含多个action creator的对象, 返回一个包含多个分发action的函数的对象
      根据mapDispatchToProps返回一个包含分发action的函数的对象
       */
      /* 版本1 */
      bindActionCreators = (mapDispatchToProps) => {
       
        const dispatchProps = {} // 准备一个保存分发action函数的对象容器
        // 遍历每个action creator
        Object.keys(mapDispatchToProps).forEach((key) => {
         
          const actionCreator = mapDispatchToProps[key] // 得到某个action creator
          //定义包含分发action代码的函数, 并只在到准备好的容器中
          dispatchProps[key] = (...args) => { // 透传: 将函数接收到的参数，原样传内部函数调用
            this.context.store.dispatch(actionCreator(...args))
          }
        })
       
        return dispatchProps // 返回dispatch代码函数容器对象
      }

      /* 版本2 */
      bindActionCreators = (mapDispatchToProps) => {
        const keys = Object.keys(mapDispatchToProps)
        return keys.reduce((preDispatchProps, key) => {
          // 添加一个包含dispatch语句的方法
          preDispatchProps[key] = (...args) => { // 透传: 将函数接收到的参数，原样传内部函数调用
            // 分发action
            this.context.store.dispatch(mapDispatchToProps[key](...args))
          }
          return preDispatchProps
        }, {})
      }
      
      // 监听
      componentDidMount() {
        const store = this.context.store // 得到store
        // 订阅监听
        store.subscribe(() => {
          // 一旦store中的state有变化, 更新组件状态, 从而导致被包装组件重新渲染
          this.setState(mapStateToProps(store.getState()))
        })    
      }

      render() {
        // 返回被包装组件标签, 传入整合的所有属性
        return <WrapComponent {...this.state} {...this.dispatchProps}/>
      }
    }
  }
}
```

# 脚手架

安装：

```
npm install -g create-react-app
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

创建项目：

```
create-react-app 项目名称    项目名称不能包含大写字母
yarn start 启动项目
```

目录结构：

```javascript
test-react
├─ README.md // 说明文档
├─ package.json // 对整个应用程序的描述：包括应用名称、版本号、一些依赖包、以及项目的启动、打包等等
├─ public
│ 	├─ favicon.ico // 应用程序顶部的icon图标
│		├─ index.htm1 // 应用的index.htm1入口文件
│ 	├─ 1ogo192 .png // PWA相关，在manifest.json中使用
│ 	├─ 1ogo512.png // PWA相关，在manifest.json中使用
│ 	├─ manifest.json // PWA相关，和web app配置相关
│ 	└─ robots.txt // 搜索引擎（爬虫）可以或者无法爬取哪些文件
├─ src
│ 	├─ App.css // App组件相关的样式
│ 	├─ App.js // App组件的代码文件
│ 	├─ App.test.js // App组件的测试代码文件
│ 	├─ index.css // 全局的样式文件
│ 	├─ index.js // 整个应用程序的入口文件
│ 	├─ 1ogo.svg // 启动项目，所看到的React图标
│ 	├─ serviceWorker.js // 默认帮助我们写好的注册PWA相关的代码
│ 	└─ setupTests.js // 测试初始化文件
└─ yarn.1ock // 详细的依赖文件说明
```

React的脚手架使用的是yarn

![1592793223480](React.assets/1592793223480.png)







# 知识点笔记

1. 为了便于阅读，我们会将 JSX 拆分为多行。同时，我们建议将内容包裹在括号中，虽然这样做不是强制要求的，但是这可以避免遇到[自动插入分号](http://stackoverflow.com/q/2846283)陷阱。

2. React 元素是不可变对象。一旦被创建，你就无法更改它的子元素或者属性。一个元素就像电影的单帧：它代表了某个特定时刻的 UI。 

3. Props是具有只读性，不能修改自身的props， **所有 React 组件都必须像纯函数一样保护它们的 props 不被更改。** 

4. 不会修改传入的参数的叫做“纯函数”

5. React组件需要大写开头，小写的如div会被认为是自带的元素

6. State 与 props 类似，但是 state 是私有的，并且完全受控于当前组件。 

7. Class 组件应该始终使用 `props` 参数来调用父类的构造函数。构造函数是唯一可以给 `this.state` 赋值的地方 

8. `Clock` 组件第一次被渲染到 DOM 中 成为“挂载 （mount） ”， 当 DOM 中 `Clock` 组件被删除的时候被称为“卸载（unmount）”。 

9. 重新渲染必须使用  setState() 

10. 出于性能考虑，React 可能会把多个 `setState()` 调用合并成一个调用。因为 `this.props` 和 `this.state` 可能会异步更新，所以你不要依赖他们的值来更新下一个状态。 例如，此代码可能会无法更新计数器 

    ```
    // 错误
    this.setState({
      counter: this.state.counter + this.props.increment,
    });
    
    // 正确
    this.setState((state, props) => ({
      counter: state.counter + props.increment
    }));
    ```

11. 调用 `setState()` 的时候，React 会把你提供的对象合并到当前的 state 

12. 不管是父组件或是子组件都无法知道某个组件是有状态的还是无状态的，并且它们也并不关心它是函数组件还是 class 组件。这就是为什么称 state 为局部的或是封装的的原因。除了拥有并设置了它的组件，其他组件都无法访问。组件可以选择把它的 state 作为 props 向下传递到它的子组件中：

    ```
    <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
    <FormattedDate date={this.state.date} />
    ```

    但是传递后无法知道来源是哪里，这叫做“单向数据流”， 只能单向地向下流动。 

13. React中的自定义事件需要用驼峰命名，且是一个函数而不是字符串

    ```
    // 原生
    <button onclick="activateLasers()">
    // React
    <button onClick={activateLasers}>
    ```

14. React中不能return false来阻止默认行为，必须

    ```
    function handleClick(e) {
        e.preventDefault();
        console.log('The link was clicked.');
      }
    ```

15. 用箭头函数解决this问题
    而箭头函数有两种方式：

    ```javascript
    class LoggingButton extends React.Component {
      // 此语法确保 `handleClick` 内的 `this` 已被绑定。
      // 注意: 这是 *实验性* 语法。
      handleClick = () => {
        console.log('this is:', this);  ///////////////////////////////
      }
    
      render() {
        return (
          <button onClick={this.handleClick}>
            Click me
          </button>
        );
      }
    }
    /* -------------------------------------------------- */
    class LoggingButton extends React.Component {
      handleClick() {
        console.log('this is:', this);
      }
    
      render() {
        // 此语法确保 `handleClick` 内的 `this` 已被绑定。
        return (
          <button onClick={() => this.handleClick()}> ///////////////////////////////
            Click me
          </button>
        );
      }
    }
    
    /*
    	更推荐第一种（public class fields 语法），第一种相当于在构造函数中用bind，
    	而第二种（在回调函数中使用箭头函数）则：
          此语法问题在于每次渲染 LoggingButton 时都会创建不同的回调函数。在大多数情况下，这没什么问 				题，但如果该回调函数作为 prop 传入子组件时，这些组件可能会进行额外的重新渲染。我们通常建					议在构造器中绑定或使用 class fields 语法来避免这类性能问题。
    */
    ```

16. 条件渲染：

    ```javascript
    // if...else条件渲染
    if (isLoggedIn) {
        return <UserGreeting />;
      }
      return <GuestGreeting />;
    }
    ```

    ```javascript
    // &&条件渲染
    {unreadMessages.length > 0 &&
      <h2>
      	You have {unreadMessages.length} unread messages.
      </h2>
    }
    ```

    ```javascript
    // 三目运算符
    {isLoggedIn
       ? <LogoutButton onClick={this.handleLogoutClick} />
       : <LoginButton onClick={this.handleLoginClick} />
    }
    ```

17. 阻止组件渲染（隐藏组件）： 让 `render` 方法直接返回 `null`，而不进行任何渲染。 

    ```javascript
    // 定义组件，然后在组件内条件判断返回null还是返回其他的
    function WarningBanner(props) {
      if (!props.warn) { // 判断
        return null;
      }
      return (
        <div className="warning">
          Warning!
        </div>
      );
    }
    
    class Page extends React.Component {
      constructor(props) {
        super(props);
        this.state = {showWarning: true}// 初始化值
        this.handleToggleClick = this.handleToggleClick.bind(this);
      }
    
      handleToggleClick() { // 其他的方法改变warn的值，例如按钮点击
        this.setState(prevState => ({
          showWarning: !prevState.showWarning
        }));
      }
      
      render() {
        return (
          <div>
            <WarningBanner warn={this.state.showWarning} />
            <button onClick={this.handleToggleClick}> // 监听点击
              {this.state.showWarning ? 'Hide' : 'Show'}
            </button>
          </div>
        );
      }
    }
    
    ReactDOM.render(
      <Page />,
      document.getElementById('root')
    );
    ```

     在组件的 `render` 方法中返回 `null` 并不会影响组件的生命周期。 

18. 用map方法渲染多个组件：

    ```javascript
    const numbers = [1, 2, 3, 4, 5];
    const listItems = numbers.map((number) =>
      <li>{number}</li>
    );
    
    ReactDOM.render(
      <ul>{listItems}</ul>,
      document.getElementById('root')
    );
    ```

19. key的位置： 元素的 key 只有放在就近的数组上下文中才有意义。  比方说，如果你[提取](https://reactjs.bootcss.com/docs/components-and-props.html#extracting-components)出一个 `ListItem` 组件，你应该把 key 保留在数组中的这个 元素上，而不是放在 `ListItem` 组件中的元素上。 

    一个好的经验法则是：在 `map()` 方法中的元素需要设置 key 属性。

    数组元素中使用的 key 在其兄弟节点之间应该是独一无二的。然而，它们不需要是全局唯一的。当我们生成两个不同的数组时，我们可以使用相同的 key 值 。
     key 会传递信息给 React ，但不会传递给你的组件。如果你的组件中需要使用 `key` 属性的值，请用其他属性名显式传递这个值 。

20. 受控组件与非受控组件
    受控组件：

    ​				将数据保存在state中，并通过setState()来更新，使 state 成为“唯一数据源” 。 渲染表单的 React 组				件还控制着用户输入过程中表单发生的操作。被 React 以这种方式控制取值的表单输入元素就叫				做“受控组件”。 
    非受控组件： 

    ​				不需要设置它的state属性，而通过ref来操作真实的DOM。

21. 状态提升：两个组件间的数据state是不共享的，将想要共享的状态提升到最近的共同父组件中去 ，便可实现共享 state，用props传入。

22. 一个组件原则上只能负责一个功能。如果它需要负责更多的功能，这时候就应该考虑将它拆分成更小的组件。 

23. 通过问自己以下三个问题，你可以逐个检查相应数据是否属于 state：

    1. 该数据是否是由父组件通过 props 传递而来的？如果是，那它应该不是 state。
    2. 该数据是否随时间的推移而保持不变？如果是，那它应该也不是 state。
    3. 你能否根据其他 state 或 props 计算出该数据的值？如果是，那它也不是 state。

24. 

25. 

26. 

27. 

28. 

29. 

30. 

31. 

32. 

33. 

    

    