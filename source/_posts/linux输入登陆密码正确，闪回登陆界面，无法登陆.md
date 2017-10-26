---
title: linux输入登陆密码正确，闪回登陆界面，无法登陆
date: 2017-06-03 20:13:25
tags: [linux,无法登陆]
categories: linux

---
最近遇到一个头疼的问题，在登陆linux系统时输入密码正确，但却老是闪回登陆界面，无法登陆。我参考了网上的方法，却仍然发现走了一些弯路，主要网上有的解释太过简略或者有点局限，甚至还是错的。在此，我结合我解决的过程，主要做个总结。
##### 产生原因
问题的原因：主目录下的.Xauthority文件拥有者变成了root，从而以用户登陆的时候无法都取.Xauthority文件 。
**注意：这里的主目录是你登陆的用户，不是root用户**
**说明：**Xauthority，是**startx**脚本记录文件。Xserver启动时，读文件~/.Xauthority,读入对应其display 的记录。当一个需要显示的客户程序启动调用XOpenDisplay()也读这个文 件，并把找到的magic code 发送给 Xserver。当Xserver验证这个magic code正确以后，就同意连接啦。观察startx脚本也可以看到，每次startx 运行，都在调用xinit以前使用了xauth的add命令添加了一个新的记录到~/.Xauthority，用来这次运行X使用认证。(**注意：一般sudo startx后就这样不能登陆了**)
##### 解决方法
1.在需要输入密码的登陆界面，按下ctrl+alt+F1组合键，进入tty1终端
![](http://ols4zt49w.bkt.clouddn.com/1496493167%281%29.png)
![](http://ols4zt49w.bkt.clouddn.com/1496492971%281%29.png)
2.输入登陆ubuntu时的用户名和密码，在tty1终端出现登陆成功的提示，这里我的用户名是topeet，默认进入当前用户主目录（我的是/home/topeet）,可以用pwd命令查看，若不是，则切换至：
cd  或者
cd /home/topeet/ 注意：我这里的用户名是topeet,根据自己的实际情况更改。
![](http://ols4zt49w.bkt.clouddn.com/1496493023%281.png)
3.执行命令，将Xauthority的拥有者改为用户topeet
$ sudo chown topeet:topeet .Xauthority
备注：本人的用户名为topeet，因此 chown后面跟了topeet:topeet若为其他用户名修改成相应的名称即可）
(我的用户和用户组都是topeet,如果你不知道可以用` more /etc/group `及`more /etc/passwd`查看)
![](http://ols4zt49w.bkt.clouddn.com/1496493060%281.png)
4.然后在终端提示符后输入：$ ls .Xauthority -l
正确无误后显示如下：-rw------- 1 topeet topeet 51  May 22 23:14 .Xauthority
此时拥有者已经变为用户。
![](http://ols4zt49w.bkt.clouddn.com/1496493097%281.png)
5.按下ctrl + alt + F7切换回图形登陆界面登陆即可。
**易错点：**
一开始我是切换到root用户下修改.Xauthority文件的拥有者的，但是试了几次都没成功，在次特别强调按下ctrl+alt+F1组合键，进入tty1终端后登入你想登入的账户，而不是root用户。