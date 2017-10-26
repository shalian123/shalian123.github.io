---
title: QT/Qte/Qtopia三者的区别
date: 2017-03-16 11:44:33
categories: QT
tags:

---
### Qt/Qte/Qtopia三者的区别
##### Qt
**泛指 Qt software的所有版本的图像界面库**,包括 Qt/X11（Unix/Linux),Qt Windows, Qt Mac 等，但这只是相对于二进制来说的。Qt作为一个**跨平台**的应用程序和GUI 框架(C++图形用户界面应用程序开发框架)，在源码上对所有平台都是一致的。Unix/Linux上最流行的桌面环境之一KDE就是采用Qt来写的。


(Nokia 在2008年收购 Trolltech 后，将 Qt 更名为 Qt Software，随后改为 Qt Development Frameworks。而后 Nokia 开发了 IDE 工具 **Qt Creator**，于是Qt FrameWork + Qt Creator = Qt SDK。)


##### Qte：Qt/Embeded for linux
**它是用于嵌入式 Linux 系统的 Qt 版本**，**也是一套界面库**,Qt/Embeded 也简称 Qte 或 Qt/E，Qte 去掉了 X Lib 的依赖而直接工作在 Framebuffer 上，而且Qte在此基础上实现了自己的窗口管理系统QWS（Qt Windows System),这是Qte与Qt/X11最大的区别。因此Qte可以在嵌入式Linux系统中没有X11库的环境下构建独立的图形用户界面，而且不会占用太多的嵌入式系统资源。


Qte为方便嵌入式Qt应用的开发，还提供qvfb工具和makeqpf工具。qvfb工具可以实现Qte的应用能在PC上进行调试和测试，避开X11库的干扰。makeqpf工具则是用来制作qpf字体文件，用来在嵌入式界面中显示特殊渲染字体。


##### Qtopia
Qtopia 是一个**基于 Qte 的类似桌面系统的应用环境**,同时又为开发者为嵌入式设备编写程序提供了一套面向对象的API，包含有 PDA 版本和 Phone 版本。**请注意是基于Qte 的应用环境**,Qtopia 是用 Qte 这个库开发出来的应用程序，**实际上Qtopia就相当于是嵌入式设备上的桌面环境**，也就是类似于PC上的KDE，提供有自己的窗口管理、控制等GUI接口，简化了其上Qte应用的开发。就算不使用Qtopia也可以使用Qte创建自己的图形界面。


Qtopia早期是一个sf.net上的开源项目，构建于Qte之上（*在QT4版本前要安装Qtopia需要先装Qt/E，4之后的Qtopia 已经带有QT/E库了*）。从Qt4.1开始，**Qt/Embedded改名为Qtopia Core**，又**从Qt4.4.1开始，Qtopia Core又改名为Qt for Embedded Linux，就是现在的Qte（eveywhere）**。


Qtopia Platform
Qtopia平台由Qt/E, libqpe, libqtopia1, qtopiapim这些库和Qtopia server/launcher组成。应用开发者通过使用这些库提供的API来为Qtopia设备开发应用程序。Qtopia server/launcher作为主程序负责窗口系统的控制、进程间的通讯、启动所有的应用及其它的任务。


Qtopia/Qte的版本
Qtopia1.7.0 / Qte 2.3.7
Qtopia2.1.1 / Qte 2.3.10
Qtopia2.1.2 / Qte 2.3.11
Qtopia2.2.0 / Qte 2.3.12 (包含在qtopia2.2源码包中，2005年，最后一个免费的版本)
qtopia 2的应用基于qte 2.3.x的，qtopia 4的应用基于qtopia core(相当于原来的qte) 4.x。

##### 总结
所以总的来说，QT也就三种：面向桌面的x11、面向嵌入式的Qt/E、以及面向嵌入式带各种应用程序的Qtopia桌面系统！！！

注：本博客参考了一些网友的看法，最终加上了自己的理解与总结等。参考链接如下：
http://m.blog.csdn.net/article/details?id=50577358