# 初见less
- - -
看如下代码：
```javascript
<head>
	<meta charset="UTF-8">
	<title></title>
	<style type="text/less">
		*{
			margin: 0;
			padding: 0;
		}
		#wrap{
			position: relative;
			width: 300px;
			height: 400px;
			border: 1px solid;
			margin: 0 auto;
		}
		#wrap .inner{
				position: absolute;
				left: 0;
				right: 0;
				top: 0;
				bottom: 0;
				margin: auto;
				background: pink;
				height: 100px;
				width: 100px;
			}
		}
	</style>
</head>
<body>
	<div id="wrap">
		<div class="inner">
			
		</div>
	</div>
</body>
```
当代码量较少时，这样的代码还是容易阅读的，但是随着代码的不断增加，就很难通过CSS看出HTML的嵌套结构。

再看下面的代码：
```javascript
<head>
	<meta charset="UTF-8">
	<title></title>
	<style type="text/less">
		*{
			margin: 0;
			padding: 0;
		}
		#wrap{
			position: relative;
			width: 300px;
			height: 400px;
			border: 1px solid;
			margin: 0 auto;
			.inner{
				position: absolute;
				left: 0;
				right: 0;
				top: 0;
				bottom: 0;
				margin: auto;
				background: pink;
				height: 100px;
				width: 100px;
			}
		}
	</style>
</head>
<body>
	<div id="wrap">
		<div class="inner">
			
		</div>
	</div>
</body>
```
此时，在CSS中很容易看出HTML的嵌套结构。但是这样的格式原生CSS并不认识，因此，需要用特定的工具进行编译或其他的辅助手段。
- - -
- - -
```
###less
	less是一种动态样式语言，属于css预处理器的范畴，它扩展了 CSS 语言，
	增加了变量、Mixin、函数等特性，使 CSS 更易维护和扩展
	LESS 既可以在 客户端 上运行 ，也可以借助Node.js在服务端运行。
 
	less的中文官网：http://lesscss.cn/
	bootstrap中less教程：http://www.bootcss.com/p/lesscss/
 
###Less编译工具
	koala 官网:www.koala-app.com 
```
- - -
因为现在还没有学过node，因此，先看原生CSS的版本(浏览器端用法)：

详细可以看[官网中的介绍](http://lesscss.cn/)。
提示我们需要引入一个脚本，借助这个JS文件进行编译，文件在上面的官网中可以下载。

如果我们的代码依赖于一个JS文件，则需要将JS文件放在代码的前面。但是我们编写CSS代码并不需要依赖这个JS文件，我们需要的是这个JS文件来对我们写好的CSS代码进行编译，因此，我们首先得有代码，然后才能进行编译，因而，我们需要将这个JS文件放在CSS代码的后面。
```javascript
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<!-- 1、告诉浏览器下面所写的是less格式的代码，需要编译，如果为"text/css"则不进行编译 -->
		<style type="text/less">
			*{
				margin: 0;
				padding: 0;
			}
			#wrap{
				position: relative;
				width: 300px;
				height: 400px;
				border: 1px solid;
				margin: 0 auto;
				.inner{
					position: absolute;
					left: 0;
					right: 0;
					top: 0;
					bottom: 0;
					margin: auto;
					background: pink;
					height: 100px;
					width: 100px;
				}
			}
		</style>
	</head>
	<body>
		<div id="wrap">
			<div class="inner">
				
			</div>
		</div>
	</body>
	<!-- 2、引入相应的文件 -->
	<script src="less/less.min.js"></script>
</html>
```
可以发现，这种方法并不好，因为需要引入JS文件，并且这是运行时编译，不是预编译。
- - -
- - -
下面使用编译工具：

新建less文件
<img src="https://img-blog.csdnimg.cn/20190901124528162.gif" width="50%">
在新建的less文件中编写less格式的CSS代码并保存：
```javascript
*{
    margin: 0;
    padding: 0;
}
#wrap{
    position: relative;
    width: 300px;
    height: 400px;
    border: 1px solid;
    margin: 0 auto;
    .inner{
        position: absolute;
        left: 0;
        right: 0;
        top: 0;
        bottom: 0;
        margin: auto;
        background: pink;
        height: 100px;
        width: 100px;
    }
}
```
然后将文件所在文件夹放入Koala：
（注意：放入的是文件夹，不是文件）
<img src="Less.assets/20190901125219972.png" width="50%">

然后点击Refresh刷新，就会出现编译好的CSS文件：
<img src="Less.assets/2019090112525014.png" width="50%">
最终我们使用的是编译后的CSS文件，之后对CSS文件的操作与其他普通的CSS文件相同，只需要引入即可，less文件只是让我们提高编写CSS的效率。


# less基础
## 注释
```
###less中的注释
   	以//开头的注释，不会被编译到css文件中
   	以/**/包裹的注释会被编译到css文件中  
```
less文件如下：
```javascript
/*这是想暴露出去的注释*/
//这是见不得人的注释
*{
    margin: 0;
    padding: 0;
}
```
编译后的css文件如下：
```javascript
/*这是想暴露出去的注释*/
*{
    margin: 0;
    padding: 0;
}
```

## 变量
```
###less中的变量
	使用@来申明一个变量
	1.作为普通属性值只来使用：直接使用@pink
		例如：
			@color: pink;
			*{
				background: @color;
			}
				
	2.作为选择器和属性名：#@{selector的值}的形式
		例如：
			@m:margin;
			@selector:#wrap;
			@{selector}{
				@{m}: 0;
			}
				
	3.作为URL：@{url}
		例如：
			@img: url(12345.jpg);
			*{
				background: @img;
			}
				
	4.变量的延迟加载
		less中以括号{}区分分为块级作用域
		例如：
			@var: 0;
			.class {
			@var: 1;							
			    .brass {
			      @var: 2;
			      three: @var;//结果为：3   brass{}中的内容全部读完才会加载@var，所以最终为@var: 3，值为3
			      @var: 3;
			    }
			  one: @var;  
			}
			
			编译后的结果为：
						.class {
						  one: 1;
						}
						.class .brass {
						  three: 3;
						}
```

## 嵌套规则
```
###less中的嵌套规则
	1.基本嵌套规则
		即与HTML结构相同(父子级关系)
		例如：
			#wrap{
			    margin: 0 auto;
			    .inner{
			        width: 100px;
			    }
			}		
			
	2.&的使用
		用&表示平级关系
		例如：
			#wrap{
			    margin: 0 auto;
			    .inner{
			        width: 100px;
			        &:hover{
			            background: pink;
			        }
			    }
			}
			
			编译后的结果为：#wrap .inner:hover{...}
			如果没有&则编译后的结果为：#wrap .inner :hover{...} 
									在 .inner 和 :hover 之间多一个空格，
									而这个空格是我们不想要的
```





# less混合
```
###less中的混合
	混合就是将一系列属性从一个规则集引入到另一个规则集的方式
	1.普通混合      
	2.不带输出的混合，加上()即可
	3.带参数的混合
	4.带参数并且有默认值的混合
	5.带多个参数的混合
	6.命名参数
	7.匹配模式
	8.arguments变量
```
- - -
## 1.普通混合
看如下所示的HTML代码，定义了2个div：
```javascript
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<link rel="stylesheet" type="text/css" href="css/01.css"/>
	</head>
	<body>
		<div id="wrap">
			<div class="inner"></div>
			<div class="inner2"></div>
		</div>
	</body>
	<script src="less/less.min.js"></script>
</html>
```
再看对应的CSS代码：
```javascript
*{
    margin: 0;
    padding: 0;
}
#wrap{
    position: relative;
    width: 300px;
    height: 400px;
    border: 1px solid;
    margin: 0 auto;
    .inner{
       position: absolute;
       left: 0;
       right: 0;
       top: 0;
       bottom: 0;
       margin: auto;
       background: pink;
       height: 100px;
       width: 100px;
    }
    .inner2{
       position: absolute;
       left: 0;
       right: 0;
       top: 0;
       bottom: 0;
       margin: auto;
       background: pink;
       height: 100px;
       width: 100px;
    }
}
```
可以发现，inner 和 inner2 有着很多的重复代码，我们希望将其提取出来。
在less中定义混合以 . 开头：
```javascript
//混合
.juzhong{
   position: absolute;
   left: 0;
   right: 0;
   top: 0;
   bottom: 0;
   margin: auto;
   background: pink;
   height: 100px;
   width: 100px;
}

*{
    margin: 0;
    padding: 0;
}
#wrap{
    position: relative;
    width: 300px;
    height: 400px;
    border: 1px solid;
    margin: 0 auto;
    .inner{
       .juzhong;
    }
    .inner2{
       .juzhong;
    }
}
```
编译后的css文件如下：
```javascript
.juzhong {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  margin: auto;
  background: pink;
  height: 100px;
  width: 100px;
}
* {
  margin: 0;
  padding: 0;
}
#wrap {
  position: relative;
  width: 300px;
  height: 400px;
  border: 1px solid;
  margin: 0 auto;
}
#wrap .inner {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  margin: auto;
  background: pink;
  height: 100px;
  width: 100px;
}
#wrap .inner2 {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  margin: auto;
  background: pink;
  height: 100px;
  width: 100px;
}
```

## 2.不带输出的混合
从 普通混合 的结果看，达到了我们需要的效果：可以将重复代码提取出来。但是我们可以发现，提取出来的代码也出现在编译后的css文件中，这不是我们想要的。

只需要在混合的后面加上()即可：
```javascript
//混合
//加上()
.juzhong(){
   position: absolute;
   left: 0;
   right: 0;
   top: 0;
   bottom: 0;
   margin: auto;
   background: pink;
   height: 100px;
   width: 100px;
}

*{
    margin: 0;
    padding: 0;
}
#wrap{
    position: relative;
    width: 300px;
    height: 400px;
    border: 1px solid;
    margin: 0 auto;
    .inner{
       .juzhong;
    }
    .inner2{
       .juzhong;
    }
}
```
此时编译后的css文件就不会再出现混合：
```javascript
* {
  margin: 0;
  padding: 0;
}
#wrap {
  position: relative;
  width: 300px;
  height: 400px;
  border: 1px solid;
  margin: 0 auto;
}
#wrap .inner {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  margin: auto;
  background: pink;
  height: 100px;
  width: 100px;
}
#wrap .inner2 {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  margin: auto;
  background: pink;
  height: 100px;
  width: 100px;
}
```

## 3.带参数的混合
现在我们想要使之拥有相同的属性和不同的值，这时就需要传入参数。
```javascript
//混合
//带参数
.juzhong(@w, @h, @c){
   position: absolute;
   left: 0;
   right: 0;
   top: 0;
   bottom: 0;
   margin: auto;
   background: @c;
   height: @h;
   width: @w;
}

*{
    margin: 0;
    padding: 0;
}
#wrap{
    position: relative;
    width: 300px;
    height: 400px;
    border: 1px solid;
    margin: 0 auto;
    .inner{
       .juzhong(100px, 100px ,pink);
    }
    .inner2{
       .juzhong(200px, 200px, deeppink);
    }
}
```
编译后的结果如下：
```javascript
* {
  margin: 0;
  padding: 0;
}
#wrap {
  position: relative;
  width: 300px;
  height: 400px;
  border: 1px solid;
  margin: 0 auto;
}
#wrap .inner {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  margin: auto;
  background: #ffc0cb;
  height: 100px;
  width: 100px;
}
#wrap .inner2 {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  margin: auto;
  background: #ff1493;
  height: 200px;
  width: 200px;
}
```

## 4.带参数并且有默认值的混合
只需要在形参后面加上 :值 即可。
```javascript
//混合
//带默认值的参数
.juzhong(@w: 10px, @h: 10px, @c: pink){
   position: absolute;
   left: 0;
   right: 0;
   top: 0;
   bottom: 0;
   margin: auto;
   background: @c;
   height: @h;
   width: @w;
}

*{
    margin: 0;
    padding: 0;
}
#wrap{
    position: relative;
    width: 300px;
    height: 400px;
    border: 1px solid;
    margin: 0 auto;
    .inner{
       .juzhong();
    }
    .inner2{
       .juzhong(200px, 200px, deeppink);
    }
}
```
编译后的结果为：
```javascript
* {
  margin: 0;
  padding: 0;
}
#wrap {
  position: relative;
  width: 300px;
  height: 400px;
  border: 1px solid;
  margin: 0 auto;
}
#wrap .inner {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  margin: auto;
  background: #ffc0cb;
  height: 10px;
  width: 10px;
}
#wrap .inner2 {
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  margin: auto;
  background: #ff1493;
  height: 200px;
  width: 200px;
}
```

## 5.带多个参数的混合
参照上面的，上面的已经是多个参数了

## 6.命名参数
现在我们想要在多个参数的时候只传一个特定的值，例如：
在```.juzhong(@w: 10px, @h: 10px, @c: pink){...}```中我们只想传入```black```作为```@c```的值，其他参数为默认值。
但是如果我们直接写```.juzhong(black);```则默认会将```black```作为```@w的参数```。

当实参和形参个数不匹配时，我们可以采用 形参: 值 的形式：
例如，上述的问题中可以写为：
```..juzhong(@c: black);```

## 7.匹配模式
首先，看如下代码：
```javascript
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
			#wrap .sjx {
			  width: 0px;
			  height: 0px;
			  overflow: hidden;
			  border-width: 40px;
			  border-style: dashed  dashed dashed solid;
			  border-color: transparent red transparent transparent;
			}
		</style>
	</head>
	<body>
		<div id="wrap">
			<div class="sjx"></div>
		</div>
	</body>
</html>
```
这段代码能画出一个三角形，具体代码这里就不再分析，如下图所示：
![在这里插入图片描述](Less.assets/20190901215549768.png)
现在，我们想要这个三角形能很方便地向上、向下、向左、向右。
我们先将其写为如下less：

```javascript
.triangle(@w, @c){
    width: 0;
    height: 0;
    border-width: @w;
    border-style: dashed solid dashed dashed;
    border-color: transparent @c transparent transparent;
    overflow: hidden;  
}

#wrap .sjx{
    .triangle(40px, red);
}
```
此时，我们可以将```.triangle```理解为一个库，可以将其写在另一个less文件中，使代码逻辑更清晰。这里我们将其放在一个叫```triangle.less```的less文件中，方便下面的分析。
然后我们需要将```triangle.less```引入：
```javascript
@import "./triangle.less";

#wrap .sjx{
   .triangle(40px,yellow)
}
```
但是，到了现在为止，仍然没有达到我们最初的要求，即能够将这个三角形很方便地向上、向下、向左、向右。
在```border-color: transparent @c transparent transparent;```中我们需要使```@c```的位置能够改变(```@c```的位置能改变三角形的方向)。
下面开始正式讲匹配模式：
less为我们提供了类似于标识符的功能。
例如，
添加一个参数```L```作为标识符
```javascript
.triangle(L,@w,@c){
    width: 0px;
    height: 0px;
    border-width: @w;
    border-style:dashed solid dashed dashed;
    border-color:transparent @c transparent transparent;
    overflow: hidden; 
}
```
然后进行调用：
```javascript
#wrap .sjx{
   .triangle(L,40px,yellow);
}
```
- - -
因此，我们可以这样写：
```javascript
.triangle(L,@w,@c){
	//向左
    width: 0px;
    height: 0px;
    border-width: @w;
    border-style:dashed solid dashed dashed;
    border-color:transparent @c transparent transparent;
    overflow: hidden; 
}

.triangle(R,@w,@c){
	//向右
    width: 0px;
    height: 0px;    
    border-width: @w;
    border-style:dashed  dashed dashed solid;
    border-color:transparent  transparent transparent @c;
    overflow: hidden; 
}

.triangle(T,@w,@c){
	//向上
    width: 0px;
    height: 0px;    
    border-width: @w;
    border-style:dashed  dashed  solid dashed;
    border-color:transparent  transparent @c transparent;
    overflow: hidden;     
}

.triangle(B,@w,@c){
	//向下
    width: 0px;
    height: 0px;    
    border-width: @w;
    border-style:solid dashed  dashed dashed;
    border-color:@c transparent  transparent transparent;
    overflow: hidden;     
}
```
然后需要哪个方向就调用相应的标识符，例如向下，则
```javascript
#wrap .sjx{
   .triangle(B,40px,red);
}
```

这样虽然达到了我们想要的功能，但是可以发现代码非常的冗余。
可以发现，以下代码是可以复用的：
```javascript
width: 0px;
height: 0px;    
overflow: hidden;   
```
因此，我们将其改为如下：
```javascript
// 定义一个相同名字、第一个参数为@_的混合，每次下面同名的混合被调用时，就会自动调用
// 注意：其他的形参也要带上。（形参名称可以不一样，但是个数要相同，但不推荐这样做）
.triangle(@_, @w, @c){
    width: 0px;
    height: 0px;
    overflow: hidden; 
}

.triangle(L, @w, @c){
    border-width: @w;
    border-style:dashed solid dashed dashed;
    border-color:transparent @c transparent transparent ;
}

.triangle(R, @w, @c){
    border-width: @w;
    border-style:dashed  dashed dashed solid;
    border-color:transparent  transparent transparent @c;
}

.triangle(T, @w, @c){
    border-width: @w;
    border-style:dashed  dashed  solid dashed;
    border-color:transparent  transparent @c transparent ;
}

.triangle(B, @w, @c){
    border-width: @w;
    border-style:solid dashed  dashed dashed;
    border-color:@c transparent  transparent transparent ;
}
```
编译后的结果为：
```javascript
#wrap .sjx {
  width: 0px;
  height: 0px;
  border-width: 40px;
  border-style: solid dashed  dashed dashed;
  border-color: #ffff00 transparent transparent transparent;
  overflow: hidden;
}
```

## 8.arguments变量
arguments表示传进来的所有参数。
```javascript
.border(@w, @style, @c){
//  border: @w @style @c;
	border: @arguments;
}

#wrap .sjx{
   .border(1px,solid,black)
}
```
编译后的结果为：
```javascript
#wrap .sjx {
  border: 1px solid #000000;
}
```

# less计算wmv
```
###less运算
	在less中可以进行加减乘除的运算
	参与计算的只需要一方带单位即可
```
```javascript
@rem: 100px;

#wrap .sjx{
   width:(100 + @rem)
}
```
编译后的结果为：
```javascript
#wrap .sjx {
  width: 200px;
}
```

# 复习
其中的部分内容详见 [尚硅谷_CSS3 笔记](https://blog.csdn.net/earthOLtainanwan/article/details/98942083) 。
```
###flex捋一捋
	0.flex frog练习
		http://flexboxfroggy.com/
		
	1.flex基础点
		---什么是容器，什么是项目，什么是主轴，什么是侧轴？
		---项目永远排列在主轴的正方向上
		---flex分新旧两个版本
			-webkit-box
			-webkit-flex / flex
	
	2.注意点
		---我们都知道新版本的flex要比老版本的flex强大很多，有没有必要学习老版本的flex？
			移动端开发者必须要关注老版本的flex
				因为很多移动端设备内核低只老版本的flex
		
		---老版本的box通过两个属性四个属性值控制了主轴的位置和方向
		      新版本的flex通过一个属性四个属性值控制了主轴的位置和方向
	
	3.老版本
		容器
			容器的布局方向
				-webkit-box-orient:horizontal/vertical
				控制主轴是哪一根
					horizontal：x轴
					vertical  ：y轴
			容器的排列方向
				-webkit-box-direction：normal/reverse
				控制主轴的方向
					normal：从左往右（正方向）
					reverse：从右往左（反方向）
			富裕空间的管理
				只决定富裕空间的位置，不会给项目区分配空间
				主轴
					-webkit-box-pack
						主轴是x轴
							start：在右边
							end:	在左边
							center：在两边
							justify：在项目之间
						主轴是y轴
							start：在下边
							end：在上边
							center：在两边
							justify：在项目之间
				侧轴
					-webkit-box-algin
						侧轴是x轴
							start：在右边
							end：  在左边
							center：在两边
						侧轴是y轴
							start：在下边
							end：   在上边	
							center：在两边
		项目
			弹性空间管理
				-webkit-box-flex：弹性因子（默认值为0）
						
	4.新版本
		容器
			容器的布局方向
			容器的排列方向
				flex-direction
				控制主轴是哪一根，控制主轴的方向
					row;		从左往右的x轴	
					row-reverse;从右往左的x轴
					column;		从上往下的y轴
					column-reverse;从下往上的y轴
			富裕空间的管理
				只决定富裕空间的位置，不会给项目区分配空间
				主轴
					justify-content
							flex-start：		在主轴的正方向
							flex-end:		在主轴的反方向
							center：			在两边
							space-between：	在项目之间
							 space-around：  在项目两边
							
				侧轴
					align-items
							flex-start：在侧轴的正方向
							flex-end：    在侧轴的反方向
							center：		在两边
							base#ne    基线对齐
         					stretch		等高布局（项目没有高度）	
		项目
			弹性空间管理
				flex-grow：弹性因子（默认值为0）
				
	5.新版本新增的
		容器
			flex-wrap：控制的是侧轴的方向
				nowrap 不换行
				wrap   侧轴方向由上而下 			（flex-shrink失效）
				wrap-reverse 侧轴方向由下而上 	（flex-shrink失效）
			
			align-content：多行/列时侧轴富裕空间的管理（把多行/列看成一个整体）
			
			flex-flow：flex-direction和flex-wrap的简写属性
				本质上控制了主轴和侧轴分别是哪一根，以及他们的方向
		
		项目
			order:控制项目的排列顺序
			align-self：项目自身富裕空间的管理
			flex-shrink：收缩因子（默认值为1）
			flex-basis：伸缩规则计算的基准值（默认拿width或height的值）
	
	6.伸缩规则
		flex-basis（默认值为auto）
		flex-grow（默认值为0）
			可用空间 = (容器大小 - 所有相邻项目flex-basis的总和)
			可扩展空间 = (可用空间/所有相邻项目flex-grow的总和)
			每项伸缩大小 = (伸缩基准值flex-basis + (可扩展空间 x flex-grow值))
		flex-shrink（默认值为1）
			   --.计算收缩因子与基准值乘的总和  
			   			var a = 每一项flex-shrink*flex-basis之和
			   --.计算收缩因数
			                     收缩因数=（项目的收缩因子*项目基准值）/第一步计算总和   
			             var b =  (flex-shrink*flex-basis)/a
			   --.移除空间的计算
			                      移除空间= 项目收缩因数 x 负溢出的空间 
			             var c =    b * 溢出的空间      
	
	7.侧轴富裕空间的管理
		单行单列
			align-items
			align-self（优先级高）
		多行多列
			align-content
	
	8.flex的简写属性
		flex:1  (flex-basis:0% flex-grow:1 flex-shrink:1)	
		等分布局	

###响应式布局核心 CSS3媒体查询选择器
	@media 媒体类型  and(关键字) 带条件的媒体属性 and 带条件的媒体属性 {
		//规则
	}

	媒体类型
		all
		screen
		print
		
	媒体属性
		
		width：浏览器的窗口尺寸
		device-width：设备尺寸
		device-pixel-ratio（必须加webkit前缀）：像素比
			---以上三个媒体属性可加 min 和 max 前缀
					min：最小值为谁
					max：最大值为谁
		
		横竖屏切换
			orientation：landscape（横屏）
			orientation：portrait （竖屏）
		
	关键字
		only：处理浏览器尽量问题
		and：连接一条查询规则
		,：连接多条查询规则
		not：取反
		
###多列布局

###规范
	HTML
		什么叫html5?   是一个强大的技术集（html5 ~ html+css+js）
	CSS
		什么是css3？
			css3其实就是html5的一部分，而且现代前端中已经没有版本的概念，都是级别
	JS
		ECMASCRIPT
		DOM
		BOM（没有规范，window）

###预处理器（less）
	变量
		@
		变量的延迟加载
		变量是块级作用域
	嵌套
		&:平级
	混合
		什么是混合？
			将一系列的规则集引入另一个规则集中（ctrl c+ctrl v）
			混合的定义在less规则有明确的指定，使用.的形式来定义
		普通混合（编译到原生css中的）
		不带输出的混合（加双括号）
		带参数的混合
		带默认值的混合
		匹配模式
		arguments
	计算
		加减乘除   计算的一方带单位就可以
	继承
	
```

# less继承
相当于Java的继承。允许一个选择器继承另一个选择器的样式。
继承的性能比混合高，但没有混合来得灵活。
less的继承将选择器组合后使用公共样式。

```javascript
/* 居中 */
.juzhong{
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    top: 0;
    margin: auto;
}

//.inner:extend 冒号后面不能有空格
.inner:extend(.juzhong){
  /*inner自己的代码*/
  background: red;
}
```
编译后的css代码为：
```javascript
.juzhong,
.inner {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  top: 0;
  margin: auto;
}
.inner {
  /*inner自己的代码*/
  background: red;
}
```
- - -
也有以下写法：

```javascript
&:extend(.juzhong);
```
这是```&```与```extend```结合使用。
- - -
all关键字用来继承所有的状态。
```javascript
.juzhong{
    position: absolute;
    left: 0;
    right: 0;
    bottom: 0;
    top: 0;
    margin: auto;
}

.juzhong:hover{
    background: red!important;
}
```
如果只是```.inner:extend(.juzhong)```则不会继承```.juzhong:hover```。
如果是```&:extend(.juzhong all);```则会继承hover。








# less避免编译
有的时候我们想要把一些代码直接交给浏览器去计算，但是写在less文件中时，less会帮我们计算出来，例如：
```javascript
*{
    margin: 100 *  10px;
    padding: cacl(100px + 100);
}
```
编译后的结果为：
```javascript
* {
  margin: 1000px;
  padding: cacl(200px);
}
```
但是我们想要把```cacl(100px + 100)```交给浏览器去计算，即想要less不编译这个代码。
我们可以加上```~" ... "```来让less不对此进行编译：
```javascript
*{
    margin: 100 *  10px;
    padding: ~"cacl(100px + 100)";
}
```
编译后的结果为：
```javascript
* {
  margin: 1000px;
  padding: cacl(100px + 100);
}
```