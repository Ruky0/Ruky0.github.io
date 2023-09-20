---
title: CMake
date: 2023-04-17 20:32:20
tags: Linux C++
---

# CMake

## 啥是CMake?

工程量比较大的时候手动进行链接、编译等工作十分复杂, 使用make进行批处理可以简化流程. Make使用的Makefile需要手动编写,编写这个文件有时也十分复杂, 所以CMake便出现来简化Makefile的流程.

CMake可以在文件(CMakeLists.txt)中配置编译、调试等步骤的参数, 之后调用指令便不用每次都敲g++、gdb和一堆参数.

VS中这一工作由IDE完成, IDE会帮你进行链接库之类的工作.

## 咋用CMake?

编写CMakeLists.txt文件

在相应路径下调用cmake生成Makefile

然后调用make产生可执行文件

## 常用语句

cmake_minimum_required - 指定CMake的最小版本要求

```cmake
# CMake最小版本要求为2.8.3
cmake_minimum_required(VERSION 2.8.3)
```

project - 定义工程名称，并可指定工程支持的语言

```cmake
# 指定工程名为HELLOWORLD
project(HELLOWORLD)
```

set - 显式的定义变量

```cmake
# 定义SRC变量，其值为sayhello.cpp hello.cpp
set(SRC sayhello.cpp hello.cpp)
```

include_directories - 向工程添加多个特定的头文件搜索路径 --->相当于指定g++编译器的-I参数

```cmake
# 将/usr/include/myincludefolder 和 ./include 添加到头文件搜索路径
include_directories(/usr/include/myincludefolder ./include)
```

target_include_directories-向目标添加头文件搜索路径

```cmake
target_include_directories(hello_library
        PUBLIC
        ${PROJECT_SOURCE_DIR}/include
        )
```

link_directories - 向工程添加多个特定的库文件搜索路径 --->相当于指定g++编译器的-L参数

```cmake
# 将/usr/lib/mylibfolder 和 ./lib 添加到库文件搜索路径
link_directories(/usr/lib/mylibfolder ./lib)
```

add_library - 生成库文件

```cmake
# 通过变量 SRC 生成 libhello.so 共享库(STATIC静态库)
add_library(hello SHARED ${SRC})
```

add_compile_options - 添加编译参数

```cmake
# 添加编译参数 -Wall -std=c++11 -O2
add_compile_options(-Wall -std=c++11 -O2)
```

add_executable - 生成可执行文件

```cmake
# 编译main.cpp生成可执行文件main
add_executable(main main.cpp)
```

target_link_libraries - 为 target 添加需要链接的共享库 --->相同于指定g++编译器-l参数

```cmake
# 将hello动态库文件链接到可执行文件main
target_link_libraries(main hello)
```

add_subdirectory - 向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置

```cmake
# 添加src子目录，src中需有一个CMakeLists.txt
add_subdirectory(src)
```



## 常用变量

CMAKE_CXX_FLAGS g++编译选项

```cmake
#在CMAKE_CXX_FLAGS编译选项后追加-std=c++11
set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
```

CMAKE_BUILD_TYPE 编译类型(Debug, Release)

```cmake
# 设定编译类型为debug，调试时需要选择debug
set(CMAKE_BUILD_TYPE Debug)
# 设定编译类型为release，发布时需要选择release
set(CMAKE_BUILD_TYPE Release)
```

CMAKE_C_COMPILER：指定C编译器
CMAKE_CXX_COMPILER：指定C++编译器
EXECUTABLE_OUTPUT_PATH：可执行文件输出的存放路径
LIBRARY_OUTPUT_PATH：库文件输出的存放路径

。。。

## 注意事项

1. 在项目同级目录下直接cmake会在该目录下产生Makefile和一堆杂乱的文件, 这种构建方式称为内部构建, 乱! 所以一般会新建一个build文件夹存放这一堆文件, 进入该文件夹cmake .. 即可.
2. include_directories是向整个工程添加头文件搜索路径; 而target_include_directories是为指定对象添加头文件搜索路径, 范围更小.

## 示例

```
cmake-test/                 工程主目录，main.c 调用 libhello-
├── CMakeLists.txt
├── include            生成 libhello-world.so，调用 
│   └── static               生成 libhello.so 
│       └── hello.h         libhello.so 对外的头文件
├── src      
    ├── hello.cpp
    └── main.cpp         libworld.so 对外的头文件
```



```cmake
cmake_minimum_required(VERSION 3.5)

project(hello_library)

# 生成一个静态库
add_library(hello_library STATIC
        src/hello.cpp
        )
        
#设置库所需的include
target_include_directories(hello_library
        PUBLIC
        ${PROJECT_SOURCE_DIR}/include
        )

# 生成可执行文件
add_executable(hello_binary
        src/main.cpp
        )

# 链接可执行文件所需的库文件
target_link_libraries(hello_binary
        PRIVATE
        hello_library
        )
```



