---
title: QT5.3.2连接Mysql老是失败！
date: 2017-05-18 17:00:13
tags: [	qt,Mysql]
categories: QT

---

之前在QT里一直用的文件型数据库sqlite,今天突然想试试在里面连接Mysql，结果却遇到了一个问题，问题如下：
在程序运行后，显示的驱动里明明有QMYSQL驱动，同时我也成功创建了一个mydata数据库，但是就是显示加载不了驱动。
![](http://ols4zt49w.bkt.clouddn.com/1495098343%281%29.png)
![](http://ols4zt49w.bkt.clouddn.com/1495098462%281%29.png)

网上的解决方案很多，大概有：
1.需要自己编译驱动。（qmysql.dlll,qmysqld.dll已经存在）
2.将mysql目录下lib中的libmysql.dll考到Qt安装目录bin中。
3.将plugins\sqldrivers文件夹拷贝到工程目录及其目标文件中。
结果都是一个效果，无效！！！不能不说网上有的回答并不全面或者说是牛头不对马嘴，也因此坑了我一个下午，最终得已解决，在此为了避免你们遇到跟我一样得问题，我在这里总结一下：

1.从Qt5.2版本开始，已经包含了MySQL驱动，不需要在手动编译了。你可以通过qDebug()<<QSqlDatabase::drivers()查看支持的驱动。


**2.还有一个很重要的是32位的MySQL对应32位的Qt。**

相信你们也发现了，解决方案很简单。由于**我的版本是QT5.3,而且x86的，即32位的**，虽然我的电脑是64位win10系统，但是在**电脑上必须装32位的Mysq**l。相信大家会有疑惑，64位的系统怎么能装32位的软件呢，但是奇葩的是竟然成功装上了，至少 mysql-5.5.27-win32.msi 可以。
接下来再将mysql目录下lib中的libmysql.dll考到Qt安装目录bin中。
在此编译运行，结果如下：
![](http://ols4zt49w.bkt.clouddn.com/1495100021%281%29.png)