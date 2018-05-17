---
title: docker初探
date: 2018-01-16 10:22:17
tags: docker
categories: linux
comments: true
---

## yum 安装docker
```
yum install docker -y
```
<!-- more -->
## 启动
```
sercive docker start
```

## 安装镜像
```
docker run -i -t ubuntu /bin/bash
docker  run -it  -p  30080:80  -p 30020-30023:20-23   --name 30.kiwilab.jd.com   fe:1.0  /bin/bash 
```
## 删除镜像
docker rmi
## docker中安装vim
apt-get update
apt-get install -y vim
创建文件：vi 文件名
创建文件夹： mkdir 文件夹
exit 从容器退出到宿主机 此时容器停止运行;重启容器： docker start container_name/id
进入容器：docker attach container_name

(卸载apt-get安装的软件 apt-get remove vim)
## 容器篇

### 创建容器
创建容器服务器篇

#### 1. 创建容器
docker  run -it  -p  30080:80  -p 30020-30023:20-23   --name 30.kiwilab.jd.com   fe:1.0  /bin/bash 

其中，32306:3306等表示端口映射，外部32036映射到内部的mysql的3306(mysql默认端口);同理外部的32017映射到内部的mongodb的27017(mongodb默认端口)；
这些外部的端口可以直接用来连接内部的数据库。
docker  run -it  -p  32080:80  -p 32020-32023:20-23 -p 32017:27017  -p 32306:3306  --name 32.kiwilab_database   fe:1.0  /bin/bash 

另外容器的时间默认和宿主机不同步，宿主机采用了CST时区即东八区，容器采用了UTC时区即标准时间，创建容器时，时间比宿主机早了8个小时。解决方案：

docker  run -it  -p  32080:80  -p 32020-32023:20-23 -p 32017:27017  -p 32306:3306  --name 32.kiwilab_database -v /etc/localtime:/etc/localtime:ro  fe:1.0  /bin/bash 


创建mongodb容器
docker  run -it  -p  34080:80  -p 34020-34023:20-23 -p 34017:27017  --name 34.kiwilab_mongodb -v /etc/localtime:/etc/localtime:ro  mongo:3.4  /bin/bash 
#### 2. 添加宿主机的nginx配置文件
nginx conf.d 下创建 kiwilab.jd.com的文件（名称自定义）,添加配置内容 

server {
    listen       80; #监听你自己服务器的80端口
    server_name  kiwilab.jd.com; #你的域名
    location / {
        proxy_pass       http://127.0.0.1:30080; #你的容器对外端口
        proxy_redirect   off;
        proxy_set_header Host    $host;
        proxy_set_header X-Forwarded-For $remote_addr;
    }
}

服务器的nginx的话，以mongodb为例
server {
    listen       27017; #监听你容器内部的mongodb默认端口
    server_name  kiwilab.jd.com; #你的域名
    location / {
        proxy_pass       http://127.0.0.1:32017; #你的mongodb容器对外端口，这个端口是创建容器时设置的，最好创建时就设置好端口映射
        proxy_redirect   off;
        proxy_set_header Host    $host;
        proxy_set_header X-Forwarded-For $remote_addr;
    }
}

#### 3 重启nginx  
whereis nginx  获取nginx安装路径
nginx 的sbin路径下 ./nginx -s reload 重启nginx，这样nginx的配置修改就有效了

#### 4. 进入你的容器，创建你的server
启动一个服务（启动服务的方式很多，比如容器内的nginx，或者http-server，或者手写一个如下）
var http = require('http');

http.createServer(function(req,res){
res.end('ok');

}).listen(80); // 

#### 5. 开启防火墙
可能需要开启防火墙才能“直接”访问（防火墙可能默认关闭直接访问ip端口路径） http://192.168.182.85:30080/ 返回容器里的服务响应
vi /etc/sysconfig/iptables  添加内容：
-A INPUT -p tcp -m state --state NEW -m tcp --dport 30080 -j ACCEPT   #添加30080端口的防火墙开关
service iptables restart   修改后重启防火墙

#### 6. 访问http://192.168.182.85:30080/
页面响应内容-ok
或者配置host 

### 删除容器
（如果容器正在运行需要先关掉容器 docker stop/kill container_name）
docker rm container_id/container_name

### 查看容器
docker ps -a  展示容器列表
docker attach container_name   进入容器
exit   退出容器到宿主机

### 重启容器
启动 docker start container_id
重启 docker restart container_id


## 安装git
ubuntu中 apt-get install git
### linux下 解决git提交冲突
需要执行下面的命令才能修复：

git reset --hard HEAD    
git clean -f -d    
git pull

## 下载
apt-get install/remove/update xxx 是ubuntu下的安装/卸载/更新软件方式
yum install/remove/update xxx是redhat centos下的安装/卸载/更新方式 


## linux常用操作
### 创建文件
vi filename 创建或编辑文件
rm filename 删除文件
### 进入vim编辑模式

#### 编辑模式

dd(按双下d键)   －－  删除一行
x键     －－  删除当前字母

#### 命令模式
按esc退出到命令模式
i 键     －－  从命令模式到编辑模式
u 撤销到上一步
ctrl + r 反撤销
:q! (命令)     －－  无保存退出
:wq (命令)  －－  保存后退出

## nginx篇

### nginx安装路径
whereis nginx  查找nginx安装路径，一般的端口域名配置文件在conf.d文件夹里
find /|grep nginx.conf    找到对应docker的nginx配置文件

### 重启nginx
nginx的sbin路径下，输入命令 ./nginx -s reload

 curl localhost:28080

### 命令历史记录
history 10  最近10条历史命令记录
history | more 所有历史记录(默认1000行)

server {
    listen       80; #监听你自己服务器的80端口
    server_name  kiwilab.jd.com; #你的域名
    location / {
        proxy_pass       http://127.0.0.1:30080; #你的容器对外端口
        proxy_redirect   off;
        proxy_set_header Host    $host;
        proxy_set_header X-Forwarded-For $remote_addr;
    }
}

### 上传文件内存大小限制
 Nginx出现的413 Request Entity Too Large错误,这个错误一般在上传文件的时候出现，打开nginx主配置文件nginx conf，找到http{}段，
 添加如下内容，表示设置最大上传内容为100m
client_max_body_size    100m; 