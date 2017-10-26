---
title: ARM简介
date: 2017-03-13 15:37:52
categories: "嵌入式"
tags:
---
今天，偶然在网上看到一篇帖子，主要讲解了ARM的发展及趋势，回想起我从本科开始接触单片机，从一开始的裸机，到烧录linux，安卓等系统，板子也接触了好几款了，但一直没有好好整理，借此机会，参考网友的观点及结合自己的一些看法，作此整理。

**ARM简介**
1. ARM只卖知识产权，不卖(物理的，实质的)产品。
2. 全世界100多家公司购买了ARM授权，包括三星,Freescale、NXP Semiconductors、STMicroelectronics、Texas Instruments ,Toshiba,Analog Device，atmel，microsemi...具体参看ARM官网
3. ARM processor family:ARM7,ARM9,ARM11,Cortex-A,Cortex-R,Cortex-M,SecurCore
4. 为了清楚地表达每个ARM应用实例所使用的指令集，ARM公司定义了6种主要的ARM指令集体系结构版本，以版本号V1～V6表示.         
ARM架构(或内核)：ARMv1，ARMv2...到ARMv8,比如ARM9,10架构为ARMv5，ARM11为ARMv6，Cortex系列为ARMv7
**(Cortex系列又包括Cortex-A,Cortex-R,Cortex-M三系列，架构也分别为ARMv7-A,ARMv7-R,ARMv7-M).最小的64位arm架构为ARMv8。。。（细分的ARMv4T，v5E等就不说了）**
                 处理器      内核、
                ARM7       ARMv4
                ARM9,10    ARMv5
				ARM11      ARMv6
				CortexA8   ARMv7   
				64bit      ARMv8
5. ARM7是冯 诺依曼体系结构，ARM9。ARM11等是哈佛体系结构(数据和指令分开存储，分开访问速度更快)。
6. 其他分类
经典 ARM 处理器
ARM11™ 系列 - 基于 ARMv6 架构的高性能处理器
ARM9™ 系列 - 基于 ARMv5 架构的常用处理器
ARM7™ 系列- 面向普通应用的经典处理器 
ARM 专家处理器
SecurCore™ - 面向高安全性应用的处理器。
FPGA Cores - 面向 FPGA 的处理器
ARM Cortex 应用程序处理器
ARM Cortex 嵌入式处理器
7. 授权数：
 经典 ARM 处理器 许可证数 
 ARM11 系列         82
 ARM9 系列           271 
 Cortex 处理器 许可证数 
 Cortex-A          86
 Cortex-R          22 
 Cortex-M          123

8. 经典 ARM 处理器：ARM7，ARM9，ARM11.
**ARM 11 之後分成三类：
Cortex - A/R/M
Cortex - A 系列面向尖端的基于虚拟内存的操作系统和用户应用；
Cortex - R 系列针对实时系统；
Cortex - M 系列对微控制器**

9. ARM7,9，11区别（网友）：
ARM7是冯诺依慢结构 ，
ARM9、ARM11是哈佛结构，所以性能要高一点。
ARM9和ARM11大多带内存管理器，跑操作系统好一点，ARM7适合裸奔。
不跑操作系统，价格低一点的：ARM7、cortex-M3等等。
性价比高，可跑也可不跑操作系统的：ARM9、cortex-Rx等等。
性能高的，通常要跑操作系统的：ARM10、ARM11、Cortex-A8等等。
成熟的：ARM7\ARM9\ARM11。
发展趋势：Cortex-A、Cortex-R、Cortex-M。
其实弄ARM大多还是在嵌入式领域，不过现在很多上网本也开始ARM了**

10. cortex-m3和ARM11区别：
cortex-m3的架构(ARMv7)比ARM11(ARMv6)的版本高，但是cortex—m系列的芯片的应用主要在低端(就相当于一个单片机，不跑OS)，从性能上来说ARM11要比cortex-m3要好不少.

11. 51单片机寄存器比较少，指令只有111条；而arm芯片寄存器较多，指令集也多，要掌握它需要耐心和时间，所以，为了简化嵌入式软件的编程工作量，生产公司把寄存器的操作搞定，封装成函数，这就是固件函数库。
比如意法半导体(ST.COM)的《STM32F10xxx固件函数库.pdf》。

12. **总结**
谈到这，许多人会问两个问题：
1.单片机和ARM到底有什么区别？
2.有无操作系统有什么区别？
**答**：下面我先来谈谈第一个问题，单片机在一定的程度上只能算是微控制器（MCU）,而ARM(此处指高级的ARM,注：低级的cortex-m系列也是MCU)可以称为微处理器。本质上多了MMU(内存管理)，和chche(高速缓存)。
**答**：第二个问题，我想只要将两者都尝试一下就不难发现：
单片机是通过软件直接控制硬件，因此用户想在一个新的硬件平台实现相同的功能，即软件程序会变动，软件开发者必须先了解这个新的硬件平台，才能进行软件编程。因此不难发现它有两个致命缺陷：软件开发者必须懂得硬件，降低了开发效率；软件移植性差。
而嵌入式通过文件系统间接控制硬件，当硬件平台变了，依旧与操作系统兼容，软件程序几乎不许改变，而且软件开发人员不需了解硬件，只需学会在操作系统中功能的调用，极大地提高了效率。相比单片机（无系统），嵌入式引入操作系统，有以下优点：
    软件移植性好
    软件人员不需了解硬件，提高开发效率
    操作系统提供了许多开源软件，库等
    **可以实现多任务**
    操作系统在自带一些网络协议，提供了大量网络资源，便于实现远程控制