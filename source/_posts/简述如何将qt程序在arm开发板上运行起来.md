---
title: 简述如何将qt程序在arm开发板上运行起来
date: 2017-06-27 19:01:38
tags: [qt,arm,镜像]
categories: 嵌入式

---
之前我谈论了qt的一些开发应用，他主要应用在一些嵌入式设备上，比如平板，机顶盒，路由等等。下面我简要通过我的试验和学习过程总结出：将qt程序移植到开发板（平板）上。（）
####  Qt/E4.7.1 编译器的安装
Qt/E4.7.1 使用的编译器是交叉编译器编译工具“arm-linux-gcc-4.3.2.tar.gz”
1.将“arm-linux-gcc-4.3.2.tar.gz”解压到 Ubuntu 系统的文件夹“/usr/local/arm”中
2.修改环境变量（注意：root用户下）
  输入命令“`vim .bashrc`”，打开设置环境变量的文件“.bashrc”，然后进入最后行，如下图，将环境变量修改为
  “`export PATH=$PATH:/usr/local/arm/4.3.2/bin`”，如下图：![](http://ols4zt49w.bkt.clouddn.com/1498563723%281%29.png)
  保存退出，然后更新一下环境变量，输入命令“`source .bashrc`”
 接着测试一下，编译器路径设置的对不对。如下图，在 Ubuntu 命令行中输入命令"arm",然后按键盘"Tab"，出现编译器“arm-none-linux-gnueabi-gcc-4.3.2”，这就说明编译器路径设置正确。 如下图：
 ![](http://ols4zt49w.bkt.clouddn.com/1498564244%281%29.png)
 
####  Qt/E4.7.1 镜像的编译
先将 Qt/E4.7.1 源码压缩包“qt-everywhere-opensource-src-4.7.1.tar.gz”拷贝到 Ubuntu 的文件夹“root/yizhi”中，没有这个文件夹则可以新建一个。在 Ubuntu 命令行中输入解压命令“`tar -vxf qt-everywhere-opensource-src-4.7.1.tar.gz`”，解压后得到文件夹“qt-everywhere-opensource-src-4.7.1”。
进入“qt-everywhere-opensource-src-4.7.1”文件夹中，执行编译脚本“`./build-all`”,注意这个命令有个点“`.`”如下图：
![](http://ols4zt49w.bkt.clouddn.com/1498564694%281%29.png)
输入回车，如下图所示，开始编译，编译比较耗费时间。
编译完成后，如下图，进入"/opt"目录，可以看到编译生成的“qt-4.7.1”文件夹。
![](http://ols4zt49w.bkt.clouddn.com/1498564714%281%29.png)
进入文件夹“/home/topeet/Linux+QT/root/opt”（目录 topeet 根据用户实际建立的文件夹调整），然后将“qt-4.7.1”文件夹拷贝到该目录下，如下图。红色框中的“qt-4.7.1”文件夹是 Qt/E4.7，蓝色框中的"Qtopia"就是之前编译得到的“Qtopia”文件系统。
![](http://ols4zt49w.bkt.clouddn.com/1498564966%281%29.png)
如下图，进入文件夹“/home/topeet/Linux+QT”中，输入命令“`make_ext4fs -s -l 314572800 -a root -L linux system.img root`”，编译生成二进制文件“system.img”。
![](http://ols4zt49w.bkt.clouddn.com/1498565228.png)
如下图，文件“system.img”就是 Qt/E4.7 的镜像。（注意：编译生成Qt/E4.7 的镜像的时候，文件夹“/home/topeet/Linux+QT/root/opt里不要有qtopia文件系统）
![](http://ols4zt49w.bkt.clouddn.com/1498565428%281%29.png)
其它三个文件和 Qtopia 文件系统对应的镜像一模一样。

  **补充：**
 **生成 system.img的前期工作：**
 生成可以下载的 system.img 文件需要工具“mkimage，这里在客服提供的02_编译器以及烧写工具”→“tools”文件夹下的压缩包“linux_tools.tgz”中。
拷贝压缩包到 Ubuntu 系统的“/”目录下，**注意目录是“/”**。 
进入“/”目录，然后使用命令“`tar -vxf linux_tools.tgz` ”，将压缩包解压。
解压后如下图所示，在“/usr/local/bin/”目录下生成了两个文件。
![](http://ols4zt49w.bkt.clouddn.com/1498566182%281%29.png)
使用命令“`cd /home/topeet/`”进入 topeet 目录，然后使用命令“`mkdir Linux+QT`”新建一个“Linux+QT”文件夹。如下图所示：
![](http://ols4zt49w.bkt.clouddn.com/1498566369%281%29.png)
拷贝root.tar.gz”到新建的“Linux+QT”文件夹，如下图
![](http://ols4zt49w.bkt.clouddn.com/1498566645%281%29.png)
使用命令“`tar -vxf root_20150123.tar.gz`”解压压缩包
解压后会生成文件夹“root”，如下图所示。
![](http://ols4zt49w.bkt.clouddn.com/1498566756%281%29.png)
**在编译生成镜像前先把前面编译生成的文件夹“Qtopia”或qt-4.7.1”文件夹拷贝到root解压出来的“opt”文件夹中**，在执行上面的操作后，最后执行生成二进制文件的命令，在目录“/home/topeet/Linux+QT ”中，使用命令“`make_ext4fs -s -l 314572800 -a root -L linux system.img root`”

**以上为生成system.img的前期工作或者必须了解的。**

#### 系统运行后 Qt/E4.7 和 Qtopia2.2.0 的切换
为了方便切换，只需要将编译 Qtopia 和编译 Qt/E4.7 得到的文件夹同时放到“/home/topeet/Linux+QT/root/opt”目录中，然后再使用“`make_ext4fs -s -l 314572800 -a root -L linux system.img root`”，编译可以得到“system.img”。
##### 设置开发板优先运行的文件系统
源代码编译后，**默认是运行 Qtopia**，下面讲一下如何直接运行 Qt/E4.7。
这里需要修改“root/etc/init.d/rcS”文件。如下图所示，打开“root/etc/init.d/rcS”文件
![](http://ols4zt49w.bkt.clouddn.com/1498565767%281%29.png)
打开文件“rcS”后，进入文件中的最后一行，如果要默认启动“qt-4.7.1”，则将修改为上图中的“qtopia”修改为“qt4”，如下图
所示。注意这里的修改语法和格式一定要和源代码的的一样。
![](http://ols4zt49w.bkt.clouddn.com/1498567183%281%29.png)
修改后，**重新编译生成二进制文件**，系统就会默认运行 Qt/E4.7。

**注意：**Qt/E4.7.1 的u-boot-iTOP-4412.bin、zImage 以及ramdisk-uboot.img 和Qtopia 通用，编译方法也一样。它们的区别是“Qtopia”带有一个桌面系统，“Qt/E4.7.1”只是一个强大的库。关于他们的区别，可以参考我之前的一篇博客：[QT/Qte/Qtopia三者的区别](http://shalian.site/2017/03/16/QT-Qte-Qtopia%E4%B8%89%E8%80%85%E7%9A%84%E5%8C%BA%E5%88%AB/)
##### 系统运行后 Qt/E4.7 和 Qtopia2.2.0 的切换
输入切换命令的时候如果已经打开过一个文件系统，则需要先关闭已启动文件系统的进程。下面举例说明，如何关闭文件系统的进程。
![](http://ols4zt49w.bkt.clouddn.com/1498567538%281%29.png)
![](http://ols4zt49w.bkt.clouddn.com/1498567561%281%29.png)
Qt/E4.7 文件系统启动后，再切换到 Qtopia2.2.0，也是使用和上面类似的方法，这里就不再重复讲解了。
#### 在 iTOP-4412 开发板上运行 helloworld
在ubuntu里进入源码文件夹
![](http://ols4zt49w.bkt.clouddn.com/1498567854%281%29.png)
之前我们编译生成了“/opt/qt-4.7.1/”，这个文件夹包含了移植所需要的**最重要的工具 qmake。**进入“**/opt/qt-4.7.1/bin**”，可以看到 qmake 文件。
![](http://ols4zt49w.bkt.clouddn.com/1498567953%281%29.png)
查看了 Qt/E4.7.1 的 qmake 之后，再进入 helloworld 的源码文件夹，然后，在该文件夹中运行 qmake “`#/opt/qt-4.7.1/bin/qmake`”
![](http://ols4zt49w.bkt.clouddn.com/1498568160%281%29.png)
如下图，多了一个 Makefile 文件。
![](http://ols4zt49w.bkt.clouddn.com/1498568168%281%29.png)
然后，执行编译命令“#make”，如下图所示。
![](http://ols4zt49w.bkt.clouddn.com/1498568292%281%29.png)
生成了“helloworld”
![](http://ols4zt49w.bkt.clouddn.com/1498568375%281%29.png)
然后，我们使用 file 命令测试一下。如下图，“#file helloworld”，可以看到 hellworld应用文件的基本信息，它是属于 ARM 平台的。
![](http://ols4zt49w.bkt.clouddn.com/1498568440%281%29.png)
最后，将此文件U盘或tf卡挂载。在超级终端中，先修改权限“#`chmod 777 helloworld`”，然后再运行“#`./hellworld -qws`”
结果如下：
![](http://ols4zt49w.bkt.clouddn.com/468431670471850727.jpg)
#### linux Qt系统下挂载U盘
![](http://ols4zt49w.bkt.clouddn.com/1498568714%281%29.png)

最后，欢迎志同道合者一起交流~