---
title: linux下搭建qt编译环境常遇到的问题及解决
date: 2017-06-06 18:49:14
tags: [linux,qt编译环境]
categories: QT

---
简要概括以下步骤：
一、下载QT版本以及gcc

二、解压，安装，编译，这个网上教程很多，我是参考的迅为的开发板使用手册。
**这里重点是补充报错及修复:**
1.没有安装**g++**,  **如果你是一开始安装新的linux系统定会有这样问题**， 自己`apt-get install g++`处理下。

2.没有相关的配置的库支持，可以用`apt-get isntall build-essential` 添加下载库支持

3.运行qt时候会有报错：Qt编译错误“GL/gl.h:No such file or directory或者提示找不到GL等。
**解决**：这是由于系统中没有安装OpenGL库导致的，于是在控制台中输入以下命令安装OpenGL库及其工具：`apt-get install libgl1-mesa-dev libglu1-mesa-dev freeglut3-dev` 即可。

三，注意配置下qt的编译器和qt版本。具体根据自己安装的目录来吧。
（一般在/usr/bin/**g++**）