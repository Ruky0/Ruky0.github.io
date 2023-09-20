---
title: TinyHTTP源码阅读笔记(一)
date: 2023-04-19 17:12:02
tags: TinyHTTP
---

TinyHTTP中的main函数以及start_up函数阅读笔记.

# main函数

初始化服务端、阻塞等待连接、处理请求、关闭服务端

```c
int main(void)
{
 int server_sock = -1;
 u_short port = 0;//端口号为0时是随机的、系统指定的端口
 int client_sock = -1;
 //sockaddr_in 是 IPV4的套接字地址结构, 存储了端口
 struct sockaddr_in client_name;
 int client_name_len = sizeof(client_name);
 //pthread_t newthread;

//建立服务器监听
 server_sock = startup(&port);
 printf("httpd running on port %d\n", port);

 while (1)
 {
  //阻塞等待客户端的连接，参读《TLPI》P1157，
  //等待连接建立，返回一个客户端连接符存入client_name
  client_sock = accept(server_sock,
                       (struct sockaddr *)&client_name,
                       &client_name_len);
  if (client_sock == -1)
  	error_die("accept");//出错了
     
  accept_request(client_sock);//处理请求

  //新建一个线程，处理这个客户的连接
 /*if (pthread_create(&newthread , NULL, accept_request, client_sock) != 0)
   perror("pthread_create");*/
 }

 close(server_sock);

 return(0);
}
```

# start_up函数

初始化socket、绑定socket和端口、获取端口号、建立监听

```c
int startup(u_short *port)
{ 
 //建立一个socket描述符
 int httpd = 0;
 struct sockaddr_in name;
 
 //socket()用于创建一个用于 socket 的描述符
 //这里的PF_INET其实是与 AF_INET同义，具体可以参读《TLPI》P946
 httpd = socket(PF_INET, SOCK_STREAM, 0);
 if (httpd == -1)
  error_die("socket");
  
 memset(&name, 0, sizeof(name));//将name结构体使用memset全部初始化为一。（清零）

 //设置服务器监听元素
 name.sin_family = AF_INET;//socket族
 //htons()，ntohs() 和 htonl()包含于<arpa/inet.h>, 参读《TLPI》P1199
 //将*port 转换成以网络字节序表示的16位整数
 //这个与大端小端有关
 name.sin_port = htons(*port);//端口
    
 //INADDR_ANY是一个 IPV4通配地址的常量，包含于<netinet/in.h>
 //大多实现都将其定义成了0.0.0.0 参读《TLPI》P1187
 name.sin_addr.s_addr = htonl(INADDR_ANY);//可接受的地址，此处为任意地址
 
 //将描述符与socket结构体进行绑定
 //bind()用于绑定地址与 socket。参读《TLPI》P1153
 //如果传进去的sockaddr结构中的 sin_port 指定为0，这时系统会选择一个临时的端口号
 if (bind(httpd, (struct sockaddr *)&name, sizeof(name)) < 0)
  error_die("bind");
  
 //如果传进来的port是零，说明需要系统随机分配port，需要手动获取
 if (*port == 0)  /* if dynamically allocating a port */
 {
  int namelen = sizeof(name);
  //getsockname()包含于<sys/socker.h>中，参读《TLPI》P1263
  //调用getsockname()获取系统给 httpd 这个 socket 随机分配的端口号
  if (getsockname(httpd, (struct sockaddr *)&name, &namelen) == -1)
   error_die("getsockname");
  *port = ntohs(name.sin_port);
 }  
  //建立监听，最大连接数为5，返回-1说明超了或者有错
  if (listen(httpd, 5) < 0) 
  	error_die("listen");
 return(httpd);//返回设置好的服务端socket标识符

 }
```

# Q&A

1. 为啥要使用随机分配的端口号? 手动设置用起来不方便嘛?

   手动设置的端口号不知道有无占用; 随机分配端口重启server时不用等待两个msl（Maximum Segment Lifetime，报文最大生存时间）（即虽然服务器程序关闭，连接终止，但是端口会等待2MSL的时间, 为保证数据完整送达.）

2. 上面说要等2个MSL才能释放资源, 为啥?

   TCP的四次挥手, 服务端在向客户端发送最后一个ACK后不知道客户端有没有收到. 如果客户端没收到, 则客户端会超时重传, 此时要多等2个MSL才能再次收到重传. 所以服务端要多等2个MSL才能安心睡去.

3. 为什么要使用多线程而不使用多进程?(此处代码将多线程部分注释掉了, 只能阻塞接收一个客户端的消息)

   HTTP协议一次请求只对应一次响应, 每次处理不需要复杂逻辑, 线程就够用了, 新开一个进程就是杀鸡用牛刀.

# 用到的库函数、定义等

1. sockaddr_in结构体

   ```c
   struct sockaddr_in {
   	short int sin_family;           /* Address family */
   	unsigned short int sin_port;    /* Port number */
   	struct in_addr sin_addr;        /* Internet address */
   	unsigned char sin_zero[8];      /* Same size as struct sockaddr */
   };
   ```

2. htonl(), htons()

   htonl-->**h**ost **to** **n**et (unsigned)**l**ong(s-short)

   将本机字节顺序转化为网络字节序(大端)

   相反的操作是ntohl(), ntohs()

3. AF_INET

   IPv4的协议, 用于表示该socket使用IPv4.

4. bind()

   在完成第一步创建套接字，分配了一个Socket描述符后，**服务端**的第二步就是使用在这个描述符用**Bind**绑定.

   bind()的作用是将参数sockfd和addr绑定在一起，使sockfd这个用于网络通讯的文件描述符监听addr所描述的地址和端口号

   bind()系统调用的主要用处：

   1.服务器向系统注册它的众所周知的地址。面向连接和无连接的服务器在接受客户的请求之前都必须做这一步。 

   2.客户可为自己注册一个特定的地址，以便服务器可以用这个有效的地址送回响应。

   ```c
   #include<socket.h>
   int bind( int sockfd, struct sockaddr * addr, socklen_t addrlen )
   ```

   返回0说明bind成功, -1说明bind失败

5. getsockname()

   *getsockname函数用于获取与某个套接字关联的本地协议地址*
   *getpeername函数用于获取与某个套接字关联的外地协议地址*

   ```c
   int getsockname(int sockfd, struct sockaddr *localaddr, socklen_t *addrlen);//将sockfd的本地地址放入localaddr里, 长度放入addrlen
   int getpeername(int sockfd, struct sockaddr *peeraddr, socklen_t *addrlen);//将sockfd连接的peer地址放入peeraddr里
   ```

   getsockname是会修改sockaddr中的内容的; 而bind不会, bind只是向系统注册某个文件描述符sockfd是跟某个socket结构体互相关联的; 同时系统在分配端口号等动作后也并不会主动去修改sockaddr中的内容, 而是需要手动调用getsockname进行修改.

   用途:

   - 在不调用bind的TCP客户，当connect成功返回后，getsockname返回分配给此连接的本地IP地址和本地端口号；
   - 在以端口号为0调用bind后，使用getsockname返回内核分配的本地端口号；
   - getsockname可用来获取某套接口的地址族；
   - 在捆绑了通配IP地址的TCP服务器上，当连接建立后，可以使用getsockname获得分配给此连接的本地IP地址；
   - 当一个服务器调用exec启动后，他获得客户身份的唯一途径是调用getpeername函数。

   以下为个人理解: 

   以上过程中主要涉及三个东西: 代码中的sockaddr结构体, 文件描述符sockfd, 以及系统中的socket. 文件描述符sockfd可以看作程序与系统中socket的接口, 程序可以通过sockfd使用sockgetname, sockpeername来访问系统中的socket, 并将获得的数据存储到scokaddr结构体中. 

   那为什么要bind? bind之前sockaddr中的成员已经被赋值, 是用来初始化系统中socket的参数, 使用bind来修改系统中的socket值使之符合需要(比如初始端口为零时需要系统分配端口), bind之后系统中的socket值被修改而sockaddr结构体还是原本的样子.

6. listen()

   *listen函数使用主动连接套接口变为被连接套接口，使得一个进程可以接受其它进程的请求，从而成为一个服务器进程。在TCP服务器编程中listen函数把进程变为一个服务器，并指定相应的套接字变为被动连接。*

   ```c
   #include<socket.h>
   int listen ( int sockfd,  int backlog )
   ```

   返回0成功, -1失败

   backlog指队列中连接的上限

7. accept()

   *accept函数主要用于服务器端，一般位于listen函数之后，默认会阻塞进程，直到有一个客户请求连接，建立好连接后，它返回的一个新的套接字 socketfd_new ，此后，服务器端即可使用这个新的套接字socketfd_new与该客户端进行通信，而sockfd 则继续用于监听其他客户端的连接请求。*

   ```c
   #include <socket.h>
   int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
   ```

   如果连接成功则会返回一个由系统分配的全新描述符来代表与客户端的TCP连接, 如果对addr和addrlen不感兴趣可以直接传入一个空指针(这里就是这样).



