---
layout: post
categories:
- 编程经验
tags:
- 第三方代理
---
#apache以非root用户调用root程序

**使用第三方代理**，就可以使cgi获得root权限。  
思路是 apache执行脚本----》执行 c程序（获得root）---》执行目的cgi（需要root权限）。

```c代码：
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

int main()
{
    uid_t uid ,euid;

    uid = getuid() ;
    euid = geteuid();

    //printf("my uid :%u\n",getuid());  //这里显示的是当前的uid 可以注释掉.
    //printf("my euid :%u\n",geteuid()); //这里显示的是当前的euid
    if(setreuid(euid, uid))  //交换这两个id
        perror("setreuid");
    //printf("after setreuid uid :%u\n",getuid());
    //printf("afer sertreuid euid :%u\n",geteuid());

    //system("/sbin/iptables -L"); //执行iptables -L命令
   system("/var/www/cgi-bin/test.sh"); //执行一个需root权限运行的脚本
    return 0;
}
```
重点在于交换id。。
在以root编译代码后 要赋予


给run_root赋予suid权限

    #chmod u+s run_root
    # ll run_root
    -rwsr-xr-x 1 root root 5162 03-17 12:00 run_root

suid权限：
意思就是当你执行这个命令时暂时拥有的拥有者的权限，也就是root