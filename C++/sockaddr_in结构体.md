## Socket编程地址信息结构体
```
struct sockaddr
{
	uint16_t sa_family;
	char sa_data[14];
}
```
	socket编程中比如bind, connect等函数均使用的是该结构体, 不过由于赋值问题, 通常使用的是`sockaddr_in`结构体(进行强制类型转换即可)

```
struct sockaddr_in
{
	short sin_family;
	uint16_t sin_port;
	struct in_addr sin_addr;
	char sin_zero[8];
}
```
在windows中和linux中`in_addr`结构体具有一定的差别
### Windows
```
struct in_addr
{
	union
	{
		struct
		{
			uint8_t s_b1, s_b2, s_b3, s_b4;
		}S_un_b;
		struct
		{
			uint16_t s_w1, s_w2;
		}
		uint32_t S_addr;
	}S_un;
}
```

### Linux
```
struct in_addr
{ 
    unsigned long s_addr;
}
```

### `in_addr`赋值函数
	在很久以前存在一些如`inet_addr`, `inet_ntoa()`函数, 但是由于不支持IPV6的原因, 不应该使用
```
int inet_pton(int af, const char *src, void *dst);	//将点分十进制转换为二进制整数
const char *inet_ntop(int af, const void *src, char *dst, socklen_t cnt);
```
第一个参数`af`指明类型,能够处理ipv4和ipv6; 第二个参数`src`为ip字符串, 第三个参数`dst`为in_addr或者in6_addr结构体
第二个函数, 参数与第一个相同, 多了一个socklen_t指明缓冲区dst大小, 避免溢出

相关头文件:
windows
`#include <WS2tcpip.h>`
linux
```
#include <sys/socket.h>
#include <netinet/in.h>
#include<arpa/inet.h>
``` 


