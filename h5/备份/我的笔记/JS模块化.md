# 入门介绍
## 1. 什么是模块 / 模块化?
当项目的功能越来越多，将所有的代码都写在一个JS文件中时，耦合度和关联度就会很高，不方便后期的维护，查找相应代码比较麻烦，容易污染全局环境...因此，将不同的代码根据某一原则，例如相应的功能，拆分成不同的文件。
**模块化**：将一个复杂的程序依据一定的规则(规范)封装成几个块(文件), 并进行组合在一起。块的内部数据/实现是私有的, 只是向外部暴露一些接口(方法)与外部其它模块通信。

**模块的进化史**：
![在这里插入图片描述](JS模块化.assets/20190822152105591.png)
![在这里插入图片描述](JS模块化.assets/20190822152901360.png)
![在这里插入图片描述](JS模块化.assets/20190822152908448.png)
![在这里插入图片描述](JS模块化.assets/2019082215291511.png)
- - -
## 2. 为什么要模块化?
![在这里插入图片描述](JS模块化.assets/20190822162612769.png)
- - -
## 3. 模块化的好处
 - 避免命名冲突(减少命名空间污染)
 - 更好的分离, 按需加载
 - 更高复用性
 - 高可维护性
 - - -
## 4. 页面引入加载 script
![在这里插入图片描述](JS模块化.assets/20190822162825483.png)
**问题**：
 - 请求过多

 - 依赖模糊

 - 难以维护

   

   ==因为这些问题的存在，就导致了模块化的必要。==

# 模块进化史
## 全局function模式
```javascript
/*
 * 全局函数模式: 将不同的功能封装成不同的全局函数
 * 问题: Global被污染了, 很容易引起命名冲突
 */

/* 文件名为：module.js */
let msg = 'module1';
function foo() {
	console.log('foo()', msg);
}

function bar() {
	console.log('bar()', msg);
}
```

```javascript
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>全局function模式</title>
</head>
<body>
<script type="text/javascript" src="module.js"></script>
<script type="text/javascript">
  foo();//结果为：foo() module1
  bar();//结果为：foo() module1
  /* 当某些原因当值我们忘了引入的JS文件中有msg这个变量，而后又将msg进行赋值 */
  msg = 'NBA';//造成命名冲突，污染了引入的JS文件中的msg
  foo();//结果为：foo() NBA  此时，输出结果改变
</script>
</body>
</html>
```
- - -
## namespace模式
```javascript
/*
 * namespace模式: 简单对象封装
 * 作用: 减少了全局变量
 * 问题: 不安全(数据不是私有的, 外部可以直接修改)
 */

/* 文件名为：module.js */
let obj = {
	msg: 'module2',
	foo(){
		console.log('foo()', this.msg);
	}
};
```
```javascript
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>namespace模式</title>
</head>
<body>
<script type='text/javascript' src='module.js'></script>
<script type='text/javascript'>
	obj.foo();//结果为：foo() module2
	obj.msg = 'NBA';
	obj.foo();//结果为：foo() NBA  结果仍然会改变
</script>
</body>
</html>
```
- - -
## IIFE模式
```javascript
/*
 * IIFE模式: 匿名函数自调用(闭包)
 * IIFE : immediately-invoked function expression(立即调用函数表达式)
 * 作用: 数据是私有的, 外部只能通过暴露的方法操作
 * 问题: 如果当前这个模块依赖另一个模块怎么办?
 */

/* 文件名为：module.js */
(function (window) {
	let msg = 'module';
	function foo() {
		console.log('foo()', msg);
	}
})(window)
```
```javascript
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>IIFE模式</title>
</head>
<body>
<script type='text/javascript' src='module.js'></script>
<script type='text/javascript'>
	msg = 'NBA';
	console.log(msg);//实际上，这个msg是相当于在window上定义了一个msg，而不是修改引入的module.js中的msg
	foo();//报错：Uncaught ReferenceError: foo is not defined
</script>
</body>
</html>
```
因此，此时的foo()是内部私有的，想要与外面通信，则需要将其暴露出来。因此：
```javascript
/* 文件名为：module.js */
(function (window) {
	let msg = 'module';
	function foo() {
		console.log('foo()', msg);
	}
	window.module = {foo};
	//相当于：
//	window.module = {//相当于给全局window对象添加了一个module属性，这个属性是个对象，对象里有个方法叫foo
//		foo: foo//第一个foo是定义的属性，第二个foo是上面的函数，而重名的属性可以省略不写
//	};
})(window)
```
```javascript
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>IIFE模式</title>
</head>
<body>
<script type='text/javascript' src='module3.js'></script>
<script type='text/javascript'>
	msg = 'NBA';
	console.log(msg);
//	foo();//不能直接调用，因为暴露出来的是module这个对象
	module.foo();//结果为foo() module
</script>
</body>
</html>
```
- - -
## IIFE增强模式
```javascript
/**
 * IIFE模式增强 : 引入依赖
 * 这就是现代模块实现的基石
 */

/* 文件名为：module */
(function (window) {
	let msg = 'module';
	function foo() {
		console.log('foo()', msg);
	}
	window.module = foo;
})(window);

```
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>IIFE模式增强</title>
</head>
<body>
<script type='text/javascript' src='module4.js'></script>
<script type='text/javascript'>
	module();//结果为：foo() module
</script>
</body>
</html>
```
然后，希望给背景添加背景色，这里需要用到jquery。
```javascript
(function (window, $) {
	let msg = 'module';
	function foo() {
		console.log('foo()', msg);
	}
	window.module = foo;
	$('body').css('background', 'red');
})(window, jQuery);
```
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>IIFE模式增强</title>
</head>
<body>
<script type='text/javascript' src='jquery-1.10.1.js'></script>
<script type='text/javascript' src='module4.js'></script>
<script type='text/javascript'>
	module();//结果为：foo() module
</script>
</body>
</html>
```
- - -
## load script
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>load script</title>
</head>
<body>
<!--
  1. 一个页面需要引入多个js文件
  2. 问题:
    1). 请求过多
    2). 依赖模糊
    3). 难以维护
  3. 这些问题可以通过现代模块化编码和项目构建来解决

  首先我们要依赖多个模块，那样就会发送多个请求，导致请求过多
  然后就是依赖关系模糊  我们不知道他们的具体依赖关系是什么  也就是说很容易因为依赖关系导致出错
  以上的现象就导致了这样会很难维护。很可能出现牵一发而动全身的情况导致项目出现严重的问题
-->
<script src="test1.js"></script>
<script src="test2.js"></script><!-- 依赖test1中的某些东西 -->
<script src="test3.js"></script><!-- 依赖test2中的某些东西 -->
<!-- 此时，test1、test2、test3 的引入顺序不能更改 -->
<script src="test4.js"></script>
</body>
</html>
```
## 模块化进化史教程
```
## 模块化进化史教程
1. 全局function模式
  * module1.js
```
    //数据
    let data = 'atguigu.com'
    
    //操作数据的函数
    function foo() {
      console.log(`foo() ${data}`)
    }
    function bar() {
      console.log(`bar() ${data}`)
    }
    ```
  * module2.js
    ```
    let data2 = 'other data'
    
    function foo() {  //与另一个模块中的函数冲突了
      console.log(`foo() ${data2}`)
    }
    ```
  * test1.html
    ```
    <script type="text/javascript" src="module1.js"></script>
    <script type="text/javascript" src="module2.js"></script>
    <script type="text/javascript">
    
      let data = "修改后的数据"
      foo()
      bar()
    </script>
    ```
   * 说明:
      * 全局函数模式: 将不同的功能封装成不同的全局函数
      * 问题: Global被污染了, 很容易引起命名冲突
2. namespace模式
  * module1.js
    ```
    let myModule = {
      data: 'atguigu.com',
      foo() {
        console.log(`foo() ${this.data}`)
      },
      bar() {
        console.log(`bar() ${this.data}`)
      }
    }
    ```
  * module2.js
    ```
    let myModule2 = {
      data: 'atguigu.com2222',
      foo() {
        console.log(`foo() ${this.data}`)
      },
      bar() {
        console.log(`bar() ${this.data}`)
      }
    }
    ```
  * test2.html
    ```
    <script type="text/javascript" src="module2.js"></script>
    <script type="text/javascript" src="module22.js"></script>
    <script type="text/javascript">
      myModule.foo()
      myModule.bar()
    
      myModule2.foo()
      myModule2.bar()
    
      myModule.data = 'other data' //能直接修改模块内部的数据
      myModule.foo()
    
    </script>
    ```
  * 说明
    * namespace模式: 简单对象封装
    * 作用: 减少了全局变量
    * 问题: 不安全
3. IIFE模式
  * module3.js
    ```
    (function (window) {
      //数据
      let data = 'atguigu.com'
    
      //操作数据的函数
      function foo() { //用于暴露有函数
        console.log(`foo() ${data}`)
      }
    
      function bar() {//用于暴露有函数
        console.log(`bar() ${data}`)
        otherFun() //内部调用
      }
    
      function otherFun() { //内部私有的函数
        console.log('otherFun()')
      }
    
      //暴露行为
      window.myModule = {foo, bar}
    })(window)
    ```
  * test3.html
    ```
    <script type="text/javascript" src="module3.js"></script>
    <script type="text/javascript">
      myModule.foo()
      myModule.bar()
      //myModule.otherFun()  //myModule.otherFun is not a function
      console.log(myModule.data) //undefined 不能访问模块内部数据
      myModule.data = 'xxxx' //不是修改的模块内部的data
      myModule.foo() //没有改变
    
    </script>
    ```
  * 说明:
    * IIFE模式: 匿名函数自调用(闭包)
    * IIFE : immediately-invoked function expression(立即调用函数表达式)
    * 作用: 数据是私有的, 外部只能通过暴露的方法操作
    * 问题: 如果当前这个模块依赖另一个模块怎么办?
4. IIFE模式增强
  * 引入jquery到项目中
  * module4.js
    ```
    (function (window, $) {
      //数据
      let data = 'atguigu.com'
    
      //操作数据的函数
      function foo() { //用于暴露有函数
        console.log(`foo() ${data}`)
        $('body').css('background', 'red')
      }
    
      function bar() {//用于暴露有函数
        console.log(`bar() ${data}`)
        otherFun() //内部调用
      }
    
      function otherFun() { //内部私有的函数
        console.log('otherFun()')
      }
    
      //暴露行为
      window.myModule = {foo, bar}
    })(window, jQuery)
    ```
  * test4.html
    ```
    <script type="text/javascript" src="jquery-1.10.1.js"></script>
    <script type="text/javascript" src="module4.js"></script>
    <script type="text/javascript">
      myModule.foo()
    </script>
    ```
  * 说明
    * IIFE模式增强 : 引入依赖
    * 这就是现代模块实现的基石
      
5. 页面加载多个js的问题
  * 页面:
    ```
    <script type="text/javascript" src="module1.js"></script>
    <script type="text/javascript" src="module2.js"></script>
    <script type="text/javascript" src="module3.js"></script>
    <script type="text/javascript" src="module4.js"></script>
    ```
  * 说明
    * 一个页面需要引入多个js文件
    * 问题:
        * 请求过多
        * 依赖模糊
        * 难以维护
    * 这些问题可以通过现代模块化编码和项目构建来解决
```
# CommonJS
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822192925341.png)
## CommonJS基于服务器端(node)应用
**Node.js模块化教程**
```
Node.js模块化教程

1. 下载安装node.js
2. 创建项目结构
    ``
    |-modules
    |-module1.js
    |-module2.js
    |-module3.js
    |-app.js
    |-package.json
    {
      "name": "commonJS-node",
      "version": "1.0.0"
    }
    ``
3. 下载第三方模块
  * npm install uniq --save
4. 模块化编码
  * module1.js
    ```
    module.exports = {
      foo() {
        console.log('moudle1 foo()')
      }
    }
    ```
  * module2.js
    ```
    module.exports = function () {
      console.log('module2()')
    }
    ```
  * module3.js
    ```
    exports.foo = function () {
      console.log('module3 foo()')
    }
    
    exports.bar = function () {
      console.log('module3 bar()')
    }
    ```
  * app.js 
    ```
    /**
      1. 定义暴露模块:
        module.exports = value;
        exports.xxx = value;
      2. 引入模块:
        var module = require(模块名或模块路径);
     */
    "use strict";
    //引用模块
    let module1 = require('./modules/module1')
    let module2 = require('./modules/module2')
    let module3 = require('./modules/module3')
    
    let uniq = require('uniq')
    let fs = require('fs')
    
    //使用模块
    module1.foo()
    module2()
    module3.foo()
    module3.bar()
    
    console.log(uniq([1, 3, 1, 4, 3]))
    
    fs.readFile('app.js', function (error, data) {
      console.log(data.toString())
    })
    ```
5. 通过node运行app.js
  * 命令: node app.js
  * 工具: 右键-->运行
```
​```javascript
/* package.json */
{
  "name": "commonjs-node",
  "version": "1.0.0",
  "dependencies": {
    "uniq": "^1.0.1"
  }
}
```
- - -
- 目录如下图所示：
![在这里插入图片描述](JS模块化.assets/20190823094744282.png)
- - -
**用命令创建package.json ：**
![在这里插入图片描述](JS模块化.assets/20190822195808504.png)
![在这里插入图片描述](JS模块化.assets/20190822195914607.png)
![在这里插入图片描述](JS模块化.assets/20190822200048141.png)
![在这里插入图片描述](JS模块化.assets/20190822200403585.png)
注意：包名中不能有中文、大写，可以发现此时已经自动将'JS'改为了'js'。(低版本中可能不会自动改正。使用 node -v 可以查看node版本)
![在这里插入图片描述](JS模块化.assets/20190822200821462.png)
![在这里插入图片描述](JS模块化.assets/20190822200924428.png)
此时，随便点一个位置即可，如下图所示：
![在这里插入图片描述](JS模块化.assets/20190822201144982.gif)
![在这里插入图片描述](JS模块化.assets/20190822201245926.png)
然后下载uniq ：
![在这里插入图片描述](JS模块化.assets/20190822201507987.png)
（注意：在npm3版本中，--save 是必须加的，否则不会自动将依赖加到package中；在npm5中，--save 已经是默认的属性了。）

此时，package中加入了依赖：
![在这里插入图片描述](JS模块化.assets/20190822201605369.png)

- - -
- - -
- - -
**模块操作：**
```javascript
/* module1.js */
/**
 * 使用module.exports = value向外暴露一个对象
 */
"use strict"
module.exports = {
  msg: 'module1',
  foo(){
  	console.log(this.msg);
  }
};
```
```javascript
/* module2.js */
/**
 * 使用module.exports = value向外暴露一个函数
 */
"use strict"
module.exports = function () {
  console.log('module2()')
}

/* 这里与exports.xxx不同，不能再写odule.exports = ... 因为下面的代码会覆盖上面的代码 */
//module.exports = function () {
//console.log('module2()')
//}
```
```javascript
/* module3.js */
/**
 * 使用exports.xxx = value向外暴露一个对象
 */
"use strict"
exports.foo = function () {
  console.log('module3 foo()')
}

/* 可以再写 */
exports.bar = function () {
  console.log('module3 bar()')
}
```
```javascript
/* app.js */
//将其他的模块汇集到主模块
//引用模块
let module1 = require('./modules/module1');
let module2 = require('./modules/module2');
let module3 = require('./modules/module3');

//使用模块
module1.foo();//结果为：module1
module2();//结果为：module2
module3.foo();//结果为：foo() module3
module3.bar();//结果为：bar() module3
```
- - -
- - - -
- - - -
```javascript
/* module3.js */
/**
 * 使用exports.xxx = value向外暴露一个对象
 */
"use strict"
exports.foo = function () {
  console.log('module3 foo()')
}

/* 可以再写 */
exports.bar = function () {
  console.log('module3 bar()')
}

/* 继续暴露数组 */
exports.arr = [2,4,5,2,3,5,1];
```
```javascript
/* app.js */
//将其他的模块汇集到主模块
/**
  1. 定义暴露模块:
    module.exports = value;
    exports.xxx = value;
  2. 引入模块:
    var module = require(模块名或模块路径);
 */

//默认将第三方库放到自定义库的上面 
let uniq = require('uniq');

////引用模块
//let module1 = require('./modules/module1')
//let module2 = require('./modules/module2')
//let module3 = require('./modules/module3')

////使用模块
//module1.foo();//结果为：module1
//module2();//结果为：module2
//module3.foo();//结果为：foo() module3
//module3.bar();//结果为：bar() module3

let result = uniq(module3.arr);
console.log(result);//结果为：[1, 2, 3, 4, 5]   自动去重和排序
//原数组为[2,4,5,2,3,5,1]
```
## CommonJS基于浏览器端应用
```
Browserify模块化使用教程
1. 创建项目结构
  ···
  |-js
    |-dist //打包生成文件的目录
    |-src //源码所在的目录
      |-module1.js
      |-module2.js
      |-module3.js
      |-app.js //应用主源文件
  |-index.html
  |-package.json
    {
      "name": "browserify-test",
      "version": "1.0.0"
    }
  ···
2. 下载browserify
  * 全局: npm install browserify -g
  * 局部: npm install browserify --save-dev       -dev表示是开发依赖的
3. 定义模块代码
  * module1.js
```
    module.exports = {
      foo() {
        console.log('moudle1 foo()')
      }
    }
    ```
  * module2.js
    ```
    module.exports = function () {
      console.log('module2()')
    }
    ```
  * module3.js
    ```
    exports.foo = function () {
      console.log('module3 foo()')
    }
    
    exports.bar = function () {
      console.log('module3 bar()')
    }
    ```
  * app.js (应用的主js)
    ```
    //引用模块
    let module1 = require('./module1')
    let module2 = require('./module2')
    let module3 = require('./module3')
    
    let uniq = require('uniq')
    
    //使用模块
    module1.foo()
    module2()
    module3.foo()
    module3.bar()
    
    console.log(uniq([1, 3, 1, 4, 3]))
    ```
* 打包处理js:
  
  * browserify js/src/app.js -o js/dist/bundle.js
* 页面使用引入:
  ···
  <script type="text/javascript" src="js/dist/bundle.js"></script> 
  ···
```
- - -
目录如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190823094901141.png)
- - -
首先，手动创建package.json，如图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019082309520142.png)
然后，下载browserify
  * 全局: npm install browserify -g
  * 局部: npm install browserify --save-dev      （-dev表示是开发依赖的） 

这两步都得做。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190823095844236.png)然后回车。
结果如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190823100037861.png)
dev表示是开发依赖的。
实际上，下载uniq：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190823100547140.png)，回车。
此时package.json，如图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190823100731441.png)
devDependencies表示是开发依赖的；dependencies表示是运行依赖的。
将开发依赖的放入运行依赖中，也行。
因为，事实上，这只是一个说明书，真正运行时找的是node_modules(npm下载包时才会有)中对应的包：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190823100229738.png)
- - -
- - -
- - -
**模块操作：**
​```javascript
/* module1.js */
/**
 * 使用module.exports = value向外暴露一个对象
 */
module.exports = {
  foo() {
    console.log('moudle1 foo()')
  }
}
```
```javascript
/* module2.js */
/**
 * 使用module.exports = value向外暴露一个函数
 */
module.exports = function () {
  console.log('module2()')
}
```
```javascript
/* module3.js */
/**
 * 使用exports.xxx = value向外暴露一个对象
 */
exports.foo = function () {
  console.log('module3 foo()')
}

exports.bar = function () {
  console.log('module3 bar()')
}
```
```javascript
/* app.js */
/**
  1. 定义暴露模块:
    module.exports = value;
    exports.xxx = value;
  2. 引入模块:
    var module = require(模块名或模块路径);
*/
//引用模块
let module1 = require('./module1');
let module2 = require('./module2');
let module3 = require('./module3');

//使用模块
module1.foo();
module2();
module3.foo();
module3.bar();
```
```javascript
/* 对应的页面index.html */
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script type="text/javascript" src="./js/src/app.js"></script>
</body>
</html>

<!-- 报错：Uncaught ReferenceError: require is not defined
	  因为浏览器不认识require，因此，需要提前编译打包处理     -->
```
- - -
打包处理js:   browserify js/src/app.js -o js/dist/bundle.js
-o 表示 output(输出)
js/src/app.js 是目标文件(源文件)
js/dist/bundle.js 是作用完后输出的文件路径及文件名

打包处理完后的bundle.js如下所示：
```javascript
(function e(t,n,r){function s(o,u){if(!n[o]){if(!t[o]){var a=typeof require=="function"&&require;if(!u&&a)return a(o,!0);if(i)return i(o,!0);var f=new Error("Cannot find module '"+o+"'");throw f.code="MODULE_NOT_FOUND",f}var l=n[o]={exports:{}};t[o][0].call(l.exports,function(e){var n=t[o][1][e];return s(n?n:e)},l,l.exports,e,t,n,r)}return n[o].exports}var i=typeof require=="function"&&require;for(var o=0;o<r.length;o++)s(r[o]);return s})({1:[function(require,module,exports){
/**
 1. 定义暴露模块:
    module.exports = value;
    exports.xxx = value;
 2. 引入模块:
    var module = require(模块名或模块路径);
 */
"use strict"
//引用模块
let module1 = require('./module1')
let module2 = require('./module2')
let module3 = require('./module3')

let uniq = require('uniq')

//使用模块
module1.foo()
module2()
module3.foo()
module3.bar()

console.log(uniq([1,3,1,4,3]))

},{"./module1":2,"./module2":3,"./module3":4,"uniq":5}],2:[function(require,module,exports){
/**
 * 使用module.exports = value向外暴露一个对象
 */
"use strict"
module.exports = {
    foo() {
        console.log('moudle1 foo()')
    }
}
},{}],3:[function(require,module,exports){
/**
 * 使用module.exports = value向外暴露一个函数
 */
"use strict"
module.exports = function () {
    console.log('module2()')
}
},{}],4:[function(require,module,exports){
/**
 * 使用exports.xxx = value向外暴露一个对象
 */
"use strict"
exports.foo = function () {
    console.log('module3 foo()')
}

exports.bar = function () {
    console.log('module3 bar()')
}
},{}],5:[function(require,module,exports){
"use strict"

function unique_pred(list, compare) {
  var ptr = 1
    , len = list.length
    , a=list[0], b=list[0]
  for(var i=1; i<len; ++i) {
    b = a
    a = list[i]
    if(compare(a, b)) {
      if(i === ptr) {
        ptr++
        continue
      }
      list[ptr++] = a
    }
  }
  list.length = ptr
  return list
}

function unique_eq(list) {
  var ptr = 1
    , len = list.length
    , a=list[0], b = list[0]
  for(var i=1; i<len; ++i, b=a) {
    b = a
    a = list[i]
    if(a !== b) {
      if(i === ptr) {
        ptr++
        continue
      }
      list[ptr++] = a
    }
  }
  list.length = ptr
  return list
}

function unique(list, compare, sorted) {
  if(list.length === 0) {
    return list
  }
  if(compare) {
    if(!sorted) {
      list.sort(compare)
    }
    return unique_pred(list, compare)
  }
  if(!sorted) {
    list.sort()
  }
  return unique_eq(list)
}

module.exports = unique

},{}]},{},[1]);
```
然后，将其引入index：
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<script type="text/javascript" src="js/dist/bundle.js"></script>
</body>
</html>
```
此时可以成功运行，结果如下图所示：
![在这里插入图片描述](JS模块化.assets/20190823102537414.png)

# AMD
![在这里插入图片描述](JS模块化.assets/20190823212557699.png)
## AMD规范_NoAMD
目录如下图所示：
![在这里插入图片描述](JS模块化.assets/20190823214900715.png)
```javascript
//文件名为：dataService.js
//定义一个没有依赖的模块
(function (window) {
  let name = 'dataService.js'
  function getName() {
    return name;
  }
	//暴露模块
  window.dataService = {getName};
})(window)
```
```javascript
//文件名为：alerter.js
//定义一个有依赖的模块，依赖于dataService
(function (window, dataService) {
  let msg = 'alerter.js';
  function showMsg() {
    console.log(msg, dataService.getName());
  }
  window.alerter = {showMsg};
})(window, dataService)
```
```javascript
//文件名为：app.js
(function (alerter) {
  alerter.showMsg();
})(alerter)
```
```javascript
//文件名为：test1.html
<!DOCTYPE html>
<html>
<head>
  <title>Modular Demo 1</title>
</head>
<body>
	<div>
	  <h1>Modular Demo 1: 未使用AMD(require.js)</h1>
	</div>
	
	<script type='text/javascript' src='./js/dataService.js'></script>
	<script type='text/javascript' src='./js/alerter.js'></script>
	<!-- alerter中还依赖于dataService，因此需要先引入dataService.js -->
	<script type="text/javascript" src="./js/app.js"></script>
	<!-- 如果只引入app.js 则会报错，因为此时并没有读过alert.js -->
	<!-- 可以看出，没有规范时需要分别引入 -->
</body>
</html>


运行结果为：alerter.js dataService.js
```
## AMD规范_自定义模块
目录结果为：
![在这里插入图片描述](JS模块化.assets/20190824182700518.png)
```
require.js使用教程
1. 下载require.js, 并引入
  * 官网: http://www.requirejs.cn/
  * github : https://github.com/requirejs/requirejs
  * 将require.js导入项目: js/libs/require.js 
2. 创建项目结构
  ···
  |-js
    |-libs
      |-require.js
    |-modules
      |-alerter.js
      |-dataService.js
    |-main.js
  |-index.html
  ···
3. 定义require.js的模块代码
  * dataService.js
    ···
    define(function () {
      let msg = 'atguigu.com'
    
      function getMsg() {
        return msg.toUpperCase()
      }
    
      return {getMsg}
    })
    ···
  * alerter.js
    ···
    define(['dataService', 'jquery'], function (dataService, $) {
      let name = 'Tom2'
    
      function showMsg() {
        $('body').css('background', 'gray')
        alert(dataService.getMsg() + ', ' + name)
      }
    
      return {showMsg}
    })
```
4. 应用主(入口)js: main.js
    ···
    (function () {
    //配置
    require.config({
      //基本路径
      baseUrl: "js/",
      //模块标识名与模块路径映射
      paths: {
        "alerter": "modules/alerter",
        "dataService": "modules/dataService",
      }
    })
    //引入使用模块
    require( ['alerter'], function(alerter) {
      alerter.showMsg()
    })
    })()
    ···
    
5. 页面使用模块:
  <script data-main="js/main" src="js/libs/require.js"></script>

------------------------------------------------------------------------

6. 使用第三方基于require.js的框架(jquery)
  * 将jquery的库文件导入到项目: 
    
    * js/libs/jquery-1.10.1.js
  * 在main.js中配置jquery路径
    ```
    paths: {
              'jquery': 'libs/jquery-1.10.1'
          }
    ```
  * 在alerter.js中使用jquery
    ```
    define(['dataService', 'jquery'], function (dataService, $) {
        var name = 'xfzhang'
        function showMsg() {
            $('body').css({background : 'red'})
            alert(name + ' '+dataService.getMsg())
        }
        return {showMsg}
    })
    ```
------------------------------------------------------------------------

7. 使用第三方不基于require.js的框架(angular/angular-messages)
  * 将angular.js和angular-messages.js导入项目
    * js/libs/angular.js
    * js/libs/angular-messages.js
  * 在main.js中配置
    ```
    (function () {
      require.config({
        //基本路径
        baseUrl: "js/",
        //模块标识名与模块路径映射
        paths: {
          //第三方库
          'jquery' : 'libs/jquery-1.10.1',
          'angular' : 'libs/angular',
          'angular-messages' : 'libs/angular-messages',
          //自定义模块
          "alerter": "modules/alerter",
          "dataService": "modules/dataService"
        },
        /*
         配置不兼容AMD的模块
         exports : 指定导出的模块名
         deps  : 指定所有依赖的模块的数组
         */
        shim: {
          'angular' : {
            exports : 'angular'
          },
          'angular-messages' : {
            exports : 'angular-messages',
            deps : ['angular']
          }
        }
      })
      //引入使用模块
      require( ['alerter', 'angular', 'angular-messages'], function(alerter, angular) {
        alerter.showMsg()
        angular.module('myApp', ['ngMessages'])
        angular.bootstrap(document,["myApp"])
      })
    })()
    ```
  * 页面:
    ···
    <form name="myForm">
      用户名: <input type="text" name="username" ng-model="username" ng-required="true">
      <div style="color: red;" ng-show="myForm.username.$dirty&&myForm.username.$invalid">用户名是必须的</div>
    </form>
    ···
```
​```javascript
//文件名为：dataService.js
//定义没有依赖的模块
define(function () {
  let name = 'dataService.js';
  function getName() {
    return name;
  }
  //暴露模块
  return {getName};
})
```
```javascript
//文件名为：alerter.js
//定义有依赖的模块
/* 
 * define(['dataService'] 中的'dataService' 并不指dataService文件，
 * 	 而只是想要代表dataService.js这个模块，名字为xxx都行，
 * 	    但是想要代表这个模块则必须要映射一个路径(main.js中的paths)，以此找到这个文件
 */
define(['dataService'], function (dataService) {
  let msg = 'alerter.js'
  function showMsg() {
    console.log(msg, dataService.getName());
  }
	//暴露模块
  return {showMsg};
})
```
```javascript
//文件名为：main.js
(function () {
  //配置
  requirejs.config({
    //基本路径
    //baseUrl: 'js/',
    
    //映射: 模块标识名: 路径
    paths: {//配置路径
      //自定义模块
      //例如下面的'dataService' 指的是alerter.js的define(['dataService']中的dataService，类似于形参，换成xxx也行
      'alerter': './modules/alerter',
      'dataService': './modules/dataService'
    }
  });  
  
  //引入模块使用
  require(['alerter'], function (alerter) {
    alerter.showMsg();
  })
})()
```
```javascript
//文件名为：index2.html
<!DOCTYPE html>
<html>
<head>
    <title>Modular Demo 2</title>
</head>
<body>
	<!-- 引入require.js并指定js主文件的入口 -->
	<!-- data-main指向的是当前的主模块 -->
  <script type="text/javascript" src="js/libs/require.js" data-main="js/main.js"></script>
 </body>
</html>



运行结果为：alerter.js dataService.js
```
- - -
- - -
- - -
如果我们将```main.js```中的paths注释掉一个，例如，注释掉```'dataService': './modules/dataService'```
则会报错，结果如下图所示：
![在这里插入图片描述](JS模块化.assets/20190824105404531.png)
![在这里插入图片描述](JS模块化.assets/20190824105417954.png)表示被alerter所以依赖
- - -
- - -
如果我们将```main.js```中的```baseUrl: 'js/',```不注释掉，依然能成功运行。
当不使用```baseUrl: 'js/',```时，找路径是从当前```main.js```的角度去找alerter、dataService的路径。
当使用```baseUrl: 'js/',```时，相当于是从根目录去找路径的。```'js/'```最终是要和```'./modules/alerter'```、```'./modules/dataService'```做拼接的。
- - -
- - -
如果我们给```'./modules/alerter'```、```'./modules/dataService'```的末尾加上```.js```则会报错：
![在这里插入图片描述](JS模块化.assets/20190824110024520.png)
因为此时变为![在这里插入图片描述](JS模块化.assets/20190824110158637.png)即它会默认加上```.js```，却它并不只能，如果你手动加上了```.js```，它还会自动地再添加```.js```。


## AMD规范_第三方模块
刚才是我们自定义的模块，有时，我们可能用第三方模块。首先，我们放入JQuery，目录结构如下图所示：
![在这里插入图片描述](JS模块化.assets/20190824193747864.png)（angular.js在稍后会用到）


同理，我们也需要映射一个路径以找到相应的文件。
```javascript
//文件名为：main.js
(function () {
  //配置
  requirejs.config({
    //基本路径
    baseUrl: 'js/',
    
    //映射: 模块标识名: 路径
    paths: {//配置路径
      //自定义模块
      //例如下面的'dataService' 指的是alerter.js的define(['dataService']中的dataService，类似于形参，换成xxx也行
      'alerter': './modules/alerter.js',
      'dataService': './modules/dataService.js'
      'jQuery': './libs/jquery-1.10.1'
    }
  });  
  
  //引入模块使用
  require(['alerter'], function (alerter) {
    alerter.showMsg();
  })
})()
```
```javascript
//文件名为：alerter.js
//定义有依赖的模块
/* 
 * define(['dataService'] 中的'dataService' 并不指dataService文件，
 * 	 而只是想要代表dataService.js这个模块，名字为xxx都行，
 * 	    但是想要代表这个模块则必须要映射一个路径，以此找到这个文件
 */
define(['dataService', 'jQuery'], function (dataService, $) {
  let msg = 'alerter.js'
  function showMsg() {
    console.log(msg, dataService.getName());
  }
  //添加背景色，验证是否生效
  $('body').css('background', 'deeppink');
	//暴露模块
  return {showMsg};
})
```
其他文件与自定义模块这一节保持相同。
结果为报错：
![在这里插入图片描述](JS模块化.assets/2019082419444595.png)
- - -
看jQuery的源码：
![在这里插入图片描述](JS模块化.assets/20190824194658911.png)
意思是：如果define的数据类型是function并且遵循的是amd规范，则jQuery内部会自己定义一个模块，名字为小写的```jquery```，最后暴露出来的是内部封装好的jQuery对象。
因此，刚才报错的原因是main.js和alerter.js文件中是大写的```jQuery```。

当全都改为小写时，运行成功。
- - -
- - -
要注意的是：**并不是所有的第三方库都支持AMD规范**。
比如说，angular，
需要额外加入：![在这里插入图片描述](JS模块化.assets/20190824200023423.png)

# CMD
相对于其他三个来说，CMD在市面上用的比较少。
![在这里插入图片描述](JS模块化.assets/2019082510211342.png)
从某一方面讲，可以说CMD结合了AMD和CommonJS。定义用AMD，暴露用CommonJS。
## CMD规范应用
目录结构如下图所示：
![在这里插入图片描述](JS模块化.assets/20190825110850123.png)
```
sea.js简单使用教程
1. 下载sea.js, 并引入
  * 官网: http://seajs.org/
  * github : https://github.com/seajs/seajs
  * 将sea.js导入项目: js/libs/sea.js 
2. 创建项目结构
  ···
  |-js
    |-libs
      |-sea.js
    |-modules
      |-module1.js
      |-module2.js
      |-module3.js
      |-module4.js
      |-main.js
  |-index.html
  ···
3. 定义sea.js的模块代码
  * module1.js
    ···
    define(function (require, exports, module) {
      //内部变量数据
      var data = 'atguigu.com'
      //内部函数
      function show() {
        console.log('module1 show() ' + data)
      }
    
      //向外暴露
      exports.show = show
    })
    ···
  * module2.js
    ···
    define(function (require, exports, module) {
      module.exports = {
        msg: 'I Will Back'
      }
    })
    ···
  * module3.js
    ···
    define(function (require, exports, module) {
      const API_KEY = 'abc123'
      exports.API_KEY = API_KEY
    })
    ···
  * module4.js
    ···
    define(function (require, exports, module) {
      //引入依赖模块(同步)
      var module2 = require('./module2')
    
      function show() {
        console.log('module4 show() ' + module2.msg)
      }
    
      exports.show = show
      //引入依赖模块(异步)
      require.async('./module3', function (m3) {
        console.log('异步引入依赖模块3  ' + m3.API_KEY)
      })
    })
    ···
  * main.js : 主(入口)模块
    ···
    define(function (require) {
      var m1 = require('./module1')
      var m4 = require('./module4')
      m1.show()
      m4.show()
    })
    ···
4. index.html:
  ···
  <!--
  使用seajs:
    1. 引入sea.js库
    2. 如何定义导出模块 :
      define()
      exports
      module.exports
    3. 如何依赖模块:
      require()
    4. 如何使用模块:
      seajs.use()
  -->
  <script type="text/javascript" src="js/libs/sea.js"></script>
  <script type="text/javascript">
    seajs.use('./js/modules/main')
  </script>
  ···
```
- - -
- - -
```javascript
//文件名为：module1.js
//定义没有依赖的模块
define(function (require, exports, module) {
  let msg = 'module1';
  function foo() {
  	return msg;
  }
  //暴露模块
  module.exports = {foo};
})
```
```javascript
//文件名为：module2.js
define(function (require, exports, module) {
  let msg = 'module2';
  function bar() {
  	console.log(msg);
  }
  module.exports = bar;
})
```
```javascript
//文件名为：module3.js
define(function (require, exports, module) {
  let data = 'module3';
  function fun() {
  	console.log(data);
  }
  exports.module3 = {fun};
})
```
```javascript
//文件名为：module4.js
define(function (require, exports, module) {
  let msg = 'module4';
  //同步引入
  let module2 = require('./module2');
  module2();
  //异步引入
  require.async('./module3', function (module3){
  	module3.module3.fun();
  });
  function fun2() {
  	console.log(msg);
  }
  exports.fun2 = fun2;
})
```
```javascript
//文件名为：main.js
define(function (require) {
  let module1 = require('./module1');
  console.log(module1.foo());
  let module4 = require('./module4');
  module4.fun2();
})
```
```javascript
//文件名为：index.html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Title</title>
</head>
<body>
	<!--
	使用seajs:
	  1. 引入sea.js库
	  2. 如何定义导出模块 :
	    define()
	    exports
	    module.exports
	  3. 如何依赖模块:
	    require()
	  4. 如何使用模块:
	    seajs.use()
	-->
	<!-- 引入sea库 -->
	<script type="text/javascript" src="js/libs/sea.js"></script>
	<script type="text/javascript">
	  seajs.use('./js/modules/main.js');
	</script>
</body>
</html>
```
结果为：![在这里插入图片描述](JS模块化.assets/20190825115313256.png)

那么，为什么顺序是1 2 4 3呢？
在```main.js```中第一个是```let module1 = require('./module1');```所以先输出的是module1。
然后```let module4 = require('./module4');```时，进入```module4.js```。在```module4.js```中，首先同步引入```let module2 = require('./module2');```所以先输出module2。
但是因为module3是异步的，所以回调函数被放入事件队列中，需要等其他的执行完毕后才能继续，```module4.js```读取完毕后，回到```main.js```，然后执行```module4.fun2();```。fun2()打印自己的msg，即打印module4。打印完后，```main.js```全部执行完了，开始执行异步的回调函数，然后打印module3。


# ES6
![在这里插入图片描述](JS模块化.assets/20190825140700175.png)
```
语法：
**导出模块:
1. 分多次导出模块的多个部分
		export class Emp{  }
		export function fun(){  }
		export var person = {};
2. 一次导出模块的多个部分
		class Emp{  }
		function fun(){  }
		var person = {};
		export {Emp, fun, person}
3. default导出(只能有一个)
		export default {}

**导入模块
import defaultModule from './myModule';  //导入默认的
import {Emp} from './myModule'; //导入指定的一个
import {Emp, person} from './myModule'; //导入指定的多个
import * as allFromModule from './myModule';  //导入所有
```
## ES6规范_基本使用
```
ES6-Babel-Browserify使用教程
1. 定义package.json文件
  ···
  {
    "name" : "es6-babel-browserify",
    "version" : "1.0.0"
  }
  ···
2. 安装babel-cli, babel-preset-es2015和browserify
    * npm install babel-cli browserify -g
	* npm install babel-preset-es2015 --save-dev 
3. 定义.babelrc文件
	···
	{
    "presets": ["es2015"]
  }
	···
4. 编码
  * js/src/module1.js
    ···
    export function foo() {
      console.log('module1 foo()');
    }
    export let bar = function () {
      console.log('module1 bar()');
    }
    export const DATA_ARR = [1, 3, 5, 1]
    ···
  * js/src/module2.js
    ···
    let data = 'module2 data'
    
    function fun1() {
      console.log('module2 fun1() ' + data);
    }
    
    function fun2() {
      console.log('module2 fun2() ' + data);
    }
    
    export {fun1, fun2}
    ···
  * js/src/module3.js
    ···
    export default {
      name: 'Tom',
      setName: function (name) {
        this.name = name
      }
    }
   ···
  * js/src/app.js
    ···
    import {foo, bar} from './module1'
    import {DATA_ARR} from './module1'
    import {fun1, fun2} from './module2'
    import person from './module3'
    
    import $ from 'jquery'
    
    $('body').css('background', 'red')
    
    foo()
    bar()
    console.log(DATA_ARR);
    fun1()
    fun2()
    
    person.setName('JACK')
    console.log(person.name);
    ···
5. 编译
  * 使用Babel将ES6编译为ES5代码(但包含CommonJS语法) : babel js/src -d js/lib
  * 使用Browserify编译js : browserify js/lib/app.js -o js/lib/bundle.js
6. 页面中引入测试
  ···
  <script type="text/javascript" src="js/lib/bundle.js"></script>
  ···
7. 引入第三方模块(jQuery)
  1). 下载jQuery模块: 
    * npm install jquery@1 --save
  2). 在app.js中引入并使用
    ···
    import $ from 'jquery'
    $('body').css('background', 'red')
    ···
```
首先，定义package.json文件，如下所示：
![在这里插入图片描述](JS模块化.assets/20190825152822995.png)
然后进行安装：
    * npm install babel-cli browserify -g
	* npm install babel-preset-es2015 --save-dev 
安装完毕后，package.json文件如下所示：
![在这里插入图片描述](JS模块化.assets/20190825152943193.png)
然后在根目录下创建 .babelrc 文件，内容为：
```javascript
{
   "presets": ["es2015"]
}
```
其工作原理是在插件干活之前会先读取配置文件，例如读到es2015后就知道是要去转换ES6的语法。(使用数组的原因是里面可以放置多个。)
- - -
```javascript
//文件名为：.babelrc
{
   "presets": ["es2015"]
}
```
```javascript
//文件名为：package.json
{
  "name": "es6-babel-browserify",
  "version": "1.0.0",
  "devDependencies": {
    "babel-preset-es2015": "^6.24.0"
  }
}
```
```javascript
//文件名为：module1.js
//分别暴露
export function foo() {
	console.log('foo() module1');
}

export function bar() {
	console.log('bar() module1');
}

export let arr = [1,2,3,4,5];
```
```javascript
//文件名为：module2.js
//统一暴露
function fun() {
	console.log('fun() module2');
}

function fun2() {
	console.log('fun2() module2');
}

export {fun, fun2};
```
```javascript
//文件名为：main.js
//引入其他的模块

//语法： import xxx from '路径'
import module1 from './module1';
import module2 from './module2';

console.log(module1, module2);
```
```javascript
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <script type="text/javascript" src="js/src/main.js"></script>
</body>
</html>
```
如果此时运行，会报错：
![在这里插入图片描述](JS模块化.assets/20190825163411489.png)
浏览器并不认识import。
需要将ES6的语法转换为ES5。
- - -
因此，执行第5步 编译：
![在这里插入图片描述](JS模块化.assets/20190825170153337.png)
```-d```左边的```js/src```说明找的并不是某一个文件，而是src下所有的文件，因为所有的文件都是用的ES6的语法，需要全部转换为ES5。
```-d```右边的```js/lib```则是输出的目录。其中，lib可以替换为其他的。
![在这里插入图片描述](JS模块化.assets/20190825164147838.png)
此时，自动生成build文件夹：
![在这里插入图片描述](JS模块化.assets/20190825164329912.png)
此时，如果直接在```index.html```中运行```<script type="text/javascript" src="js/build/main.js"></script>```则会报错：
![在这里插入图片描述](JS模块化.assets/20190825164431412.png)
因为其中包含require，浏览器不认识。
因此，执行下一步：
![在这里插入图片描述](JS模块化.assets/20190825164936223.png)
(注意：browserify不会自动创建当前没有的文件夹，只能在当前已有的文件夹下创建文件。)
![在这里插入图片描述](JS模块化.assets/20190825165101109.png)

此时，在```index.html```中运行```<script type="text/javascript" src="js/dist/bundle.js"></script>```
此时，结果为：
![在这里插入图片描述](JS模块化.assets/20190825165134990.png)
这两个undefined来自```main.js```的![在这里插入图片描述](JS模块化.assets/20190825165216913.png)
因此，改为如下：
```javascript
//文件名为：main.js
//引入其他的模块

//语法： import xxx from '路径'
import {foo, bar} from './module1';
import {fun, fun2} from './module2';

//console.log(module1, module2);

/*
 * 在ES6中，如果使用 分别暴露 或 统一暴露 的形式，
 * 	 则在引入时必须使用 对象结构赋值 的形式才能拿到模块对象中的数据
 */

foo();
bar();
fun();
fun2();
```
此时，再运行，结果为：
![在这里插入图片描述](JS模块化.assets/20190825165854534.png)
（注意：此时dist中的```bundle.js```是之前打包好的，是之前的代码；因此，需要进行重新打包。）

## ES6规范_默认暴露
在上述的基本使用中，必须使用解构赋值的形式，否则拿到的值就是undefined，那么怎么才能回到以前那种定义变量后直接使用，暴露什么就接收什么的模式呢？
分别暴露 和 统一暴露，合起来叫作常规暴露。
```javascript
//文件名为：module3.js
//默认暴露  可以暴露任意的数据类型，暴露什么数据，接收到的就是什么数据
//语法：export default value;
export default () => {
  console.log('我是默认暴露的箭头函数');
}
```
然后在main.js中引入
```javascript
//文件名为：main.js
//引入其他的模块

//语法： import xxx from '路径'
import {foo, bar} from './module1';
import {fun, fun2} from './module2';
//此时module3就是一个函数，可以直接调用
import module3 from './module3';

//console.log(module1, module2);

/*
 * 在ES6中，如果使用 分别暴露 或 统一暴露 的形式，
 * 	 则在引入时必须使用 对象结构赋值 的形式才能拿到模块对象中的数据
 */

foo();
bar();
fun();
fun2();

module3();
```
同样，需要重新打包编译成新的文件，运行结果如下图所示：
![在这里插入图片描述](JS模块化.assets/20190825220410837.png)
- - -
同样，可以继续暴露
```javascript
//文件名为：module3.js
//默认暴露  可以暴露任意的数据类型，暴露什么数据，接收到的就是什么数据
//语法：export default value;
export default () => {
  console.log('我是默认暴露的箭头函数');
}

export default {
  msg: '默认暴露',
  foo() {
  	console.log(this.msg);
  }
}
```
此时，重新打包编译时会报错，因为默认暴露只能写一次。
那么需要暴露很多数据怎么办呢？
对象是用来管理数据的，可以将所有数据放到default中。
```javascript
//文件名为：module3.js
//默认暴露  可以暴露任意的数据类型，暴露什么数据，接收到的就是什么数据
//语法：export default value;

export default {
  msg: '默认暴露',
  foo() {
  	console.log(this.msg);
  }
}
```
```javascript
//引入其他的模块

//语法： import xxx from '路径'
import {foo, bar} from './module1';
import {fun, fun2} from './module2';
import module3 from './module3';

//console.log(module1, module2);

/*
 * 在ES6中，如果使用 分别暴露 或 统一暴露 的形式，
 * 	 则在引入时必须使用 对象结构赋值 的形式才能拿到模块对象中的数据
 */

foo();
bar();
fun();
fun2();

module3.foo();
```
运行结果为：
![在这里插入图片描述](JS模块化.assets/20190825222011404.png)
- - -
- - -
- - -
再来说说**第三方的模块怎么使用**。
首先，下载jQuery：
![在这里插入图片描述](JS模块化.assets/20190825225705308.png)
顺便一提：@后面可以加版本号，如图是1.11.3固定死了，如果是@1则是1.几的版本中最新的。
下载后package.json如下图所示：
![在这里插入图片描述](JS模块化.assets/20190825230003103.png)
```javascript
//引入其他的模块

//语法： import xxx from '路径'

//第三方库写在自定义的上方
import $ from 'jquery';

import {foo, bar} from './module1';
import {fun, fun2} from './module2';
import module3 from './module3';

//console.log(module1, module2);

/*
 * 在ES6中，如果使用 分别暴露 或 统一暴露 的形式，
 * 	 则在引入时必须使用 对象结构赋值 的形式才能拿到模块对象中的数据
 */

$('body').css('background', 'green');

foo();
bar();
fun();
fun2();
module3.foo();
```
重新打包编译。
运行成功，背景变为绿色。

# 其他
```
JS模块化
* 模块化的理解
* 什么是模块?
  * 将一个复杂的程序依据一定的规则(规范)封装成几个块(文件), 并进行组合在一起
  * 块的内部数据/实现是私有的, 只是向外部暴露一些接口(方法)与外部其它模块通信
* 一个模块的组成
  * 数据--->内部的属性
  * 操作数据的行为--->内部的函数
* 模块化
  * 编码时是按照模块一个一个编码的, 整个项目就是一个模块化的项目
* 模块化的进化过程
  * 全局function模式 : 
    * 编码: 全局变量/函数
    * 问题: 污染全局命名空间, 容易引起命名冲突/数据不安全
  * namespace模式 : 
    * 编码: 将数据/行为封装到对象中
    * 解决: 命名冲突(减少了全局变量)
    * 问题: 数据不安全(外部可以直接修改模块内部的数据)
  * IIFE模式/增强
    * IIFE : 立即调用函数表达式--->匿名函数自调用
    * 编码: 将数据和行为封装到一个函数内部, 通过给window添加属性来向外暴露接口
    * 引入依赖: 通过函数形参来引入依赖模块
      ···
      (function(window, module2){
        var data = 'atguigu.com'
        function foo() {
           module2.xxx()
           console.log('foo()'+data)
        }
        function bar() {
           console.log('bar()'+data)
        }
        
        window.module = {foo}
      })(window, module2)
      ···
* 模块化规范
  * CommonJS
    * Node.js : 服务器端
    * Browserify : 浏览器端    也称为js的打包工具
    * 基本语法:
      * 定义暴露模块 : exports
        ···
        exports.xxx = value
        module.exports = value
        ···
      引入模块 : require
        ···
        var module = require('模块名/模块相对路径')
        ···
    * 引入模块发生在什么时候?
      * Node : 运行时, 动态同步引入
      * Browserify : 在运行前对模块进行编译/转译/打包的处理(已经将依赖的模块包含进来了), 
                  运行的是打包生成的js, 运行时不存在需要再从远程引入依赖模块
  * AMD : 浏览器端
    * require.js
    * 基本语法
      * 定义暴露模块: define([依赖模块名], function(){return 模块对象})
      * 引入模块: require(['模块1', '模块2', '模块3'], function(m1, m2){//使用模块对象})
      * 配置: 
        ···
        require.config({
          //基本路径
          baseUrl : 'js/',
          //标识名称与路径的映射
          paths : {
            '模块1' : 'modules/模块1',
            '模块2' : 'modules/模块2',
            'angular' : 'libs/angular',
            'angular-messages' : 'libs/angular-messages'
          },
          //非AMD的模块
          shim : {
            'angular' : {
                exports : 'angular'
            },
            'angular-messages' : {
                exports : 'angular-messages',
                deps : ['angular']
            }
          }
        })
        ···
  * CMD : 浏览器端
    * sea.js
    * 基本语法
      * 定义暴露模块: 
        ···
        define(function(require, module, exports){
          通过require引入依赖模块
          通过module/exports来暴露模块
          exports.xxx = value
        })
        ···
      * 使用模块seajs.use(['模块1', '模块2'])
  * ES6
    * ES6内置了模块化的实现
    * 基本语法
      * 定义暴露模块 : export
        * 暴露一个对象: 
          ···
          export default 对象
          ···
        * 暴露多个: 
          ···
          export var xxx = value1
          export let yyy = value2
          
          var xxx = value1
          let yyy = value2
          export {xxx, yyy}
          ···
              
      * 引入使用模块 : import
        * default模块:
          ···
          import xxx  from '模块路径/模块名'
          ···
        * 其它模块
          ···
          import {xxx, yyy} from '模块路径/模块名'
          import * as module1 from '模块路径/模块名'
          ···
    * 问题: 所有浏览器还不能直接识别ES6模块化的语法  
    * 解决:
        * 使用Babel将ES6--->ES5(使用了CommonJS) ----浏览器还不能直接支行
        * 使用Browserify--->打包处理----浏览器可以运行
```
![在这里插入图片描述](JS模块化.assets/20190825230643854.png)
[前端模块化开发那点历史](https://github.com/seajs/seajs/issues/588) 
[CommonJS，AMD，CMD区别](http://zccst.iteye.com/blog/2215317) 
[AMD和CMD 的区别](http://www.zhihu.com/question/20351507/answer/14859415) 
[Javascript模块化编程](http://www.ruanyifeng.com/blog/2012/10/javascript_module.html) 