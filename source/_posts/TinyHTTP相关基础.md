---
title: TinyHTTP项目基础
date: 2023-04-18 11:00:20
tags: TinyHTTP
---
欲入门C++网络编程, 遂选择简单小巧经典的TinyHTTP项目进行学习, 顺便回顾计算机网络相关知识.

本文相关的理论知识有socket、HTTP、CGI.

[TOC]

# Socket

*中文翻译为套接字(不明所以的翻译,感觉不如翻译成插座或者接口易于理解).*

服务端与客户端相互访问的接口

TCP下使用socket的流程:

1. 服务端和客户端初始化 `socket`，得到文件描述符；
2. 服务端调用 `bind`，将绑定在 IP 地址和端口;
3. 服务端调用 `listen`，进行监听；
4. 服务端调用 `accept`，等待客户端连接；
5. 客户端调用 `connect`，向服务器端的地址和端口发起连接请求；
6. 服务端 `accept` 返回用于传输的 `socket` 的文件描述符；
7. 客户端调用 `write` 写入数据；服务端调用 `read` 读取数据；
8. 客户端断开连接时，会调用 `close`，那么服务端 `read` 读取数据的时候，就会读取到了 `EOF`，待处理完数据后，服务端调用 `close`，表示连接关闭。

# HTTP协议

*超文本传输协议(Hyper Text Transfer Protocol)*

超文本传输协议（HTTP）是一个用于传输超媒体文档（例如 HTML）的应用层协议。它是为 Web 浏览器与 Web 服务器之间的通信而设计的，但也可以用于其他目的。HTTP 遵循经典的客户端—服务端模型，客户端打开一个连接以发出请求，然后等待直到收到服务器端响应。HTTP 是无状态协议，这意味着服务器不会在两个请求之间保留任何数据（状态）。

[HTTP | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)

## HTTP报文

### 客户端请求消息

#### 示例

```http
GET /hello.txt HTTP/1.1
User-Agent: curl/7.16.3 libcurl/7.16.3 OpenSSL/0.9.7l zlib/1.2.3
Host: www.example.com
Accept-Language: en, mi
```

```http
POST /contact_form.php HTTP/1.1
Host: developer.mozilla.org
Content-Length: 64
Content-Type: application/x-www-form-urlencoded

name=Joe%20User&request=Send%20me%20one%20of%20your%20catalogue
```

#### 起始行

HTTP 请求是由客户端发出的消息，用来使服务器执行动作。*起始行*（start-line）包含三个元素：

1. 一个 *[HTTP 方法](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods)*，一个动词（像 [`GET`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/GET)、[`PUT`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/PUT) 或者 [`POST`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/POST)）或者一个名词（像 [`HEAD`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/HEAD) 或者 [`OPTIONS`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/OPTIONS)），描述要执行的动作。例如，`GET` 表示要获取资源，`POST` 表示向服务器推送数据（创建或修改资源，或者产生要返回的临时文件）。

2. 请求目标

   （request target），通常是一个

    

   URL

   ，或者是协议、端口和域名的绝对路径，通常以请求的环境为特征。请求的格式因不同的 HTTP 方法而异。它可以是：

   - 一个绝对路径，末尾跟上一个`?`和查询字符串。这是最常见的形式，称为原始形式（origin form），被`GET` `POST` `HEAD`和`OPTIONS`方法所使用。

     - `POST / HTTP/1.1`
     - `GET /background.png HTTP/1.0`
     - `HEAD /test.html?query=alibaba HTTP/1.1`
     - `OPTIONS /anypage.html HTTP/1.0`
     
   - 一个完整的 URL，被称为*绝对形式*（absolute form），主要在使用 `GET` 方法连接到代理时使用。`GET http://developer.mozilla.org/en-US/docs/Web/HTTP/Messages HTTP/1.1`
   
   - 由域名和可选端口（以 `':'` 为前缀）组成的 URL 的 authority 部分，称为 *authority form*。仅在使用 `CONNECT` 建立 HTTP 隧道时才使用。`CONNECT developer.mozilla.org:80 HTTP/1.1`
   
   - *星号形式*（asterisk form），一个简单的星号（`'*'`），配合 `OPTIONS` 方法使用，代表整个服务器。`OPTIONS * HTTP/1.1`
   
3. *HTTP 版本*（HTTP version），定义了剩余消息的结构，作为对期望的响应版本的指示符。

#### 标头（Header）

来自请求的 [HTTP 标头](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers)遵循和 HTTP 标头相同的基本结构：不区分大小写的字符串，紧跟着的冒号（`':'`）和一个结构取决于标头的值。整个标头（包括值）由一行组成，这一行可以相当长。

有许多请求标头可用，它们可以分为几组：

- [通用标头（General header）](https://developer.mozilla.org/zh-CN/docs/Glossary/General_header)，例如 [`Via`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Via)，适用于整个消息。
- [请求标头（Request header）](https://developer.mozilla.org/zh-CN/docs/Glossary/Request_header)，例如 [`User-Agent`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/User-Agent)、[`Accept-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Type)，通过进一步的定义（例如 [`Accept-Language`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Language)）、给定上下文（例如 [`Referer`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Referer)）或者进行有条件的限制（例如 [`If-None`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/If-None)）来修改请求。
- [表示标头（Representation header）](https://developer.mozilla.org/zh-CN/docs/Glossary/Representation_header)，例如 [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) 描述了消息数据的原始格式和应用的任意编码（仅在消息有主体时才存在）。

#### 主体（Body）

所有的请求都有一个主体：例如获取资源的请求，像 `GET`、`HEAD`、`DELETE` 和 `OPTIONS`，通常它们不需要主体。有些请求将数据发送到服务器以便更新数据：常见的的情况是 POST 请求（包含 HTML 表单数据）。

主体大致可分为两类：

- 单一资源（Single-resource）主体，由一个单文件组成。该类型的主体由两个标头定义：[`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) 和 [`Content-Length`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Length)。
- [多资源（Multiple-resource）主体](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types#multipartform-data)，由多部分主体组成，每一部分包含不同的信息位。通常是和 [HTML 表单](https://developer.mozilla.org/zh-CN/docs/Learn/Forms)连系在一起。

### 服务端响应消息

#### 示例

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 55743
Connection: keep-alive
Cache-Control: s-maxage=300, public, max-age=0
Content-Language: en-US
Date: Thu, 06 Dec 2018 17:37:18 GMT
ETag: "2e77ad1dc6ab0b53a2996dfd4653c1c3"
Server: meinheld/0.6.1
Strict-Transport-Security: max-age=63072000
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Vary: Accept-Encoding,Cookie
Age: 7

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>A simple webpage</title>
</head>
<body>
  <h1>Simple HTML webpage</h1>
  <p>Hello, world!</p>
</body>
</html>
```

#### 状态行

HTTP 响应的起始行被称作*状态行*（status line），包含以下信息：

1. *协议版本*，通常为 `HTTP/1.1`。
2. *状态码*（status code），表明请求是成功或失败。常见的状态码是 [`200`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/200)、[`404`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/404) 或 [`302`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/302)。
3. *状态文本*（status text）。一个简短的，纯粹的信息，通过状态码的文本描述，帮助人们理解该 HTTP 消息。

一个典型的状态行看起来像这样：`HTTP/1.1 404 Not Found`。

#### 标头（Header）

响应的 [HTTP 标头](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers)遵循和任何其他标头相同的结构：不区分大小写的字符串，紧跟着的冒号（`':'`）和一个结构取决于标头类型的值。整个标头（包括其值）表现为单行形式。

许多不同的标头可能会出现在响应中。这些可以分为几组：

- [通用标头（General header）](https://developer.mozilla.org/zh-CN/docs/Glossary/General_header)，例如 [`Via`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Via)，适用于整个消息。
- [响应标头（Response header）](https://developer.mozilla.org/zh-CN/docs/Glossary/Response_header)，例如 [`Vary`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Vary) 和 [`Accept-Ranges`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Ranges)，提供有关服务器的其他信息，这些信息不适合状态行。
- [表示标头（Representation header）](https://developer.mozilla.org/zh-CN/docs/Glossary/Representation_header)，例如 [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) 描述了消息数据的原始格式和应用的任意编码（仅在消息有主体时才存在）。

#### 主体（Body）

响应的最后一部分是主体。不是所有的响应都有主体：具有状态码（如 [`201`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/201) 或 [`204`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/204)）的响应，通常不会有主体。

主体大致可分为三类：

- 单资源（Single-resource）主体，由**已知**长度的单个文件组成。该类型主体由两个标头定义：[`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Type) 和 [`Content-Length`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Length)。
- 单资源（Single-resource）主体，由**未知**长度的单个文件组成。通过将 [`Transfer-Encoding`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Transfer-Encoding) 设置为 `chunked` 来使用分块编码。
- [多资源（Multiple-resource）主体](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types#multipartform-data)，由多部分 body 组成，每部分包含不同的信息段。但这是比较少见的。

## HTTP方法

最常用的有POST和GET.

GET 方法请求一个指定资源的表示形式，使用 GET 的请求应该只被用于获取数据.

POST 方法用于将实体提交到指定的资源，通常导致在服务器上的状态变化或副作用.

# CGI

*Common Gateway Interface, 通用网关接口*

个人理解cgi为服务器上的一个可执行程序,可以被来自客户端的HTTP请求调用, 服务器接收请求并把这个请求给cgi处理, cgi处理完成后传给服务器,服务器再传给用户.

本项目中使用的是Perl编写的CGI.

POST方法往往会调用CGI.
