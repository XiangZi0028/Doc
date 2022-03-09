# ZEngine
ZEngine的学习笔记

## CMake

主要记录在学习ZEngine过程中的CMake的学习

##### CMake的函数

###### cmake_minimum_required

cmake_minimum_required(VERSION 3.1)

###### set

set(CMAKE_C_STANDARD 11)

set(CMAKE_CXX_STANDARD 11)

###### project

project(ZEngine)

###### inlcude_directories

include_directories("${PROJECT_SOURCE_DIR}/Framework/Common")

include_directories("${PROJECT_SOURCE_DIR}/Framework/Interface")

###### add_subdirectory

add_subdirectory(Framework)

add_subdirectory(Empty)

###### add_library

add_library(BaseApplication.cpp main.cpp)

###### add_executable

用来建立Empty平台的最后的可执行文件

add_executable(Empty EmptyApplication.cpp)

###### target_link_libraries

//给某个exe可执文件添加库

target_link_libraries(Empty Common)

## 项目其它

##### #pragma once

###### ①作用

当在头文件中声明#pragma once 在编译的的时候就只需要处理一次。项目太大了，同一个头文件可能会被多次包含。由于编译器处理包含文件的方式是将其展开在源文件中，所以如果不加这个条件编译指令，头文件的内容会在同一个源文件里多次展开，那么编译器就会报错，说同一个东西被多次定义。

###### ②替代

老版本的C/C++编译器不认识#pragma，就需要采用以下形式来代替

```
#ifndef __INTERFACE_H__
#define __INTERFACE_H__
<代码>
#endif //__INTERFACE_H__
```

## Linux

#### 命令

###### touch

创建文件 touch filename

###### rm

删除文件

###### cd

###### mkdir

创建一个目录

###### rm -r

删除一个目录，rm -r src 删除src目录

rm -fr /     一定不要在linux下尝试，一删就什么都没有了

###### ls(ll)

查看当前目录下的所有文件

###### pwd

像是当前所在路径

###### mv

移动移动文件 mv file dirpath

###### clear

清屏

###### reset

重新生成终端

###### history

查看历史命令

###### help

帮助

###### exit

退出

#表示注释

touch 路径/test.{hpp,cpp} 在路径下创建test.hpp和test.cpp文件

## DX

## Windows核心编程

windows系统API，俗称win32 API(64位windows的API仍然成为win32 API)

## Clang

用clang编译helloengine_win.c,因为这是一个windows程序，所以需要在windows里面编译它，并且，因为我们用到了一些windows平台的接口，需要链接相关的库

clang -l user32.lib -o helloengine_win.exe helloengine_win.c

## 技巧

vs 自动代码格式化 ctrl+a -->ctrl+k+f

环境变量，是为了在任意地方使用而已 搜索的时候会搜索系统的环境变量的路径
