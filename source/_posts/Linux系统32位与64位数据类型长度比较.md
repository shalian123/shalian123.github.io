---
title: Linux系统32位与64位数据类型长度比较
date: 2017-09-18 11:44:52
tags: [数据类型,linux]
categories: linux

---
**Linux系统32位与64位GCC编译器基本数据类型长度对照表**
**GCC 32位**
sizeof(char)=1
sizeof(double)=8
sizeof(float)=4
sizeof(int)=4
**sizeof(short)=2**
**sizeof(long)=4**
**sizeof(long long)=8**
**sizeof(long double)=12**
**sizeof(complex long double)=24**

**GCC 64位**
sizeof(char)=1
sizeof(double)=8
sizeof(float)=4
sizeof(int)=4
sizeof(short)=2
**sizeof(long)=8**
**sizeof(long long)=8**
**sizeof(long double)=16
sizeof(complex long double)=32**

32位，指针占用4个字节；
64位，指针占用8个字节。 
以下是实测结果：
```
#include <stdio.h>
#include <limits.h>
int main(void)
{
printf("size of char %d\n",sizeof(char));
printf("size of char max %d\n",CHAR_MAX);
printf("size of char min %d\n",CHAR_MIN);
printf("size of int %d\n",sizeof(int));
printf("size of int min %d\n",INT_MIN);
printf("size of int max %d\n",INT_MAX);
printf("size of long %d\n",sizeof(long));
printf("size of long min %ld\n",LONG_MIN);
printf("size of long max %ld\n",LONG_MAX);
printf("size of short %d\n",sizeof(short));
printf("size of short min %d\n",SHRT_MIN);
printf("size of short max %d\n",SHRT_MAX);
printf("size of unsigned char %d\n",sizeof(unsigned char));
printf("size of unsigned char max %d\n",UCHAR_MAX);
printf("size of unsigned long %d\n",sizeof(unsigned long));
printf("size of unsigned long max %lu\n",ULONG_MAX);
printf("size of unsigned short %d\n",sizeof(unsigned short));
printf("size of unsigned short max %u\n",USHRT_MAX);
printf("size of double %d\n",sizeof(double));
printf("size of long long %d\n",sizeof(long long));
printf("size of long double %d\n",sizeof(long double));
printf("size of float %d\n",sizeof(float));
int *p;
printf("size of pointer  %d\n",sizeof(p));
}
```
64 |32位上运行结果：
size of char 1
size of char max 127
size of char min -128
size of int 4
size of int min -2147483648
size of int max 2147483647
size of long 8  |32位是4
size of long min -9223372036854775808
size of long max 9223372036854775807
size of short 2
size of short min -32768
size of short max 32767
size of unsigned char 1
size of unsigned char max 255
size of unsigned long 8 |32位是4
size of unsigned long max 18446744073709551615
size of unsigned short 2
size of unsigned short max 65535
size of double 8
size of long long 8
size of long double 16     ///////32位是12
size of float 4
size of pointer  8  /////////32位是4 
 