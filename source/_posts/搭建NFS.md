title: 搭建NFS及注意点
date: 2017-03-03 15:39:14
categories: "linux"
tags:
---
NFS 是 Network File System 的缩写，是由 SUN 公司研制的 UNIX 表示层协议（pressentation layer protocol），NFS 是基于 UDP/IP 协议的应用。它的最大功能就是可以通过网络让不同的机器，不通的操作系统彼此共享文件，可以通过 NFS 挂载远程主机的目录，访问该目录就像访问本地目录一样，所以也可以简单的将它看做一个文件服务器。通过 NFS 服务，我们可以实现在线调试文件系统或应用程序，而不用像传统的方式生成文件系统镜像，然后烧写到板子的 eMMC 里，在启动开发板。通过 NFS 服务可以提高我们的调试效率。
   
_ _ _

如果是 Android 或者 Qt 的文件系统就太大了，启动速度慢，这么做也就没有什么意义了。但是最小 linux 文件相对其它操作系统小很多，所以使用网络文件系统，可以方便的用户调试。

_ _ _
### 一、服务器端的搭建
实现 NFS，我们需要一个主机作为 NFS 服务器，可以选择我们的虚拟机 Ubuntu 作为主机。首先我们需要在虚拟机的 Ubuntu 上安装Ubuntu NFS 服务，这是一个软件包，我们可以使用 apt 命令下载。
>  apt-get install nfs-kernel-server
>  安装 Ubuntu NFS 服务

接下来需要配置/etc/exports
> vi /etc/exports

在/etc/export 文件的最后一行添加：
/home/minilinux/system/ *(rw,sync,no_root_squash)，如下图
![](http://ols4zt49w.bkt.clouddn.com/~DGB@XPKTOPS@V%7D$%29VRQ7%7B7.png)
接下来重启 portmap 服务
> /etc/init.d/portmap restart


然后重启 nfs 服务
> /etc/init.d/nfs-kernel-server restart

至此，NFS服务器端搭建完成。
### 二、将板中的最小系统挂载到NFS
1.先把 linux 最小文件系统放到虚拟机 Ubuntu 的挂载目录下（此处我采用的是/home/minilinux/system） 目录下。（**注意：此处的文件系统必须为板子里烧录的最小系统。**）
2.实现 nfs 文件系统我们需要修改 linux 最小文件系统的 etc/init.d/ifconfig-eth0 文件，
vi命令打开并修改为如下图结果：
![](http://ols4zt49w.bkt.clouddn.com/9%284D@WU3H%29D6VDL5FK~%5BJTP.png)
改完保存并退出！
#### 3.配置内核
此处我们需要用到内核源码，打开内核源码，输入 cp config_for_linux .config 命令生成支持 linux 最小文件系统的内核配
置文件。（**注意此处 .config前有一个空格**）。然后输入 make menuconfig 命令进入 linux 配置界面。具体配置网上很多，此处只挑重点说：
在Boot options 配置界面里，我们需要在 Default kernel command 里面输入：
> root=/dev/nfs rw nfsroot=192.168.1.103:/home/minilinux/system
ip=192.168.1.230:192.168.1.103:192.168.1.1:255.255.255.0:iTOP:eth0:off
rootfstype=ext4 init=/linuxrc console=ttySAC2,115200

**
第一句表示挂载的nfs服务器 ip是192.168.1.103，挂载的目录是/home/minilinux/system， （注意：/home/minilinux/是前面我们搭建 nfs 服务器设置的）。
第二句里第一个 ip192.168.1.230 是我们开发板的 ip 地址，第二个 ip192.168.1.103 是 nfd 服务器的 ip，第三个ip192.168.1.1 是开发板的网关，255.255.255.0 是子网掩码，iTOP 是开发主机的名字（一般无关紧要，可以随便填写），eth0 是网卡设备的名称。
**
#### 4.待配置完内核后，重新编译内核
> make zImage

5.将新生成的内核烧写到开发板，然后重启开发板，就可以使用 NFS 文件系统了。
### 三、注意点
1.这里编译zImage时，要注意用的是前面我们做的最小linux系统（板子里烧录的是我们对应编译生成的系统镜像，这两者得对应）,通过NFS挂载它。
2.在测试行x86 的服务器与板子上的客户端是否连通时，要将虚拟机的网络配置改为桥接模式。（之前我没有路由器，因此将它设置成了自定义模式中的VMnet8（NAT）,在调试网络通信socket及tftp文件传输和NFS时都应将它改为桥接模式。）
3.在NFS调试时，因为路由器网络设置成了动态ip，所以有时ubuntu重启会导致它的ip变了，此时会导致板子启动不了（一直卡在启动内核那），这时我们有两种解决方法：一、将ubuntu设置成静态ip   二、将配置内核里的ip改了，重新编译zImage，然后重新烧写到板中。