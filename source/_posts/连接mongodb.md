---
title: 使用mongoose连接mongodb
date: 2018-01-08 15:25:47
tags: 数据库
categories: mongodb
comments: true
---
## mongodb数据库
MongoDB是一个高效的基于分布式文件存储的数据库，将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组，很适合和nodejs搭配。

Mongoose是基于nodejs的一个mongodb对象模型工具，可以很方便的对mongodb进行操作。

### momngodb和mysql比较
对比项	| MongoDB	| MySQL
---|---|---
数据库 | 数据库(dataBase) | 数据库(dataBase)
表 | 集合(collection)| 二维表(table)
表在中的一行数据 | 文档(document) | 一条记录(record)
表字段 | 键(key) | 列(column)
主外键 | 无 | Primary Key
灵活度扩展性 | 高 | 差
表的关联性 | 差 | 高


## mongodb的安装

### windows下的安装
#### 安装
根据电脑操作系统位数[下载](https://www.mongodb.com/download-center#community)对应的安装包。安装过程几乎一路下一步就可以安装完成。一般默认安装在C:\Program Files\MongoDB。

安装完mongodb成后，在此，以安装在c盘路径下为例，创建文件夹 C:\data\db。

在mongodb的安装目录的bin文件下，执行命令C:\Program Files\MongoDB\Server\3.2\bin    mongod --dbpath c:\data\db，如果出现如下提示一般表示安装成功，这样mongodb的服务就启动了，ctrl+c可关闭服务。
![](/images/1.png)

再创建文件夹C:\data\log和文件C:\data\mongod.cfg。
mongod.cfg文件的内容为下：
```
systemLog:
    destination: file
    path: c:\data\log\mongod.log
storage:
    dbPath: c:\data\db
```
运行命令行执行mongod.cfg来制定数据库和日志文件目录：
C:\Program Files\MongoDB\bin\mongod.exe --config "C:\data\mongod.cfg" --install
#### 启动服务
C:\Program Files\MongoDB\Server\3.2\bin  net start MongoDB
启动服务状态下，浏览器打开 [http://localhost:27017](http://localhost:27017),如果出现以下内容，表示安装及启动mongodb服务成功。
![](/images/2.png])
#### 关闭服务
C:\Program Files\MongoDB\Server\3.2\bin  net stop MongoDB

#### 常用命令

安装路径下，运行mongo进入命令模式，C:\Program Files\MongoDB\Server\3.2\bin  mongo
db.help() 查看命令提示
db 查看当前所在数据库的名字 
use 数据库名   表示切换或是创建数据库
show dbs 显示有数据的库
db.stats() 查看当前数据的状态 
db.version() 查看mongoDB版本 
db.[数据库名].insert() 插入数据
db.[数据库名].find() 查找数据
![](/images/3.png)

### linux下的安装

#### 1. 首先查你的linux版本 
x86_64 表示你的操作系统为64位
$ uname -a    

#### 2.下载安装包
$ curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.0.6.tgz

#### 3. 解压
$ tar -zxvf mongodb-linux-x86_64-3.0.6.tgz  

#### 4. 将解压包拷贝到指定目录
$ mv  mongodb-linux-x86_64-3.0.6/ /usr/local/mongodb  

#### 5. 将可执行文件添加到path路径
export PATH=<mongodb-install-directory>/bin:$PATH
本文的安装路径在  /usr/local/mongodb，
所以运行 
$ export PATH=/usr/local/mongodb/bin:$PATH

#### 6. 创建数据库目录
mongodb的安装需手动撞见data存储目录，在此我们在根目录下创建data文件夹
$ mkdir -p /data/db

#### 7. 运行mongodb服务
在/usr/local/mongodb/bin路径下  
$ ./mongod

运行后打印如下，表示启动mongodb成功
2018-04-24T08:55:06.486+0000 I JOURNAL  [initandlisten] journal dir=/data/db/journal
2018-04-24T08:55:06.487+0000 I JOURNAL  [initandlisten] recover : no journal files present, no recovery needed
2018-04-24T08:55:11.438+0000 I JOURNAL  [initandlisten] preallocateIsFaster check took 4.951 secs
2018-04-24T08:55:11.536+0000 I JOURNAL  [durability] Durability thread started
2018-04-24T08:55:11.536+0000 I JOURNAL  [journal writer] Journal writer thread started

#### 8. mongodb 容器相关





## mongoose
### mongoose的核心概念
mongoose通过关系型数据库思维来设计数据库，有几个重要的概念，Schema，Model，Documents。其中，Schema用来定义数据库collection的结构，包含该字段和类型；Model是有Schema实例出的构造器，可以对数据库进行增删查改等操作；Document就是model的每一条数据。

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
```

### 查
Model.find(conditions, [projection], [options], [callback]).第一个参数conditions为查询条件，第二个参数doc为需要修改的数据，第三个参数options为控制选项，第四个参数是回调函数

find()找出列表-数组；
findById() 通过id找；
findOne找出列表第一个-文档对象
```js
const Answer = require('answer.js');

Answer.find({name: 'kiwi'}, (err, docs) => {
    console.log(docs);
})
```


### 改
Model.update(conditions, doc, [options], [callback])第一个参数conditions为查询条件，第二个参数doc为需要修改的数据，第三个参数options为控制选项，第四个参数是回调函数

update()；　
updateOne()-只能更新找到的第一条数据，即使设置{multi:true}也无法同时更新多个文档
find()/findOne()/findById() + save()；操作比较复杂的话可以用这种方法
```js
const Answer = require('answer.js');

Answer.update({name: 'kiwi'}, {name: 'kiwi2'} (err, docs) => {
    console.log(docs);
})
```

### 删
model.remove(conditions, [callback]).第一个参数conditions为查询条件，第二个参数回调函数的形式如function(err){}　
```js
const Answer = require('answer.js');

Answer.remove({name: 'kiwi'}, (err) => {
    
})
```　




