---
layout: post
categories:
- 编程经验
tags:
- Snort
---
##修改snort的socket模式的输出内容  

###snort的输出方式复用
实践发现snort可以同时应用fast和unsock两种模式。
-A fast -A unsock 不过必须fast后unsock
###snort源码部分函数学习

阅读snort源码中snort自带socket发送日志功能，可以说非常方便。  
但是如果需要用snort的检测模块输出我们想要的数据包某段值，就需要改snort的源码了。  
下面是我之前跟踪过的记录，简单点说，我们可以根据fast日志输出中的特殊符号，如->来找ip和端口等。特殊位置，这样我们就可以找到snort中对应解析后的结构体。  

*结构体 in_addr 用来表示一个32位的IPv4地址.

extern void *memcpy(void *destin, void *source, unsigned n);  ###由source指向地址为起始地址的连续n个字节的数据复制到以destin指向地址为起始地址的空间内。

```
public: int SendTo( 
　　SOCKET s; 
　　unsigned char buffer __gc[], 
　　int size, 
　　SocketFlags socketFlags, 
　　sockaddr FAR *addr 
　　int len 
　　); 
```
>返回值：实际发送数据的长度。   
>parameter :   
　　s 套接字   
　　buff 待发送数据的缓冲区   
　　size 缓冲区长度   
　　Flags 调用方式标志位, 一般为0, 改变Flags，将会改变Sendto发送的形式   
　　addr （可选）指针，指向目的套接字的地址   
　　len addr所指地址的长度  

    size_t fwrite(const void*buffer,size_t size,size_t count,FILE*stream); 

　　注意：这个函数以二进制形式对文件进行操作，不局限于文本文件   
　　返回值：返回实际写入的数据块数目   
　　（1）buffer：是一个指针，对fwrite来说，是要输出数据的地址。   
　　（2）size：要写入内容的单字节数；   
　　（3）count:要进行写入size字节的数据项的个数；   
　　（4）stream:目标文件指针。 



 

通过寻找unsock下snort的ip输出和时间输出

找此结构是通过->ip之间的符号找到LogIpAddrs函数，输出ip->ip。

printf("!!!!!!!!!!!!!%s:%d\n",inet_ntoax(GET_SRC_ADDR(p)), p->sp);可以在输出结构体p中的ip和端口

在packet结构体中有：x->ip4_header->source 和x->iph->ip_src 存放原ip地址。

顺藤摸瓜找到：inet_ntoax(GET_SRC_ADDR(p)), p->sp   ########输出ip和端口。（怀疑inet_ntoax是inet_ntoa的变体函数，在snort的头文件中实现）

　linux下：   
　　函数声明：char *inet_ntoa (struct in_addr);   
　　返回点分十进制的字符串在静态内存中的指针。

printf("@@@@@@@@@@@@%s:%d\n",inet_ntoax(GET_SRC_ADDR(p)), p->sp);   ##可以把p中的ip和端口输出。

查找pkt是否有ip。。等信息

（2）时间戳LogTimeStamp(data->log, p);在这里实现。同理也很简单。  

建立Alertpkt_txt的结构体，装入msg ip 时间戳 端口。用socket传结构体，目的达到。
