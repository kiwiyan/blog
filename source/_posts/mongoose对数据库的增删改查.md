---
title: mongoose对数据库的增删改查
date: 2018-05-12 10:43:58
tags: mongodb
categories: 数据库
comments: true
---

## mongoose
### mongoose的核心概念
mongoose通过关系型数据库思维来设计数据库，有几个重要的概念，Schema，Model，Documents。其中，Schema用来定义数据库collection的结构，包含该字段和类型；Model是有Schema实例出的构造器，可以对数据库进行增删查改等操作；Document就是model的每一条数据。
<!-- more -->
###  mongoose的使用
安装mongoose
```
npm install mongoose -S
```

### 连接数据库
通过connet()方法链接数据库，mongodb的默认端口是27017
```js
const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost:27017/test'， err => {
    if (err) {
        console.log('connect fail:', err)
    } else {
        console.log('connect success');
    }
});
```

## mongoose的常用操作
### 创建collection集合
```js
// answer.js
const mongoose = require('mongoose');

let schema = new mongoose.Schema({
    id: String,
    pic: String,
    name: String,
    answer: Array
});

module.exports = mongoose.model('answer', schema);
```
### 增 save(); create(); insertMany()
```js
// save()
const Answer = require('answer.js');
let answer = new  Collection.Answer({
    id: 'wx9528',
    pic: 'xx/t2.png',
    name: 'rose',
    answer: [1,1,3,2,3]
});
answer.save((err, doc) => {
    if (err) {
        console.log('保存失败')
    } else {
        console.log('save ok: ', doc)
    }
})

// create()
let insertData = {
    id: 'wx9528',
    pic: 'xx/t2.png',
    name: 'rose',
    answer: [1,1,3,2,3]
}
Collection.Answer.create(insertData, (err, docs) => {
    if (err) throw err;
    console.log(docs)
})
```

### 查
Model.find(conditions, [projection], [options], [callback]).第一个参数conditions为查询条件，第二个参数doc为需要修改的数据，第三个参数options为控制选项，第四个参数是回调函数。

常用方法：
find()找出列表-数组；
findById() 通过id找；
findOne找出列表第一个-文档对象

options参数解析：
safe (boolean)： 默认为true。安全模式。
upsert (boolean)： 默认为false。如果不存在则创建新记录。
multi (boolean)： 默认为false。是否更新多个查询记录。
runValidators： 如果值为true，执行Validation验证。
setDefaultsOnInsert： 如果upsert选项为true，在新建时插入文档定义的默认值。
strict (boolean)： 以strict模式进行更新。
overwrite (boolean)： 默认为false。禁用update-only模式，允许覆盖记录。
```js
const Answer = require('answer.js');

Answer.find({name: 'kiwi'}, (err, docs) => {
    console.log(docs);
})

// or
let data = await Collection.Answer.findOne({name: 'rose'});
ctx.body = data;
```


### 改
Model.update(conditions, doc, [options], [callback])第一个参数conditions为查询条件，第二个参数doc为需要修改的数据，第三个参数options为控制选项，第四个参数是回调函数

update();
updateOne()-只能更新找到的第一条数据，即使设置{multi:true}也无法同时更新多个文档
find()/findOne()/findById() + save()；操作比较复杂的话可以用这种方法
```js

Answer.update({name: 'kiwi'}, {name: 'kiwi2'} (err, docs) => {
    console.log(docs);
})

// or 操作比较复杂时，先修改值，再保存值
let kiwiData = await Collection.Answer.findOne({name: 'kiwi2333'});
kiwiData.name += '3';
kiwiData.save((err, doc) => {
    if (err) throw err;
    console.log('save doc', doc)
    ctx.response.body = doc;
})
```

### 删
model.remove(conditions, [callback]).第一个参数conditions为查询条件，第二个参数回调函数的形式如function(err){}　
```js

Answer.remove({name: 'kiwi'}, (err) => {
    
})
```　