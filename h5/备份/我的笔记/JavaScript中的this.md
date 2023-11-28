# this

## 为什么要使用this

JavaScript 中的 this 比常见的面向对象编程语言中的 this 更为灵活。

**this 能以便捷的方式来引用对象**，使代码更为简洁和容易复用。

以下述代码为例：

- 在方法中，通过 `this` 即可直接获取引用对象，而不必关心引用的对象的名称是什么。
- 当变量名称（`obj1`）改为其他名称时时，不需要对使用了变量名称（`obj1`）的方法逐个替换。否则，如果使用了10次变量名称，则需要替换10次。

```javascript
// 不使用 this
var obj1 = {  
  name: "obj1",  
  running: function() { console.log(obj1.name + " running"); }
}

// 使用 this
var obj2 = {  
	name: "obj2",  
  running: function() { console.log(this.name + " running"); } // this 指向 obj 对象
}
```

this在**函数运行时进行绑定**，取决于函数调用时的各种条件，并不是在编写时绑定，因此**与函数定义/声明的位置没有任何关系**，只取决于函数**调用的方式**及**调用的位置**。

## this的绑定规则

### 默认绑定

独立的函数调用（函数没有被绑定到某个对象上进行调用）会使用默认绑定。 

在严格模式（use strict）下，无法使用默认绑定，this会指向`undefined`。  

```javascript
function foo(func) {  
  func(); // 真正的函数调用的位置  通过指针找到bar函数然后进行调用，和obj没有关系
}

var obj = {  
  name: "obj",  
  bar: function() {    // 存放了一个bar函数的引用（指针），
    console.log(this); 
  }
}

foo(obj.bar); // window  只是将函数的引用作为参数传入进行调用，并没有与任何对象相关联
// obj.bar 只是通过obj对象拿到bar函数的引用（指针），再通过该引用（指针）找到存放的bar函数，和obj没有关系
// 就像我(foo)去银行(obj对象)取了50块钱(bar函数)，然后去买东西吃(调用bar函数)，实际上，银行(obj对象)只是帮我(foo)保管钱(bar函数)，我(foo)花钱(调用bar函数)的时候和银行(obj)没有关系。
```

**1.直接调用**

函数直接被调用，没有关联任何的对象，此时会默认绑定，函数中的this指向全局对象`window` 。

```javascript
function func() {  
  console.log(this);
}
func(); // window  调用的时候没有和任何对象进行关联
```

**2.函数调用链**

函数内部调用另外一个函数，所有的函数调用都没有被绑定到某个对象上。

```javascript
function test1() {  
  console.log(this); // window  
  test2(); // 调用时没有与任何对象相关联
}

function test2() {  
  console.log(this); // window  
  test3();  // 调用时没有与任何对象相关联
}

function test3() {  
  console.log(this); // window
}

test1();  // 调用时没有与任何对象相关联
```

**3. 函数作为参数**

```javascript
function foo(func) {  
  func()
}

function bar() {  
  console.log(this);
}

foo(bar); // window  只是将函数的引用作为参数传入进行调用，并没有与任何对象相关联
```

### 隐式绑定

函数的调用由一个对象发起，那么JavaScript内部会将this指向这个函数。

准确来说：谁发起的函数调用，this就指向到谁。

隐式绑定有**前提**条件：

- 函数的调用由一个对象发起。

- 必须在调用的对象内部有一个对函数的引用（比如一个属性）。

- 如果没有这样的引用，在进行调用时，会报找不到该函数的错误。

  ```javascript
  // 例如
  function foo() {
  	console.log(this);
  }
  
  var obj = {
  	name: "obj"
  //  foo: foo  没有引用
  }
  
  obj.foo(); // 报错
  ```

- 正是通过这个引用，间接的将this绑定到了这个对象上。

**1.通过对象调用函数**

this会绑定到调用它的那个对象上。

```javascript
function foo() {  
  console.log(this);
}
var obj = {  
  name: "obj",  
  foo: foo
}

obj.foo(); // obj1对象  因为是obj对象发起的函数调用，所以this隐式绑定到obj对象上
```

**2.对象嵌套调用**

当对象嵌套调用时，this绑定到**直接**调用该方法的对象上。

```javascript
function foo() {  
  console.log(this.name);
}
var obj1 = {  
  name: "obj1",  
  foo: foo
}
var obj2 = {  
  name: "obj2",  
  obj1: obj1
}

obj2.obj1.foo(); // obj1  obj2只是拿到obj1对象，最终发起函数调用的是obj1，因此this绑定到obj1
// 就像我(obj1)脚受伤了不能走路(不能直接取得obj1)，叫姐姐(obj2)帮我(obj1)拿一瓶农夫山泉(foo函数)给我喝(调用foo函数)，真正喝农夫山泉(调用foo函数)的人是我(obj1)，姐姐(obj2)只是帮我拿一下，是个工具人。
```

```javascript
function foo() {
  console.log(this)
}

var obj1 = {
  name: "obj1",
  foo: foo
}

var obj2 = {
  name: "obj2",
  demo: obj1.foo
}

obj2.demo(); // obj2对象  不用管内部的调用，只要看是谁发起的函数调用
```

扩展：作用域链 & 原型链

```javascript
// 注释掉 obj1 中的 name
function foo() {  
  console.log(this.name);
}
var obj1 = {  
//  name: "obj1",
  foo: foo
}
var obj2 = {  
  name: "obj2",  
  obj1: obj1
}

obj2.obj1.foo(); // undefined  如果注释掉obj1中的name，则结果为 undefined 。不要将作用域链和原型链弄混淆了，obj1对象虽然obj2的属性，但它们两个的原型链并不相同，并不是父子关系，因此obj1没有在自身找到name属性时并不会查找obj2中的属性，由于obj1未提供name属性，所以是undefined。
```

```javascript
function aaa() {};
aaa.prototype.name = 'xxx';

function bbb() {
  console.log(this.name);
};

var obj1 = new aaa();
obj1.func = bbb;

var obj2 = {
  name: 'yyy',
  obj: obj1
};

obj2.obj.func() // xxx
// 虽然obj1对象并没有name属性，但顺着原型链，找到了产生自己的构造函数aaa，由于Fn原型链存在name属性xxx，所以结果为xxx
```

#### 隐式丢失

隐式丢失有2种：

- 参数传递
- 变量赋值

```javascript
/* 参数传递 */
var obj1 = {
  name: "obj1",
  fn: function () {
    console.log(this);
  }
};
function fn1(func) {
  func();
};

fn1(obj1.fn); // window
// 将obj1的foo作为参数传递，相当于只是单纯传递了一个函数而已，this并没有跟函数绑在一起，所以this丢失，默认绑定指向了window。obj1中的fn只是保存了一个指向fn的指针，fn1(obj1.fn) 相当于 fn1(fn的指针)，只是通过obj1来获取这个指针，本质上函数的调用和obj1没有关系。
// 就像我(fn1)脚受伤了不能走路(不能直接拿到fn的指针)，叫姐姐帮我拿一瓶农夫山泉给我喝，然后我(fn1)再从姐姐(obj1)那里拿到农夫山泉(fn的指针)来喝(调用fn函数)，真正喝农夫山泉(调用fn函数)的人是我(fn1)，姐姐(obj1)只是帮我拿一下，是个工具人。
```

```javascript
/* 变量赋值 */
function foo() {  
  console.log(this); // window
}
var obj1 = {  
  name: "obj1",  
  foo: foo
}

var bar = obj1.foo; // 将obj1的foo赋值给bar
bar(); // 调用时没有绑定任何的对象，相当于默认绑定
```

```javascript
// 隐式丢失并不都会指向全局对象window
var obj1 = {
  name: 'obj1',
  fn: function () {
    console.log(this);
  }
};
var obj2 = {
  name: 'obj2'
}

obj2.fn = obj1.fn; // 虽然丢失了 obj1 的隐式绑定，但是在赋值的过程中，又建立了新的隐式绑定，这里this就指向了对象 obj2。
obj2.fn(); // obj2
```

### 显示绑定

可以通过 `call()` 、 `apply()` 及 `bind()` 方法来将 this 明确、强制绑定到传入的对象上。

#### call、apply、bind

```javascript
var obj1 = { name: 'obj1' };
var obj2 = { name: 'obj2' };
var obj3 = { name: 'obj3' };
function fn() {
  console.log(this);
};

fn(); // window

fn.call(obj1); // obj1
fn.apply(obj2); // obj2
fn.bind(obj3)(); // obj3

fn(); // window
```

三者的区别：

1. `call`、`apply`在改变this指向的同时还会执行函数，而`bind`返回一个新的改变了this的函数，不会改变原方法的this，也不会自动执行，这也是为什么下方例子中`bind`后还加了一对括号 `()` 的原因。

2. `bind`属于硬绑定，返回的函数的 this 指向无法再次通过`bind`、`apply`或`call`修改；`call`与`apply`的绑定只适用当前调用，调用完就没了，下次要用还得再次绑定。

3. `call`与`apply`功能完全相同，唯一不同的是`call`传递函数调用形参是以散列形式，而`apply`的形参是一个数组。在传参的情况下，`call`的性能要高于`apply`，因为`apply`在执行时还要多一步解析数组。

   ```javascript
   // call 与 apply 传参不同
   xxx.call(obj, 123, "abc", "def")
   xxx.apply(obj, [123, "abc", "def"]) // 要放在数组中
   ```

```javascript
var obj1 = { name: 'obj1' };
var obj2 = { name: 'obj2' };
var obj3 = { name: 'obj3' };
function fn() {
  console.log(this);
};

// call() 之类的方法，如果指向参数提供的是null或者undefined，那么this将指向全局对象
fn.call(null);  // window
fn.apply(null); // window
fn.bind(undefined)(); // window  bind不会自动执行，因此需要加()

var bar = fn.bind(obj2); // bind返回的是一个函数，不会改变原函数的this，因此用bar来接受
bar() // obj2  bind不会自动执行，因此需要手动使用
bar.bind(obj3) // 无效，this不能再改变
bar() // obj2  bind不会自动执行，因此需要手动使用

fn.call(obj2); // obj2  this可以继续改变
fn.call(obj3); // obj3

fn.apply(obj2); // obj2  this可以继续改变
fn.apply(obj3); // obj3

fn() // window
```

#### JS API 的内置函数

在JavaScript提供的部分API中也内置了显式绑定

**1.setTimeout**

setTimeout中会传入一个函数，这个函数中的this通常是window。这个和setTimeout源码的内部调用有关，V8引擎中setTimeout内部通过 `apply` 将this绑定为全局对象window。

```javascript
setTimeout(function() {  
  console.log(this); // window
}, 1000);
```

**2.数组的forEach**

高阶函数 `forEach` ，用于函数的遍历：

- 在 `forEach` 中的this也指向window，因为默认情况下传入的函数是自动调用函数（默认绑定）：

```javascript
var names = ['a', 'b', 'c'];
names.forEach(function(item) {  
  console.log(this); // 打印3次window
});

// 可以传入第2个参数，用于指定this绑定
var obj = {name: "d"};
names.forEach(
  function(item) {  
  	console.log(this); // 打印3次obj对象
	}, 
  obj // 第2个参数
);
```

```javascript
// forEach的源码（伪代码）
var fn = 传入的函数;
var arg = obj || window;
var newFn = fn.bind(arg);
newFn();
```

**3.div的点击**

- 在点击事件的回调中，执行传入的回调函数时，会将this绑定到相应的元素。

```javascript
<div class="box"></div>

var box = document.querySelector(".box");
box.onclick = function() {  
  console.log(this); // box对象
}
```

### new绑定

JavaScript中的函数可以当作一个类的构造函数来使用，也就是使用new关键字。

使用new关键字来调用函数时，会执行如下的操作：

1. 创建一个新的空对象 
2. 链接到原型 
3. this指向新的空对象
4. 返回新对象

```javascript
function Animal(name){
  this.name = name;
}
var cat = new Animal("cat"); // this指向创建的 cat

/* 使用new关键字后，所执行的操作 */
var obj = {}; // 创建一个空对象

obj.__proto__ = Animal.prototype; // 链接到原型  把obj的__proto__指向Animal的原型对象prototype，此时便建立了obj对象的原型链：obj -> Animal.prototype -> Object.prototype->null

var result = Animal.call(obj, "cat"); // 绑定this值  将构造函数内部的this绑定到新建的对象obj

return typeof result === 'obj'? result : obj; // 返回新对象  若构造函数没有返回非原始值（即不是引用类型的值），则返回新建的对象obj（默认会添加return this）；否则，返回引用类型的值。简单来说，如果构造函数Animal没有return语句，则将新创建的额对象obj返回，如果有return语句，则执行return语句。
```

下面回到this：

```javascript
// 创建Person
function Person(xxx, yyy) {
  console.log(this); // Person {}
  setTimeout(console.log(this), 1000); // Person {}
  this.xxx = xxx; 
  console.log(this); // Person {xxx: "111"}
  setTimeout(console.log(this), 1000); // Person {xxx: "111"}
  this.yyy = yyy;
  console.log(this); // Person {xxx: "111", yyy: "222"}
}

var p = new Person('111','222');
console.log(p); // Person {xxx: "111", yyy: "222"}
```

### 规则优先级

如果一个函数同时符合多条规则，就需要使用优先级进行判断。

**结论：**

- **显式绑定 > 隐式绑定 > 默认绑定**
- **new绑定 > 隐式绑定 > 默认绑定** 
- **new绑定 > bind()**
- **new绑定 不能与 `call()`、`apply()` 同时使用，无法比较**



**1.默认规则的优先级最低**

默认规则的优先级是最低的。

**2.显示绑定优先级高于隐式绑定**

```javascript
function foo() {
  console.log(this);
}

var obj1 = {
  name: "obj1",
  foo: foo
}

var obj2 = {
  name: "obj2",
  foo: foo
}

// 隐式绑定
obj1.foo(); // obj1

// 隐式绑定和显示绑定同时存在
obj1.foo.call(obj2); // obj2  说明显式绑定优先级更高
```

**3.new绑定优先级高于隐式绑定**

```javascript
function foo() {
  console.log(this);
}

var obj = {
  name: "obj",
  foo: foo
}

obj.foo(); // obj对象
new obj.foo(); // foo对象  说明new绑定优先级高于隐式绑定
```

**4.new绑定优先级高于bind**

new绑定和call、apply是不允许同时使用的，所以不存在谁的优先级更高。

```javascript
function foo() {
  console.log(this);
}

var obj = {
  name: "obj"
}

// new绑定和call、apply不允许同时使用
var foo = new foo.call(obj); //报错：Uncaught TypeError: foo.call is not a constructor 

// 但是new绑定可以和bind后的函数同时使用
var bar = foo.bind(obj);
bar(); // obj  bind绑定后结果为obj
var foo = new bar(); // foo  说明new绑定优先级高于bind
```

## this的特殊情况

### 忽略显示绑定

如果在显示绑定中，传入一个 `null` 或者 `undefined` ，那么这个显示绑定会被忽略，并使用默认规则。

```javascript
function foo() {  
  console.log(this);
}

var obj = {  
  name: "obj"
}

foo.call(obj); // obj对象
foo.call(null); // window
foo.call(undefined); // window

foo.bind(null)(); // window
```

### 间接函数引用

创建一个函数的间接引用，会使用默认绑定规则。

```javascript
// (num2 = num1) 的结果是num1的值
var num1 = 100;
var num2 = 0;
var result = (num2 = num1);
console.log(result); // 100
```

```javascript
// 赋值 (obj2.foo = obj1.foo) 的结果是foo函数
// foo函数被直接调用，那么是默认绑定
function foo() {  
  console.log(this);
}

var obj1 = {  
  name: "obj1",  
  foo: foo
}; 

var obj2 = {  
  name: "obj2"
}

obj1.foo(); // obj1
(obj2.foo = obj1.foo)();  // window  (obj2.foo = obj1.foo)相当于foo函数，因此，(obj2.foo = obj1.foo)(); 相当于 foo(); 独立函数调用
```

### ES6箭头函数

箭头函数没有自己的this，箭头函数的this不是调用的时候决定的，而是在定义的时候处在的对象（外层作用域）就是它的this。

通俗理解： 箭头函数的this看外层的是否有函数：

- 如果有，外层函数的this就是内部箭头函数的this；
- 如果没有，则this是window。

```javascript
<button id="btn">测试箭头函数this_1</button>
<button id="btn2">测试箭头函数this_2</button>

let btn1 = document.getElementById('btn1');
let btn2 = document.getElementById('btn2');

btn1.onclick = function () {
  console.log(this); // btn1  谁调用，this就是谁
}
btn2.onclick = () => {
  console.log(this); // window  定义时所处的对象是window，所以this是window
}

let obj = {
  name: '箭头函数',
  getName: function () {
    btn2.onclick = () => {
      console.log(this);
    }
  }
}
obj.getName(); // obj  箭头函数向上寻找外层作用域，getName的作用域为obj，因此沿用getName的作用域，所以为结果为obj

let obj1 = {
  name: '箭头函数',
  getName: () => { // 相当于obj.getName = () => {}; 
    btn2.onclick = () => {
      console.log(this);
    }
  }
}
obj.getName(); // window  箭头函数向上寻找外层作用域，getName也是箭头函数，因此getName没有自己的作用域，它也是沿用外层作用域，obj1的外层作用域为window，因此沿用obj1的作用域，所以为结果为window
```

## this面试题

### 面试题一

```javascript
var name = "window";
var person = {  
  name: "person",  
  sayName: function () {    
    console.log(this.name);  
  }
};

function sayName() {  
  var sss = person.sayName; 
  sss();  
  
  person.sayName();   
  (person.sayName)();   
  (b = person.sayName)(); 
}

sayName();
```

下面是代码解析：

```javascript
function sayName() {  
  
  var sss = person.sayName;  
  sss(); // window  独立函数调用，没有和任何对象关联，属于隐式丢失的变量赋值情况，相当于默认绑定

  person.sayName(); // person  隐式绑定
  (person.sayName)(); // person  如果是拿到函数sayName，然后用()进行调用，则结果是window。但是，这里把(person.sayName)当作一个整体，这里的()并不是执行的作用，只是告诉解析器这是一个整体，并不是调用执行，这与JavaScript的语法解析器有关，等价于 (person.sayName)(); 
  (b = person.sayName)(); // window  这里又与上面这一条语句不同，(b = person.sayName)这里的括号是执行效果，把执行之后的结果赋值给b，最终，相当于 b(); 。是隐式丢失的变量赋值情况。
}
```

### 面试题二

```javascript
var name = 'window'
var person1 = {  
  name: 'person1',  
  foo1: function () {    
    console.log(this.name)  
  },  
  foo2: () => console.log(this.name),  
  foo3: function () {    
    return function () {      
      console.log(this.name)    
    }  
  },  
  foo4: function () {    
    return () => {      
      console.log(this.name)    
    }  
  }
}
var person2 = { 
  name: 'person2' 
}

person1.foo1(); 
person1.foo1.call(person2); 

person1.foo2();
person1.foo2.call(person2);

person1.foo3()();
person1.foo3.call(person2)();
person1.foo3().call(person2);

person1.foo4()();
person1.foo4.call(person2)();
person1.foo4().call(person2);
```

下面是代码解析：

```javascript
person1.foo1(); // person1  隐式绑定
person1.foo1.call(person2); // person2  显示绑定优先级高于隐式绑定

person1.foo2() // window  foo2()是箭头函数，沿用的是person1的this，而person1的this是window，所以foo2()的this也是window
person1.foo2.call(person2) // window  foo2()是箭头函数，this的优先级规则对于箭头函数无效

person1.foo3()() // window  person1.foo3()拿到函数，然后在window中进行调用，相当于隐式丢失中的变量赋值
person1.foo3.call(person2)() // window  person1.foo3.call(person2)返回一个this指向person2的foo3函数，但是返回的函数真正的调用依然是在window中，所以依然是独立函数调用，默认绑定window，这与上面这一条语句是相同的，只是用call改变了foo3的this的指向，与返回的那个函数无关
person1.foo3().call(person2) // person2  person1.foo3()拿到foo3返回的一个函数，然后通过call()将返回的函数的this显示绑定到person2中，之后call会自动运行这个改变this后的函数，所以是person2。注意：这条语句与上面那条语句中的call()改变的this是不同的，上面那条语句call改变的是foo3的this，而这条语句的call改变的是foo3返回的那个函数的this

person1.foo4()() // person1  person1.foo4()的函数返回的是一个箭头函数，沿用的是foo4的作用域，而foo4的作用域是person1，因此结果为person1
person1.foo4.call(person2)() // person2  先通过person1.foo4.call(person2)将foo4的this指向person2，然后()执行返回箭头函数，箭头函数沿用foo4的this，而此时foo4的this已经变为person2，因此结果为person2
person1.foo4().call(person2) // person1  person1.foo4()返回箭头函数，而箭头函数使用call改变this的指向是不起作用的，然后然后的箭头函数被call()执行，相当于person1.foo4()()，箭头函数沿用foo4的this，而foo4的this指向person1，因此结果为person1
```

### 面试题三

```javascript
var name = 'window'
function Person (name) {
  this.name = name
  this.foo1 = function () {
    console.log(this.name)
  },
  this.foo2 = () => console.log(this.name),
  this.foo3 = function () {
    return function () {
      console.log(this.name)
    }
  },
  this.foo4 = function () {
    return () => {
      console.log(this.name)
    }
  }
}
var person1 = new Person('person1')
var person2 = new Person('person2')

person1.foo1()
person1.foo1.call(person2)

person1.foo2()
person1.foo2.call(person2)

person1.foo3()()
person1.foo3.call(person2)()
person1.foo3().call(person2)

person1.foo4()()
person1.foo4.call(person2)()
person1.foo4().call(person2)
```

下面是代码解析：

```javascript
person1.foo1() // peron1  隐式绑定
person1.foo1.call(person2) // person2  显示绑定优先级大于隐式绑定 

person1.foo2() // person1  foo2是一个箭头函数，沿用foo2的this，foo2的this是person1，因此结果为person1
person1.foo2.call(person2) // person1  foo是一个箭头函数，call对箭头函数无效

person1.foo3()() // window  person1.foo3()返回一个函数，并在window下调用，所以结果为window，相当于隐式丢失的变量赋值
person1.foo3.call(person2)() // window  person1.foo3.call(person2)将返回后的函数的this指向person2，但依然是在window下调用执行，所以结果是window
person1.foo3().call(person2) // person2  person1.foo3()返回的函数后，通过call绑定到person2

person1.foo4()() // person1  person1.foo4()返回一个箭头函数，person1.foo4()()执行返回的箭头函数，上层作用域是person1
person1.foo4.call(person2)() // person2  将person1.foo4的this指向person2，此时箭头函数的上层作用域是改变后的person2
person1.foo4().call(person2) // person1  person1.foo4()返回一个this指向person1的箭头函数，call(person2)对箭头函数无效
```

### 面试题四

```javascript
var name = 'window'
function Person (name) {
  this.name = name
  this.obj = {
    name: 'obj',
    foo1: function () {
      return function () {
        console.log(this.name)
      }
    },
    foo2: function () {
      return () => {
        console.log(this.name)
      }
    }
  }
}
var person1 = new Person('person1')
var person2 = new Person('person2')

person1.obj.foo1()()
person1.obj.foo1.call(person2)()
person1.obj.foo1().call(person2)

person1.obj.foo2()()
person1.obj.foo2.call(person2)()
person1.obj.foo2().call(person2)
```

下面是代码解析：

```javascript
person1.obj.foo1()() // window  person1.obj.foo1()返回一个函数，在window下执行，所以结果为window，相当于隐式丢失的变量赋值
person1.obj.foo1.call(person2)() // window  person1.obj.foo1.call(person2)将foo1的this指向person2，并执行，返回一个函数，虽然花里胡哨的，但返回后的函数依然在window下执行，所以结果为window，本质上和上一条语句相同
person1.obj.foo1().call(person2) // person2  person1.obj.foo1()返回函数，call()将返回的函数的this绑定到person2，然后执行将this指向person2的函数

person1.obj.foo2()() // obj  person1.obj.foo2()返回一个箭头函数，箭头函数的this寻找上层作用域，即obj
person1.obj.foo2.call(person2)() // person2  person1.obj.foo2.call(person2)将foo2的this指向person2，然后再执行，返回一个箭头函数，此时箭头函数的上层作用域变为person2
person1.obj.foo2().call(person2) // obj  person1.obj.foo2()返回一个箭头函数，call对箭头函数无效，相当于person1.obj.foo2()()
```

## 主要的参考来源

1.https://mp.weixin.qq.com/s?__biz=Mzg5MDAzNzkwNA==&mid=2247483847&idx=1&sn=fe8089ded81098b35461d3c14bb85cde&chksm=cfe3f238f8947b2e734221c5131e3a6bc42f2dae66b9640cc0f038e9dffef45dd4a52d8dd930&mpshare=1&scene=23&srcid=&sharer_sharetime=1591262412714&sharer_shareid=ac0071abab7ee9d90efd1d6c4cb28f72#rd

2.https://www.cnblogs.com/echolun/p/11962610.html