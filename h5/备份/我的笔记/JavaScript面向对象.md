# JavaScript面向对象详解（一）

## 一、JavaScript的对象

### 1.1 传统对象 vs JavaScript对象

传统的面向对象

- 面向对象语言的一个标志就是类。
- 类是所有对象的统称，是更高意义上的一种抽象，对象是类的实例。通过类可以创建任意多个具体的对象。
- C++/OC/Java/Python等编程语言，都可以按照这种方式创建类和对象。

JavaScript的面向对象

- ES6之前，JavaScript中没有类的概念，因此通常称为基于对象，而不是面向对象。
- 虽然JavaScript中的基于对象也可以实现类似于类的封装、继承、甚至是多态。但是和传统意义的面向对象还是有一些差异。
- ECMA中对象的定义：无序属性的集合，属性可以包含基本值、对象或者函数。也就是说，对象是一组没有顺序的值组成的集合而已。对象的每个属性或者方法都有一个名字，而名字对应一个值，其实就是经常看到和使用的映射（或者有些语言称为字典，通常会使用哈希表来实现）。

### 1.2 简单的方式创建对象

在**早期**，创建自定义对象最简单的方式就是创建一个Object实例，然后添加属性和方法：

```javascript
// 1.创建person的对象
var person = new Object(）

// 2.给person对象赋值了一些动态的属性和方法
person.name = "xxx"
person.age = 18
person.height = 1.88

person.sayHello = function （） {
  alert("My name is " + this.name)
}

// 3.调用方法，查看结果
person.sayHello(） // xxx
```

**后来**，对象字面量成为创建对象的首选方式

```javascript
// 1.创建对象的字面量
var person = {
    name: "xxx",
    age: 18,
    height: 1.88,
    sayHello: function （） {
        alert("My name is " + this.name)
    }
}

// 2.调用对象的方法
person.sayHello(） // xxx
```

### 1.3 属性的特性（属性描述符）

虽然在实际开发中，大多数情况不会去使用属性描述符，但某些情况下，也确实会用到。

JavaScript中关于属性有一个比较重要的概念：**属性描述符**，用于描述属性特征的特性，根据特性的不同，可以把属性分成两种类型：数据属性和访问器属性。

常见的属性特性：

- [[Configurable]] // true or false

- - 表示能否通过delete删除属性从而重新定义属性，能否修改属性的特性，或者能否把属性修改为访问器属性。像前面例子中那样直接在对象上定义的属性，这个特性默认值为true。

- [[Writable]] // true or false

- - 表示能否修改属性的值。像前面例子中那样直接在对象上定义的属性，这个特性默认值为true。

- [[Enumerable]] // true or false

- - 表示能否通过for-in循环返回属性。像前面例子中那样直接在对象上定义的属性，这个特性默认值为true。

- [[Value]] // everty thing

- - 包含这个属性的数据值。读取属性值的时候，从这个位置读；写入属性值的时候，把新值保存在这个位置。这个特性的默认值为undefined。

- [[set]] // function or undefined

- - 在写入属性时调用的函数。默认值为undefined。

- [[get]] // function or undefined

- - 在读取属性时调用的函数。默认值为undefined。

什么是属性特征?

- 每个特征都有自己特定的用途。比如Configurable配置为false时，就无法使用delete来删除该属性。

- 设置属性特定

- - obj：将要被添加属性或修改属性的对象

  - prop：对象的属性

  - descriptor：对象属性的特性

  - 要想修改属性的特性，必须通过两个Object方法，即 `Object.defineProperty` 和 `Object.defineProperties` ，这两个方法都是用来定义（修改）属性的，前者一次只能定义一个属性，后者则可以多个。

  - 案例练习：

    ```javascript
    var person = {}
    Object.defineProperty(person, "birth"，{
      writable: false,
      value: 2000
    }）
    
    alert(person.birth) // 2000
    person.birth = 1999
    alert(person.birth) // 2000
    ```

    注意：在使用defineProperty方法定义新属性时（非修改旧属性），如果不指定，configurable, enumerable和writable特性的默认值都是false。

    上面的代码等同于：

    ```javascript
    var person = {};
    Object.defineProperty(person, "birth"，{
      configurable: false,
      enumerable: false,
      writable: false,
      value: 2000
    }）; 
    ```

数据属性：

- 数据属性包含一个数值的位置，在这个位置可以读取和写入值。
- 数据属性拥有4个特性：[[Configurable]] 、 [[Enumerable]] 、 [[Writable]] 、 [[Value]]
- 按照上面的方式，定义的属性就是数据属性

访问器属性：

- 访问器属性不包含数据值，包含一对 `getter` 和 `setter` 函数。
- 访问器属性不能直接定义，需要使用 `Object.defineProperty` 函数定义。
- 访问器属性拥有4个特性：[[Configurable]] 、 [[Enumerable]] 、 [[Get]] 、 [[Set]]

定义一个访问器属性：

```javascript
var person = {
  birth: 2000,
  age:  17
};
Object.defineProperty(person, 'year'，{
  get: function （） {
    return this.birth + this.age;
  },
  set: function (newValue) {
    this.age = newValue - this.birth;
  }
}）;

person.year = 2088
alert(person.age) // 88
person.age = 30
alert(person.year) // 2030
```

注意：`getter` 和 `setter` 都是可选的，在非严格模式下，只指定了 `getter` 但进行写入操作，写入的值会被忽略；只指定了 `setter` 但进行读取操作，读取到的属性值为undefined。在严格模式下，则都会报错。

## 二、JavaScript创建对象

> 虽然Object构造函数或对象字面量可以用来创建单个对象，但都有个明显的缺点：使用同一个接口创建多个对象，会产生大量的重复代码。

### 2.1 工厂函数模式

工厂模式是一种常见的设计模式，抽象了创建具体对象的过程。

JavaScript中没法创建类，因此用函数来封装以特定接口创建对象的细节。

工厂模式创建对象：

```javascript
// 创建工厂函数
function createPerson(name, age, height) {
  var o = new Object(）
  o.name = name
  o.age = age
  o.height = height
  o.sayHello = function （） {
    alert("My name is " + this.name)
  }
  
  return o
}

// 创建两个对象
var person1 = createPerson("Coderwhy"，18, 1.88)
var person2 = createPerson("Kobe"，30, 1.98)
person1.sayHello(） // My name is Coderwhy
person2.sayHello(） // My name is Kobe
```

代码解析：

- 函数createPerson(）能够根据接受的参数来构建一个包含所有必要信息的Person对象。
- 可以无数次地调用这个函数，每次都会返回一个包含三个属性一个方法的对象。
- 工厂模式虽然解决了创建多个相似对象的问题，但却没有解决对象识别的问题（怎样知道一个对象的类型）。
- 随着JavaScript的发展，又一个新模式出现。

### 2.2 构造函数模式

JavaScript中的构造函数可用来创建特定类型的对象。

- 像Object和Array这样的原生构造函数，在运行时会自动出现在执行环境中。
- 此外，也可以创建自定义的构造函数，从而定义自定义对象类型的属性和方法。

使用构造函数模式创建对象：

```javascript
// 构造函数
function Person(name, age, height) {
  this.name = name
  this.age = age
  this.height = height
  this.sayHello = function （） {
    alert(this.name)
  }
}

// 使用构造函数创建对象
var person1 = new Person("Coderwhy"，18, 1.88)
var person2 = new Person("Kobe"，30, 1.98)
person1.sayHello(） // Coderwhy
person2.sayHello(） // Kobe
```

代码解析：

- 在这个例子中，Person(）函数取代了createPerson(）函数。

- Person(）函数有一些不太一样的地方：

- - 没有显式地创建对象，比如创建一个Object对象
  - 直接将属性和方法赋给了this对象
  - 没有return语句

- 函数名Person使用的是大写字母P。

- - 按照惯例，构造函数始终都应该以一个大写字母开头，而非构造函数则应该以一个小写字母开头。这个做法借鉴自其他面向对象语言，主要是为了区别于ECMAScript中的其他函数
  - 构造函数本身也是函数，只不过可以用来创建对象

- 在调用函数时，不再只是简单的函数加上（），而是使用了new关键字

- 这种方式调用构造函数实际上会经历以下4个步骤：

- - 创建一个新对象，这个新的对象类型其实就是Person类型
  - 将构造函数的作用域赋给新对象，因此this就指向了这个新对象，也就是this绑定
  - 执行构造函数中的代码，为这个新对象添加属性和方法
  - 返回新对象，但是是默认返回的，不需要使用return语句

在前面例子的最后，person1和person2分别保存着Person的一个不同的实例。

- 这两个对象都有一个constructor（构造函数）属性，该属性指向Person
- 后面我们会详细说道constructor到底从何而来，所以你需要特别知道一下这里有这个属性

```javascript
// constructor属性
alert(person1.constructor === Person) // true
alert(person2.constructor === Person) // true
```

也可以通过instanceof来查看它的类型：

​	注意：person1和person2既是Person类型，也是Object类型。因为默认所有的对象都继承自Object

```javascript
// 使用instanceof查看是否是person或者Object类型
alert(person1 instanceof Object) // true
alert(person1 instanceof Person) // true
alert(person2 instanceof Object) // true
alert(person2 instanceof Person) // true
```

### 2.3 关于构造函数

关于构造函数

- 构造函数也是一个函数，只是使用new关键字进行调用，但也可以像普通的函数一样使用
- 其他任何的函数，也可以通过new关键字来调用，此时这个函数也可以被称为构造函数

```javascript
// 作为构造函数调用
var person = new Person("Coderwhy"，18, 1.88) // person对象
person.sayHello(）

// 作为普通函数调用
Person("Kobe"，30, 1.98) // window对象
window.sayHello(）

// 在另外一个对象的作用域调用
var o = new Object(）
Person.call(o, "Curry"，28, 1.93) // o对象
o.sayHello(）
```

使用构造函数来创建对象也有缺陷：

- 每个方法都要在每个实例上重新创建一遍
- 在前面的例子中，personl和person2都有一个名为sayName(）的方法，但那两个方法不是同一个Function的实例。
- JavaScript中的函数也是对象，因此每定义一个函数，也就是实例化了一个对象

构造函数的换一种形式，也就是上面的代码类似于下面的写法：

```javascript
function Person(name, age, height) {
  this.name = name
  this.age = age
  this.height = height
  this.sayHello = new Function("alert(this.name)"）
}
```

- 从这个角度上来看构造函数，更容易明白每个Person实例都包含一个不同的Function实例
- 但是，有必要创建多个Function实例吗？它们执行的代码完全相同
- 你也许会考虑，它们需要区分不同的对象，不过，在调用函数时，我们传入的this就可以区分了，没有必要创建出多个Function的实例

我们可以验证一下这是两个不同的函数：

```javascript
alert(person1.sayHello === person2.sayHello) // false
```

有没有办法让它们是同一个函数呢? 使用全局函数即可

```javascript
// 定义全局和函数
function sayHello(） {
  alert(this.name)
}

// 构造函数
function Person(name, age, height) {
  this.name = name
  this.age = age
  this.height = height
  this.sayHello = sayHello
}

// 使用构造函数创建对象
var person1 = new Person("Coderwhy"，18, 1.88)
var person2 = new Person("Kobe"，30, 1.98)

alert(person1.sayHello === person2.sayHello) // true
```

新的问题：

- 这样做解决了两个函数做同一件事的问题，但是，在全局作用域中定义的函数的目的却是只能被某个对象调用，这让全局作用域有点名不副实。
- 进一步思考：如果对象需要定义很多方法，那么就要定义很多个全局函数，就没有封装性可言了。

新的解决方案就是使用原型模式。

# JavaScript面向对象详解（二）

## 一、理解原型模式

### 1.1 什么是原型呢?

每个函数都有一个prototype（原型）属性；这个属性是一个指针，指向一个对象；这个对象的作用是存放这个类型创建的所有实例共享的属性和方法；指向的这个对象，就是我们的所谓的原型对象。

原型对象的作用：可以让所有对象实例共享它所包含的属性和方法。换句话说，不必在构造函数中定义对象实例的信息，而是可以将这些信息直接添加到原型对象中。

```javascript
// 创建对象的构造函数
function Person(） {}

// 通过原型对象来设置一些属性和值，可以让所有的对象来共享这些属性和方法
Person.prototype.name = "Coderwhy"
Person.prototype.age = 18
Person.prototype.height = 1.88
Person.prototype.sayHello = function （） {
  alert(this.name)
}

// 创建两个对象，并且调用方法
var person1 = new Person(）
var person2 = new Person(）

person1.sayHello(） // Coderwhy
person2.sayHello(） // Coderwhy
```

### 1.2 深入原型对象

原型对象的创建：

- 无论什么时候，只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个prototype属性，这个属性指向函数的原型对象。

原型上的constructor属性：

- 默认情况下，所有原型对象都会自动获得一个constructor（构造函数）属性，这个属性包含一个指向prototype属性所在函数的指针。
- 用我们上面的例子来说，`Person.prototype.constructor` 指向Person。
- 也就是原型对象自身来说，自带的只有一个 `constructor` 属性，而其他属性可以由我们自行添加或者从`Object` 中继承。

新的实例创建时，原型对象在哪里呢？

- 当调用构造函数创建一个新实例后，该实例的内部将包含一个内部属性，该属性的指针，指向构造函数的原型对象。
- 简单说，每个实例中，其实也会有一个属性，该属性是指向原型对象的，这个属性是  `__proto__`

```javascript
// 原型对象中有一个constructor属性，指向Person函数
console.log(Person.prototype.constructor); // Person函数
 
// 对象实例也有一个`__proto__`属性指向原型
console.log(person1.__proto__); // 原型对象
console.log(Person.prototype); // 原型对象
console.log(person1.__proto__ === Person.prototype); // true
```

通过一个图来解释上面的概念：

 ![img](JavaScript面向对象.assets/640-1592708655981.webp) 

解析：

- 上图解析了 Person构造函数、Person的原型属性 以及 Person现有的两个实例 之间的关系。
- `Person.prototype` 指向原型对象，而 `Person.prototype.constructor` 指回了 Person。
- 原型对象中除了包含constructor属性之外，还包括后来添加的其他属性。
- Person的每个实例 person1 和 person2 都包含一个内部属性 `__proto__`，该属性也指向原型对象。

对象搜索属性和方法的过程：

- 每当代码读取某个对象的某个属性时，都会执行一次搜索，也就是要找到给定名称的属性。

- 搜索首先从*对象实例本身*开始

- - 如果在实例中找到了具有给定名字的属性，则返回该属性的值。

- - 如果没有找到，则继续搜索指针指向的原型对象，在原型对象中查找具有给定名字的属性。如果在原型对象中找到了这个属性，则返回该属性的值。

- 也就是说，在调用 `personl.sayHello(）` 的时候，会先后执行两次搜索，一次搜索实例，如果没有找到，则继续搜索原型对象。

- 现在我们也能理解，为什么所有的实例中都包含一个constructor属性，这是因为默认所有的原型对象中都包含了该属性。

可以通过`__proto__`来修改原型的值（通常不会这样修改，知道即可）

```javascript
person1.sayHello(） // Coderwhy
person2.sayHello(） // Coderwhy

person1.__proto__.name = "Kobe" // person1修改了原型上的name后，person2也会修改

person1.sayHello(） // Kobe
person2.sayHello(） // Kobe
```

但是要注意下面的情况：

- 当我们给 `person1.name` 进行赋值时，其实在给person1实例添加一个name属性。
- 这个时候再次访问时，就不会访问原型中的name属性了。

```javascript
// 创建两个对象，并且调用方法
var person1 = new Person(）
var person2 = new Person(）

person1.sayHello(） // Coderwhy
person2.sayHello(） // Coderwhy

// 给person1实例添加属性
person1.name = "Kobe"
person1.sayHello(） // Kobe  来自实例
person2.sayHello(） // Coderwhy  来自原型
```

可以通过 `hasOwnProperty` 判断属性属于实例还是原型：

```javascript
// 判断属性属于谁
alert(person1.hasOwnProperty("name"）） // true
alert(person2.hasOwnProperty("name"）） // false
```

### 1.3 简洁的原型语法

- 按照前面的做法，每添加一个原型属性和方法，都要敲一遍 ` Person.prototype` 。
- 为了减少不必要的输入，以及更好的封装性，通常是用一个包含所有属性和方法的对象字面量来一次性重写整个原型对象。

```javascript
// 定义Person构造函数
function Person(） {}

// 重写Person的原型属性
Person.prototype = {
  name: "Coderwhy",
  age: 18,
  height: 1.88,
  sayHello: function （） {
    alert(this.name)
  }
}
```

注意：

- 我们将 `Person.prototype` 赋值了一个新的对象字面量，最终结果和原来是一样的。
- 但是constructor属性不再指向Person。
- 前面说过，每创建一个函数，就会同时创建它的prototype对象，这个对象也会自动获取constructor属性。
- 而这里相当于给prototype重新赋值了一个对象，那么这个新对象的constructor属性，会指向Object构造函数，而不是Person构造函数了。

```javascript
// 创建Person对象
var person = new Person(）

alert(person.constructor === Object) // true
alert(person.constructor === Person) // false

alert(person instanceof Person) // true
```

如果在某些情况下，确实需要用到constructor的值，可以手动给constructor赋值

```javascript
// 定义Person构造函数
function Person(） {}

// 重写Person的原型属性
Person.prototype = {
  constructor: Person, // 手动给constructor赋值
  name: "Coderwhy",
  age: 18,
  height: 1.88,
  sayHello: function （） {
    alert(this.name)
  }
}

// 创建Person对象
var person = new Person(）

alert(person.constructor === Object) // false
alert(person.constructor === Person) // true

alert(person instanceof Person) // true
```

上面的方式虽然可以，但是也会造成constructor的 `[[Enumerable]]` 特性被设置了true。

默认情况下，原生的constructor属性是不可枚举的。如果希望解决这个问题，可以使用前面介绍的 `Object.defineProperty(）` 函数。

```javascript
// 定义Person构造函数
function Person(） {}

// 重写Person的原型属性
Person.prototype = {
  name: "Coderwhy",
  age: 18,
  height: 1.88,
  sayHello: function （） {
    alert(this.name)
  }
}

Object.defineProperty(Person.prototype, "constructor"，{
  enumerable: false,
  value: Person
}）
```

### 1.4 修改原型属性

考虑下面的代码执行是否会有问题：

```javascript
// 定义Person构造函数
function Person(） {}

// 创建Person的对象
var person = new Person(）

// 给Person的原型添加方法
Person.prototype.sayHello = function （） {
  alert("Hello JavaScript"）
}

// 调用方法
person.sayHello(）
```

代码解析：

- 我们发现代码的执行没有任何问题。
- 因为在创建person的时候，person的 `__proto__` 也是指向的Person.prototype。
- 所以，当动态的修改了Person.prototype中的sayHello属性时，person中也可以获取到该属性。

图解上面的过程：

![img](JavaScript面向对象.assets/640.webp)

再来看下面的代码会不会有问题：

```javascript
// 定义Person构造函数
function Person(） {}

// 创建Person的对象
var person = new Person(）

// 给Person的原型添加方法
Person.prototype = {
  constructor: Person,
  sayHello: function （） {
    alert("Hello JavaScript"）
  }
}
// 调用方法
person.sayHello(）
```

代码解析：

- 代码不能正常运行。
- 因为Person的prototype指向了一个新的对象。
- 而最初创建的person依然指向原来的原型对象，原来的原型对象没有sayHello(）函数。
- 当然，如果在此之后，再创建的Person对象，是可以调用sayHello(）的，但是在此之前创建的，没有该方法。

图解上面的过程：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)![img](JavaScript面向对象.assets/640-1592708535552.webp) 

### 1.5 原型对象缺点

原型对象也有一些缺点：

- 它不再有为构造函数传递参数的环节，所有实例在默认情况下都将有相同的属性值。
- 原型中所有的属性是被很多实例共享的，这种共享对于函数来说非常适合，对于基本属性通常情况下也不会有问题（因为通过person.name直接修改时，会在实例上重新创建该属性名，不会在原型上修改。除非使用person.__proto__.name修改）。但是，对于引用类型的实例，就必然会存在问题。

考虑下面代码的问题：

```javascript
// 定义Person构造函数
function Person(） {}

// 设置Person原型
Person.prototype = {
  constructor: Person,
  name: "Coderwhy",
  age: 18,
  height: 1.88,
  hobby: ["Basketball"，"Football"],
  sayHello: function （） {
    alert("Hello JavaScript"）
  }
}

// 创建两个person对象
var person1 = new Person(）
var person2 = new Person(）

alert(person1.hobby) // Basketball,Football
alert(person2.hobby) // Basketball,Football

person1.hobby.push("tennis"）

alert(person1.hobby) // Basketball,Football,tennis
alert(person2.hobby) // Basketball,Football,tennis
```

我们会发现，我们明明给person1添加了一个爱好，但是person2也被添加到一个爱好

- 因为它们是共享的同一个数组。
- 但是，我们希望每个人有属于自己的爱好，而不是所有的Person爱好都相同。

## 二、组合构造函数和原型模式

> 创建自定义类型的最常见方式，就是组合使用构造函数模式与原型模式。
>
> 构造函数模式用于定义实例属性，
>
> 原型模式用于定义方法和共享的属性。
>
> 结果，每个实例都会有自己的一份实例属性的副本，但同时又共享着对方法的引用，最大限度地节省了内存。
>
> 另外，这种混成模式还支持向构造函数传递参数；可谓是集两种模式之长。

组合构造函数和原型模式的代码

```javascript
// 创建Person构造函数
function Person(name, age, height) {
  this.name = name
  this.age = age
  this.height = height
  this.hobby = ["Basketball"，"Football"]
}

// 重新Peron的原型对象
Person.prototype = {
  constructor: Person,
  sayHello: function （） {
    alert("Hello JavaScript"）
  }
}

// 创建对象
var person1 = new Person("Coderwhy"，18, 1.88)
var person2 = new Person("Kobe"，30, 1.98)

// 测试是否共享了函数
alert(person1.sayHello == person2.sayHello) // true

// 测试引用类型是否存在问题
person1.hobby.push("tennis"）
alert(person1.hobby)
alert(person2.hobby)
```

person1和person2各有一份自己的属性，但是方法是共享的。

事实上，还有一些其他的变种模式来实现基于对象的封装，但是这种方式是最常用的，这里不再展开讨论其他的模式，后续需要我们再深入讨论。

# JavaScript面向对象详解（三）

> 继承是面向对象中非常重要的特性。
>
> ES5中和类的实现一样，不能直接实现继承，实现继承主要是依靠原型链来实现的。

## 一、原型链

> 原型链是ES5中实现继承的主要手段，因此相对比较重要，我们需要深入理解原型链。

### 1.1 深入理解原型链

先来回顾一下构造函数、原型和实例的关系：

- 每个构造函数都有一个原型对象，通过prototype指针指向该原型对象。
- 原型对象都包含一个指向构造函数的指针，通过constructor指针，指向构造函数。
- 而实例都包含一个指向原型对象的内部指针，该内部指针我们通常使用 `__proto__` 来描述。

思考如下情况：

- 我们知道，可以通过 `Person.prototype = {}` 的方式来重写原型对象。
- 假如，后面赋值的不是一个 `{}`，而是另外一个类型的实例，此时的原型对象将包含一个指向另一个原型的指针，相应地，另一个原型中也包含着一个指向另一个构造函数的指针。
- 假如另一个原型又是另一个类型的实例，那么上述关系依然成立，如此层层递进，就构成了实例与原型的链条。这就是所谓原型链的基本概念。

通过代码来理解：

```javascript
// 创建Person构造函数
function Person(） {
}

// 设置Animal的原型
Person.prototype = {
}
```

我们将代码修改成原型链的形式：

```javascript
// 1.创建Animal的构造函数
function Animal(） {
  this.animalProperty = "Animal"
}

// 2.给Animal的原型中添加一个方法
Animal.prototype.animalFunction = function （） {
  alert(this.animalProperty)
}

// 3.创建Person的构造函数
function Person(） {
  this.personProperty = "Person"
}

// 4.给Person的原型对象重新赋值
Person.prototype = new Animal(）

// 5.给Person添加属于自己的方法
Person.prototype.personFunction = function （） {
  alert(this.personProperty)
}

// 6.创建Person的实例
var person = new Person(）
person.animalFunction(）
person.personFunction(）
```

代码解析：

- 重点是第4步代码：给Person.prototype赋值了一个Animal的实例，也就是Person的原型变成了Animal的实例。

- Animal实例本身有一个` __proto__` 可以指向Animal的原型。

- 那么，思考一个问题：如果现在搜索一个属性或者方法，这个时候会按照什么顺序搜索呢?

- 1. 在person实例中搜索，搜索到直接返回或者调用函数。如果没有执行第二步。

- 2. 在Person的原型中搜索，Person的原型是谁？Animal的实例，所以会在Animal的实例中搜索，无论是属性还是方法，如果搜索到则直接返回或者执行。如果没有，执行第三步。

- 3. 在Animal的原型中搜索，搜索到返回或者执行，如果没有，搜索结束。当然其实还有Object，但是先不考虑。

画图解析可能更加清晰：

当代码执行到第3步（上面代码的序号）的时候，如图所示：

![img](JavaScript面向对象.assets/640-1592879253666.webp)

当代码执行第4步（上面代码的序号）时，发生了如图所示的变化：

注意图片中的红色线，原来指向的是谁，现在指向的是谁。

![img](JavaScript面向对象.assets/640-1592879253727.webp)

代码继续执行

- `Person.prototype.personFunction = function （）{}`
- 当执行第5步，也就是给Person的原型赋值了一个函数时，事实上在给new Animal（Animal的实例）赋值了一个新的方法。

![img](JavaScript面向对象.assets/640-1592879253677.webp)

代码继续执行，我们创建了一个Person对象

- 创建Person对象，person对象会有自己的属性 personProperty 。
- 另外，person对象有一个 `__prototype__` 指向Person的原型，即之前的new Animal（Animal的一个实例），所以会指向它。

原型链简单总结：

- 通过实现原型链，本质上扩展了本章前面介绍的原型搜索机制。
- 当以读取模式访问一个实例属性时，首先会在实例中搜索该属性。如果没有找到该属性，则会继续搜索实例的原型。在通过原型链实现继承的情况下，搜索过程就得以沿着原型链继续向上。
- 在找不到属性或方法的情况下，搜索过程总是要一环一环地前行到原型链末端才会停下来。

### 1.2 原型和实例的关系

如果我们希望确定原型和实例之间的关系，有两种方式：

1. 使用 `instanceof` 操作符，只要用这个操作符来测试实例与原型链中出现过的构造函数，结果就会返回true。

   ```javascript
   // instanceof
   alert(person instanceof Object) // true
   alert(person instanceof Animal) // true
   alert(person instanceof Person) // true
   ```

2. 使用 `isPrototypeOf(）` 方法。同样，只要是原型链中出现过的原型，都可以说是该原型链所派生的实例的原型，结果就会返回true。

   ```javascript
   // isPrototypeOf函数
   alert("isPrototypeOf函数函数"）
   alert(Object.prototype.isPrototypeOf(person)） // true
   alert(Animal.prototype.isPrototypeOf(person)） // true
   alert(Person.prototype.isPrototypeOf(person)） // true
   ```

### 1.3 添加新的方法

在第5步操作中，我们为子类型添加了一个新的方法。但是这里有一个注意点，无论是子类中添加新的方法，还是对父类中方法进行重写。都一定要将添加方法的代码，放在替换原型语句之后，否则，添加的方法将会无效。

错误代码引起的代码：

```javascript
// 1.定义Animal的构造函数
function Animal(） {
  this.animalProperty = "Animal"
}

// 2.给Animal添加方法
Animal.prototype.animalFunction = function （） {
  alert(this.animalProperty)
}

// 3.定义Person的构造函数
function Person(） {
  this.personProperty = "Person"
}

// 4.给Person添加方法
Person.prototype.personFunction = function （） {
  alert(this.personProperty)
}

// 5.给Person赋值新的原型对象
Person.prototype = new Animal(）

// 6.创建Person对象，并且调用方法
var person = new Person(）
person.personFunction(） // 不会有任何弹窗，因为找不到该方法
```

代码解析：

- 执行上面的代码不会出现任何的弹窗，因为我们添加的方法是无效的，被赋值的新的原型覆盖了。
- 正确的办法是将第4步和第5步操作换一下位置即可。

### 1.4 原型链的问题

原型链对于继承来说：

- 原型链似乎对初学JavaScript原型的人来说，已经算是比较高明的设计技巧了，有些人理解起来都稍微有些麻烦。
- 但是，这种设计还存在一些缺陷，不是最理性的解决方案。（但是后续的解决方案也是依赖原型链，无论如何都需要先理解它）

原型链存在的问题：

- 原型链存在最大的问题是关于引用类型的属性。
- 通过上面的原型实现了继承后，子类的person对象继承了（可以访问)Animal实例中的属性(animalProperty)。
- 但是如果这个属性是一个引用类型（（如数组或者其他引用类型），就会出现问题。

引用类型的问题代码：

```javascript
// 1.定义Animal的构造函数
function Animal(） {
  this.colors = ["red"，"green"]
}

// 2.给Animal添加方法
Animal.prototype.animalFunction = function （） {
  alert(this.colors)
}

// 3.定义Person的构造函数
function Person(） {
  this.personProperty = "Person"
}

// 4.给Person赋值新的原型对象
Person.prototype = new Animal(）

// 5.给Person添加方法
Person.prototype.personFunction = function （） {
  alert(this.personProperty)
}

// 6.创建Person对象，并且调用方法
var person1 = new Person(）
var person2 = new Person(）

alert(person1.colors) // red,green
alert(person2.colors) // red,green

person1.colors.push("blue"）

alert(person1.colors) // red,green,blue
alert(person2.colors) // red,green,blue
```

代码解析：

- 第6步的操作，创建了两个对象。
- 查看它们的colors属性。
- 给person1中的colors属性添加一个新的颜色blue。
- 再次查看两个对象的colors属性，会发现person2的colors属性也发生了变化。
- 两个实例应该是相互独立的，这样就可能导致一些问题。

原型链的其他问题：

- 在创建子类型的实例时，不能向父类型的构造函数中传递参数。
- 实际上，应该说是没有办法在不影响所有对象实例的情况下，给父类型的构造函数传递参数。
- 从而可以修改父类型中属性的值，在创建构造函数的时候就确定一个值。

## 二、经典继承

> 为了解决原型链继承中存在的问题，开发人员提供了一种新的技术： constructor stealing（有很多名称：借用构造函数或经典继承或伪造对象）, steal是偷窃的意思，但是这里可以翻译成借用。

### 2.1 经典继承的思想

经典继承的做法非常简单：在子类型构造函数的内部调用父类型构造函数。

- 因为函数可以在任意的时刻被调用
- 因此通过 `apply(）` 和 `call(）` 方法也可以在新创建的对象上执行构造函数。

经典继承代码如下：

```javascript
// 创建Animal的构造函数
function Animal(） {
  this.colors = ["red"，"green"]
}

// 创建Person的构造函数
function Person(） {
  Animal.call(this) // 继承Animal的属性
  this.name = "Coderwhy" // 给自己的属性赋值
}

// 创建Person对象
var person1 = new Person(）
var person2 = new Person(）

alert(person1.colors) // red,greem
alert(person2.colors) // red,greem
person1.colors.push("blue"）
alert(person1.colors) // red,green,blue
alert(person2.colors) // red,green
```

代码解析：

- 通过在Person构造函数中，使用call函数，将this传递进去。
- 这个时候，当Animal中有相关属性初始化时，就会在this对象上进行初始化操作。
- 这样就实现了类似于继承Animal属性的效果。

这个时候，我们也可以传递参数，修改上面的代码：

```javascript
// 创建Animal构造函数
function Animal(age) {
  this.age = age
}

// 创建Person构造函数
function Person(name, age) {
  Animal.call(this, age)
  this.name = name
}

// 创建Person对象
var person = new Person("Coderwhy"，18)
alert(person.name)
alert(person.age)
```

### 2.2 经典继承的问题

经典继承的问题：

- 对于经典继承理解比较深入，你已经能发现：经典继承只有属性的继承，无法实现方法的继承。
- 因为调用call函数，将this传递进去，只能将父构造函数中的属性初始化到this中。
- 但是如果函数存在于父构造函数的原型对象中，this中是不会有对应的方法的。

回顾原型链和经典继承：

- 原型链存在的问题是引用类型问题和无法传递参数，但是方法可以被继承。
- 经典继承是引用类型没有问题，也可以传递参数，但是方法无法被继承。
- 因此，可以将两者结合。

## 三、组合继承

> 如果你认识清楚了上面两种实现继承的方式存在的问题，就可以很好的理解组合继承了。
>
> 组合继承（combination inheritance, 有时候也称为伪经典继承），就是将原型链和经典继承组合在一起，从而发挥各自的优点。

### 3.1 组合继承的思想

组合继承：

- 组合继承就是发挥原型链和经典继承各自的优点来完成继承的实现。
- 使用原型链实现对原型属性和方法的继承。
- 通过经典继承实现对实例属性的继承，以及可以在构造函数中传递参数。

组合继承的代码：

```javascript
/* 1.创建构造函数的阶段 */
// 1.1.创建Animal的构造函数
function Animal(age) {
  this.age = age
  this.colors = ["red"，"green"]
}

// 1.2.给Animal添加方法
Animal.prototype.animalFunction = function （） {
  alert("Hello Animal"）
}

// 1.3.创建Person的构造函数
function Person(name, age) {
  Animal.call(this, age)
  this.name = name
}

// 1.4.给Person的原型对象重新赋值
Person.prototype = new Animal(0)

// 1.5.给Person添加方法
Person.prototype.personFunction = function （） {
  alert("Hello Person"）
}

/* 2.验证和使用的代码 */
// 2.1.创建Person对象
var person1 = new Person("Coderwhy"，18)
var person2 = new Person("Kobe"，30)

// 2.2.验证属性
alert(person1.name + "-" + person1.age) // Coderwhy,18
alert(person2.name + "-" + person2.age) // Kobe,30

// 2.3.验证方法的调用
person1.animalFunction(） // Hello Animal
person1.personFunction(） // Hello Person

// 2.4.验证引用属性的问题
person1.colors.push("blue"）
alert(person1.colors) // red,green,blue
alert(person2.colors) // red,green
```

### 3.2 组合继承的分析

组合继承是JavaScript最常用的继承模式之一。但是它依然不是很完美，存在一些细节问题：

- 无论在什么情况下，都会调用两次父类构造函数：
  1. 一次在在创建子类原型的时候；
  2. 另一次在子类构造函数内部（也就是每次创建子类实例的时候）。
- 所有的子类实例事实上会拥有两份父类的属性，当然，这两份属性我们无需担心访问出现问题，因为默认一定是访问实例本身这一部分的：
  1. 一份在当前的实例自己里面（也就是person本身的）；
  2. 另一份在子类对应的原型对象中（也就是`person.__proto__`里面）

怎么解决呢?

# JavaScript面向对象详解（四）

## 一、原型式继承

### 1.1 原型式继承的思想

原型式继承的渊源：

- 这种模式要从道格拉斯·克罗克福德（Douglas Crockford, 著名的前端大师, JSON的创立者）在2006年写的一篇文章说起: Prototypal Inheritance in JavaScript(在JS中使用原型式继承）
- 在这篇文章中，它介绍了一种继承方法，而且这种继承方法不是通过构造函数来实现的。
- 为了理解这种方式，先再次回顾一下JavaScript想实现继承的目的：重复利用另外一个对象的属性和方法。

原型式继承的核心函数:

```javascript
// 封装object(）函数
function object(o) {
  function F(） {}
  F.prototype = o
  return new F(）
}
```

代码解析:

- 在object(）函数内部，先创建一个临时的构造函数。
- 然后将传递的对象作为这个构造函数的原型。
- 最后返回这个临时类型的一个新的实例。
- 事实上，object(）对传入的对象执行了一次浅复制。

### 1.2 原型式继承的使用

使用原型式继承:

```javascript
// 使用原生式继承
var person = {
  name: "Coderwhy",
  colors: ["red", "green"]
}

// 通过person去创建另外一个对象
var person1 = object(person)
person1.name = "Kobe"
person1.colors.push("blue"）

alert(person1.name) // Kobe
alert(person1.colors) // red,green,blue

alert(person.name) // Coderwhy
alert(person.colors) // red,green,blue
```

代码解析:

- 这种方式和我们传统意义上理解的继承有些不同。 它做的事情是通过一个对象去创建另外一个对象（利用person去创建person1）。
- 当然，person1中继承过来的属性是放在了自己的原型对象中的。
- 也可以给person1自己再次添加name属性，这个时候name才是在实例本身中。
- 但是如果是修改或者添加引用类型的内容，还是会引起连锁反应。
- 可能暂时你看不到这些代码的意义，但是这些代码是我们后续终极方案的前提思想。

针对这种思想，ES5中新增了Object.create(）方法来规范化了原型式继承。

也就是上面的代码可以修改成这样（只是将object函数修改成了Object.create）：

```javascript
// 使用原生式继承
var person = {
  name: "Coderwhy",
  colors: ["red", "green"]
}

// 通过person去创建另外一个对象
var person1 = Object.create(person)
person1.name = "Kobe"
person1.colors.push("blue")

alert(person1.name) // Kobe
alert(person1.colors) // red,green,blue

alert(person.name) // Coderwhy
alert(person.colors) // red,green,blue
```

Object.create(）还可以传入第二个参数，用于每个属性的自定义描述。

比如person1的name我们希望修改成"Kobe"，就可以这样来做：

```javascript
// 使用原型式继承
var person = {
  name: "Coderwhy",
  colors: ["red", "green"]
}

// 通过person去创建另外一个对象
var person1 = Object.create(person, {
  name: {
    value: "Kobe"
  }
})
person1.colors.push("blue")

alert(person1.name) // Kobe
alert(person1.colors) // red,green,blue

alert(person.name) // Coderwhy
alert(person.colors) // red,green,blue
```

### 1.3 原型式继承的问题

原型式继承的的优点和缺点:

- - 如果我们只是希望一个对象和另一个对象保持类似的情况下，原型式继承完全可以胜任，这是它的优势。
  - 但是，原型式继承依然存在属性共享的问题，就像使用原型链一样。

## 二、寄生式继承

### 2.1 寄生式继承的思想

寄生式(Parasitic)继承

- 寄生式(Parasitic)继承是与原型式继承紧密相关的一种思想。
- 思路是结合原型类继承和工厂模式。即创建一个封装继承过程的函数，该函数在内部以某种方式来增强对象，最后再将这个对象返回。

寄生式函数多增加了一个核心函数:

```javascript
// 封装object函数
function object(o) {
  function F() {}
  F.prototype = o
  return new F()
}

// 封装创建新对象的函数
function createAnother(original) {
  var clone = object(original)
  clone.sayHello = function () {
    alert("Hello JavaScript")
  }
  return clone
}
```

### 2.2 寄生式继承的应用

来使用一下寄生式继承：

```javascript
// person对象
var person = {
  name: "Coderwhy",
  colors: ["red", "green"]
}

// 新的对象
var person1 = createAnother(person) // 基于person对象，创建了另外一个对象person1
person1.sayHello() // 在最新的person1对象中，不仅会拥有person的属性和方法，而且还有自己定义的方法
```

### 2.3 寄生式继承的问题

寄生式继承存在的问题：

- 寄生式继承和原型式继承存在一样的问题，引用类型会共享（因为是在原型式继承基础上的一种封装）。
- 另外，寄生式继承还存在函数无法复用的问题，因为每次createAnother一个新的对象，都需要重新定义新的函数。

## 三、寄生组合式继承

### 3.1 寄生组合式继承的思想

寄生组合式继承

- 现在我们来回顾一下之前提出的比较理想的组合继承

- ​	组合继承是比较理想的继承方式，但是存在两个问题：

- 1. 构造函数会被调用两次：一次在创建子类型原型对象的时候，一次在创建子类型实例的时候。

- 2. 父类型中的属性会有两份：一份在原型对象中，一份在子类型实例中。

- 事实上，我们现在可以利用寄生式继承将这两个问题给解决掉。

- - 需要先明确一点：当我们在子类型的构造函数中调用父类型.call(this, 参数) 这个函数的时候，就会将父类型中的属性和方法复制一份到了子类型中。所以父类型本身里面的内容，我们不再需要。
  - 这个时候，我们还需要获取到一份父类型的原型对象中的属性和方法。
  - 能不能直接让子类型的原型对象 = 父类型的原型对象呢？
  - 不要这么做，因为这么做意味着以后修改了子类型原型对象的某个引用类型的时候，父类型原生对象的引用类型也会被修改。
  - 我们使用前面的寄生式思想就可以了。

寄生组合式的核心代码:

```javascript
// 定义object函数
function object(o) {
  function F() {}
  F.prototype = o
  return new F()
}

// 定义寄生式核心函数
function inhreitPrototype(subType, superType) {
  var prototype = object(superType.prototype)
  prototype.constructor = subType
  subType.prototype = prototype
}
```

### 3.2 寄生组合式继承的应用

在ES5中，普遍认为寄生组合式继承是最理想的继承范式，也是我们以后使用继承的最终方式：

```javascript
// 定义Animal构造函数
function Animal(age) {
  this.age = age
  this.colors = ["red", "green"]
}

// 给Animal添加方法
Animal.prototype.animalFunction = function () {
  alert("Hello Animal")
}

// 定义Person构造函数
function Person(name, age) {
  Animal.call(this, age)
  this.name = name
}

// 使用寄生组合式核心函数
inhreitPrototype(Person, Animal)

// 给Person添加方法
Person.prototype.personFunction = function () {
  alert("Hello Person")
}
```

优点：

- 高效，只调用了一次Animal的构造函数。
- 避免了在原型上面多出的多余属性，而且原型之间不会产生任何的干扰（子类型原型和父类型原型之间）。



# 来源

- https://mp.weixin.qq.com/s?__biz=Mzg5MDAzNzkwNA==&mid=2247483858&idx=1&sn=bea62eda72b5004f02ab97ca60a72182&chksm=cfe3f22df8947b3bd450d5b7a267828d4f4af1b504e7c955404d47540620bde817771ca83b55&mpshare=1&scene=23&srcid=0613mtT8wW2GtLCLnB7u2zh2&sharer_sharetime=1592015438668&sharer_shareid=ac0071abab7ee9d90efd1d6c4cb28f72#rd
- https://mp.weixin.qq.com/s?__biz=Mzg5MDAzNzkwNA==&mid=2247483867&idx=1&sn=f6989f438101afc9d314352bfc28ae0e&chksm=cfe3f224f8947b3208233f3979b4349ae95cd7f6232d29df14182a94ea81c1197eb09a185c81&mpshare=1&scene=23&srcid=0613Fdl3BVdUB2Bw1Fh7N53I&sharer_sharetime=1592015529190&sharer_shareid=ac0071abab7ee9d90efd1d6c4cb28f72#rd
- https://mp.weixin.qq.com/s?__biz=Mzg5MDAzNzkwNA==&mid=2247483876&idx=1&sn=d9241b9be5462019dbff10fb539e593f&chksm=cfe3f21bf8947b0db29ef6cc5c85acb7d2e60ecc6fb110d897db65af09962a5f34f334c4a4b0&mpshare=1&scene=23&srcid=06136UbON54AEN55oKKjng5l&sharer_sharetime=1592015540610&sharer_shareid=ac0071abab7ee9d90efd1d6c4cb28f72#rd
- https://mp.weixin.qq.com/s?__biz=Mzg5MDAzNzkwNA==&mid=2247483882&idx=1&sn=7a7010bd70d2b61f13c5c419087f2db5&chksm=cfe3f215f8947b03d5b2e51b93b458f31bcada2c827482b91c7d248b41e278fe0f7e2428b39c&mpshare=1&scene=23&srcid=0613QkBmbhLMI7GkZNh9gIAI&sharer_sharetime=1592015554337&sharer_shareid=ac0071abab7ee9d90efd1d6c4cb28f72#rd

