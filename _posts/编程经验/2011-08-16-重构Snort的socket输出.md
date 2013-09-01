---
layout: post
categories:
- 编程经验
tags:
- tcpdump
---
##Linux下网络抓包命令tcpdump详解

tcpdump采用命令行方式，它的命令格式为：  
    　　tcpdump[ -adeflnNOpqStvx ] [ -c 数量 ] [ -F 文件名 ]  
　　　　　　　　　　[ -i 网络接口 ] [ -r 文件名] [ -s snaplen ]  
　　　　　　　　　　[ -T 类型 ] [ -w 文件名 ] [表达式 ]  

    

 - -a 　　　将网络地址和广播地址转变成名字；
 - -d 　　　将匹配信息包的代码以人们能够理解的汇编格式给出；
 - -dd 　　 将匹配信息包的代码以c语言程序段的格式给出；
 - -ddd 　　将匹配信息包的代码以十进制的形式给出；
 - -e 　　　在输出行打印出数据链路层的头部信息；
 - -f 　　　将外部的Internet地址以数字的形式打印出来；
 - -l 　　　使标准输出变为缓冲行形式；
 - -n 　　　不把网络地址转换成名字；
 - -t 　　　在输出的每一行不打印时间戳；
 - -v 　　　输出一个稍微详细的信息，例如在ip包中可以包括ttl和服务类型的信息；
 - -vv 　　 输出详细的报文信息；
 - -c 　　　在收到指定的包的数目后，tcpdump就会停止；
 - -F 　　　从指定的文件中读取表达式,忽略其它的表达式；
 - -i 　　　指定监听的网络接口；
 - -r 　　　从指定的文件中读取包(这些包一般通过-w选项产生)；
 - -w 　　　直接将包写入文件中，并不分析和打印出来；
 - -T 　　　将监听到的包直接解释为指定的类型的报文，常见的类型有rpc

（远程过程调用）和snmp（简单网络管理协议；）

示例：

1、如果要抓eth0的包，命令格式如下：

tcpdump -i eth0 -w /tmp/eth0.cap

2、如果要抓192.168.1.20的包，命令格式如下：

tcpdump -i etho host 192.168.1.20 -w /tmp/temp.cap

3、如果要抓192.168.1.20的ICMP包，命令格式如下：

tcpdump -i etho host 192.168.1.20 and icmp -w /tmp/icmp.cap

4、如果要抓192.168.1.20的除端口10000,10001,10002以外的其它包，命令格式如下：

tcpdump -i etho host 192.168.1.20 and ! port 10000 and ! port 10001 and ! port 10002 -w /tmp/port.cap

5、假如要抓vlan 1的包，命令格式如下：

tcpdump -i eth0 port 80 and vlan 1 -w /tmp/vlan.cap

6、假如要抓pppoe的密码，命令格式如下：

tcpdump -i eht0 pppoes -w /tmp/pppoe.cap

7、假如要抓eth0的包，抓到10000个包后退出，命令格式如下：

tcpdump -i eth0 -c 10000 -w /tmp/temp.cap

8、在后台抓eth0在80端口的包，命令格式如下：

nohup tcpdump -i eth0 port 80 -w /tmp/temp.cap &

########################

自己用的命令：：tcpdump -i eth0 -w /tmp/eth0.cap -s0   否则抓的包不全，没有内容