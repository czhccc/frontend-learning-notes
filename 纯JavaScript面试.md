# 纯JavaScript

- “阻塞和非阻塞”与“同步和异步”之间的区别是什么

  阻塞/非阻塞是关于程序在等待时是继续运行还是暂停执行的问题，而同步/异步是关于如何处理和接收操作完成的通知的问题。

- 关于“闭包”

  即内部函数访问外部函数中的变量。

  当外部函数返回内部函数时，即使外部函数的执行上下文已经消失，内部函数仍然可以访问外部函数的变量。这是因为变量仍然存储在闭包中。

  - **内存泄漏**：如果不正确使用闭包，可能导致内存泄漏，因为闭包中的变量在闭包存在的整个生命周期内都不会被垃圾回收。
  - **性能**：频繁创建闭包可能会影响程序的性能，因为每个闭包都占用内存空间。

- Node.js架构中各部分的简要概述：

  1. **V8 引擎**：这是核心，负责执行JS代码。V8是谷歌开发的高性能JS和WebAssembly引擎，它将JS代码编译成机器代码，使其执行速度更快。
  2. **libuv**：这是一个跨平台的C库，提供了异步I/O操作的支持。它是Node.js非阻塞I/O模型的关键部分，允许Node.js处理大量并发操作，例如文件系统操作、网络读写等。
  3. **C++ APIs**：这些是Node.js的C++接口，使Node.js能够访问底层系统资源。通过这些API，Node.js可以调用C/C++库以及其他系统级资源，增强了Node.js的功能和效率。
  4. **事件循环**：这是Node.js的事件处理机制，用于处理外部事件（如用户输入、文件I/O事件等）并将它们转换为回调函数调用。事件循环是Node.js异步非阻塞架构的核心，允许Node.js在处理大量并发请求时保持高效。
  5. **JavaScript标准库**：这是Node.js内置的模块和函数集合。它提供了一系列实用的工具和功能，如文件系统操作、HTTP请求处理、流处理等，这些都是构建Node.js应用程序的基础。

- `Proxy` 之所以能有效地监听数组变化，是因为它在整个对象级别上工作，能够拦截数组的各种操作，包括通过方法修改数组的行为。而 `Object.defineProperty()` 只能作用于单个属性，无法全面地拦截数组的动态变化。因此，对于需要监听数组和其他复杂对象变化的场景，`Proxy` 是更合适的选择。

- 什么是变量提升？函数提升？

  - 变量提升

  ```js
  // 在 JavaScript 中，所有的变量声明（但不是赋值）都会被提升到其所在作用域的顶部。这意味着无论在哪里声明变量，它们都会被视为在函数或代码块的顶部声明的，而不是在实际声明的位置。这个过程发生在代码执行之前。
  
  console.log(name) // undefined
  var name = 'Sunshine_Lin'
  
  if (false) {
    var age = 23
  }
  console.log(age) // undefined 不会报错
  ```

  - 函数提升

  ```js
  // 与变量提升类似，函数声明也会被提升到其所在作用域的顶部。这意味着你可以在函数定义之前调用函数，而不会引发错误。
  
  sayHello(); // "Hello, World!"
  
  function sayHello() {
    console.log("Hello, World!");
  }
  ```

  - 函数提升优先级 > 变量提升优先级

  ```js
  console.log(fun) // function fun() {}
  var fun = 'Sunshie_Lin'
  function fun() {}
  console.log(fun) // 'Sunshie_Lin'
  ```

- isNaN 与 Number.isNaN的区别？

  ### `isNaN`

  - **定义**：`isNaN` 函数是全局函数。

  - **行为**：它**首先尝试将非数字的值转换为数字，然后才检测**这个值是否是 `NaN`。这意味着对于非数字值（例如字符串），即使它们不是 `NaN`，`isNaN` 也会返回 `true`，因为在转换过程中产生了 `NaN`。

  - 示例

    ```js
    javascriptCopy codeisNaN("Hello"); // true，因为 "Hello" 转换为数字是 NaN
    isNaN(undefined); // true，因为 undefined 转换为数字是 NaN
    isNaN({}); // true，因为 {} 转换为数字是 NaN
    isNaN(NaN); // true
    ```

  ### `Number.isNaN`

  - **定义**：`Number.isNaN` 是 ES6 中引入的，作为 `Number` 对象的一个方法。

  - **行为**：它**不会尝试转换参数**，**而是直接检测**这个值是否严格等于 `NaN`。这意味着除了 `NaN` 本身，`Number.isNaN` 对于所有其他值都返回 `false`。

  - 示例

    ```js
    javascriptCopy codeNumber.isNaN("Hello"); // false，因为 "Hello" 本身不是 NaN
    Number.isNaN(undefined); // false
    Number.isNaN({}); // false
    Number.isNaN(NaN); // true
    ```

  ### 选择使用哪一个

  - 使用 `Number.isNaN` 当你需要准确判断一个值是否为 `NaN` 本身，不希望有任何类型转换。
  - 使用 `isNaN` 当你需要判断一个值在转换为数字后是否为 `NaN`，例如在处理用户输入或任何可能不是数字的数据时。

  在大多数现代JavaScript应用中，推荐使用 `Number.isNaN` 以避免因自动类型转换导致的意外行为。

- 解决遍历对象时，把原型上的属性遍历出来了咋办？

  遍历出原型上的属性通常发生在使用 `for...in` 循环遍历对象时。这是因为 `for...in` 循环不仅遍历对象自身的属性，还会遍历其原型链上的所有可枚举属性。

  下面是几种常见的解决方法：

  ### 使用 `hasOwnProperty`

  在 `for...in` 使用 `Object.hasOwnProperty()` 判断属性是否是对象自身属性。`hasOwnProperty` 方法不会检查对象的原型链，只会检查属性是否直接属于该对象。

  ```js
  javascriptCopy codeconst myObject = {/*...对象的属性...*/};
  
  for (const key in myObject) {
    if (myObject.hasOwnProperty(key)) {
      console.log(key, myObject[key]);
    }
  }
  ```

  ### 使用 `Object.keys()`

  `Object.keys()` 方法返回一个包含对象自身（非继承）所有可枚举属性名称的数组。

  ```js
  javascriptCopy codeconst myObject = {/*...对象的属性...*/};
  
  Object.keys(myObject).forEach(key => {
    console.log(key, myObject[key]);
  });
  ```

  ### 使用 `Object.entries()`

  `Object.entries()` 方法返回一个给定对象自身可枚举属性的键值对数组。

  ```js
  javascriptCopy codeconst myObject = {/*...对象的属性...*/};
  
  Object.entries(myObject).forEach(([key, value]) => {
    console.log(key, value);
  });
  ```

- JavaScript变量在内存中具体存储形式？

  基础类型存放在栈内存中，引用类型存放在堆内存中。而如果像字符串数据量很大的话，JS引擎会考虑将这个字符串也存放到堆内存中。不同的JS引擎也有不同的管理方式。

- 去重除了set还有哪些方法

  1. **使用`filter`方法**： 使用数组的`filter`方法结合`indexOf`方法可以实现去重。这种方法在每次遇到元素时检查它在数组中的第一次出现的位置，如果当前位置与首次出现位置相同，则保留该元素。

     ```js
     const array = [1, 2, 2, 3, 4, 4, 5];
     const uniqueArray = array.filter((item, index) => array.indexOf(item) === index);
     ```

  2. **使用`reduce`方法**： `reduce`方法也可以用于累积数组中的唯一项。在这种方法中，您可以创建一个新数组，并在每一步检查当前元素是否已经存在于这个新数组中。

     ```js
     const uniqueArray = array.reduce((unique, item) => {
       return unique.includes(item) ? unique : [...unique, item];
     }, []);
     ```

  3. **使用`forEach`和对象属性**： 使用一个对象来记录出现过的元素。对象的属性名对应元素的值，属性值可以是任意值。这种方法在处理大型数组时比较高效。

     ```js
     const uniqueObject = {};
     array.forEach(item => {
       uniqueObject[item] = true;
     });
     const uniqueArray = Object.keys(uniqueObject).map(key => parseInt(key));
     ```

  4. **使用`Map`对象**： 类似于使用普通对象，但`Map`对象允许键为任意类型，在处理非字符串元素时很有用。

     ```js
     const uniqueMap = new Map();
     array.forEach(item => {
       uniqueMap.set(item, true);
     });
     const uniqueArray = Array.from(uniqueMap.keys());
     ```

- 用Promise 设计为请求5秒后即判断为超时

  可以使用`Promise.race`，接受一个`Promise`数组作为输入，并返回一个新的`Promise`，这个新的`Promise`在输入的任何一个`Promise`完成（或拒绝）时就完成（或拒绝）。

  创建两个`Promise`对象：一个是你的实际请求，另一个是一个在5秒后拒绝的`Promise`。然后使用`Promise.race`来返回这两个`Promise`中首先完成的那个。

  以下是一个示例代码：

  ```js
  function requestWithTimeout(requestFunction, timeout = 5000) {
    // 创建一个在指定时间后拒绝的Promise
    let timeoutPromise = new Promise((resolve, reject) => {
      setTimeout(() => {
        reject(new Error('Request timed out'));
      }, timeout);
    });
  
    // 用Promise.race返回早期完成的Promise（请求或超时）
    return Promise.race([requestFunction(), timeoutPromise]);
  }
  
  // 示例用法
  requestWithTimeout(() => {
    // 在这里实现你的请求逻辑，例如fetch API或其他
    return new Promise((resolve) => {
      // 模拟请求
      setTimeout(() => {
        resolve('Request successful');
      }, 3000); // 假设请求在3秒内完成
    });
  })
  .then(response => console.log(response))
  .catch(error => console.error(error));
  ```

  注意，即使超时`Promise`首先完成，原始的请求仍然会在后台执行，超时不会取消请求，而只是导致`Promise.race`返回超时错误。如果你需要取消正在进行的请求，你可能需要使用更复杂的机制。

- 什么是单点登录

  单点登录（SSO，Single Sign-On）是一种身份验证服务，允许用户使用一套登录凭据（如用户名和密码）来访问多个应用或系统。

  单点登录的关键特点和优势包括：

  1. **简化用户体验**：用户只需记住一套凭据，就可以访问多个服务和应用程序。
  2. **提高安全性**：减少密码泄露的风险，因为用户只需要记住一组凭据。此外，企业可以集中管理用户凭证和访问权限，实施统一的安全策略。
  3. **减少管理工作**：变更时只需在一个地方更新其状态和权限。
  4. **集中的身份验证和审计**：SSO 允许集中跟踪用户活动和身份验证尝试，有助于合规性和安全监控。
  5. **降低支持成本**：由于用户密码问题减少，可以减轻 IT 支持负担。

  SSO 的实现通常依赖于标准的身份验证协议，如 OAuth、OpenID Connect、SAML（安全断言标记语言）等。这些协议允许不同系统和服务之间安全地交换用户身份和凭证信息。

  然而，单点登录也有其挑战和缺点，例如，如果 SSO 解决方案本身受到攻击，那么攻击者可能获得对所有连接服务的访问权限。因此，实施 SSO 需要考虑到综合的安全措施和风险管理策略。

- unhandlerejection

  `unhandledrejection` 用于捕获未处理的 Promise 拒绝（没有`.catch()`处理函数的 Promise）的机制。是一个全局事件，可以在`window`上监听。

  当一个 Promise 被拒绝，且没有`.catch()`或在`async`函数中没有使用`try...catch`来捕获错误），`unhandledrejection`事件就会被触发。

  ```js
  // main.js 或 main.ts
  import { createApp } from 'vue'
  import App from './App.vue'
  
  const app = createApp(App);
  
  window.addEventListener('unhandledrejection', event => {
    event.preventDefault(); // 防止默认的控制台错误日志
    console.error('捕获到未处理的 Promise 拒绝:', event.reason); // 这里可以添加其他错误处理逻辑
  });
  
  app.mount('#app');
  ```

  - 如果一个 Promise 最初被拒绝但稍后被`.catch()`捕获了，不会触发`unhandledrejection`。
  - 一些环境（如Node.js）可能有自己的机制来处理未捕获的 Promise 拒绝，所以在不同环境中使用`unhandledrejection`可能会有所不同。

- undefined、null、NaN 这些进行数据类型之间的运算是否有什么规律

  ### `undefined`

  - `undefined` 表示变量已声明但尚未被赋值。
  - 算术运算中，将 undefined 视为 NaN
    - 例如，`undefined + 1` 会得到 `NaN`。
  - 在与其他类型（如字符串）进行运算时，undefined 通常被转换为字符串 "undefined"
    - 例如，`"Value is " + undefined` 会得到 `"Value is undefined"`。

  ### `null`

  - `null` 表示一个“无”的值，通常被用来表示意图上的“空值”。
  - 在算术运算中，null 被视为 0
    - 例如，`null + 1` 会得到 `1`。
  - 在与字符串进行运算时，null 会被转换为字符串 "null"
    - 例如，`"Value is " + null` 会得到 `"Value is null"`。

  ### `NaN`

  - `NaN` 表示“非数字值”，是一个特殊的浮点数值，用于表示无法定义或不可表示的数值。
  - NaN 与任何值（包括它自己）进行算术运算都会得到 NaN
    - 例如，`NaN + 1` 或 `NaN + NaN` 都会得到 `NaN`。
  - 在逻辑运算中，`NaN` 被视为 `false`。
  - `NaN` 是唯一一个不等于自己的值，即 `NaN !== NaN` 为 `true`。

  ### 相互运算

  - null 和 undefined 相互比较时相等，但与其他值比较时不相等。
    - `null == undefined` 是 `true`，但 `null === undefined` 是 `false`（因为严格等于运算符还会比较类型）
  - 当 `null`、`undefined` 或 `NaN` 与其他类型的值进行运算时，通常会发生类型转换，遵循 JS 的弱类型特性和类型转换规则

  ### 注意事项

  - 实际编程中，最好避免涉及 `undefined`、`null` 或 `NaN` 的复杂运算，以减少错误和混淆。
  - 使用诸如 `typeof`、`isNaN()`、`Number.isNaN()`、`null` 和 `undefined` 的严格比较（使用 `===` 和 `!==`）等可以帮助更精确地处理这些特殊值。

- Monorepo

  想象有一个大书架放着很多不同的书，每本书代表一个不同的项目。小说、教科书、工具书或手册。这些书虽然内容不同，但都放在同一个书架上，这就类似于 Monorepo。

  在 Monorepo 中，不同的项目都放在同一个代码仓库中。可以很方便地从一个项目中借鉴另一个项目的代码（就像你可以很容易地从一个书架上取下一本书参考里面的内容）。同时，如果要对书架进行整理（比如对代码库进行更新或维护），只需要处理这一个大书架，而不是到处去找分散在不同地方的多个小书架。

  实际开发中，Monorepo 允许团队在一个统一的环境中开发、测试和部署多个相关的项目。有助于简化工作流程，比如代码共享、依赖管理和版本控制。但是，就像一个大书架如果放得太满可能难以管理一样，Monorepo 也会随着项目的增多而变得更加复杂，需要更好的组织和工具来维护。

  -----

  Monorepo（单仓库）与 Git 或 SVN 这样的版本控制系统（VCS）之间的区别，主要在于它们各自的组织和管理代码的方式。

  Monorepo

  - **概念**：Monorepo 是一种代码仓库的结构方式，指的是在一个单一的版本控制仓库中维护多个相关或不相关的项目。它是关于如何组织代码和项目的一种策略。
  - 特点：
    - **单一仓库**：所有项目（可能包括库、应用、服务等）都存放在同一个版本控制仓库中。
    - **统一版本历史**：所有项目共享同一个提交历史，便于跟踪跨项目的更改和依赖关系。
    - **统一的工作流程**：构建、测试和部署等操作可以在单个仓库中统一处理。

  Git 和 SVN

  - **概念**：都是版本控制系统，用于管理文件的更改历史。提供工具和命令来帮助记录、比较、合并和回退更改。
  - 特点：
    - **多仓库和单仓库**：Git 和 SVN 都可以用来实现多仓库（每个项目一个仓库）或单仓库（Monorepo）的结构。它们本身不限定仓库中代码的组织方式。
    - **分布式和集中式**：Git 是分布式的（每个开发者有完整的仓库副本），而 SVN 是集中式的（一个中央仓库，开发者有这个仓库的不同版本）。
    - **功能和操作**：Git 和 SVN 提供了一系列相似但在细节上不同的功能来管理代码的版本，如分支、合并、标签等。

  区别

  1. **概念范畴**：Monorepo 是关于项目结构的策略，而 Git 和 SVN 是实现这种策略的工具
  2. **使用方式**：可以使用 Git 或 SVN 来管理 Monorepo，也可以用它们来管理多个小的仓库
  3. **侧重点**：Monorepo 侧重于如何在一个仓库中组织多个项目，而 Git 和 SVN 侧重于如何记录和管理任何类型仓库中的更改历史
  4. **Monorepo是一种管理的理念，而Git或SVN是具体的工具**

- 关于“事件委托”

  事件委托是一种利用事件冒泡原理来处理事件的技术。在 DOM 中，当一个事件发生后，它会从触发事件的元素开始，向上冒泡至最外层的元素。事件委托就是把原本需要绑定在子元素上的事件处理器委托给父元素，由父元素统一处理多个子元素的事件。例如列表中不再是每个元素都监听一个事件，而是在最外层的父元素中对事件进行统一处理

  优点：

  1. **减少内存消耗**：不必为每个子元素都绑定事件处理器。
  2. **动态元素处理**：对于动态添加的子元素，无需重新绑定事件处理器。

  ```jsx
  // 假设有一个列表，希望在点击每个列表项时执行某些操作，而不是为每个列表项单独绑定事件
  
  <ul id="myList">
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
  </ul>
  
  document.getElementById('myList').addEventListener('click', e => {
    if (e.target.tagName === 'LI') {
      console.log('Clicked on item:', e.target.textContent);
    }
  });
  
  // 甚至可以动态添加新的列表项
  let newItem = document.createElement('li');
  newItem.textContent = 'Item 4';
  document.getElementById('myList').appendChild(newItem);
  
  // 补充：target是触发事件的元素，currentTarget的进行事件处理的元素
  ```

  #### Vue 3

  在 Vue 3 中，事件委托通常是通过自定义的方式实现的，因为 Vue 的事件绑定机制主要集中在组件的模板内。但是，你可以在父元素上使用 `v-on` 或 `@` 语法来实现类似的效果。

  ```vue
  <template>
    <ul @click="handleClick">
      <li v-for="item in items" :key="item.id">{{ item.text }}</li>
    </ul>
  </template>
  
  <script>
  export default {
    data() {
      return {
        items: [{ id: 1, text: 'Item 1' }, { id: 2, text: 'Item 2' }]
      };
    },
    methods: {
      handleClick(e) {
        if (e.target.tagName === 'LI') {
          console.log('Clicked on item:', e.target.textContent);
        }
      }
    }
  }
  </script>
  ```

  #### React

  在 React 中，事件委托是自动实现的。React 的事件系统实际上是在顶层使用了事件委托，所有的事件处理器都绑定到了文档根节点上。当事件发生时，React 根据其内部事件映射来调用相应的处理器。

  ```js
  class MyList extends React.Component {
    handleClick = e => {
      if (e.target.tagName === 'LI') {
        console.log('Clicked on item:', e.target.textContent);
      }
    };
  
    render() {
      return (
        <ul onClick={this.handleClick}>
          <li>Item 1</li>
          <li>Item 2</li>
          <li>Item 3</li>
        </ul>
      );
    }
  }
  ```

  

- 

- 

- 

- 

- 

- 

- 