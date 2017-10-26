---
title: QT移植到am437x开发板
date: 2017-08-23 18:42:00
tags: [QT,AM437X,跨平台]
categories: QT

---
最近在把之前开发好的应用程序移植到板子上，由于公司用的是TI的am437x系列，网上的资料也比较少，所以前前后后摸索了一个多星期，中间遇到了一些问题，通过自己的摸索还算是最终移植成功了。下面就此做一个简单的小结：
首先贴出我参考的以些资料：
[【创龙AM4379 Cortex-A9试用体验】之移植QT5.4.1与tslib1.4到TL-4379开发板](http://bbs.elecfans.com/forum.php?mod=viewthread&tid=905966&extra=&ordertype=2)
[Qt for ARM_Linux环境搭建-Qt5.7+iTop4412嵌入式平台移植 ](http://blog.csdn.net/hechao3225/article/details/52981148)
[【转帖】嵌入式4412开发板QT5.7编译安装到arm ](http://www.cnblogs.com/topeet/p/5711957.html)
[Qt5.7.0移植到4412 ](http://www.cnblogs.com/t1029901995/p/6046600.html)
[Qt for ARM_Linux环境搭建-Qt5.7+iTop4412嵌入式平台移植 ](http://blog.csdn.net/hechao3225/article/details/52981148)

在移植的过程中遇到的问题：
1.linux下qt不能编译？
  解决：（1）权限的问题：工程代码需要权限；qtcreator也需要权 （2）linux下目录是‘/’（没有双杠，且左低右高）
2.移植的注意点：
  （1）如果是原生的ubuntu（刚装好的），移植前需要先安装x11的库，见下图：
  
  ![](http://ols4zt49w.bkt.clouddn.com/%E5%AE%89%E8%A3%85x11,%E5%8F%AF%E8%83%BD%E5%AE%89%E8%A3%85%E6%BA%90%E9%9C%80%E8%A6%81%E6%94%B9.png)
  （2）安装跟板子配套的交叉编译工具链，注意：三星的4412是arm-none-linux-gnueabi-gcc，ti用的是arm-linux-gnueabihf，由于板子出厂固件中运行的是基于Qt5.7,因此我下载的也是跟5.7配套的交叉编译工具链`gcc-linaro-6.2.1-2016.11-x86_64_arm-linux-gnueabihf.tar`
  （3）修改交叉编译架构用到的信息：
   ![](http://ols4zt49w.bkt.clouddn.com/1503489195%281%29.png)
   
  （3）创建脚本文件用于生成Makefile文件时，要有`-no-iconv  \`, 否则字库会出问题。
   ![](http://ols4zt49w.bkt.clouddn.com/1503489408%281%29.png)
  
  （4）在开发板上运行校正程序时出现No raw modules loaded
```
解决方法是把  tslib/etc目录下的ts.conf 的 "#module_raw input"的注释符号“＃”去掉。但记住不要在前面留有 空格 ，否则会出现错误Segmentation fault
```
  （5）程序运行起来，仍报错：
QIconvCodec::convertFromUnicode: using Latin-1 for conversion, iconv_open failed
QIconvCodec::convertToUnicode: using Latin-1 for conversion, iconv_open failed​
解决 ：下载 http://ftp.gnu.org/gnu/libiconv/libiconv-1.14.tar.gz   
                ./configure -prefix=$PWD/_install -host=arm-linux-gnueabihf   
                make   
                make install 
                把_install/lib 下的preloadable_libiconv.so 拷到系统的/system/lib 下,  
                export LD_PRELOAD=/system/lib/preloadable_libiconv.so

   （6）在环境变量配置时，如果不想把板子自带的DT Demo覆盖掉，可以不配置QT_ROOT
   配置如下：
   ![](http://ols4zt49w.bkt.clouddn.com/1503489027%281%29.png)
   
   （7）要使板子支持中文，可以下载中文字库，放在ARM板的fonts目录下，这里我放在`/usr/share/fonts/ttf`下。此处我提供下载连接。
   [QT中文字库，直接放在ARM板的fonts目录下](http://ols4zt49w.bkt.clouddn.com/DroidSansFallback.ttf)
   
   （8）移植sqlite到arm板上，参考：http://www.cnblogs.com/lidabo/p/5851752.html

最后，附上一张移植后的图片：
![](http://ols4zt49w.bkt.clouddn.com/881346359011543039.jpg)