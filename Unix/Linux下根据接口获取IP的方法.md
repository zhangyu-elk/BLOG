## Linux根据接口获取IP
> 最近遇到一个问题, 需要根据接口获取相应的IP, 查询了一下网上的解决方法, 做一下记录

### 接口说明
> 通常查询IP的时候在Linux下可以使用`ifconfig`, 以下是Centos下的结果
```
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.56.99  netmask 255.255.255.0  broadcast 192.168.56.255
        inet6 fe80::976c:f683:736c:c13a  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:9a:e8:6a  txqueuelen 1000  (Ethernet)
        RX packets 53  bytes 6171 (6.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 52  bytes 7183 (7.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.113  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::b986:5019:f3b0:31c6  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:30:cb:51  txqueuelen 1000  (Ethernet)
        RX packets 2908  bytes 204037 (199.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 58  bytes 5325 (5.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
以下代码就是根据如`enp0s3`,`enp0s8`这样的参数获取对应的IP

### `getifaddrs`接口API
> 该接口可以获取一个类似链表的结果
#### `ifaddrs`结构体
```
struct ifaddrs {
		struct ifaddrs  *ifa_next;    /* Next item in list */
		char            *ifa_name;    /* Name of interface */
		unsigned int     ifa_flags;   /* Flags from SIOCGIFFLAGS */
		struct sockaddr *ifa_addr;    /* Address of interface */
		struct sockaddr *ifa_netmask; /* Netmask of interface */
		union {
		   struct sockaddr *ifu_broadaddr;
		                    /* Broadcast address of interface */
		   struct sockaddr *ifu_dstaddr;
		                    /* Point-to-point destination address */
		} ifa_ifu;
		#define              ifa_broadaddr ifa_ifu.ifu_broadaddr
		#define              ifa_dstaddr   ifa_ifu.ifu_dstaddr
		void            *ifa_data;    /* Address-specific data */
		};
```
`ifa_name`: `enp0s3`这样的接口名
`ifa_addr`: 对应的地址参数
`ifa_next`: 链表的下一个

#### 运行代码(从man手册中复制下来的)
```
#include <arpa/inet.h>
#include <sys/socket.h>
#include <netdb.h>
#include <ifaddrs.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int
main(int argc, char *argv[])
{
	struct ifaddrs *ifaddr, *ifa;
	int family, s;
	char host[NI_MAXHOST];

	if (getifaddrs(&ifaddr) == -1)
	{
		perror("getifaddrs");
    	exit(EXIT_FAILURE);
	}

	for (ifa = ifaddr; ifa != NULL; ifa = ifa->ifa_next) {
       if (ifa->ifa_addr == NULL)
           continue;

		family = ifa->ifa_addr->sa_family;

		if (family == AF_INET)
		{
			printf("interfac: %s, ip: %s\n", ifa->ifa_name, inet_ntoa(((struct sockaddr_in*)ifa->ifa_addr)->sin_addr));
		}
   }

   freeifaddrs(ifaddr);
   exit(EXIT_SUCCESS);
}
```
***结果***:
```
interfac: lo, ip: 127.0.0.1
interfac: enp0s3, ip: 192.168.56.99
interfac: enp0s8, ip: 192.168.1.113
```

> ps: **如果时使用pppoe拨号上网的方式, 是获取到不到IP的, 可以获取到相应名字的`ifaddrs`结构体,但是结构体中的ifa_addr是NULL**

#### Linux原始方式(ioctl)
```
#include <sys/socket.h>
#include <netinet/in.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <errno.h> 
#include <unistd.h> 
#include <netdb.h>  
#include <net/if.h>  
#include <arpa/inet.h> 
#include <sys/ioctl.h>  
#include <sys/types.h>  
#include <sys/time.h> 
// 获取本机ip  eth_inf网卡名称 调用方法get_local_ip("apcli0", ip);
int get_local_ip(const char *eth_inf)
{
    int sd;
    struct sockaddr_in sin;
    struct ifreq ifr;
 
    sd = socket(AF_INET, SOCK_DGRAM, 0);
    if (-1 == sd)
    {
        printf("socket error: %s\n", strerror(errno));
        return -1;
    }
 
    strncpy(ifr.ifr_name, eth_inf, IFNAMSIZ);
    ifr.ifr_name[IFNAMSIZ - 1] = 0;
 
    // if error: No such device  
    if (ioctl(sd, SIOCGIFADDR, &ifr) < 0)
    {
        printf("ioctl error: %s\n", strerror(errno));
        close(sd);
        return -1;
    }
    
    printf("interfac: %s, ip: %s\n", eth_inf, inet_ntoa(((struct sockaddr_in*)&ifr.ifr_addr)->sin_addr)); 

    close(sd);
    return 0;
}

int main(int argc, const char *argv[])
{
    get_local_ip(argv[1]);
    return 0;
}
```

> 该种方式不存在上一个的问题

***结果***:
```
./a.out enp0s3
interfac: enp0s3, ip: 192.168.56.99
```