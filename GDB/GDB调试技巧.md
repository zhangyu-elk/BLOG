## GDB调试技巧(一)
> GDB时Linux下C/C++开发必备的调试工具, Linux开发工具往往就是 编辑器 + gcc(g++) + gdb。
> 在windows下也存在MINGW工具, 也提供了GDB调试工具

### 调试符号
	在使用GCC编译时需要加上`-g`选项, 才会带上调试程序需要的符号信息
`gcc -g -o hello_server hello_server.c`
	实际开发过程中可能会使用`strip`命令将这些信息去掉, 以减小体积. 可以使用`file`命令查看文件是否带有debug信息

ps: 在调试时开启`-g`选项, 同时建议关闭优化选项

## GDB调试技巧(二)
> 调试`core`文件
	使用`ulimit -a`查看是否开启core文件(用于定位程序突然崩溃的原因), 一下是在Centos7下的命令执行结果
```
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7262
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 4096
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```
第一行`core file size`为0表示未开启
可以使用:
`ulimit -c unlimited`
将其改为不限制, 也可以在-c后指定大小

***文件名: core.pid(进程号)***

## 调试命令(以Redis调试作为例子)
> 这里只是简单介绍GDB, 而不是详细调试Redis

### run命令
```
[zhangyu@bogon src]$ gdb redis-server 
GNU gdb (GDB) Red Hat Enterprise Linux 7.6.1-114.el7
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /home/zhangyu/code/Redis/src/redis-server...done.
(gdb) 
```
请注意这里仅仅是导入文件, 并没有开始运行
ps: ***倒数第二行说明了该程序具有调试信息, 否则可能显示`no debugging symbols found`***

```
(gdb) r
Starting program: /home/zhangyu/code/Redis/src/redis-server 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
```
正式执行整个程序, 注意对于`gdb`来说, 在不重复的情况下可以使用缩写

### continue命令
> 在使用`Ctrl+C`或者遇到断点时, 程序会被中断, 可以使用该命令重新开始运行
```
^C
Program received signal SIGINT, Interrupt.
0x00007ffff71e3183 in epoll_wait () from /lib64/libc.so.6
Missing separate debuginfos, use: debuginfo-install glibc-2.17-222.el7.x86_64
(gdb) c
Continuing.
```

### break命令(断点)

#### 使用介绍
*	break funcname	: 在对应函数开头添加断点
*	break line 		: 在当前文件对应行号断点
*	break file:line : 在对应文件对应行号添加断点

#### 例子
```
(gdb) b main
Breakpoint 1 at 0x437032: file server.c, line 4040
```
在main函数开头添加一个断点, 注意在程序运行中是不可以添加的

重新使用 `r`命令开始运行
```
(gdb) r
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/zhangyu/code/Redis/src/redis-server 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".

Breakpoint 1, main (argc=1, argv=0x7fffffffeaa8) at server.c:4040
```
可以使用`continue`继续执行