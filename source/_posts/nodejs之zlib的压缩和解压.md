---
title: nodejs之zlib的压缩和解压
date: 2018-06-12 18:44:38
tags: nodejs模块
categories: nodejs
comments: true
---


nodejs的zlib模块可以很方便的实现文件的压缩和解压，常用于服务端的gzip压缩，用来提高网页加载速度。
<!-- more -->

## 压缩
通过zlib的createGzip方法实现压缩。
```js
const fs = require('fs');
const zlib = require('zlib');

let gzip = zlib.createGzip();

let inFile = fs.createReadStream(`${__dirname}/test.js`);
let out = fs.createWriteStream(`${__dirname}/test.js.zip`);

inFile.pipe(gzip).pipe(out);
```
将本地的test.js文件打包成压缩包test.js.zip。
其他压缩方式还有zlib.createDeflate()等方式。

## 解压
通过zlib的createGunzip方法实现压缩。
```js
const fs = require('fs');
const zlib = require('zlib');

let gzip = zlib.createGunzip();

let inFile = fs.createReadStream(`${__dirname}/test.js.zip`);
let out = fs.createWriteStream(`${__dirname}/test.js`);

inFile.pipe(gzip).pipe(out);
```
将本地的test.js.zip文件解压为test.js。
其他解压方法是有zlib.createUnzip()等。

## 服务端gzip压缩
开启网站的 gzip 压缩功能，可以减小响应文件内存大小，也就是说，如果你的网页的一个js文件有300K，压缩之后可能就只有100K， 对于大部分网站，显然可以明显提高浏览速度。

```js
const http = require('http'); 
const fs = require('fs');
const zlib = require('zlib');

http.createServer((req, res) => {
    let acceptEncoding = req.headers['accept-encoding'];
    if (acceptEncoding.indexOf('gzip') >= 0){ // 判断是否需要gzip压缩
        console.log('to zlib');
        res.writeHead(200, {
            'Content-Encoding': 'gzip'
        });
        let path = `${__dirname}/test.html`;
        fs.createReadStream(path).pipe(zlib.createGzip()).pipe(res);
    
    } else {
        fs.createReadStream(filepath).pipe(res);
    }
}).listen(5001);

console.log('http://localhost:5001')
```

当客户端发送Accept-Encoding:gzip这个request header，服务器即认为其能接受gzip压缩，就响应一个Content-Encoding:gzip，并发送压缩内容；假如客户端没有发送 Accept-Encoding，那么服务器就返回未压缩的源码。一般在实际应用中，并不是对所有文件进行压缩，通常只是压缩静态文件。 
```js
// Request Headers  请求头  
Accept-Encoding: gzip, deflate, br

// Response Headers 响应头
Content-Encoding: gzip

```

gzip 压缩功能需要浏览器支持。HTTP协议上的gzip编码是一种用来改进web应用程序性能的技术，web服务器和客户端（浏览器）必须共同支持gzip。目前主流的浏览器，Chrome,firefox,IE等都支持该协议。常见的服务器如Apache，Nginx，IIS同样支持gzip。
