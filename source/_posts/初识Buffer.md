---
title: 初识Buffer
date: 2018-04-03 20:53:45
tags: node核心模块
categories: nodejs
comments: true
---


## Buffer对象

### 什么是Buffer？
Buffer 是 Node.js 中用于处理二进制数据的类, 其中与 IO 相关的操作 (网络请请求/文件读写等) 均基于 Buffer. Buffer 类的实例非常类似整数数组, 但其大小是固定不变的, 并且其内存在 V8 堆栈外分配原始内存空间。 Buffer 类的实例创建之后, 其所占用的内存大小就不能再进行调整。

## Buffer对象的创建
Buffer对象类似于数组，每个元素由两个16进制的整数组成。
Node 6.x之后，new Buffer()的方式被弃用。创建Buffer对象常用的方法有：

api | 解释
---|---
Buffer.from() | 根据已有数据生成一个 Buffer 对象
Buffer.alloc() | 创建一个初始化后的 Buffer 对象


```js
//创建一个根据字符串生成的buffer
let str = '床前明月光，疑是地上霜，举头望明月，低头思故乡';
let buffer1 = Buffer.from(str);
console.log(buffer1); //<Buffer e7 aa 97 e5 89 8d e6 98 8e e6 9c 88 e5 85 89 ef bc 8c e7 96 91 e6 98 af e5 9c b0 e4 b8 8a e9 9c 9c ef bc 8c e4 b8 be e5 a4 b4 e6 9c 9b e6 98 8e e6 9c 88 ef bc 8c e4 bd 8e e5 a4 b4 e6 80 9d e6 95 85 e4 b9 a1>

// 创建一个长度为 10、且用 0x1 填充的 Buffer。
let buffer2 = Buffer.alloc(10, 1);
console.log(buffer2); //<Buffer 01 01 01 01 01 01 01 01 01 01>

// 创建一个长度为 10、且未初始化的 Buffer。
let buffer3 = Buffer.allocUnsafe(10);
console.log(buffer3); //<Buffer 00 00 00 00 00 00 00 00 00 00>
```

## Buffer的转换
### Buffer转成字符串 
```js
// 读取文件数据
fs.readFile('./test.txt', (err, bufferData) => {
    console.log(bufferData.toString()); //通过toString 转换成字符串，toString可传一个参数，包含ASCII，utf-8,base64,binary等，默认为utf-8
});
```
### 将图片转成base64格式字符串
```js
// 读取文件将图片转为base64格式的字符串
let bufferFile = fs.readFileSync('test.png');
let baseData = data.toString('base64);
let base64Data = `data:image/png;base64,${baseData}`;
console.log(base64Data); // data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAI...

```


## Buffer拼接
### 中文乱码问题。
中文在utf-8编码下占3个元素，英文和半角符占1个字符。还是以“床前明月光，疑是地上霜，举头望明月，低头思故乡”这首诗举例，这首诗文本生成的Buffer打印出来为
```
<Buffer e7 aa 97 e5 89 8d e6 98 8e e6 9c 88 e5 85 89 ef bc 8c e7 96 91 e6 98 af e5 9c b0 e4 b8 8a e9 9c 9c ef bc 8c e4 b8 be e5 a4 b4 e6 9c 9b e6 98 8e e6 9c 88 ef bc 8c e4 bd 8e e5 a4 b4 e6 80 9d e6 95 85 e4 b9 a1>
```
e7 aa 97表示汉字“床”，e6 9c 88则表示第4个字“月”，但在此设置buffer对象长度为11，第10个元素即第4个字被截断了，就出现了乱码。
```js
let fs = require('fs');

// 假设test.txt 文本内容为“床前明月光，疑是地上霜，举头望明月，低头思故乡”
let rs = fs.createReadStream('./test.txt', {
    highWaterMark: 11 //最大限制11个字节读取
});
let txt = '';
rs.on('data',(chunk) => {
    txt += chunk; // 隐含一个操作，相当于 txt = txt.toString() + chunk.toString()
});
rs.on('end',() => {
    console.log(txt); //窗前明��光，疑���地上霜，举头��明月，���头思故乡
});

```
### 如何解决中文乱码问题？
一个方法是将多个小Buffer拼接成一个大Buffer，避免了单个转换造成的部分中文字节不完整造成的乱码现象。
```js
let fs = require('fs');

let rs = fs.createReadStream('./test.txt', {
    highWaterMark: 11
});
let buf = [];
let size = 0;
rs.on('data',chunk => {
    buf.push(chunk);
    size += chunk.length;
});
rs.on('end',() => {
    let buffer1 = Buffer.concat(buf, size);
    console.log(buffer1.toString());  //窗前明月光，疑是地上霜，举头望明月，低头思故乡
});

```

### Buffer 其他api

concat 连接buffer，slice 截取，equal 比较是否相同等,api操作与数组比较相似。



