---
layout: post
title: 技术总结与回忆
categories:
- Programming Review
tags:
- 爬虫Larbin
---
---------


##通过Larbin学Makefile
基本知识点：  
**自动变量**  
自动变量也不需要我们定义，但我们可以在Makefile中使用，如前面看到的那两个特殊的变量$@和$^，还有一些常用的自动变量列入下表，其含义如下：

$@ : 规则中目标体的名字

$^ : 规则中所有的依赖体

$< : 规则中第一个依赖体

$? : 规则中所有时间晚于目标体的依赖体集合

如对应下面这条规则：

example: main.o file1.o file2.o  
gcc $^ -o $@

我们先假定main.o和file2.o比example要新，则 $@ 的值为 example , $^ 的值为 main.o file1.o file2.o , $< 的值为 main.o , $? 的值为 main.o file2.o  

注意：Makefile种的自动变量的作用范围是每一条规则，也就是说对另一条规则来说，这些自动变量的值也会有所不同，还有我们不能给这些变量赋值

**预定义变量**

预定义变量跟自动变量差不多，也是由make程序自动引入的，但它与自动变量有一点不同，预定义变量的作用范围是在整个Makefile文件，同一个预定义变量在不同的规则里其值是不会改变的，除非我们自己给它重新赋值(我们可以给预定义变量赋值，但不能给自动变量赋值)。常见的自定义变量有：

CC : C编译器，默认值为 cc

CPP : C预处理程序，默认值为 cpp

CFLAGS : 传给C编译器的标志，无默认值

CPPFLAGS : 传给C预处理程序的标志，无默认值

LDFLAGS : 传给连接器的标志，无默认值
AR : 归档程序，默认值为 ar
RM : 文件删除命令，默认值为 rm -f
For example: ar cq libfoo.a *.o creates a new library named libfoo.a from all .o files in a directory. Normally you'll use the ar                        command in a makefile so you'll probably use a makefile variable rather than *.o.


larbin目录下结构  
├── adns  
│   └── Makefile        【二 】 
├── config.h            （configure 中生成）  
├── config.make     （configure 中生成）  
├── Makefile        入口： 【一  】
└── src  
    ├── fetch  
    │   └── Makefile  【四】  
    ├── interf  
    │   └── Makefile  【五】  
    ├── larbin               
    ├── larbin.make  
    ├── Makefile         【三  】
    └── utils  
        └── Makefile    【六】  

  1 从configure 开始，7-8行是清空 config.h和 config.make  
  2  定义了自己的函数 exists 『功能是 判断命令$1是否存在』  
  3   
  4 19-34 是将自己的编译环境 写入config.make  
  5   
  6 37 行 写入test.c文件  
  7   
  8 40 行 编写函数 testlib{功能是测试 所要的库是否包含}  
  9   
 10 54-59 行 写入.depend  
 11   
 12 下一步是make  
 13 larbin ->Makefile  
1行 .PHONY (建立伪目标：all clean。。。)   以后可以用make all 或是 make   clean 执行一口气生成多个目标的目的。如果只有make则默认从第一个开始。  
3-4行包含 /src/larbin.make (里面是 larbin的所有.o 文件的变量)  
 9行 在adns目录下make   [进入二]  
11 执行完第9行后，执行src目录下的 make all。【三入口】  
12行 拷贝larbin（bin）到.  
结束。

【二】   执行adns下的Makefile   
跳到第一个依赖 all：libadns.a  
  libadns.a: $(LIBOBJS)  
                  rm -f $@              //删除libadns.a  
                 $(AR) cq $@ $(LIBOBJS)   //将 LIBOBJS (N个.o文件 归档成   libadns.a) 【返回一，执行三】  

【三】执行 src 下的Makefile   make all   
废话不多说：直接到 all: config.h options.h subdirs-all larbin   
       欲生成两个 二进制 文件 subdirs-all 和 larbin  
跳到  subdirs-all:       【四，分别进入src下的子目录，执行Makefile】  
             for d in $(SUBDIR); do (cd $$d; $(MAKE) all); done  
  for 跟shell 命令一样。   
 
\$d 是为了使用 d变量。$$是真正意义上的$.  分别进入了 SUBDIR=utils interf fetch 目录。执行   make all  
【四】 完成，返回三中，执行五】  

【五】之后再 生成larbin  
   larbin: $(ABS-UTILS-OBJ) $(ABS-FETCH-OBJ) $(ABS-INTERF-OBJ) $(ABS-MAIN-OBJ)  #为larbin.make下定义的.o文件  
         $(CXX) $(MF) $(LIBS) -o larbin $(ABS-UTILS-OBJ) \  
       $(ABS-FETCH-OBJ) $(ABS-INTERF-OBJ) $(ABS-MAIN-OBJ)   ../adns/libadns.a    ###所有依赖的 .o文件 和 ../adns下的.o  
