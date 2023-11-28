# TypeScript

## 类型的种类

```typescript
语法：
var/let/const 标识符: 数据类型 = 赋值;
												↑
                     类型注解

例子：
const message: string = "hello"

注意：
const message: String = "hello"
当类型注解为大写时，表示的是JavaScript中的包装类的类型，例如String、Number等

类型推导：
			不主动进行类型注解时，会将赋值的值的类型作为前面标识符的类型，这个过程称为类型推导/类型推断
      let foo = "foo"
      foo = 123 // 报错
```

### number

```typescript
// number
let num: number = 100
let num: number = 0b110 // 二进制，也可以是其他进制
```



### boolean

```typescript
// boolean
let flag: boolean = true
flag = 20>30
```



### string

```typescript
// string
let message: string = "Hello"
let info = `my ame is ${message}` // 模板字符串
```



### array

```typescript
// array
// 数组有两种定义方式，注意：开发中推荐第一种写法，因为第二种语法与React的jsx语法有冲突
let names1: string[] = ["a", "b", "c"]
let names2: Array<string> = ["a", "b", "c"] // Array 表示是个数组类型，string表示数组里的元素是字符串类型
names2.push("abc")
```



### object

```typescript
// object
// 对象类型最好是不主动指定而让其推导
const info = {
  name: "aaa",
  age: 18
}

const info: object = {
  name: "aaa",
  age: 18
}
console.log(info.name) // 报错:Property 'name' does not exist on type 'object' 
```



### null & undefined

```typescript
// null & undefined
const n1: null = null // null类型只能赋值null
const n2: undefined = undefined

const n1 = null // 注意，最好主动指定，此时推导的是any类型
```



### symbol

```typescript
// symbol
const s1: symbol = Symbol("title")
const s2: symbol = Symbol("title")
const person = {
  [s1]: "程序员"
  [s2]: "老师"
}
```



### any

```typescript
// any
// 可以对any类型的变量进行任何的操作，包括获取不存在的属性、方法。给一个any类型的变量赋值任何的值，比如数字、字符串的值
// 如果对于某些情况的处理过于繁琐不希望添加规定的类型注解，或者在引入一些第三方库时，缺失了类型注解，这个时候可以使用any：包括在Vue源码中，也会使用到any来进行某些类型的适配
let a: any = "aaa"
a = 123
a = true
const aArray: any[] = ["abc", 18]
```



### unknown

```typescript
// unknown
// 用于描述类型不确定的变量，且只能赋值给unknown的类型，而不能赋值给其他明确的类型
const flag = true
let result: unknown
if (flag) {
  result = foo()
} else {
  result = bar()
}

// unknown可以防止乱用
let message: string = result // 报错
let num: number = result // 报错

if (typeof result === 'string') {
  console.log(result.length)
}
```



### void

```typescript
// void
// void通常用来指定一个函数是没有返回值的，那么它的返回值就是void类型

// 可以将null和undefined赋值给void类型，也就是函数可以返回null或者undefined
function sum(num1: number, num2: number) {
  console.log(num1 + num2)
}

							↓↓↓↓
              
// 这个函数我们没有写任何类型，那么它默认返回值的类型就是void的，我们也可以指定返回值是void
function sum(num1: number, num2: number): void {
  console.log(num1 + num2)
}
```



### never

```typescript
// never
// 表示“无法返回”、"永远不可能"，用于指定一个永远无法返回的函数的返回类型，比如一个函数是一个死循环或者抛出一个异常，那么这个函数不会返回东西，所以写void类型或者其他类型作为返回值类型都不合适，就可以使用never类型
// 用来告知编译器某处代码永远不会执行,从而获得编译器提示, 以避免程序员在一个永远不会(或不应该)执行的地方编写代码

// ----------------1、检查「Unreachable code」---------------------
function foo(): never {
  while(true) {
    console.log('123')
  }
}
foo()
console.log("!!!")  // Error: Unreachable code detected.ts(7027)

// -----

function throwError(msg: string): never {
	throw new Error(msg)
}
throwError("some error")
console.log("!!!")  // Error: Unreachable code detected.ts(7027)


// ---------------2、间接收窄其他数据的类型---------------------
// 后者用never避免了一次空检查
function throwError(){
	throw new Error()
}

function firstChar(msg: string|undefined){
	if (msg == undefined)
		throwError()
	let chr = msg.charAt(1) // ❌Object is possibly 'undefined'.

// ---
    
function throwError(): never{
	throw new Error()
}

function firstChar(msg: string|undefined){
	if (msg == undefined)
		throwError()
	let chr = msg.charAt(1) // ✅

// ---------------3、借助never检查,避免业务性错误---------------------
function handleMessage(message: number | string) {
  switch (typeof message) {
    case 'string':
      console.log('string')
      break
    case 'number':
      console.log('number')
      break
  }
}
// 此时，雅儿贝德如果传入了一个boolean类型
handleMessage(true) // 报错
// 发现报错了，然后雅儿贝德偷偷把handleMessage函数改为：
function handleMessage(message: number | string | boolean) {
  ...
  // 但是雅儿贝德忘了增加相应的case 'boolean'语句进行处理，所以这个函数的代码已经变得不严谨了，但是代码依然是能够运行的
}
  
// 但是如果一开始的函数是这样的：
function handleMessage(message: number | string) {
  switch (typeof message) {
    case 'string':
      console.log('string')
      break
    case 'number':
      console.log('number')
      break
    default:
      let check: never = message // 用never标识，表示 “永远不可能发生/执行” 这条语句，因为上面string和number两种case已经将参数message的所有情况涵盖了，所以default是永远不会发生并执行的
  }
}
// 此时函数变为：
function handleMessage(message: number | string | boolean) {
  ...
}
// 此时，当参数message为新增加的boolean类型时，default中的check就能够被赋值为boolean类型，这就与never表示的 “永远不可能发生/执行” 相冲突，就会在编译阶段就报错，提醒雅儿贝德加上case 'boolean'语句
```



### tuple 

```typescript
// tuple 元组类型
const info: [string, number, number] = ["aaa", 18, 20]
const name = info[0]
console.log(name.length) // ✅
const age = info[1]
console.log(age.length) // ❌报错

// 使用场景
// 例如React中的useState
function useState<T>(state: T): [T, (newState: T) => void] {
  let currentState = state
  const changeState = (newState: T) => {
    currentState = newState
  }
  
  return [currentState, changeState]
}

const [counter, setCounter] = useState(10)
```



### 函数类型

```typescript
// 函数类型

// 函数的参数
function sum(num1: number, num2: number): number {
  return num1 + num2                        ↑
}																				返回值的类型

// 在开发中，通常不写返回值的类型，使其自动推导。但某些第三方库为了方便别人的使用，会主动注明

sum("abc", "bcd") // ❌报错，参数的类型不对
sum(1)  // ❌报错，参数的数量不对，在ts中，函数的参数个数也是要对应的


// ---匿名函数（上下文中的函数）的参数类型---
// 在匿名函数（上下文中的函数）中，可以不添加参数的类型注解
const names = ["abc", "cba", "nba"]
// 参数来自数组，而数组的类型是明确的，是根据上下文的环境推断出来的
// names.forEach(function(item: string) {
names.forEach(function(item) {
  console.log(item.split(""))
})

// ----------------------------------------------------------------------------

// 接收两个参数的函数：num1和num2，并且都是number类型。
// 当返回值是 void 即表示可以返回任何类型，包括null和undefined
type AddFnType = (num1: number, num2: number) => number
const add: AddFnType = (a1: number, a2: number) => {
  return a1 + a2
}

// 注意：在别的语言中，或许可以写成如下，即省略参数名称，只写上参数类型：
type AddFnType = (number, number) => void
// 但是在TypeScript中不行，编译器会将number当成形参的名字，即 number: any

// ---案例---
function calc(n1: number, n2: number, fn: (num1: number, num2: number) => number) {
  return fn(n1, n2)
}

calc(20, 30, function(a1, a2) {
	return a1 + a2     
}
calc(20, 30, function(a1, a2) {
	return a1 * a2     
}   

// 参数的可选类型
// 注意：可选类型必须写在必选类型的后面
// function foo(x?: number, y: number){} // 错误的
function foo(x: number, y?: number){}
foo(20, 30)
foo(20)
```



### 对象类型

```typescript
// 对象类型
// 函数的参数是个对象
// ? 表示是可选的，可传可不传
function printPoint(point: {x:number, y: number}) {
  console.log(point.x)
  console.log(point.y)
}

printPoint({x: 123, y: 321})
```



### 联合类型

```typescript
// 联合类型（Union Type）
// TypeScript的类型系统允许我们使用多种运算符，从现有类型中构建新类型。
// 联合类型是由两个或者多个其他类型组成的类型，表示可以是这些类型中的任何一个值，联合类型中的每一个类型被称之为联合成员（union's members）

function printId(id: number | string | boolean) {
  if (typeof id === 'string') { // narrow
    // 在这种if判断中，TypeScript可以根据缩小的代码结构，推断出更加具体的类型
    console.log(id.toUpperCase())
  } else {
    console.log(id)
  }
}
printId(10)
printId("abc")
printId(true)
```



### 可选类型

```typescript
// 可选类型
// ? 表示是可选的，可传可不传
// 注意：可选类型必须写在必选类型的后面
// function foo(x?: number, y: number){} // 错误的
function printPoint(point: {x:number, y: number, z?: number}) {
  console.log(point.x)
  console.log(point.y)
  console.log(point.z) // 当没有传入值时，为 undefined
}

printPoint({x: 123, y: 321, z: 111})


// 其实，可选类型可以“看作”是 具体类型 和 undefined 的联合类型
// 最主要区别是：当写为联合类型时，是必须传入某个值的，而不能像可选类型一样不传第2或第n个参数

function print(message?: string) {// 相当于 function print(message: string | undefined)
  console.log(message)
}

print()
print("abc")
print(undefined)

print(null) // ❌报错：error: Argument of type 'null' is not assignable to parameter of type 'string | undefined'
```



## 类型的操作

### 类型别名

```typescript
// 类型别名（type alias）
// type 关键字 用于定义类型别名
type IDType = string | number | boolean
function printId(id: IDType) {
  
}

type PointType = {
  x: number,
  y: number,
  z?: number
}
function printPoint(id: PointType) {
  
}
```



### 类型断言

```typescript
// 类型断言（Type Assertions）
// 有时候TypeScript无法获取具体的类型信息，此时需要使用类型断言
// TypeScript只允许类型断言转换为 更具体 或者 不太具体(any/unknwon) 的类型版本，此规则可防止不可能的强制转换

// 案例1
const el = document.getElementById("my-img")
// TypeScript只知道该函数会返回HTMLElement，但并不知道它具体的类型，例如img、div等
el.src = "url地址" // 无法赋值

const el = document.getElementById("my-img") as HTMLImageElement
el.src = "url地址"

// 案例2
class Person {}
class Student extends Person {
  study() {}
}
function sayHello(p: Person) {
  p.study() // 报错
  (p as Student).study() // 可以
}
const stu = new Student()
sayHello(stu) // 传入的是Student的实例

// 3.转换
const message = "Hello"
const num: number = message // 报错
const num: number = (message as any) as number
const num: number = (message as unknown) as number
```



### 非空类型断言

```typescript
// 非空类型断言

function print(message?: string) {
  console.log(message.toUpperCase())
  // 报错，因为传入的message可能为undefined
}

function print(message?: string) {
  console.log(message!.toUpperCase()) // 用 ! 表示可以确定某个标识符是有值的，跳过ts在编译阶段对它的检测
}
```



### 可选链

```typescript
// 可选链
// 可选链事实上并不是TypeScript独有的特性，它是ES11（ES2020）中增加的特性
// 可选链操作符 ?.
// 它的作用是当对象的属性不存在时，会短路，直接返回undefined，如果存在，那么才会继续执行

type Person = {
  name: string
  friend?: {
  	name: string
    age?: number
  	girlFriend?: {
  		name: string
		}
	}
}

const info: Person = {
  name: "why",
  friend: {
    name: "kobe",
    girlFriend: {
      name: "lily"
    }
  }
}

console.log(info.friend.name) // 报错，friend可能为空
console.log(info.friend?.name)
console.log(info.friend?.age)
console.log(info.friend?.girlFriend?.name)
```



### 字面量类型

```typescript
// 字面量类型

// 字符串、数字本身也可以作为类型，即 字面量类型
let message: "Hello" = "Hello"
message = "hhh" // 报错
let num: 123 = 123
num = 321 // 报错

// 使用场景：结合联合类型
type = Alignment = 'left' | 'right' | 'center'
function changeAlign(align: Alignment) {
  console.log('修改方向:', align)
}
changeAlign('left')
changeAlign('hhh') // 报错

// -----------------------------------------------

// 字面量推理
function request(url: string, method: "GET" | "POST") {
  console.log(url, method)
}

const options = {
  url: "www.baidu.com",
  method: "POST"
}
request(options.url, options.method) // options.method会报错，因为推导出的是options.method是个string类型，是可以被赋值成任意字符串的，例如"hhh"，不符合method: "GET" | "POST"

// 方式一：断言
request(options.url, options.method as Method)

// 方式二：
type Request = {
  url: string,
  method: Method
}
const options: Method = {
  url: "www.baidu.com",
  method: "POST"
}

// 方式三：字面量推理
const options = {
  url: "www.baidu.com",
  method: "POST"
} as const
```



### 类型缩小

```typescript
// 类型缩小（Type Narrowing）
// 可以通过类似于 typeof padding === "number" 的判断语句，来改变TypeScript的执行路径
// 在给定的执行路径中，我们可以缩小比声明时更小的类型，这个过程称之为 缩小
// 编写的 typeof padding === "number 可以称之为 类型保护（type guards）
// 常见的类型保护：ptypeof、平等缩小（比如===、!==）、instanceof、in、等等..

// 在 TypeScript 中，检查返回的值typeof是一种类型保护：因为 TypeScript 对如何typeof操作不同的值进行编码
function printId(id: number | string) {
  // console.log(id.toUpperCase()) // 报错
  if (typeof id === 'string') {
    console.log(id.toUpperCase())
  } else {
    console.log(id)
  }
}

// 可以使用Switch或者相等的一些运算符来表达相等性（比如===, !==, ==, and != ）
function turnDirection(direction: 'left' | 'center' | 'right') {
  switch (direction) {
    case 'left':
      console.log('left')
      break
    case 'center':
      console.log('center')
      break
    case 'right':
      console.log('right')
      break
    default: 
      console.log('default')
  }
}

// JavaScript 有一个运算符来检查一个值是否是另一个值的“实例”
function printTime(data: Date | string) {
  if (date instanceof Date) {
    console.log(date.toLocaleString())
  } else {
    console.log(date)
  }
}

// Javascript 有一个运算符，用于确定对象是否具有带名称的属性：in运算符
// 如果指定的属性在指定的对象或其原型链中，则in运算符返回true
type Fish = {swim: () => void}
type Dog = {run: () => void}
function move(animal: Fish | Dog) {
  if ('swim' in animal) {
    animal.swim()
  } else {
    animal.run()
  }
}
```



### 默认参数

```typescript
// 默认参数
// 与ES6的默认参数相同
function foo (x: number, y: number = 6) {}
foo(10)
```



### 剩余参数

```typescript
// 剩余参数
// 与ES6的剩余参数相同
// 会将参数放到一个nums数组中
function sum(...nums: number[]) {
  let total = 0
  for (const num of nums) {
    total += num
  }
  return total
}
sum(10, 20, 30, 40)
```



### 函数的重载

```typescript
// 函数的重载

// 错误写法：
function sum(a1: number|string, a2: number|string): number|string {
  return a1 + a2 // 报错
} // 即使是在函数中使用if判断，也不好，因为：1.需要写很多的if语句；2.调用函数时显示的返回值类型是any


// 正确写法，使用函数的重载：函数的名称相同，但参数不同
// TypeScript中函数的重载没有具体的实现
function sum(a1: number, a2: number): number;
function sum(a1: string, a2: string): string;

function sum(a1: any, a2: any): any { // 注意是any，具体的实现的函数用any
  // 可以是复杂的逻辑
  if (typeof a1 === 'string' && typeof a2 === 'string') {
    return a1.length + a2.length
  }
  return a1 + a2
}

sum({name: "why"}, {age：18}) // 报错，必须能匹配到对应的类型

console.log(sum(20, 30))
console.log(sum('aaa', 'bbb'))

// ---------------------------------------------------
// 使用联合类型来实现
function getLength(a: string | any[]) {
	return a.length
}
// 使用函数重载来实现
function getLength(a: string): number;
function getLength(a: any[]): number;
function getLength(a: any) {
	return a.length 
}
// 开发中，在可能的情况下，尽量选择使用联合类型来实现
```





## this

```typescript
// this

const info = {
  name: "why",
  sayHello() {
    console.log(this.name)
  }
}
info.sayHello() // 可以

// ----------------不明确的this----------------
function sayHello() {
  console.log(this.name)
}
const info = {
  name: "why",
  sayHello
}
info.sayHello() // 报错
// 这里对于sayHello的调用来说，虽然将其放到了info中，通过info去调用，this依然是指向info对象的；但是对于TypeScript编译器来说，这个代码是非常不安全的，因为也有可能直接调用函数，或者通过别的对象来调用函数，例如info.sayHello.bind、info.sayHello.call

// ----------------指定this的类型----------------
// 第一个参数可以传入this
type ThisType = { name: string }
// function sayHello(this: {name: string}, message: string) {
function eating(this: ThisType, message: string) {
  console.log(this.name + " eating " + message)
}
const info = {
  name: "why",
  eating: eating
}
info.eating("hhh") // 可以 隐式绑定
eating("hhh") // 报错
eating.call({name: "kobe"}, "hhh") // 可以 显式绑定
eating.apply({name: "kobe"}, ["hhh"]) // 可以 显式绑定
```



## 类

```typescript
// 类
// 使用class关键字来定义一个类；
// 可以声明一些类的属性：在类的内部声明类的属性以及对应的类型，如果类型没有声明，那么它们默认是any的；也可以给属性设置初始化值；
// 在默认的strictPropertyInitialization模式下面我们的属性是必须初始化的，如果没有初始化，编译时就会报错；如果在strictPropertyInitialization模式下确实不希望给属性初始化，可以使用 name!: string 语法；
// 类可以有自己的构造函数constructor，当通过new关键字创建一个实例时，构造函数会被调用；构造函数不需要返回任何值，默认返回当前创建出来的实例；
// 类中可以有自己的函数，定义的函数称之为方法；

class Person {
	// name!: string 
  name: string
	age: number = 18
	constructor (name: string, age: number) {
		this.name = name
		this.age = age
	}
	running() {
		console.log(this.name + ” running")
	}
	eating() {
		console.log(this.name + ” eating")
	}
}

const p = new Person("why", 18)

// ----------------类的继承----------------
class Student extends Person {
	sno?: number
  
	constructor (name: string, age: number, sno?: number) {
		super(name, age)
		this.sno = sno
	}
  
  eating(){
    super.eating() // 调用父类中的方法
  	... // 子类方法自己的代码
	} // 可以对父类的方法进行重写
  
	studying() {
		console.log(this.name + " studying")
	}
}

new student = new Student('hhh', 18, 123)
student.name = 'hhh'
student.sno = 123
studen.eating()
student.studying()

// ------------------修饰符------------------
// 在TypeScript中，类的属性和方法支持三种修饰符： public、private、protected
// public 在任何地方可见、公有的属性或方法，默认编写的属性就是public的；（默认）
// private 仅在同一类中可见、私有的属性或方法；
// protected 仅在类自身及子类中可见、受保护的属性或方法；
class Person {
  private name: string
  constructor(name: string) {
    this.name = name
  }
}
const p = new Person("why")
console.log(p.name) // 报错


// ------------------只读属性readonly------------------
class Person {
	readonly name: string
  // 只读属性可以在构造器中赋值，赋值之后不能再修改
	constructor(name: string) { 
		this.name = name
	}
}
const p = new Person( "why")
p.name = 'aaa' // 报错

// 注意：readonly只表示本身是不能被修改的，但如果是对象类型，则对象中的属性是可以被修改的

// ------------------getters/setters------------------
// 私有属性不能直接访问，或者某些属性我们想要监听它的获取(getter)和设置(setter)的过程，这时可以使用存取器。
class Person {
	private _name: string
	set name(newName) {
		this._name = newName
	}
	
	get name() {
		return this._name
	}
	constructor(name: string) {
		this.name = name
	}
}
const p = new Person("why")
p.name = "coderwhy"
console.log(p.name)

// ------------------静态成员------------------
// 前面在类中定义的成员和方法都属于对象级别的, 有时候也需要定义类级别的成员和方法。
// 在TypeScript中通过关键字static来定义
class Student {
	static time: string = "20:00"
	static attendClass() {
		console.log("去上课")
	}
}
console.log(Student.time)
Student.attendClass()

// ------------------抽象类abstract------------------
abstract class Shape {
	abstract getArea(): number
}

class Circle extends Shape {
	private r: number
	constructor(r: number) {
		super() 
		this.r = r
  }
  
  getArea() {
		return this.r * this.r * 3.14
	}
}

class Rectangle extends Shape {
	private width: number
	private height: number
  
	constructor (width: number, height: number) {
		super()
		this.width = width
		this.height = height
	}
  
	getArea() {
		return this.width * this.height
	}
}

const circle = new Circle(10)
const rectangle = new Rectangle(20, 30)

function calcArea(shape: Shape) {
	console.log(shape.getArea())
}

calcArea(circle) 
calcArea(rectangle)

// ------------------类的类型------------------
// 类本身也是可以作为一种数据类型
class Person {
	name: string;
	constructor(name: string) {
		this.name = name;
	}
	running() { 
		console.log(this.name + ” running");
	}
}
const p1: Person = new Person("why");
const p2: Person = {
	name: "kobe",
	running: function() {
		console.log(this.name + ” running");
	},
};

```



## 接口

```typescript
// 接口
// 除了通过type声明一个对象类型，还可以通过接口来声明
interface IInfoType {
  readonly name: string // 只读属性
  age?: number // 可选属性
}

const info: IInfoType = {
  name: "aaa"
}

// ----------------索引类型----------------
interface IIndexLanguage {
  [index: number]: string
}

const frontLanguage: IIndexLanguage = {
  0: "HTLM",
  1: "CSS",
  "abc": "cab" // 报错
}

// ----------------函数类型----------------
// type CalcFn = (n1: number, n2: number) => number
interface CalcFn { // 可调用的接口
  (n1: number, n2: number): number
}

function calc(num1: number, num2: number, calcFn: CalcFn) {
  return calcFn(num1, num2)
} 

const add: CalcFn = (num1, num2) => {
  return num1 + num2
}

calc(20, 30, add)

// ----------------接口继承----------------
interface Person { 
	name: string 
	eating: () => void
}
interface Animal {
	running: () => void
}

interface Student extends Person, Animal { // 接口是支持多继承的
	sno: number
}

const stu: Student = {
	sno: 110
	name: "why", 
	eating: function() {},
	running: function() {}
}

// ----------------交叉类型----------------
interface Person { 
	name: string 
	eating: () => void
}
interface Animal {
	running: () => void
}

type Type1 = Person | Animal // 联合类型
type Type2 = Person & Animal // 交叉类型，需要同时满足，相当于 extends Person, Animal

// ----------------接口的实现----------------
interface ISwim {
  swimming: () => void
}
interface IEat {
  eating: () => void
}
class Animal {
  
}

class Fish extends Animal implements ISwim, IEat {
  swimming() {
    
  }
  eating() {
    
  }
}

// 编写一些公共的API：面向接口编程
function swimAction(swimable: ISwim) {
  swimable.swimming()
}
swimAction(new Fish()) // 只要实现了对应的ISwim接口，就可以传入
swimAction({swimming: function() {}})

// ----------------interface和type的区别----------------
// 如果是定义非对象类型：比如Direction、Alignment、一些Function，通常推荐使用type
// 如果是定义对象类型：interface 可以重复的对某个接口来定义属性和方法；而type定义的是别名，别名是不能重复的
interface IPerson {
	name: string
	running: () => void
}
interface IPerson { // 同名的接口会进行合并
	age: number
}

type Person = {
  name: string
  running: () => void
}
type Person = { // 报错
  age: number
}

// ----------------字面量赋值----------------
interface IPerson {
	name: string
	eating: () => void
}

const obj: IPerson = { // 报错
	name: "why",
	age: 18,
	eating: function() {}
}

const obj = { 
	name: "why",
	age: 18,
	eating: function() {}
} // 不会报错，实际上的操作是把对象的引用（指针）进行赋值
// freshness擦除 赋值的过程中在类型检测时，如果擦除多余的属性后依然赋值类型检测，就可以赋值
const p: IPerson = obj

```



## 枚举

```typescript
// 枚举
// 枚举类型是为数不多的TypeScript特性有的特性之一：
// 枚举其实就是将一组可能出现的值，一个个列举出来，定义在一个类型中，这个类型就是枚举类型；
// 枚举允许开发者定义一组命名常量，常量可以是数字、字符串类型；
enum Direction {
	LEFT，
	RIGHT,
	TOP,
	BOTTOM
}

let d: Direction = Direction.LEFT

function turnDirection(direction: Direction) {
	switch (direction) {
		case Direction.LEFT:
			console.log("转向左边")
			break;
		case Direction.RIGHT:
			console.log("转向右边")
			break;
		...
		default:
			const myDirection: never = direction;
	}
}

// -----------------枚举类型的值-----------------
enum Direction {
  LEFT = 0，
	RIGHT = "RIGHT",
	TOP,
	BOTTOM
}
```



## 泛型

```typescript
// 泛型
// 在定义函数时，不决定参数的类型，让函数的调用者以参数的形式传入参数的类型
function sum<Type>(num: Type): Type {  
  return num
}
// 方式1：明确传入类型
sum<number>(20)
sum<{name: string}>({name: "aaa"})
sum<any[]>(["abc"])
// 方式2：类型推导
sum(50) // 字面量类型

// 可以传入多个参数
function foo<T, E>(a1: T, a2: E) {}
foo<number, string>(10, "abc")

// 平时在开发中我们可能会看到一些常用的名称：
// T：Type的缩写，类型
// K、V：key和value的缩写，键值对
// E：Element的缩写，元素
// O：Object的缩写，对象

// ----------------------------泛型接口----------------------------
interface IPerson<T1 = string, T2r> {
  name: T1,
  age: T2
}
const p: IPerson<string, number> = {
  name: "why",
  age: 18
}
// ----------------------------泛型类----------------------------
class Point<T> {
  x: T
  y: T
  z: T
  constructor(x: T, Y: T, z: T) {
    this.x = x
    this.y = y
    this.z = z
  }
}
const p1 = new Point(1, 2, 3)
const p2 = new Point<number>(1, 2, 3)
const p2: Point<number> = new Point(1, 2, 3)

// ----------------------------泛型约束----------------------------
// 有时候希望传入的类型有某些共性，但是这些共性可能不是在同一种类型中：比如string和array都是有length的，或者某些对象也是会有length属性的；那么只要是拥有length的属性都可以作为参数类型
interface ILength {
	length: number
}

function getLength<T extends ILength>(args: T) {
	return args.length
}

console.log(getLength("abc"))
console.log(getLength(["abc", "cba"]))
console.log(getLength({length: 100, name: "why"}))

```



## 声明

```typescript
// 声明
// .d.ts 文件，用来做类型的声明(declare)，仅仅用来做类型检测，告知typescript有哪些类型，不会被编译成.js文件
// 查找类型声明：1、内置类型声明；2、外部定义类型声明；3、自己定义类型声明；
// 内置类型声明是typescript自带的，通常在安装typescript的环境中会带有的
/*	外部类型声明通常是使用一些库（比如第三方库）时，需要的一些类型声明。
		这些库通常有两种类型声明方式：
  		方式一：在自己库中进行类型声明（编写.d.ts文件），比如axios
  		方式二：通过社区的一个公有库DefinitelyTyped存放类型声明文件
  		该库的GitHub地址：https://github.com/DefinitelyTyped/DefinitelyTyped/
			该库查找声明安装方式的地址：https://www.typescriptlang.org/dt/search?search=
			比如我们安装react的类型声明：npm i @types/react --save-dev										
				什么情况下需要自己来定义声明文件
					情况一：我们使用的第三方库是一个纯的JavaScript库，没有对应的声明文件；比如lodash
					情况二：我们给自己的代码中声明一些类型，方便在其他地方直接进行使用；		
*/

// 声明不需要写具体的实现代码

// 声明变量、函数、类
declare let wAge: number;
declare function wFoo(): void
declare class Person : {
	name: string
	age: number
	constructor (name: string, age: number)
}

// 声明模块
declare module "lodash" { // 声明模块的语法: declare module '模块名' {}
  // 在声明模块的内部，我们可以通过 export 导出对应库的类、函数等
  export function join(args: any[]): any;
}

// declare文件
// 在某些情况下，我们也可以声明文件：
// 		比如在开发vue的过程中，默认是不识别.vue文件的，那么就需要对其进行文件的声明；
// 		比如在开发中使用了.jpg这类图片文件，默认typescript也是不支持的，也需要对其进行声明；
declare module '*.vue' {
	import { DefineComponent } from 'vue'
	const component: DefineComponent

	export default component
}

declare module '*.g'
declare module '*.jpg' {
	const src: string
	export default src
}

```



## 补充

### !! 和 ??

```typescript
// !!操作符
// 将一个其他类型转换成boolean类型，类似于Boolean(变量)的方式
const messgae = ""
let flag2 = !!messgae // !messge 相当于直接将message作为boolean类型使用，但是是取反的，因此再来一个 ! 表示取正
// 相当于 let flag1 = Boolean(message)
```

```typescript
// ??操作符
// 是ES11增加的新特性，空值合并操作符（??）是一个逻辑操作符，当操作符的左侧是 null 或者 undefined 时，返回其右侧操作数，否则返回左侧操作数
const message = "321"
const result = message ?? "123"
console.log(result)
// 相当于 const result = message ? message : '123'
```



