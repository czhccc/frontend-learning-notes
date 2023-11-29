# node

![image-20231129154640590](nodejs.assets/image-20231129154640590.png)

## 实现一个自用脚手架



## 知识点

### Buffer

#### 是什么

计算机中的所有内容最终由二进制数据进行表示。

JavaScript可以处理非常直观的数据，比如字符串；但像图片的显示则其实并不是由JavaScript来处理的，而是交给浏览器完成的

8位的二进制数据 -> RGB值 -> 图片 -> 视频

我们做的事情是告诉浏览器一个图片地址，然后浏览器将图片下载后进行解析和显示



但是现在使用nodejs开发后台的话，就需要使JavaScript具备处理各类文件的能力而不止是字符串，但是使用JavaScript处理二进制数据是一件非常麻烦的事情，此时可以用node中提供的Buffer来处理二进制数据

【简单来说，在node中想要处理二进制的数据，就使用Buffer】



可以将Buffer看成是一个存储二进制的数组，每一项可以保存8位二进制：00000000 ~ 11111111

为什么是8位？

- 一位二进制只能表示0或1，能表示的数据太少了，因此通常将8位合在一起作为一个单元，成为一个字节byte（1byte = 8bit；1kb = 1024byte；1M = 1024kb）
- 其他许多编程语言中，int类型是4个字节，long类型是8个字节
- 比如TCP传输的是字节流，写入和读取时都需要说明字节的个数
- RGB值中一个单位是0~255，即2^8，即一个字节
- 总之就是为了能与其他地方更好的兼容性

#### 做什么

```js
const message = "Hello"

// 创建方式1，已过期
const buffer1 = new Buffer(message)
console.log(buffer1) // <Buffer 48 65 6c 6c 6f> 【默认显示的是十六进制】

// 创建方式2
const buffer2 = Buffer.from(message)
console.log(buffer2) // <Buffer 48 65 6c 6c 6f> 【默认显示的是十六进制】

const message2 = "你好啊" // 中文
const buffer3 = Buffer.from(message)
console.log(buffer3) // <Buffer e4 bd a0 e5 a5 bd e5 95 8a> // 在utf8中，一个中文汉字在编码时一般是3个字节
const buffer4 = Buffer.from(message, 'utf16le') // 第二个参数可以指定编码方式
console.log(buffer4) // <Buffer 60 4f 7d 59 4a 55>
const message3 = buffer4.toString('utf16le') // 进行解码，可以传入解码方法
console.log(message3) // “你好啊”
```

## 事件队列

事件循环像是一个桥梁，连接应用程序JS代码和系统调用：

- 文件IO、数据库、网络IO、定时器、子进程，在完成对应的操作后，都会将对应的结果和回调函数放到事件循环（任务队列中）
- 事件循环会不断的从任务队列中取出对应的事件（回调函数）来执行

在node中，一次完整的事件循环Tick分成很多歌阶段：

- 定时器（Timers）：本阶段执行已经被setTimeout()和setInterval()的调度回调函数
- 待定回调（Pending Callback）：对某些系统操作（如TCP错误类型）执行回调，比如TCP连接时收到ECONNREFUSED
- idle，prepare：仅系统内部使用
- 轮询（Poll）：检索新的I/O事件；执行与I/O相关的回调【相比而言，这个阶段会非常长，因为node想要I/O事件能尽可能早地响应；甚至有I/O事件是会进行阻塞】
- 检测：setImmediate() 回调函数在这里执行
- 关闭的回调函数：一些关闭的回调函数，如：socket.on('close'm ...)

