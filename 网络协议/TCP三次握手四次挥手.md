
## TCP的握手和挥手
> 对于TCP的三次握手和四次挥手一直以来都知道, 但是没有深入实验了解. 看来网上的一些文章, 今天做一个实验并记录一下

### 简介

* 三次握手

![20180717202520531](E:\Code\Note\picture\20180717202520531.png)

* 4次挥手

  ![20180717204202563](E:\Code\Note\picture\20180717204202563.png)

需要注意的是, 为什么握手时三次和挥手时四次呢, 因为 在建立连接阶段可以直接把SYN和ACK一次发送过去

而对于断开连接时, 可能我发送完数据后, 发送`SYN`试图断开连接, 但是另外一端还在接受数据(SOCKET还在工作)!

### 代码

> 写了一段简单的测试代码, 一个服务端, 一个测试端

**服务端**
```
#include <winsock.h>
#include <inaddr.h>
#include <stdio.h>

int main(){
    WORD sockVersion = MAKEWORD(2,2);
    WSADATA wsaData;
    if(WSAStartup(sockVersion, &wsaData)!=0)
    {
        return 0;
    }

    int ret = 0;
    char buff[1024] = {0};
    SOCKET fd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(6000);
    addr.sin_addr.S_un.S_addr = INADDR_ANY;
    if(bind(fd, (const struct sockaddr *)&addr, sizeof(struct sockaddr_in)) != 0)
    {
        printf("%s\n", "bind errro!");
    }
    if(0 != listen(fd, 5))
    {
        printf("%s\n", "listen errro!");
    }
    struct sockaddr caddr;
    int len = sizeof(sockaddr);
    SOCKET newfd = accept(fd, &caddr, &len);
    if(newfd == INVALID_SOCKET)
    {
        printf("%s %d\n", "accept errro!", WSAGetLastError ());
    }
    else
    {
        printf("%s\n", "accept succ!");
    }

    while(1){
        ret = recv(newfd, buff, 1024, 0);
        if (ret <= 0){
            Sleep(10);
            break;
        }
    }
    closesocket(newfd);
    closesocket(fd);
    Sleep(5000);
    WSACleanup();
    return 0;
}
```

**客户端**
```
#include <stdio.h>
#include <winsock2.h>
#include <inaddr.h>

int main(){
    WORD sockVersion = MAKEWORD(2,2);
    WSADATA wsaData;
    if(WSAStartup(sockVersion, &wsaData)!=0)
    {
        return 0;
    }

    SOCKET fd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(6000);
    addr.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
    int ret = connect(fd,  (const struct sockaddr *)&addr, sizeof(struct sockaddr_in));
    if(0 != ret){
        printf("connect error: %d\n", WSAGetLastError ());
    }
    else
    {
        printf("%s\n", "connect succ!");
    }
    closesocket(fd);
    WSACleanup();

    Sleep(5000);
    return 0;
}
```

### 抓包分析
![抓包](E:\Code\Note\picture\抓包.png)

**握手**: ASYN->BSYN/ACK->AACK

**挥手**:AFIN->BACK->BFIN->AACK

ps: 开头一个AB表示方向

***我看到过一篇文章说, FIN本身携带ACK号, 如果双方都立刻关闭, 那么可能仅需三个报文即可完成挥手(第一个FIN可能被第二个FIN的ACK所确认), 但是我实验并没有成功验证这个! 敬请指教***

