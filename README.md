# HTTP协议
【TOC】
## 前言
### 5层协议
#### 1，物理层（硬件）
#### 2，数据链路层
```
主机间的传输
比特（0，1）
```
#### 3，网络层
```
分组转发
路由选择
```
#### 4，传输层（TCP/IP,UDP）
```
端对端的，如自己的主机到某个网址
```
#### 5, 应用层（HTTP,FTP）
```
构建于TCP/IP协议之上，为应用软件提供服务
```

### HTTP协议简介
```
目前通用版本是HTTP/1.1版本

HTTP方法是定义上的，但不用语意化的方法就要承担用户在不知情的情况下更改数据所产生的安全隐患，常用方法是GET/POST

HTTP/0.9:
只有一个GET命令
没有持久连接（服务器发送完毕，就关闭TCP连接）
没有pipeline（同一个连接里发送多个请求）

HTTP/1.0:
增加了status code和header
增加了命令和字符集
增加了多部分发送
增加了权限和缓存

HTTP/1.1:
增加了持久链接
增加了host

HTTP/2:
将传输形式由以前的字符串转换成二进制形式（同一连接里发送多个请求无需按照顺序来执行）
头信息压缩
推送（并行执行）

HTTPS:
安全版本
```

## HTTP协议三次握手（三次网络传输）
```
主要规避网络延迟导致服务器开销的问题

名词解释（个人解释，具体理解请参考原单词）：
1，SYN：同步
2，ACK：反馈
3，send：发送状态
4，recived：接收状态
5，established：确定状态

第一次握手：
client发送SYN(J)给sever，等待sever的ACK回复，进入SYN-send状态

第二次握手：
sever接收到SYN(J)，返回一个ACK(J+1),并发送一个自己的SYN(K)包，等待slient的ACK回复，进入SYN-recived状态

第三次握手：
client接收到sever发回的SYN(J+1)包后，进入established状态,然后接收sever发来的SYN(K),并返回给sever端一个ACK(K+1),后把sever状态设置为established状态

为什么要三次握手？
因为服务器如果创建失败了，自己不知道，会导致服务器一直开销
```

## URL详解
```
https://github.com:80/ajun568?id=1#hash

https：协议
github.com：域名（找到服务器所在物联网的位置）
80：端口，默认80，可省略（每个服务器都有很多个端口，但端口不方便记忆，所以用域名来区分，而不是端口）
/ajun568：路由
?id=1：query，用来传值，如修改时传ID
#hash：哈希，文档中的片段，用来锚点定义

URN：不成熟，内容更换位置还能找到
```

## HTTP状态值和状态码
```
状态值(readyState):ajax的运行步骤
状态码(status):服务器对请求的反馈

readyState:对于访问XHR(XMLHttpRequest)并不是一次请求的，而是经历多种状态后取得的结果
    0:(未初始化)还没有调用send()方法
    1:(载入)已经发送send()方法，正在发送请求
    2:(载入完成)send()方法执行完成，已经接收到全部响应内容
    3:(交互)正在解析响应内容
    4:(完成)响应内容解析完成，可以在客户端调用了
status
    2xx:表示成功处理请求
        200:请求成功
    3xx:请求重定向和缓存(浏览器直接跳转)
        301:永久重定向
        302:临时重定向
        304:缓存(服务器告诉客户，原来缓存的内容还可以继续使用)
    4xx:客户端请求错误
        401:请求授权失败(被请求的页面需要用户名和密码)
        403:服务器拒绝访问
        404:资源找不到
    5xx:服务器端请求错误
        500:服务器内部错误
```

## 报文(客户端与服务端的交流方式)
### 概念
```
客户端向服务端是流入
不管请求报文还是响应报文，所有报文都会向下流流动，所有报文的发送者都在接收者的上游

```
### 报文的格式
```
请求报文:
<method>空格<request-url>空格<version>(HTTP版本)
<headers>
空行
<entity-body>

响应报文:
<version>空格<status>空格<reason-phrase>(描述)
<headers>
空行
<entity-body>
```

## curl(客户端查看HTTP命令)
```
curl -v https://github.com/ajun568
```

## 跨域
```
浏览器遵守同源策略(协议，域名，端口)

浏览器发送请求时不知道是否会跨域，如果未设置跨域，浏览器会将请求内容忽略，并报错。
```
### 实现跨域
#### 设置请求头：（response.writeHead(状态码, {})）
```
'Access-Control-Allow-Origin': '*'
设置定向请求时注意从浏览器角度并不知道127.0.0.1和localhost是同一个
```
#### JSONP(script标签里嵌套src)
```
浏览器在像link标签，img标签，script标签里写路径加载内容时，是允许跨域的，并不在乎你是否设置请求头。

利用script标签不受跨域限制去加载一个链接，并通过这个链接访问内容。
```
#### CORS
```
简单请求不会发送预请求，复杂的请求会发送cors预请求。
简单请求包括：GET,HEAD,POST
Content-Type包括：text/plain;multipart/form-data;application/x-www-form-urlencoded

修改项：
‘Access-Control-Allow-Headers’,
'Access-Control-Allow-Methods',
'Access-Control-Max-Age'(再次发送预请求的时间间隔)
```

## 缓存(Cache-Control)
```
一般情况下都指的是代理服务器的缓存

分为：public(http经过的地方都进行缓存),private(发起请求的浏览器进行缓存)和no-cache(不进行缓存)

设置缓存后，在客户端缓存了，不会在服务端验证，在实际打包后，会携带一串时间戳的hash码，若内容变化，则hash码变化，ETag这个标识发生变化，重新请求资源。

缓存到期时间：max-age
s-maxage(为代理服务器设置的缓存，在有代理服务器时会先走该设置)
max-stale(在该时间内，可以使用过期的缓存，无需从原服务器请求)
```
### 缓存策略
#### 强缓存
```
其实就是在HTTP头部设置了缓存(如缓存时间)，下一次使用时，根据缓存时间观察是否过期，未过期直接使用缓存加载文件。
Last-Modified：文件最后一次的修改时间。
```
#### 协商缓存
```
协商缓存其实就是缓存已经过期，服务器通过If-Moified-Since和If-None-Match这两个字段判断内容是否有修改，没有修改则返回状态码304，协商缓存结束。
```
#### 启发式缓存
```
遇到默认缓存的情况，都是启发式缓存的锅。
```

## 长连接
```
基于Keep-Alive请求头的约定，其实还是基于TCP的

短连接：请求发送，连接建立，数据返回，连接关闭。
长连接：连接发送后，在请求关闭前，客户端和服务端都保持连接

小tip：浏览器tcp连接的并发限制是6个
```

## 数据协商
```
包括请求(Accept)和返回(Content)两种：包括Type,Encoding和Language,其中Encoding用的最多的是gulp
User-Agent:可以用来区分访问设备类型，PC端or移动端
```
[User-Agent用法小结跳转链接](https://blog.csdn.net/crper/article/details/52005514)

## 重定向
[302跳转的URL劫持，本人很懒，直接甩链接](https://blog.csdn.net/sinat_37059404/article/details/77412748of)

#### 不厚道的补两个关于端口占用的命令，方便自己查找
```
查看进程：lsof -i :端口号
杀死进程：kill -9 端口ID
```
#### 开启过滤跨域模式
```
open -n /Applications/Google\ Chrome.app/ --args --disable-web-security  --user-data-dir=/Users/wuyijun/Documents/chromeDevUserData
```
