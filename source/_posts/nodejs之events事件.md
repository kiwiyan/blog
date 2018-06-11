---
title: nodejs之events事件
date: 2018-06-11 18:42:39
tags: nodejs模块
categories: nodejs
comments: true
---
events模块是node的核心模块，几乎所有常用的node模块都继承了events模块，比如http、fs等。
EventEmitter是events的一个类，这个类在node的内置模块和第三方模块中大量使用，因此理解和掌握EventEmitter的用法非常重要。

<!-- more -->

```js
// events模块的EventEmitter属性指向该模块本身
console.log(events.EventEmitter === events); 

// 输出
// true
```

## 事件监听和触发

要使用EventEmitter，首先需要先继承它。
使用on方法来监听，使用emit方式来触发事件。
on还有另一个别名，即addListener，也可通过addListener监听事件，和on使用方式一致。

```js
const events = require('events');

// es6继承。当然也可以使用es5的继承方式，或nodejs的util模块里的inherits方法继承。
class Person extends events.EventEmitter {
    constructor() {
        super();
    }
}

let person = new Person();

person.on('say', function(){
    console.log('hello kiwi!');
});

person.emit('say');

// 输出
// hello kiwi!

```


### 同一个事件，多个监听器

每个事件默认最多注册10个监听器，可以通过emitter.setMaxListeners(n) 方法改变。

```js
const events = require('events');

class Person extends events {
    constructor() {
        super();
    }
}

let person = new Person();

person.on('say', function(){
    console.log('hello kiwi!');
});

person.on('say', function(){
    console.log('yo yo,Check it out!');
});

person.emit('say');
// 输出
// hello kiwi!
// yo yo,Check it out!

```


### once监听一次
通过once注册事件，事件只会触发一次。
```js
const events = require('events');

class Person extends events {
    constructor() {
        super();
    }
}

let person = new Person();

person.once('say', function(){
    console.log('hello kiwi!');
});

person.emit('say');
person.emit('say'); 

// 输出
// hello kiwi!
```

### 追踪现有监听器
EventEmitter有一个特殊的事件叫newListener,可以追踪监听器何时被添加，或者查询现有的监听器。
```js
const events = require('events');

class Person extends events {
    constructor() {
        super();
    }
}

let person = new Person();

person.on('newListener', (name, listener) => {
    console.log(`eventName:${name}, listener: ${listener}`);
})

person.on('say', function(){
    console.log('hello kiwi!');
});

person.on('eat', function(){
    console.log('eating kiwi~~');
});

// 输出
// eventName:say, listener: function (){
//     console.log('hello kiwi!');
// }
// eventName:eat, listener: function (){
//     console.log('eating kiwi~~');
// }
```
如果把newListener的监听器放在所有监听器之后，看看会发生什么？

### 移除监听器
通过removeListener移除监听器。
注意removeListener只能移除有命名的监听器。
另外，通过person.removeAllListeners可移除某个事件的所有监听。

```js
const events = require('events');

class Person extends events {
    constructor() {
        super();
    }
}

let person = new Person();

person.on('say', function(){
    console.log('hello kiwi!');
});

function sayHi() {
    console.log('yo yo,Check it out!');
}
person.on('say', sayHi);


person.removeListener('say', sayHi);
// person.removeAllListeners('say') 移除所有say事件的监听器

person.emit('say'); 

// 输出
// hello kiwi!
```



## 应用
### 异常管理
在error事件上添加一个监听器，可以在有异常情况时触发来管理异常。


```js
const events = require('events');

class Person extends events {
    constructor() {
        super();
    }
}

let person = new Person();

person.on('error', err => {
    console.log('err:', err);
});

person.emit('error', 'some error');

// 输出
// err: some error
```
koa框架的错误监听就是基于这种方式实现。

```js
const Koa = require('koa');
const app = new Koa();

const main = ctx => {
  ctx.throw(500);
};

app.on('error', (err, ctx) => {
  console.error('server error', err);
});

app.use(main);
app.listen(3000);

```

### 组件通信
一个项目的模块之间可能需要通信，通过on和emit的方式可以实现。

```js
const koa  = require('koa');

const app = new koa();

app.use(async (ctx, next) => {
    if (ctx.request.path === '/') {

        ctx.app.emit('hello_kiwi', 'kiwi msg...'); // koa的app实例继承自events。可以将参数通过emit传递到另一个模块。
        ctx.response.body = 'kiwi';
    }
});

app.on('hello_kiwi', msg => {
    console.log('hi:', msg)
});

app.listen(5000);
console.log('http://localhost:5000');

// 打开浏览器访问 http://localhost:5000， 输出
// hi: kiwi msg...

```

### 处理数据流 
包含http，fs等nodejs模块对数据流的监听，也是基于events实现的。
在此实现一个大型文件的拷贝方法。通过监听data，error，end事件，实现对数据流的管理。
```js
const fs = require('fs');

function fileCopy(filename1, filename2, done) {
    var input = fs.createReadStream(filename1);
    var output = fs.createWriteStream(filename2);

    input.on('data', function (d) {
        output.write(d);
    });
    input.on('error', function (err) {
        throw err;
    });
    input.on('end', function () {
        output.end();
        if (done) done();
    });
}
```