## CMake入门
> 最近决定使用`Clion`编辑器直接在`windows`下编译, 这是使用`cmake`作为构建工具的, 所以决定学习一下

### `Clion`
> 我用的是`CLion+GCC`的组合
	Clion从网上的下的, 然后发现淘宝15块可以买一个license, 也懒得弄了, 直接买了用了
	GCC是用`scoop`下的, 比较推荐使用`scoop`可以用来管理这些`gcc`,`make`之类的

### `CMakeFiles.txt`(第一版)
> 决定随着工程慢慢学习, 接触到什么语法, 记录什么
```
cmake_minimum_required(VERSION 3.14)
project(Graph)

set(CMAKE_CXX_STANDARD 11)

add_executable(Graph Graph.cpp)

LINK_LIBRARIES(-lws2_32)

add_executable(serv serv.cpp)

add_executable(clie clie.cpp)
```
**cmake_minimum_required**: cmake版本要求
**project**: 工程名, 目前暂时不知道用处
**set(CMAKE_CXX_STANDARD 11)**: 指定C++标准为11
**LINK_LIBRARIES**:相当与g++中的`-l`,(`-lws2_32`中的`-l`去掉也可以)
**add_executable**: 把后面的C文件编译为前面的可执行文件名(windows下会追加`.exe`)


> ***2019年9月18日22点40分遇到新的cmake文件, 继续添加***

### `CMakeFiles.txt`(第二版)

```
add_library(A SHARED A.c global.h)

add_library(B SHARED B.c global.h)

link_directories("../cmake-build-debug")
```

`add_library`: 添加了SHARED, 最后会同时编译出静态库和动态库
`link_directories`: 相当于`-L`

***拓展:***
`include_directories`: 相当于`-I`, 头文件路径
