---
title: http协议之报文首部和状态码
date: 2018-07-06 09:44:39
tags: http
categories: http
---

常用的响应状态码：
## 2xx 响应成功
- 200 OK 请求被正常处理，返回请求资源。
- 204 No Content 请求处理成功，但没有资源可返回。用于不需项客户端发送新资源的情况下用。
- 206 Partial Content 客户端对资源某一部分进行请求。响应报文中包含由Content-Range制定范围的实体内容。
<!-- more -->

## 3xx 重定向
- 301 Moved Permanently 永久性重定向，请求资源的URI已经更改了。
- 302 Found 临时重定向。资源被分配了新的URI，希望用户（本次）能使用新的URI访问。
- 303 See Other 请求资源存在另一个URI，应使用GET方法(和302的区别)定向获取请求的资源。
- 304 Not Modified 请求的资源在客户端已经缓存了，而且服务器资源没有更新，不返回响应资源。(和重定向没有关系)
- 307 Temporary Rediret 临时重定向，类似302

## 4xx 客户端错误
- 400 Bad Request 请求存在语法错误，服务器无法理解请求。(但浏览器会像对待200 OK一样去对待该状态码)
- 401 Unauthoriaed 表示发送的请求需要通过http认证，第一次收到401响应，会弹出认证的对话窗口。
- 403 Forbidden 请求资源服务器拒绝了。未授权的IP源发送请求等情况可能会发生403响应。
- 404 Not Found 服务器上没找到请求的资源。

## 5xx 服务端错误
- 500 Interna Server Error web应用出了bug或故障，响应资源失败。(有时返回的状态码响应式错了，web服务出现了故障，但状态码可能还是200 OK。)
- 503 Service Unavailable 服务器处于超负载或正在停机维护，暂停营业。
