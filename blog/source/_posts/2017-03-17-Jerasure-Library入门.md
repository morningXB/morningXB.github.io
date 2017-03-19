---
title: Jerasure Library入门
date: 2017-03-17 09:09:17
tags: [erasure code,Jerasure]
categories: 纠删码
---
# Jerasure Library入门 #

## 介绍 ##
Jerasure是由C语言编写的实现了纠删码编解码的开源库，提供了范德摩尔RS，柯西RS，优化柯西RS，以及针对RAID-6的Liberation,Liber8tion,Boomth编码，以及基于VRS的optRaid-6编码。
纠删码起源通信中防止信息丢失，现在被广泛应用到存储系统中包括分布式存储系统，内存存储系统，闪存存储系统，磁盘阵列中，以防止系统出错导致的信息丢失，它的优点是：相较于传统的副本存储机制，一份数据在系统中复制三份存储，纠删码编码后数据有效的减少了冗余数据大小，并且拥有相同的容错率，有效减少存储空间，举个列子一块size为M的数据块，在副本机制中要占据3*M的空间，容错为2，而纠删码(6,2)系统来说，只占据1.3 M空间拥有相同容错，减少了40% 的空间，但 是它的缺陷是：在编解码上的耗时，以及增加了系统的网络负载。

github：[https://github.com/tsuraan/Jerasure](https://github.com/tsuraan/Jerasure "https://github.com/tsuraan/Jerasure")
doc：[http://jerasure.org/jerasure-2.0/](http://jerasure.org/jerasure-2.0/ "http://jerasure.org/jerasure-2.0/")
Online-home：[http://jerasure.org/jerasure/jerasure](http://jerasure.org/jerasure/jerasure "http://jerasure.org/jerasure/jerasure")
<!-- more -->

## 安装 ##
在github中打开或者项目首页中可看见最新的版本的是Jeraure2.0，通过README可以知道，为了增加Jerasure的灵活性，version2.0中将gf运算与编码库分离，所以使用Jeraure2.0之前，我需要先安装GF-Complete库。
Online-home：[http://jerasure.org/jerasure/gf-complete](http://jerasure.org/jerasure/gf-complete "http://jerasure.org/jerasure/gf-complete")

``` bash
git clone http://lab.jerasure.org/jerasure/gf-complete.git
cd gf-complete
./autogen.sh #(如果通过tar包安装这步忽略)
./congfigure
make
sudo make install
```
在相关目录下(一般是/usr/local/lib和/usr/local/include),有相应的头文件以及.so库文件，即安装成功。

接下来安装Jeraure2.0：
``` bash
git clone http://lab.jerasure.org/jerasure/jerasure.git
cd jerasure
autoreconf --force --install -I m4 #(如果通过tar包安装这步忽略)
./configure
make
sudo make install
```
同样的方法查看是否安装成功。

之后就可以在程序中应用头文件，并且通过 -lJeraure来链接库文件编译。
## Jeraure2.0库的使用 ##
首先当然要仔细阅读文档地址上面已经给了，通过文档你可以对纠删码的编码流程有所了解，请务必知晓库中各参数的意义。
我们主要阅读example文件夹下的encoder.c和decoder.c源文件，这两个文件中包含了所有编码的编解码过程 ，以及相关函数的调用，具体的API会在下一篇文章中讲解，这里我们运行生成的obj文件，看一下Jerasure纠删码的效果，达到一个直观的了解。
encoder.o参数
``` bash
./encoder "inputfilepath" k m "tech_method_name" w packetize buffersize
```
inputfilepath:编码文件路径
k:源数据块个数
m:冗余块个数
w:数据字长
tech_method_name:{"reed_sol_van", "reed_sol_r6_op", "cauchy_orig", "cauchy_good", "liberation", "blaum_roth", "liber8tion", "no_coding"};
packetsize:数据块大小
buffersize:数据缓冲区大小
程序流程大致是：
1.根据packetsize以及运行环境来调整buffersize，计算blocksize即每个块的大小，malloc空间
2.根据不同的方法，准备编码矩阵/位矩阵/调度表
3.根据不同的方法，执行不同的编码方法
4.完成文件生成
### 编码 ###
``` bash
./encoder "/home/xubin/图片/test02.jpg" 4 2 "reed_sol_van" 8 8 1024
```
执行后,在相应的目录下，产生test02_k1.jpg，test02_k2.jpg，test02_k3.jpg，test02_k4.jpg
test02_m1.jpg，test02_m2.jpg,以及test02_meta.txt配置文件。
{% asset_img 编码.png %}
### 解码 ###
删除一个原始数据块和一个冗余块
{% asset_img 删除数据块.png %}
``` bash
./decoder "test02.jpg"
```
{% asset_img 解码.png %}
执行后在相对目录下，生成test02decoder.jpg
可以看到命令的相应效率以及编码效率，对于各个编码的效率问题主要涉及到编码方法以及一吸血参数的选择上，会在以后的博客中对比讲解。
有不解或错误的地方记得留言或者@我呀。