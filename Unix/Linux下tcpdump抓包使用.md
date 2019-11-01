# tcpdump的基本命令和使用

## 介绍
&emsp;&emsp;tcpdump是linux下的抓包命令, 可以指定网卡/port/host等等, 甚至可以保存数据导出在wireshark中打开分析; 在服务器上可以用该命令抓包然后用wireshark分析

## `help`结果
```
Usage: tcpdump [-aAbdDefhHIJKlLnNOpqRStuUvxX] [ -B size ] [ -c count ]
                [ -C file_size ] [ -E algo:secret ] [ -F file ] [ -G seconds ]
                [ -i interface ] [ -j tstamptype ] [ -M secret ]
                [ -P in|out|inout ]
                [ -r file ] [ -s snaplen ] [ -T type ] [ -V file ] [ -w file ]
                [ -W filecount ] [ -y datalinktype ] [ -z command ]
                [ -Z user ] [ expression ]
```

**命令格式**:`tcpdump [ -i interface]`

## 基础选项(介绍一下几个使用较多的选项)

* `-i interface`: 指定网卡, `interface`指的网卡名, 比如使用`ifconfig`查询到的`eth1`等等;
* `-c num`: 指定抓取多少个包, 可能处理了很多的包但是只用`num`个符合条件的包; 这个用的比较少, 实际上我们可以随时终止
* `-P inout`: 抓取流入或者流出的包, in|out|inout, 默认两个方向
* `-v`: 产生更详细的输出, 还有`-vv,-vvv`每一级都会更加详细
* `-w file`: 输出到文件, 可以使用wireshark解析该文件
* `-n`: 对地址以数字方式显式，否则显式为主机名，也就是说-n选项不做主机名解析。
* `-nn`: 除了-n的作用外，还把端口显示为数值，否则显示端口服务名

## tcpdump表达式
> tcpdump除了上述的选项还可以由也给或多个`单元`组成, 用于指定类型/方向/协议等等。 每个单元一般包含修饰符和参数。

* 1 指定参数类型
&emsp;&emsp;可用的值包括`host/net/port/portrange`, 后面跟随一个相应的值, 比如`host test`指定主机为`test`的报文. 通常是配合`host`使用

* 2 指明方向
&emsp;&emsp; 给定的值包括`src/dst/src or dst/src and dst`, 比如`dst host test`指定到`test`主机的报文; 默认所有

* 3 指定协议
&emsp;&emsp;常用协议`tcp/udp/icmp`等等, 默认所有

## 逻辑运算符
```
取非运算：not  和 ！

与运算：and 和 &&

或运算：or 和 ||
```
## 一个常用命令说明
```
tcpdump -c 200 -i any -w dump.pcap -v '((tcp or icmp) and (port 80) and (src host 14.215.177.39))'
```
上述报文, 前面指定一些选项; 后面跟限制端口协议等等. 

**注意**: 如果不加引号需要注意括号的使用, 可能需要进行转义




