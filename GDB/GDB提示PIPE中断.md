# GDB提示PIPE终端

&emsp;&emsp;有时候用GDB调试的时候发现会提示`SIGPIPE`中断, 但是实际我们在代码也许已经屏蔽了`SIGPIPE`信号; 原因是: 该错误是因为信号被GDB先截取了, 默认GDB会中断进程, 并不是进程本身的问题.

## 解决方法

`handle SIGPIPE nostop`
在GDB命令行输入该命令即可; `handle`是用于操作信号的命令, 可选的参数有`stop/nostop/print`等.