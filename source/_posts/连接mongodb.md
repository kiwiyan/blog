---
title: 使用mongoose连接mongodb
date: 2018-01-08 15:25:47
tags: mongodb
categories: 数据库
comments: true
---
## mongodb数据库
MongoDB是一个高效的基于分布式文件存储的数据库，将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组，很适合和nodejs搭配。

Mongoose是基于nodejs的一个mongodb对象模型工具，可以很方便的对mongodb进行操作。
<!-- more -->

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

安装路径(/usr/local/mongodb/bin)下，运行mongo(./mongo)进入命令模式，C:\Program Files\MongoDB\Server\3.2\bin  mongo
```
db.help() 查看命令提示
db 查看当前所在数据库的名字 
use 数据库名   表示切换或是创建数据库
show dbs 显示有数据的库
db.stats() 查看当前数据的状态 
db.version() 查看mongoDB版本 
db.[数据库名].insert() 插入数据
db.[数据库名].find() 查找数据
![](/images/3.png)
```

### linux下的安装

#### 1. 首先查你的linux版本 
x86_64 表示你的操作系统为64位
```
$ uname -a    
```

#### 2.下载安装包
```
$ curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.0.6.tgz
```
#### 3. 解压
```
$ tar -zxvf mongodb-linux-x86_64-3.0.6.tgz  
```

#### 4. 将解压包拷贝到指定目录
```
$ mv  mongodb-linux-x86_64-3.0.6/ /usr/local/mongodb  
```

#### 5. 将可执行文件添加到path路径
export PATH=<mongodb-install-directory>/bin:$PATH
本文的安装路径在  /usr/local/mongodb，
所以运行 
```
$ export PATH=/usr/local/mongodb/bin:$PATH
```

#### 6. 创建数据库目录
mongodb的安装需手动创建data存储目录，在此我们在根目录下创建data/mongo文件夹
```
$ mkdir -p /data/mongo
```

然后在mongo文件夹里创建mongodb.cnf配置文件，用来设置mongodb的数据库路径以及启动参数等。
内容可以如下。
其中，dbpath表示mongodb数据库的路径，logpath是数据库日志路径，port表示连接端口，auth为true表示通过密码验证。

其中注意，auth不设置或设置为false的话，数据库表示没有密码，可以直接连接。自己测试的话可以不设密码，但正式项目中一定要设置密码验证，
否则任何人都可以轻易侵入你的数据库。

```
dbpath=/data/mongo/

logpath=/data/mongo/mongo.log

logappend=true

fork=true

port=27017

auth=true
```


#### 7. 运行mongodb服务
在/usr/local/mongodb/bin路径下，运行命令，-f 后的路径为自己创建的mongodb配置文件(见第6步) 
```
$ ./mongod -f /data/mongo/mongodb.cnf
```

如果不创建mongodb.cnf的话，也可以用加参数的方式启动mongodb,效果和启动配置文件一样。但推荐配置文件的方式，修改和启动都方便。

```
$ mongod -dbpath /data/mongo/ -logpath /data/mongo/mongo.log -logappend -fork -port 27017 -auth
```

运行后打印类似如下，表示启动mongodb成功
```
2018-04-24T08:55:06.486+0000 I JOURNAL  [initandlisten] journal dir=/data/db/journal
2018-04-24T08:55:06.487+0000 I JOURNAL  [initandlisten] recover : no journal files present, no recovery needed
2018-04-24T08:55:11.438+0000 I JOURNAL  [initandlisten] preallocateIsFaster check took 4.951 secs
2018-04-24T08:55:11.536+0000 I JOURNAL  [durability] Durability thread started
2018-04-24T08:55:11.536+0000 I JOURNAL  [journal writer] Journal writer thread started
```

### 8. 数据库的重启

#### 关闭数据库：
切入admin数据库中，然后运行db.shutdownServer() 关闭数据库。
有时关闭失败，可能有其他误操作，这时在/data/mongo下运行rm -f mongod.lock删掉 /data/mongo/mongod.lock文件，再重启就好了。

```
$ use admin
$ db.shutdownServer() 
```

但有时运行db.shutdownServer()依然不能关闭数据库，这是可以采用一种强力的方法，用命令强行杀死mongodb的进程。
```
$ netstat -tln    // 查看进程
$ netstat -tln | grep 27017  //查看mongo进程
$ lsof -i:27017 // 查看27017端口被哪个应用占用，找出pid (如果没有lsof，centos下先安装 yum install lsof)
$ kill -9 9527  // 9527是mongodb进程的pid

```
#### 启动数据库
```
mongod -f /data/mongo/mongodb.cnf
```
启动成功后会有successfully的提示符


#### 8. 设置数据库用户及密码

默认安装mongodb，如果直接创建数据库的话，是没有密码的，在真实开发环境中，很有必要为数据库设置密码来保障安全。

Mongodb用户有全局用户和数据库对应用户。

mongodb有一个admin数据库，用于创建全局用户账号，这个账号可以访问可连接到其他数据库。

创建admin账号
```
$ use admin  // 切到admin数据库
$ db.createUser({user: 'root', pwd: 'test', roles: ['root']})  // 创建账号和密码
$ db.auth('root', 'test') // 验证用户身份 返回1为正确, 0为错误
```

可以在可视化工具里通过admin账号连接到所有的数据库。

创建普通数据库账号，方式同创建admin账号，只是要先切进到对应的数据库。
```
$ use somedbname  // 切到普通数据库
$ db.createUser({user:'root',pwd:'test2',roles: [{role:'readWrite',db:'somedbname'}]})  // 创建账号和密码,设置权限为可读写
$ db.auth('root', 'test2') // 验证用户身份 返回1为正确, 0为错误
```

项目开发中比如要连接某个数据库：
```
mongodb://root:test2@127.0.0.1:27017/somedbname
```
注意一点，如果密码有@的话，需要转译成'%40'

