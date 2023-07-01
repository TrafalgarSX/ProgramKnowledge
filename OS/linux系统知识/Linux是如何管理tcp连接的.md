### Linux管理tcp连接的代码实现
---
#### 操作系统中tcp， socket相关结构体
首先，在linux内核的网络模块里维护着一个**全局实例**，用来存储所有和tcp相关的socket：

```c
struct inet_hashinfo tcp_hashinfo;
```

其次，在该实例的内部，又根据socket类型的不同，划分成四个hashtable：
```c
// include/net/inet_hashtables.h
struct inet_hashinfo {
        // key是由本地地址、本地端口、远程地址、远程端口组成的四元组
        // value是正在建立连接或已经建立连接的socket
        // 比如，当内核收到一个tcp消息时，它先从消息头里读出地址和端口等信息
        // 然后用该信息到ehash里获取对应的socket
        // 最后把剩余的tcp数据添加到该socket的recv buf中供用户程序读取
        struct inet_ehash_bucket        *ehash;

        // key是本地端口
        // value是使用这个端口的所有socket
        // 比如，当我们用socket监听一个端口时，该socket就在bhash里
        // 同理，由该监听端口建立的连接对应的那些socket也在这里
        // 因为它们也都是使用同样的本地端口
        struct inet_bind_hashbucket     *bhash;

        // key是本地地址和端口组成的二元组
        // value是对应的处于listen状态的socket
        struct inet_listen_hashbucket   *lhash2;

        // key是本地端口
        // value是对应的处于listen状态的socket
        struct inet_listen_hashbucket   listening_hash[INET_LHTABLE_SIZE];
};
```

在系统启动时，这个全局的tcp_hashinfo实例会在下面的方法中被初始化：
```c
```text
// net/ipv4/tcp.c
void __init tcp_init(void)
{
        // 初始化tcp_hashinfo里的四个hashtable等信息
}
```


### 操作系统 tcp 操作
在tcp编程中一般都分为客户端和服务端，我们先来看下服务端对应的操作。

首先，**一个socket想要监听一个端口，必须要先bind一个地址，然后再执行listen操作。**

其中bind操作就用到了上面的tcp_hashinfo实例里的bhash这个字段，用来判断该端口**是否被占用。**

[linux操作系统是如何管理tcp连接的？ - 知乎](https://zhuanlan.zhihu.com/p/571947344)