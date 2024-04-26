1、众所周知，现在主流网络用的还是IPV4协议，理论上一共有2^32=43亿个地址，除去私有网段、网络ID、广播ID、保留网段、本地环回127.0.0.0网段、组播224.0.0.0网段、实际可用就是36.47亿个；全球的服务器、PC机、手机、物联网设备等需要通信的设备加起来远不止36.47亿，怎么才能尽可能让多的设备联网了？IPV6的地址有128位，理论上可以包含地球上每一粒沙子。但目前IPV4还是主流，过度到IPV6是个非常漫长的过程，所以目前“节约”IP地址最常见的方式：NAT

2、NAT大家肯定不陌生: 在家里、公司上网，一般都是通过路由器的，这么做的好处有：

      （1）上述的节约IP地址。只需要给路由器分配公网IP，路由器内部的设备用内网网段的地址，不同路由器内网网段的地址能复用。比如一个家用路由器，一般最多能支持10来个设备同时连接路由器，都通过路由器上网，运营商只需要给路由器分配一个公网IP即可 

      （2）内网的设备并不直接暴露在公网，只能让路由器对外直接通信，在一定程度上保障了内网设备收不到外界的各种扫描、探测类的数据包，保障内网安全。 路由器收到外界这些主动连接的数据包会直接丢弃（除非网关设置了反向代理，比如服务器前端的网关一般会转发80端口的数据包），这也是客户端之间通信需要“穿透、打洞”的根本原因！

      （3）对于部分网关而言：只要内网发送数据的IP不变，那么自己对外映射转发的端口就不变；比如内部192.168.0.10对外发数据，网关的公网地址是111.111.111.111，此时随机选一个端口比如111，那么对外数据包的源信息就是111.111.111.111:111; 后续网关凡是收到目的是111.111.111.111:111的数据包，不管这个包来自哪（因为是UDP协议，源和目的没有建立连接），都认为是响应192.168.0.10这个节点的，一律转发到该节点，这就是所谓的full cone NAT，这种方式NAT并不验证源IP和端口就直接转发到内网主机（包括内网主机并未访问过的IP和端口），不安全。下面介绍的UDP打洞必须依赖这个特性！

3、NAT原理： NAT转发的种类与很多，这里仅说明最想见的两种情况，分别作为客户端和服务端的转发，下面简单介绍一下整个访问过程。
![[2052730-20210120204518482-1992630750.png]]

正常情况下，是客户端主动向服务端请求数据，比如：

         （1）客户端内网的PC1给服务器发送消息，内容是hello，消息格式（为了突出重点，这里简化一下成IP:PORT:MESSAGE）192.168.1.6:1234:hello -> 139.186.199.148:80

         （2）网关收到这条消息后，对消息做转换和映射，新格式为：106.186.53.107: 5678:hello -> 139.186.199.148:80; 此时路由器内部会维护一个映射表192.168.1.6:1234->106.186.53.107: 5678, 表明内网哪个ip的哪个端口往外发数据包时自己转换成了哪个端口（类似交换机的MAC映射表）；

           (3)   服务器的网关接收到数据包，发现是80端口，并且入栈规则这里也配置了允许80端口的数据包，于是乎这个包通过（这个就是所谓的反向代理）；大型站点的网关内部一般都不止1台服务器，这些服务器组成集群。网关会有一套负载均衡的算法决定把这个数据请求发给哪个内网服务器，比如这里选中了10.0.1.10:1234，消息格式就变成了10.0.1.1:80:hello->10.0.1.10:1234，此时网关同样也会维护一个映射表，记录哪个源IP：端口发过来的数据（也就是hello这条消息）被自己转发到了哪个服务器，比如这里就是106.186.53.107: 5678->10.0.1.10:1234；

                  内网服务器响应请求，给网关回复数据包，格式为：10.0.1.10:1234:nihao -> 10.0.1.1:80

         （4）网关收到服务器的包后，从自己先前维护的映射表查找这个请求包来自那里，一查发现是106.186.53.107: 5678，于是乎消息变成了：139.186.199.148:80:nihao -> 106.186.53.107: 5678

         （5）客户端的网关收到服务器回应，同样查自己维护的映射表，发现192.168.1.1:1234（也就是PC1）曾经向139.186.199.148:80发送过请求包，于是乎消息格式又变成了：139.186.199.148:80:nihao ->192.168.1.6:1234

          整个数据传输的过程不算复杂，最关键的两个地方：客户端的网关、服务端的网关。这两个网关在转发IP和端口的同时维护了映射表，每收发一个数据包都要查找这个映射表来确认该往哪个端口转发！

    对于服务端的网关，因为要对外提供服务（比如这里的http），所以网关收到80端口的数据包请求后会继续转发到内部服务器；
    对于客户端，因为内部的PC1主动对外发送了数据包，网关这里有映射记录的；所以服务端返回包后，网关才会继续转发给PC1；如果PC1从未主动对外发送任何数据，网关也不会有任何PC1的地址转换映射记录，外部发过来的数据包一律会被网关直接丢弃，这是某些场景下需要内网穿透的根本原因！

4、内网穿透方案

     网络分了5层，传输层有tcp和udp协议；tcp是面向连接的可靠数据传输协议，udp是面向消息的无连接传输协议。相比之下，udp协议简单，不需要建立连接，打洞的时候对硬件资源消耗小，所以做内网穿透时首选UDP协议！

     如上所述：两个PC平时都通过网关上网，网关也只记录了PC发到服务器的映射。如果PC1通过网关1给PC2发消息，首先要经过网关2.  但是网关2并没有PC2发数据包给PC1的记录，所以认为来自PC1的数据包是“不请自来”，直接丢弃，导致PC1与PC2无法直接通信！为了在PC1和PC2之间“牵线搭桥”，就需要要有公网地址的服务器了！为了突出重点，这里简化了网络拓扑，只保留关键的节点，如下：            

![[2052730-20210121101527039-866110896.png]]

让PC1和PC2之间互相通信，最简单的就是让公网服务器“传话”：PC1和PC2都主动连接服务器，都给都服务器发消息，服务器来转发双方的数据。原理倒是简单，不过服务器的带宽和流量压力就大了，有可能从成为通信的瓶颈，最好是能让PC1和PC2之间直接通信，不再通过服务器长期中转，该怎么操作了？

-     PC1和PC2分别通过各自的网关主动联系服务器，这一步本质是**在服务器那里备案（服务器保存了222.222.222.222:222和333.333.333.333:333两个PC网关的地址）**，让服务器知道有两个客户端连上了；
-     服务器给PC2发送PC1的网关地址222.222.222.222:222；由于PC2曾经主动给服务器发送过数据，网关2有地址的映射记录，所以服务器给PC2的消息能顺利送达PC2；同理，服务器也给PC1发送PC2网关的地址和端口，即:333.333.333.333:333; 这一步的本质是**让PC1和PC2知道对方网关的IP和端口**，方便后续通信；
-     此时PC2根据服务器提供的PC1的端口和IP，给PC1网关发消息，此时**PC2的网关就会新增一条映射：192.168.0.10:123->222.222.222.222:222**； 消息到达网关1后，由于**网关1没有PC1给网关2发消息的记录，**觉得这个数据包是“不请自来”，**不知道该转发给内网哪个设备，只能丢掉**；
-    PC1接着给PC2的网关2发消息，此时网关1新增一条映射: 192.168.1.10:123->333.333.333.333:333； **网关2收到这条消息后，查看自己的映射表，发现自己内网的192.168.0.10:123曾经给222.222.222.222：222发过消息，觉得这个会是网关1回复的，直接转发给内网的192.168.0.10:123**，此时PC1终于联系上了PC2！
-   PC2继续给PC1发消息，由于PC1上一步给PC2发送了消息，网关1有映射记录，所以网关1直接把这个包转发给PC1！ 至此: PC1和PC2终于可以互相收发消息！

       以上便是利用UDP协议穿透内网（俗称打洞）的全过程！整个过程最核心的就是穿透各自的网关，办法也不复杂，就是**PC主动给另一个PC发消息，让网关有地址的映射记录，对方回复消息时自己的网关才知道转发到内网的哪个节点**！所以说**内网穿透、打洞的本质就是让网关有内、外部地址的映射和转换记录，外网来数据后网关能顺利准发到内网正确的节点**！

 5、代码实现

      UDP打洞技术早在10几年前bt这种P2P软件流行时就已经成熟了，github上代码一大堆，我这里参考[https://blog.csdn.net/yxc135/article/details/8541563](https://blog.csdn.net/yxc135/article/details/8541563) 做个简单的介绍；

    （1）服务端：服务端本质上就是个“中间商”，PC1和PC2连接后把自己的公网IP和端口都在服务端登记备案；服务端把PC1和PC2的公网IP分别发给对方，核心代码如下：

```c
/*
某局域网内客户端C1先与外网服务器S通信，S记录C1经NAT设备转换后在外网中的ip和port；
然后另一局域网内客户端C2与S通信，S记录C2经NAT设备转换后在外网的ip和port；S将C1的
外网ip和port给C2，C2向其发送数据包；S将C2的外网ip和port给C1，C1向其发送数据包，打
洞完成，两者可以通信。(C1、C2不分先后)
测试假设C1、C2已确定是要与对方通信，实际情况下应该通过C1给S的信息和C2给S的信息，S
判断是否给两者搭桥。（因为C1可能要与C3通信，此时需要等待C3的连接，而不是给C1和
C2搭桥）
编译：gcc UDPServer.c -o UDPServer -lws2_32
*/
#include <Winsock2.h>
#include <stdio.h>
#include <stdlib.h>
 
#define DEFAULT_PORT 5050
#define BUFFER_SIZE 100
 
int main() {
    //server即外网服务器
    int serverPort = DEFAULT_PORT;
    WSADATA wsaData;
    SOCKET serverListen;
    struct sockaddr_in serverAddr;
 
    //检查协议栈
    if (WSAStartup(MAKEWORD(2,2),&wsaData) != 0 ) {
        printf("Failed to load Winsock.\n");
        return -1;
    }
    
    //建立监听socket
    serverListen = socket(AF_INET,SOCK_DGRAM,0);
    if (serverListen == INVALID_SOCKET) {
        printf("socket() failed:%d\n",WSAGetLastError());
        return -1;
    }
 
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(serverPort);
    serverAddr.sin_addr.s_addr = htonl(INADDR_ANY);
 
    if (bind(serverListen,(LPSOCKADDR)&serverAddr,sizeof(serverAddr)) == SOCKET_ERROR) {
        printf("bind() failed:%d\n",WSAGetLastError());
        return -1;
    }
 
    //接收来自客户端的连接，source1即先连接到S的客户端C1
    struct sockaddr_in sourceAddr1;
    int sourceAddrLen1 = sizeof(sourceAddr1);
    SOCKET sockC1 = socket(AF_INET,SOCK_DGRAM,0);
    char bufRecv1[BUFFER_SIZE];
    int len;
 
    len = recvfrom(serverListen, bufRecv1, sizeof(bufRecv1), 0,(struct sockaddr*)&sourceAddr1,&sourceAddrLen1);
    if (len == SOCKET_ERROR) {
        printf("recv() failed:%d\n", WSAGetLastError());
        return -1;
    }
 
    printf("C1 IP:[%s],PORT:[%d]\n",inet_ntoa(sourceAddr1.sin_addr)
                ,ntohs(sourceAddr1.sin_port));
 
    //接收来自客户端的连接，source2即后连接到S的客户端C2
    struct sockaddr_in sourceAddr2;
    int sourceAddrLen2 = sizeof(sourceAddr2);
    SOCKET sockC2 = socket(AF_INET,SOCK_DGRAM,0);
    char bufRecv2[BUFFER_SIZE];
 
    len = recvfrom(serverListen, bufRecv2, sizeof(bufRecv2), 0,(struct sockaddr*)&sourceAddr2,&sourceAddrLen2);
    if (len == SOCKET_ERROR) {
        printf("recv() failed:%d\n", WSAGetLastError());
        return -1;
    }
 
    printf("C2 IP:[%s],PORT:[%d]\n",inet_ntoa(sourceAddr2.sin_addr)
                ,ntohs(sourceAddr2.sin_port));    
    
    //向C1发送C2的外网ip和port
    char bufSend1[BUFFER_SIZE];//bufSend1中存储C2的外网ip和port
    memset(bufSend1,'\0',sizeof(bufSend1));
    char* ip2 = inet_ntoa(sourceAddr2.sin_addr);//C2的ip
    char port2[10];//C2的port
    itoa(ntohs(sourceAddr2.sin_port),port2,10);//10代表10进制
    for (int i=0;i<strlen(ip2);i++) {
        bufSend1[i] = ip2[i];
    }
    bufSend1[strlen(ip2)] = '^';
    for (int i=0;i<strlen(port2);i++) {
        bufSend1[strlen(ip2) + 1 + i] = port2[i];
    }
 
    len = sendto(sockC1,bufSend1,sizeof(bufSend1),0,(struct sockaddr*)&sourceAddr1,sizeof(sourceAddr1));
    if (len == SOCKET_ERROR) {
        printf("send() failed:%d\n",WSAGetLastError());
        return -1;
    } else if (len == 0) {
        return -1;
    } else {
        printf("send() byte:%d\n",len);
    }
 
    //向C2发送C1的外网ip和port
    char bufSend2[BUFFER_SIZE];//bufSend2中存储C1的外网ip和port
    memset(bufSend2,'\0',sizeof(bufSend2));
    char* ip1 = inet_ntoa(sourceAddr1.sin_addr);//C1的ip
    char port1[10];//C1的port
    itoa(ntohs(sourceAddr1.sin_port),port1,10);
    for (int i=0;i<strlen(ip1);i++) {
        bufSend2[i] = ip1[i];
    }
    bufSend2[strlen(ip1)] = '^';
    for (int i=0;i<strlen(port1);i++) {
        bufSend2[strlen(ip1) + 1 + i] = port1[i];
    }
    
    len = sendto(sockC2,bufSend2,sizeof(bufSend2),0,(struct sockaddr*)&sourceAddr2,sizeof(sourceAddr2));
    if (len == SOCKET_ERROR) {
        printf("send() failed:%d\n",WSAGetLastError());
        return -1;
    } else if (len == 0) {
        return -1;
    } else {
        printf("send() byte:%d\n",len);
    }
 
    //server的中间人工作已完成，退出即可，剩下的交给C1与C2相互通信
    closesocket(serverListen);
    closesocket(sockC1);
    closesocket(sockC2);
    WSACleanup();
 
    return 0;
}
```

（2）客户端

      内网穿透和打洞的本质是**路由器能够转换和映射地址**，但这个是路由器内部的功能，貌似路由器厂家也并未开放API来新增或删除这种映射（这种API很危险，一旦被黑客利用，**轻则导致断网，中则导致内部不同设备之间数据错乱，重则导致外部数据随随便便进入内网**），开发人员是没法直接增加映射的，只能通过发送UDP的数据包让路由器增加地址映射（原理上讲：这像不像CPU的L1\L2\L3缓存啊？开发人员没法直接修改缓存，但是可以通过读写数据、mov cr3，eax等方式间接刷新缓存），核心代码如下：

-     连接服务器，让服务器保存自己的公网IP和端口
-     从服务器接受另一个PC的公网IP和端口
-     向另一个PC的公网IP和端口发送数据包，**来刷新自己网关的地址转换映射表**；

```c
/*
客户端C1，连接到外网服务器S，并从S的返回信息中得到它想要连接的C2的外网ip和port，然后
C1给C2发送数据包进行连接。
*/
#include<Winsock2.h>
#include<stdio.h>
#include<stdlib.h>
 
#define PORT 7777
#define BUFFER_SIZE 100
 
//调用方式:UDPClient1 10.2.2.2 5050 (外网服务器S的ip和port)
int main(int argc,char* argv[]) {
    WSADATA wsaData;
    struct sockaddr_in serverAddr;
    struct sockaddr_in thisAddr;
 
    thisAddr.sin_family = AF_INET;
    thisAddr.sin_port = htons(PORT);
    thisAddr.sin_addr.s_addr = htonl(INADDR_ANY);
 
    if (argc<3) {
        printf("Usage: client1[server IP address , server Port]\n");
        return -1;
    }
 
    if (WSAStartup(MAKEWORD(2,2),&wsaData) != 0) {
        printf("Failed to load Winsock.\n");
        return -1;
    }
 
    //初始化服务器S信息
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(atoi(argv[2]));
    serverAddr.sin_addr.s_addr = inet_addr(argv[1]);
    
    //建立与服务器通信的socket和与客户端通信的socket
    SOCKET sockS = socket(AF_INET,SOCK_DGRAM,0);
    if (sockS == INVALID_SOCKET) {
        printf("socket() failed:%d\n",WSAGetLastError());
        return -1;
    }
    if (bind(sockS,(LPSOCKADDR)&thisAddr,sizeof(thisAddr)) == SOCKET_ERROR) {
        printf("bind() failed:%d\n",WSAGetLastError());
        return -1;
    }
    SOCKET sockC = socket(AF_INET,SOCK_DGRAM,0);
    if (sockC == INVALID_SOCKET) {
        printf("socket() failed:%d\n",WSAGetLastError());
        return -1;
    }
 
    char bufSend[] = "I am C1";
    char bufRecv[BUFFER_SIZE];
    memset(bufRecv,'\0',sizeof(bufRecv));
    struct sockaddr_in sourceAddr;//暂存接受数据包的来源，在recvfrom中使用
    int sourceAddrLen = sizeof(sourceAddr);//在recvfrom中使用
    struct sockaddr_in oppositeSideAddr;//C2的地址信息
 
    int len;
    
    //C1给S发送数据包
    len = sendto(sockS,bufSend,sizeof(bufSend),0,(struct sockaddr*)&serverAddr,sizeof(serverAddr));
    if (len == SOCKET_ERROR) {
        printf("sendto() failed:%d\n", WSAGetLastError());
        return -1;
    }
 
    //C1从S返回的数据包中得到C2的外网ip和port
    len = recvfrom(sockS, bufRecv, sizeof(bufRecv), 0,(struct sockaddr*)&sourceAddr,&sourceAddrLen);
    if (len == SOCKET_ERROR) {
        printf("recvfrom() failed:%d\n", WSAGetLastError());
        return -1;
    }
 
    //下面的处理是由于测试环境（本机+两台NAT联网的虚拟机）原因，若在真实环境中不需要这段处理。
    /*
      关闭与服务器通信的socket，并把与C2通信的socket绑定到相同的端口，真实环境中，路由器的NAT会将客户端
      对外的访问从路由器的外网ip某固定端口发送出去，并在此端口接收
    */
    closesocket(sockS);
    if (bind(sockC,(LPSOCKADDR)&thisAddr,sizeof(thisAddr)) == SOCKET_ERROR) {
        printf("bind() failed:%d\n",WSAGetLastError());
        return -1;
    }
 
    char ip[20];
    char port[10];
    int i;
    for (i=0;i<strlen(bufRecv);i++)
        if (bufRecv[i] != '^')
            ip[i] = bufRecv[i];
        else break;
    ip[i] = '\0';
    int j;
    for (j=i+1;j<strlen(bufRecv);j++)
        port[j - i - 1] = bufRecv[j];
    port[j - i - 1] = '\0';
 
    oppositeSideAddr.sin_family = AF_INET;
    oppositeSideAddr.sin_port = htons(atoi(port));
    oppositeSideAddr.sin_addr.s_addr = inet_addr(ip);
 
    //下面的处理是由于测试环境（本机+两台NAT联网的虚拟机）原因，若在真实环境中不需要这段处理。
    /*
      此处由于是在本机，ip为127.0.0.1，但是如果虚拟机连接此ip的话，是与虚拟机本机通信，而不是
      真实的本机，真实本机即此实验中充当NAT的设备，ip为10.0.2.2。
    */
    oppositeSideAddr.sin_addr.s_addr = inet_addr("10.0.2.2");
 
    //设置sockC为非阻塞
    unsigned long ul = 1;
    ioctlsocket(sockC, FIONBIO, (unsigned long*)&ul);
 
    //C1向C2不停地发出数据包，得到C2的回应，与C2建立连接
    while (1) {
        Sleep(1000);
        //C1向C2发送数据包
        len = sendto(sockC,bufSend,sizeof(bufSend),0,(struct sockaddr*)&oppositeSideAddr,sizeof(oppositeSideAddr));
        if (len == SOCKET_ERROR) {
            printf("while sending package to C2 , sendto() failed:%d\n", WSAGetLastError());
            return -1;
        }else {
            printf("successfully send package to C2\n");
        }
    
        //C1接收C2返回的数据包，说明C2到C1打洞成功，C2可以直接与C1通信了
        len = recvfrom(sockC, bufRecv, sizeof(bufRecv), 0,(struct sockaddr*)&sourceAddr,&sourceAddrLen);
        if (len == WSAEWOULDBLOCK) {
            continue;//未收到回应
        }else {
            printf("C2 IP:[%s],PORT:[%d]\n",inet_ntoa(sourceAddr.sin_addr)
                ,ntohs(sourceAddr.sin_port));
            printf("C2 says:%s\n",bufRecv);
            
        }
    }
 
    closesocket(sockC);
 
    
}
```

6、UDP内网穿透/打洞应用场景：

（1）P2P文件传输：避免所有流量都经过服务器，减轻服务器带宽和流量压力，也不用收集用户隐私

（2）微信、企业微信、qq等IM用户之间的视频、语音聊天；或则用户之间互相传文件

说明：（1）内网穿透、UDP打洞能够成功，主要**取决于网关的NAT策略**！
### 参考
---
[UDP内网穿透和打洞原理与代码实现 - 第七子007 - 博客园](https://www.cnblogs.com/theseventhson/p/14304321.html)

1、[https://www.zhihu.com/question/20168985](https://www.zhihu.com/question/20168985)  为什么IPV4的地址不够用

2、[https://www.bilibili.com/video/BV16b411e78C?from=search&seid=8594225681123793506](https://www.bilibili.com/video/BV16b411e78C?from=search&seid=8594225681123793506)  C++实战UDP打洞原理及实现

3、[https://blog.csdn.net/yxc135/article/details/8541563](https://blog.csdn.net/yxc135/article/details/8541563)  C语言实现UPD打洞

4、[https://www.cnblogs.com/dyufei/p/7466924.html](https://www.cnblogs.com/dyufei/p/7466924.html)   CONE NAT 和 Symmetric NAT