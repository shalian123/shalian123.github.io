---
title: 怎么去备份你的hexo博客
date: 2017-10-26 22:44:07
tags: [hexo备份,换电脑]
categories: Hexo教程

---
#### 前言
作为一个写代码的人来说，保存和备份就像是吃饭和喝水一下重要，所以随手保存和存有备份已经成为我的习惯了。使用Hexo在github搭建的博客，仓库里只有生成的静态网页文件，是没有Hexo的源文件的，如果现在这个电脑出现了什么问题，那就麻烦了，所以我就研究了一下怎么备份。备份的教程和搭建的教程比起来简直是少的可怜，好不容易找到一份教程，却是写的非常高深，个人觉得不应该是那么复杂的，所以果断放弃，就在我准备自己瞎搞得时候，终于让我看到了一份简单明了的教程，因为测试成功了，所以我需要把过程记录下来，以后忘了还可以有个参考


作者：程序媛JY
链接：http://www.jianshu.com/p/baab04284923
來源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
#### 正文
针对博客已经搭建并发布过文章的。

1、在你的博客仓库创建一个分支hexo（这个命名随意）；

2、设置hexo为默认分支（不知道的参考http://blog.csdn.net/zwx2445205419/article/details/66970640）；

3、将博客仓库clone至本地，将之前的hexo文件夹中的

_config.yml，themes/，source，scffolds/，package.json，.gitignore复制到你克隆下来的仓库文件夹，即Username.github.io；（Username是你自己的用户名）

4、将themes/next/(我用的是NexT主题)中的.git/删除，否则无法将主题文件夹push；

5、在Username.github.io；文件夹执行npm install，npm install hexo-deployer-git(这里可以看看分支是不是显示为hexo)
![](http://ols4zt49w.bkt.clouddn.com/76YE31%7B%60HJ5C$3%5DYN%29%5DD@CQ.png)

6、执行`git add .`，`git commit -m "提交文件"`，`git push origin hexo`来提交hexo网站源文件；

7、执行hexo g -d 生成静态网页部署到github上。
这样，Username.github.io仓库就有master分支保存静态网页，hexo分支保存源文件。

#### 修改
在本地对博客修改（包括修改主题样式、发布新文章等）后

1、执行`git add .`，`git commit -m "提交文件"`，`git push origin hexo`来提交hexo网站源文件；

2、执行hexo g -d 生成静态网页部署到github上；

（每次发布重复这两步，它们之间没有严格的顺序）
#### 恢复
换电脑想改博客：
1、安装git；
2、安装Nodejs和npm；
3、使用克隆命令将仓库拷贝至本地；
4、在文件夹内执行命令`npm install hexo-cli -g`、`npm install`、`npm install hexo-deployer-git`；
#### 附录
Hexo的源文件说明：

1、_config.yml站点的配置文件，需要拷贝；

2、themes/主题文件夹，需要拷贝；

3、source博客文章的.md文件，需要拷贝；

4、scaffolds/文章的模板，需要拷贝；

5、package.json安装包的名称，需要拷贝；

6、.gitignore限定在push时哪些文件可以忽略，需要拷贝；

7、.git/主题和站点都有，标志这是一个git项目，不需要拷贝；

8、node_modules/是安装包的目录，在执行npm install的时候会重新生成，不需要拷贝；

9、public是hexo g生成的静态网页，不需要拷贝；

10、.deploy_git同上，hexo g也会生成，不需要拷贝；

11、db.json文件，不需要拷贝。

不需要拷贝的文件正是.gitignore中所忽略的。

注：本文主要是为了避免以后换电脑或电脑出问题，具体操作参考了一些博客：
[怎么去备份你的Hexo博客](http://www.jianshu.com/p/baab04284923)

[使用Hexo搭建博客，备份至GitHub过程 ](http://blog.csdn.net/u012195214/article/details/72721065)

[Hexo+github个人博客搭建+异地管理 ](http://blog.csdn.net/zwx2445205419/article/details/66970640)