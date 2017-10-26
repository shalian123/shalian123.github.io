---
title: 搭建web服务器并控制外设
date: 2017-03-06 20:43:03
categories: "嵌入式"
tags:
---
本文主要分为两部分：web服务器的搭建、实现web对外设的控制
### 一、web服务器的搭建
我们需要用到boa，boa 是一个小型的 web 服务器，可执行代码只有约 60KB，可以用于多种平台，它一个单任务 web 服务器，只能依次完成用户的请求，在嵌入式中比较常见。boa 的官方网站为www.boa.org，可以在上面下载最新版本的 boa，这里我们使用的是 boa-0.94.13.tar.gz。
#### 第一步：boa的拷贝解压

-我们需要先将 boa-0.94.13.tar.gz kao'dao拷到我们的ubuntu系统中。（可以使用ssh）
-解压 tar -vxf boa-0.94.13.tar.gz
-cd boa-0.94.13
#### 第二步：boa生成Makefile编译文件
cd src
运行#./configure
当前目录下会生成Makefile编译文件 ，如下图：
![](http://ols4zt49w.bkt.clouddn.com/@4ETKSJIK67%60XU%7D7SV8I$%25E.png)
#### 第三步：修改Makefile里面的两个参数

-vi Makefile（或者#vim Makefile）
-查找到有CC = gcc的行，在31行左右（可以使用vi编辑器的查找命令#/CC = gcc），替换为CC = arm-none-linux-gnueabi-gcc -static	即修改编译器
-查找到有CPP = gcc -E的行，在32行左右，替换为CPP = arm-none-linux-gnueabi-gcc -E -static
-保存退出
-输入make 开始编译
#### 第四步：编译过程中会出现错误
-vi compat.h
-查找到有#define TIMEZONE_OFFSET(foo) foo##->tm_gmtoff的行，在123行左右，替换为#define TIMEZONE_OFFSET(foo) foo->tm_gmtoff（即把#去掉）。
-保存退出
-继续编译
#### 第五步：生成boa文件，给boa瘦身
-ls查看boa是否生成
-ll boa查看其大小
-使用瘦身命令#arm-none-linux-gnueabi-strip boa
#### 第六步：NFS系统文件处理
-拷贝编译生成的boa到NFS文件系统的bin目录下（这里根据自己实际设置的共享目录，具体参考我之前的文章）
-在NFS文件系统的etc目录下建立boa文件夹
-在NFS文件系统的目录下建立www文件夹
-在NFS文件系统的www目录下面建立文件夹cgi-bin目录

#### 第七步：添加配置文件和用户组
-拷贝boa-0.94.13目录下面默认的boa.conf到NFS文件系统的etc/boa中
-拷贝虚拟机下面/etc目录下的mime.types到NFS文件系统的etc目录下面（注意是主目录/etc）
#### 第八步：用户组，如果使用的是QT裁剪过来的则不需要执行这一步
-在NFS文件系统的etc目录下用命令#vi group命令建立group文件
-在group文件中添加 root:*:0:
#### 第九步：修改配置文件boa.conf
-进到NFS文件系统的etc/boa目录，打开boa.conf文件#vi boa.conf
-查找到有Group nogroup的行，在50行左右，替换为Group root
-查找到有#ServerName www.your.org.here的行，在95行左右，替换为ServerName www.your.org.here
-查找到有DocumentRoot /var/www的行，在115行左右，替换为DocumentRoot /www（注意：这里的“/www”就是前面步骤我们使用mkdir创建的www目录）
-查找到有ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/的行，在194行左右，替换为ScriptAlias /cgi-bin/ /www/cgi-bin/ (此处的目录记住，后面需要跟它一致)
#### 第十步：添加自动运行脚本，建立网页
-打开etc/init.d/rcS文件，定位到最后一行
-添加boa &到自动启动，保存退出，如图：
![](http://ols4zt49w.bkt.clouddn.com/F_%60RA%7DP%5B4%5DK1P40A%5BFD3FEU.png)
-进入到前面创建的www目录
-vi index.html命令建立index.html
-将我们的网页代码拷贝进index.html（网页代码可以自己写或改，也可以到网上找）
![](http://ols4zt49w.bkt.clouddn.com/$4X%7B5EMT5@I3Y~D19A0%7DYHS.png)
#### 测试
至此，基于 boa 的 web 服务器就搭建完成了，上面我们创建的 index.html 是一个简单的 wen 网页，用于我们测试。现在启动我们的开发板（开发板是挂载 NFS 网络文件系统），开发板起来以后我们输入 ps 命令，可以看到 boa 程序在运行，如下图
![](http://ols4zt49w.bkt.clouddn.com/UHG~HC6TS%25101U4C%28ADA%29@R.png)
-然后使用任意内网设备（注意在同一个网段，一般连在同一个路由），登录http://192.168.1.230 默认地址（此地址为开发板系统的ip），如果修改过则需要使用修改过的地址。结果如下：
![](http://ols4zt49w.bkt.clouddn.com/_PU_%299T%7DS5MAJQ%7BZP1K4E6N.png)

### 二、用web实现对外设控制
本文我们主要实现对开发板上的led和蜂鸣器实现开与关。我们需要了解CGI编程，并能对字符设备实现控制（至少知道对应设备节点目录和ioctl函数）
**CGI编程**
CGI（Common Gateway Interface）是外部应用扩展，应用程序与www服务器交互的一个标准接口。按照CGI标准编写的外部扩展应用程序可以处理客户端浏览器输入的数据，从而完成客户端与服务器的交互操作。而CGI规范就定义了web服务器如何向扩展应用程序发送消息，在收到扩展应用程序的信息后又如何进行处理等内容。通过CGI可以提供许多静态的HTM L网页无法实现的功能。比如搜索引擎、基于web的数据库访问等等。


#### 第一步：替换index.html代码
-进入之前建立的www目录中，打开index.html
-拷贝新的程序代码到index.html中
注意：上面输入的是HTML格式的代码，主要是用到了通过表单向服务器提交信息，在表单里面指定了服务器端处理接收到信息的CGI程序是myled.cgi，这是在form表单的属性里设置的，代码是“form action="/cgi-bin/myled.cgi" method="get”，使用的传递数据的方式是get方法，如下图：
![](http://ols4zt49w.bkt.clouddn.com/34OLZLI3%7B6%5DM6%280UW7CL%5BOY.png)

#### 第二步：CGI程序
-打开在etc/boa中的boa.conf文件
-查找ScriptAlias /cgi-bin/ /www/cgi-bin/，这是指定的CGI程序的存储目录
#### 第三步：myled.c
-进入www/cgi-bin目录（**注意：跟etc/boa中的boa.conf文件中设置的CGI程序的存储目录一致**）
-将myled.c拷贝进去（或创建myled.c,把代码拷进去也行）
-编译#arm-none-linux-gnueabi-gcc -o myled.cgi myled.c -static (**注意：myled.cgi跟index.html中form表单里设置一致**)
-修改权限#chmod 777 myled.cgi
此处附上本人移植buzzer（蜂鸣器）后的myled.c：
> 	#include <stdio.h>
	#include <stdlib.h>
    int main()
	{
	 char *data;
     int leds[3] = {0, 0, 0};
     long m, n;
	int exit=0,i,fd,fd1;
	printf("Content-Type:text/html;charset=gb2312\n\n");
	printf("<html>\n"); 
	printf("<body>\n");
	printf("<title>iTOP-4412</title> ");
	printf("<h3>iTOP-4412</h3> ");
	data = getenv("QUERY_STRING");
	printf("<p>receive data:%s</p>",data);
	while(*data != '\0')
	{
		if(*data=='=')
		switch(*(data+1))
		{
			case '1':leds[0]=1;break;
			case '2':leds[1]=1;break;
			case '3':leds[2]=1;break;
			default:exit=1;break;
		}
		if(exit == 1)
			break;
		data++;
	}
	fd=open("/dev/leds",0);
    fd1=open("/dev/buzzer_ctl",0);
	for(i=0;i<2;i++)
	{
		if(leds[i]==1)
			printf("<p>%d\t</p>",i+1);
		ioctl(fd,leds[i],i);
	}
      if(leds[2]==1) //buzzer
		{
			printf("<p>%d\t</p>",3);
		    ioctl(fd1,leds[2],2);
		}
	  if(leds[2]!=1)
		{
		    ioctl(fd1,0,2);
		}
	printf("</body>\n");
	printf("</html>\n");
	return 0;
	}



#### 第四步：启动开发板，然后打开 PC 的浏览器（连内网的手机也行），输入开发板的 ip 地址
结果如下：
![](http://ols4zt49w.bkt.clouddn.com/T%28XCF%60M%7B@O$76%7BSI@8%29%5BHK7.png)
我们可以通过勾选右边的复选框，按提交实现控制led和蜂鸣器的开关。
##### 最后，欢迎志同道合网友一起交流学习！



