---
title: 连接mongodb
date: 2018-01-08 15:25:47
tags: 数据库
categories: mongodb
comments: true
---
## mongodb数据库概念

### momngodb和mysql比较
SQL术语/概念	|MongoDB术语/概念	|解释/说明
---|---|---
database	    |database	        |数据库
table	        |collection	        |数据库表/集合
row	            |document	        |数据记录行/文档
column	        |field	            |数据字段/域
index	        |index	            |索引
table joins	 	|                    |表连接,MongoDB不支持
primary key	    |primary key	         |主键,MongoDB自动将_id字段设置为主键

## mongodb的安装

### windows下的安装
### linux下的安装


##  mongoose的使用

### mongoose的核心概念
Schema,Model

### 连接数据库
以nodejs为例
在安装路径下设置数据库集合的位置
D:\setUp\mongodb\bin>mongodb.exe -dbpath d:\data\db

## mongoose的常用操作
### 增
save(); create(); insertMany()

### 查
find()找出列表-数组；findById() 通过id找；findOne找出列表第一个-文档对象，第一个参数conditions为查询条件，第二个参数doc为需要修改的数据，第三个参数options为控制选项，第四个参数是回调函数

### 改
update(conditions, doc, [options], [callback])；
updateOne()-只能更新找到的第一条数据，即使设置{multi:true}也无法同时更新多个文档
find()/findOne()/findById() + save()；操作比较复杂的话可以用这种方法

### 删

### 其他操作



