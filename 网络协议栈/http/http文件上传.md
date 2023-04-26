### http如何实现文件上传

![[Pasted image 20230308152621.png]]
通过wireshark抓包，查看上传文件postman发送的请求包。
可以看到Content-Type是multipart/form-data;boundary=---------...\r\n类型
然后将整个文件字节流传送过去。

传送body信息，后面也有boundary来标记结束。
![[Pasted image 20230308152835.png]]
