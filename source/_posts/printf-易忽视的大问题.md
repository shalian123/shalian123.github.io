---
title: printf 易忽视的大问题
date: 2017-09-20 21:37:01
tags: [c/c++]
categories: c/c++

---
  今天看到了  printf 的一些知识  感觉挺好  下面 我就来吐槽一下 ： 《该博文以下内容的 测试平台及环境 win  64bit  VS2012》
 http://www.cnblogs.com/hnrainll/archive/2011/08/05/2128496.html      //从printf谈可变参数函数的实现

首先，使用 uint32_t  别忘了加头文件 ` #include <stdint.h>`
```
int64_t a = 1;
    printf("%d\n", a);         
    //输出结果 ： 1  果然是1！但是你会不会以为是 a 首先被自动转化成了 int 类型，然后输入为 1的呢？
    //如果真这么简单，本文到此也该结束了。我们换一个写法：  

    int64_t b = 1;
    int c = 2;
    printf("%d, %d\n", b, c);   
    //这次的结果是多少呢？1 和 2？真的吗？我们来看一下结果：
    //输出结果： 1, 0 
    //好吧，你可能该惊讶了。然而这个结果的确是对的。
    //如果你还是觉得不可相信，我们再来看一个代码：
    /*
    这就涉及到 printf 的设计了，printf的第一个参数永远是字符串，他会解析每一个类似 %d 的结构，然后对指针做对应长度的偏移，如%d是4，%lld就是8。（为什么要偏移，请参看这里 从printf谈可变参数函数的实现）
    实际上两个%d分别取得是 a 的低4字节和高4字节，从而分别是1和0（这里还涉及到大小端的问题，本机是小端存储）。
    */

    uint32_t uin0 = 1;
    printf("uin0 = %llu\n", uin0);
    //原文输出一个很大的随机数  我现在的平台与环境是正常输出 1

    uint32_t uin = 1;
    uint32_t uin2 = 2;
    printf("%llu\n", uin, uin2);

    uint64_t uin3 = uin2;
    uin3 = uin3 << 32;
    uin3 += uin;
    printf("%llu\n",uin3);
    //uin2比uin先入栈，所以uin2会在高位，uin会在低位。
    //如果按照我们所解释的，那两个结果应该完全一致，对不对？我们来看一下输出： 
    // 8589934593
    // 8589934593
```
```
llu 64位无符号
d，lx，ld，，lu，这几个都是输出32位的
hd，hx，hu，这几个都是输出16位数据的，
hhd，hhx，hhu，这几个都是输出8位的，
lld，ll，llu，llx，这几个都是输出64位的，

printf（ "%llu ",.....）
%llu   是64位无符号
%llx才是64位16进制数
 

Dev-C++下基本数据类型学习小结

环境: Dev-C++ 4.9.6.0 (gcc/mingw32), 使用-Wall编译选项

基本类型包括字节型（char）、整型（int）和浮点型（float/double）。

定义基本类型变量时，可以使用符号属性signed、unsigned（对于char、int），和长度属性short、long（对

于int、double）对变量的取值区间和精度进行说明。

下面列举了Dev-C++下基本类型所占位数和取值范围：

符号属性     长度属性     基本型     　　所占位数     取值范围       　　　　输入符举例      输出符举例

--            --          char         8         -2^7 ~ 2^7-1        %c          %c、%d、%u

signed        --          char         8         -2^7 ~ 2^7-1        %c          %c、%d、%u

unsigned      --          char         8         0 ~ 2^8-1           %c          %c、%d、%u

[signed]      short       [int]        16        -2^15 ~ 2^15-1              %hd

unsigned      short       [int]        16        0 ~ 2^16-1             %hu、%ho、%hx

[signed]      --           int         32        -2^31 ~ 2^31-1              %d

unsigned      --          [int]        32        0 ~ 2^32-1              %u、%o、%x

[signed]      long        [int]        32        -2^31 ~ 2^31-1              %ld

unsigned      long        [int]        32        0 ~ 2^32-1             %lu、%lo、%lx

[signed]      long long   [int]        64        -2^63 ~ 2^63-1             %I64d

unsigned      long long   [int]        64        0 ~ 2^64-1          %I64u、%I64o、%I64x

--            --          float        32       +/- 3.40282e+038         %f、%e、%g

--            --          double       64       +/- 1.79769e+308  %lf、%le、%lg   %f、%e、%g

--            long        double       96       +/- 1.79769e+308        %Lf、%Le、%Lg

几点说明：

1. 注意! 表中的每一行，代表一种基本类型。“[]”代表可省略。

例如：char、signed char、unsigned char是三种互不相同的类型；

int、short、long也是三种互不相同的类型。

可以使用C++的函数重载特性进行验证，如:

void Func(char ch) {}

void Func(signed char ch) {}

void Func(unsigned char ch) {}

是三个不同的函数。

2. char/signed char/unsigned char型数据长度为1字节；

char为有符号型，但与signed char是不同的类型。

注意! 并不是所有编译器都这样处理，char型数据长度不一定为1字节，char也不一定为有符号型。

3. 将char/signed char转换为int时，会对最高符号位1进行扩展，从而造成运算问题。

所以,如果要处理的数据中存在字节值大于127的情况，使用unsigned char较为妥当。

程序中若涉及位运算，也应该使用unsigned型变量。

4. char/signed char/unsigned char输出时，使用格式符%c（按字符方式）；

或使用%d、%u、%x/%X、%o，按整数方式输出；

输入时，应使用%c，若使用整数方式，Dev-C++会给出警告，不建议这样使用。

5. int的长度，是16位还是32位，与编译器字长有关。

16位编译器（如TC使用的编译器）下，int为16位；32位编译器（如VC使用的编译器cl.exe）下，int为32

位。

6. 整型数据可以使用%d（有符号10进制）、%o（无符号8进制）或%x/%X（无符号16进制）方式输入输出。

而格式符%u，表示unsigned，即无符号10进制方式。

7. 整型前缀h表示short，l表示long。

输入输出short/unsigned short时，不建议直接使用int的格式符%d/%u等，要加前缀h。

这个习惯性错误，来源于TC。TC下，int的长度和默认符号属性，都与short一致，

于是就把这两种类型当成是相同的，都用int方式进行输入输出。

8. 关于long long类型的输入输出：

"%lld"和"%llu"是linux下gcc/g++用于long long int类型(64 bits)输入输出的格式符。

而"%I64d"和"%I64u"则是Microsoft VC++库里用于输入输出__int64类型的格式说明。

Dev-C++使用的编译器是Mingw32，Mingw32是x86-win32 gcc子项目之一，编译器核心还是linux下的gcc。

进行函数参数类型检查的是在编译阶段，gcc编译器对格式字符串进行检查，显然它不认得"%I64d"，

所以将给出警告“unknown conversion type character `I' in format”。对于"%lld"和"%llu"，gcc理

所当然地接受了。

Mingw32在编译期间使用gcc的规则检查语法，在连接和运行时使用的却是Microsoft库。

这个库里的printf和scanf函数当然不认识linux gcc下"%lld"和"%llu"，但对"%I64d"和"%I64u"，它则是

乐意接受，并能正常工作的。

9. 浮点型数据输入时可使用%f、%e/%E或%g/%G，scanf会根据输入数据形式，自动处理。

输出时可使用%f（普通方式）、%e/%E（指数方式）或%g/%G（自动选择）。

10. 浮点参数压栈的规则：float(4 字节)类型扩展成double(8 字节)入栈。

所以在输入时，需要区分float(%f)与double(%lf)，而在输出时，用%f即可。

printf函数将按照double型的规则对压入堆栈的float(已扩展成double)和double型数据进行输出。

如果在输出时指定%lf格式符，gcc/mingw32编译器将给出一个警告。

11. Dev-C++(gcc/mingw32)可以选择float的长度，是否与double一致。

12. 前缀L表示long（double）。

虽然long double比double长4个字节，但是表示的数值范围却是一样的。

long double类型的长度、精度及表示范围与所使用的编译器、操作系统等有关。
```