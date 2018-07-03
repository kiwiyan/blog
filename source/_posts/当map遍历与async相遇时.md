---
title: 当map遍历与async相遇时
date: 2018-07-03 20:25:38
tags: js
categories: es6
comments: true
---

通过nodejs读取数据库，然后想遍历修改下获取的数据，很自然的想到通过async异步获取数据，然后map遍历修改数据，如下：
<!-- more -->

## async await与promise
```js
let arr1 = ['1', '2', '3'];
let result = [];

function findById(id) {
    let database = {
        '1': 'one',
        '2': 'two',
        '3': 'three'
    };
    return new Promise((resolve, reject) => {
        resolve(database[id]);
    });
}

result = arr1.map(async item => {
    let itemValue = await findById(item);
    return `item-${itemValue}`;
});

console.log(result)
```

本以为正常的输出结果是: 
```js
['iten-one', 'item-two', 'item-three]
```
但是实际结果却是
```js
[ 
    Promise { <pending> },
    Promise { <pending> },
    Promise { <pending> } 
]
```
是的，输出的是一个Promise对象的数组。
其实这也不难理解，因为map方法遍历时，每一次循环操作是异步操作，通过await 将每项结果处理成了Promise对象。

## Promise.all()
Promise.all()可传入一个Promise对象数组，如果我们想获得正常期待的结果，那么可以通过Promise.all()，即等所有异步Promise状态都完成时才获取结果。
```js
let arr1 = ['1', '2', '3'];
let result = [];

function findById(id) {
    let database = {
        '1': 'one',
        '2': 'two',
        '3': 'three'
    };
    return new Promise((resolve, reject) => {
        resolve(database[id]);
    });
}

result = Promise.all(arr1.map(async item => {
    let itemValue = await findById(item);
    return `item-${itemValue}`;
})).then(data => {
    console.log(data); //['iten-one', 'item-two', 'item-three]
})
```

或者通过await和async搭配来获取，需要在外层包一层async 方法，因为await关键字使用只能在async方法里。
```js
let arr1 = ['1', '2', '3'];
let result = [];

async function init() {
    function findById(id) {
        let database = {
            '1': 'one',
            '2': 'two',
            '3': 'three'
        };
        return new Promise((resolve, reject) => {
            resolve(database[id]);
        });
    }

    result = await Promise.all(arr1.map(async item => {
        let itemValue = await findById(item);
        return `item-${itemValue}`;
    }));
    console.log(result); // ['iten-one', 'item-two', 'item-three]
}

init();
```

## Promise.race()

Promise.race()可传入一个Promise对象数组，race是比赛的意思，顾名思义，Promise.race()返回率先改变状态的那个Promise对象。
async常使用try catch来捕获异常，不仅可以捕获reject状态，也可捕获其他比如语法等错误。

```js

let promise1 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve('test1')
    }, 100)
});

let promise2 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve('test2')
    }, 200)
});

let promise3 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve('test1')
    }, 300)
});

async function getValue() {
    try {
        let data = await Promise.race([promise1, promise2, promise3]);
        console.log(data)
    } catch (err) {
        console.log('err:', err);
    }
};
getValue(); // 'test1'

```