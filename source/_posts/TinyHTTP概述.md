---
title: TinyHTTP概述
date: 2023-04-18 22:58:41
tags: TinyHTTP
---
TinyHTTP是一个1999年的小项目, 五百多行代码写明了使用C/C++进行网络编程的基本流程, 非常适合入门. 本文将从TinyHTTP要干什么, 怎么干成的, 以及如何运行TinyHTTP几个方面进行阐述.

# TinyHTTP的目标

1. 实现一个多线程的服务器, 时刻监听请求, 每接收到一个新的请求便会新建一个进程处理该请求.
2. 分析HTTP请求报文, 分辨是GET还是POST(仅实现这两种较为典型的方法).
3. 根据方法的不同从报文中读出相关参数, 根据参数进行应答.
4. 具体功能为根据用户输入的颜色改变显示的背景颜色, 该功能由一个CGI实现. 
5. 错误消息处理.

## TinyHTTP的工作流程

1. 服务器启动，在指定端口或随机选取端口绑定 httpd 服务。

2. 收到一个 HTTP 请求时（其实就是 listen 的端口 accpet 的时候），派生一个线程运行 accept_request 函数。

3. 取出 HTTP 请求中的 method (GET 或 POST) 和 url,。对于 GET 方法，如果有携带参数，则 query_string 指针指向 url 中 ？ 后面的 GET 参数。

4. 格式化 url 到 path 数组，表示浏览器请求的服务器文件路径，在 tinyhttpd 中服务器文件是在 htdocs 文件夹下。当 url 以 / 结尾，或 url 是个目录，则默认在 path 中加上 index.html，表示访问主页。

5. 如果文件路径合法，对于无参数的 GET 请求，直接输出服务器文件到浏览器，即用 HTTP 格式写到套接字上，跳到（10）。其他情况（带参数 GET，POST 方式，url 为可执行文件），则调用 excute_cgi 函数执行 cgi 脚本。

6. 读取整个 HTTP 请求并丢弃，如果是 POST 则找出 Content-Length. 把 HTTP 200  状态码写到套接字。

7. 建立两个管道，cgi_input 和 cgi_output, 并 fork 一个进程。

8. 在子进程中，把 STDOUT 重定向到 cgi_outputt 的写入端，把 STDIN 重定向到 cgi_input 的读取端，关闭 cgi_input 的写入端 和 cgi_output 的读取端，设置 request_method 的环境变量，GET 的话设置 query_string 的环境变量，POST 的话设置 content_length 的环境变量，这些环境变量都是为了给 cgi 脚本调用，接着用 execl 运行 cgi 程序。

9. 在父进程中，关闭 cgi_input 的读取端 和 cgi_output 的写入端，如果 POST 的话，把 POST 数据写入 cgi_input，已被重定向到 STDIN，读取 cgi_output 的管道输出到客户端，该管道输入是 STDOUT。接着关闭所有管道，等待子进程结束。

## TinyHTTP运行时注意事项

*总之先跑起来, 有助于理解整个项目*

1. Linux环境下注意删掉相关不兼容语句

   ```markdown
    To compile for Linux:
     1) Comment out the #include <pthread.h> line.
     2) Comment out the line that defines the variable newthread.
     3) Comment out the two lines that run pthread_create().
     4) Uncomment the line that runs accept_request().
     5) Remove -lsocket from the Makefile.
   ```

2. 安装Perl-cgi

   Perl在Ubuntu上已默认安装

   ```shell
   perl -MCPAN -e shell
   (cpan)install CGI.pm
   perl -MCGI -e 'print "CGI.pm version $CGI::VERSION\n";'# 检查是否安装成功
   ```

3. 修改color.cgi和check.cgi文件

   ```shell
   which perl #查找一下perl路径
   ```

   将上述两文件第一句改成`#!+perl路径 -Tw`, 如:

   ```perl
   #!/usr/bin/perl -Tw
   ```

4. 设置文件权限

   在这步卡了很久,不搞这步能成功链接但是浏览器页面上一篇空白

   在`htdocs`路径下修改`index.html`访问权限为不可执行

   ```shell
   sudo chmod 600 index.html
   ```

   同时要保证两个cgi可以执行,否则输入颜色后一片空白

   ```shell
   sudo chmod 777 color.cgi
   ```

5. 上述工作搞完之后直接make出可执行文件运行, 使用浏览器访问对应端口即可. 
