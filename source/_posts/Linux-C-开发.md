---
title: G++与GDB，编译与调试
date: 2023-04-17 19:18:21
tags: Linux C++
---
# G++与GDB，编译与调试

## G++编译器

### 快速使用

```shell
g++ test.cpp -o test
```

将test.cpp编译为名为test的二进制文件，-o参数后为指定的文件名。

### 常用参数

```shell
g++ -g test.cpp 
#生成带调试信息的可执行文件，可供GDB进行调试

g++ -O2 test.cpp 
#优化源代码(O0不优化,O1默认优化,O2额外优化,O3更多优化)

g++ -lglog test.cpp 
# 链接glog库(直接使用小写l用于连接系统lib中有的库)

g++ -L/home/code/mytestlibfolder -lmytest test.cpp 
#L用于指定库文件目录,l指定库文件

g++ -I/myInclude test.cpp 
#指定头文件搜索目录

g++ -Wall test.cpp
#-W打印警告信息,all打印所有警告信息

g++ -w test.cpp
#关闭所有警告

g++ -std=c++11 test.cpp
#使用c++11标准进行编译
```

## GDB调试器

### 进入调试程序

```shell
gdb test #进入test的调试
```

### 常用参数

```shell
#括号内为参数简写

$(gdb)help(h) 
# 查看命令帮助，具体命令查询在gdb中输入help + 命令

$(gdb)run(r)
# 重新开始运行文件（run-text：加载文本文件，run-bin：加载二进制文件）

$(gdb)start
# 单步执行，运行程序，停在第一行执行语句

$(gdb)list(l)
# 查看原代码（list-n,从第n行开始查看代码。list+ 函数名：查看具体函数）

$(gdb)set
# 设置变量的值

$(gdb)next(n)
# 单步调试（逐过程，函数直接执行）

$(gdb)step(s)
# 单步调试（逐语句：跳入自定义函数内部执行）

$(gdb)backtrace(bt)
# 查看函数的调用的栈帧和层级关系

$(gdb)frame(f)
# 切换函数的栈帧

$(gdb)info(i)
# 查看函数内部局部变量的数值

$(gdb)finish
# 结束当前函数，返回到函数调用点

$(gdb)continue(c)
# 继续运行

$(gdb)print(p)
# 打印值及地址

$(gdb)quit(q)
# 退出gdb

$(gdb)break+num(b)
# 在第num行设置断点

$(gdb)info breakpoints
# 查看当前设置的所有断点

$(gdb)delete breakpoints num(d)
# 删除第num个断点

$(gdb)display 
# 追踪查看具体变量值

$(gdb)undisplay
# 取消追踪观察变量

$(gdb)watch
# 被设置观察点的变量发生修改时，打印显示

$(gdb)i watch
# 显示观察点

$(gdb)enable breakpoints
# 启用断点

$(gdb)disable breakpoints
# 禁用断点

$(gdb)x
# 查看内存x/20xw 显示20个单元，16进制，4字节每单元

$(gdb)run argv[1] argv[2]
# 调试时命令行传参

$(gdb)set follow-fork-mode child
#Makefile项目管理：选择跟踪父子进程（fork()）
```

