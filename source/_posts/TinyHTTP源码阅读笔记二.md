---
title: TinyHTTP源码阅读笔记(二)
date: 2023-04-23 23:35:12
tags: TinyHTTP
---

TinyHTTP中accept_request函数和execute_cgi函数阅读笔记

# accept_request函数

本部分包含了对HTTP协议的简单实现, 读取报文并根据请求类型做出回应.

```c
void accept_request(int client)
{
  char buf[1024];
  int numchars;
  char method[255];
  char url[255];
  char path[512];
  size_t i, j;
  struct stat st;
  int cgi = 0; /* becomes true if server decides this is a CGI
                * program */
  char *query_string = NULL;

  // 读http 请求的第一行数据（request line），把请求方法存进 method 中
  numchars = get_line(client, buf, sizeof(buf)); // 从client这个socket里读出一行存到buf里
  i = 0;
  j = 0;
  // 找出请求方法
  while (!ISspace(buf[j]) && (i < sizeof(method) - 1))
  {
    method[i] = buf[j];
    i++;
    j++;
  }
  method[i] = '\0'; // 字符串结束符

  // 如果请求的方法不是 GET 或 POST 任意一个的话就直接发送 response 告诉客户端没实现该方法
  if (strcasecmp(method, "GET") && strcasecmp(method, "POST"))
  {
    unimplemented(client);
    return;
  }

  // 如果是 POST 方法就将 cgi 标志变量置一(true)
  if (strcasecmp(method, "POST") == 0)
  { // 忽略大小写比较字符串
    cgi = 1;
    // printf("this is a POST\n");
  }

  i = 0;
  // 跳过所有的空白字符(空格)
  while (ISspace(buf[j]) && (j < sizeof(buf)))
    j++;

  // 然后把 URL 读出来放到 url 数组中
  while (!ISspace(buf[j]) && (i < sizeof(url) - 1) && (j < sizeof(buf)))
  {
    url[i] = buf[j];
    i++;
    j++;
  }
  url[i] = '\0';

  // 如果这个请求是一个 GET 方法的话
  if (strcasecmp(method, "GET") == 0)
  {
    // 用一个指针指向 url
    query_string = url;

    // 去遍历这个 url，跳过字符 ？前面的所有字符，如果遍历完毕也没找到字符 ？则退出循环
    while ((*query_string != '?') && (*query_string != '\0'))
      query_string++;

    // 退出循环后检查当前的字符是 ？还是字符串(url)的结尾
    if (*query_string == '?')
    {
      // 如果是 ？ 的话，证明这个请求需要调用 cgi，将 cgi 标志变量置一(true)
      cgi = 1;
      // 从字符 ？ 处把字符串 url 给分隔会两份
      *query_string = '\0';
      // 使指针指向字符 ？后面的那个字符
      query_string++;
    }
  }

  // 将前面分隔两份的前面那份字符串，拼接在字符串htdocs的后面之后就输出存储到数组 path 中。相当于现在 path 中存储着一个字符串
  sprintf(path, "htdocs%s", url);

  // 如果 path 数组中的这个字符串的最后一个字符是以字符 / 结尾的话，就拼接上一个"index.html"的字符串。首页的意思
  if (path[strlen(path) - 1] == '/')
    strcat(path, "index.html");

  // 在系统上去查询该文件是否存在
  if (stat(path, &st) == -1)
  {
    // 如果不存在，那把这次 http 的请求后续的内容(head 和 body)全部读完并忽略
    while ((numchars > 0) && strcmp("\n", buf)) /* read & discard headers */
      numchars = get_line(client, buf, sizeof(buf));
    // 然后返回一个找不到文件的 response 给客户端
    not_found(client);
  }
  else
  {
    // 文件存在，那去跟常量S_IFMT相与，相与之后的值可以用来判断该文件是什么类型的
    // S_IFMT参读《TLPI》P281，与下面的三个常量一样是包含在<sys/stat.h>
    if ((st.st_mode & S_IFMT) == S_IFDIR)
      // 如果这个文件是个目录，那就需要再在 path 后面拼接一个"/index.html"的字符串
      strcat(path, "/index.html");

    // S_IXUSR, S_IXGRP, S_IXOTH三者可以参读《TLPI》P295
    if ((st.st_mode & S_IXUSR) ||
        (st.st_mode & S_IXGRP) ||
        (st.st_mode & S_IXOTH))
      // 如果这个文件是一个可执行文件，不论是属于用户/组/其他这三者类型的，就将 cgi 标志变量置一
      cgi = 1;

    if (!cgi)
      // 如果不需要 cgi 机制的话，
      serve_file(client, path);
    else
      // 如果需要则调用
      execute_cgi(client, path, method, query_string);
  }

  close(client);
}
```

