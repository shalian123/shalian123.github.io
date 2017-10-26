---
title: 逐行读写字符串数组到文本txt文件
date: 2017-06-02 14:31:30
tags: [读写文本,c++]
categories: c/c++

---
在c和c++代码的操作中，我们有时会用到一些文件操作，为了方便以后的调用，我在此总结了一些简单的文件读写操作，并依次展示效果。
#### C语言方式
在代码中，主要用到了以下几个头文件：为了方便，此处将几次代码的头文件汇到了一起，你可以直接复制过去。
```
#include<iostream> //c++的
#include<stdio.h>  //c的
#include<string>
#include<fstream> //c++文件读写操作函数
using namespace std;

```
##### fprintf()函数将字符串数组写入到txt文件中
此处的fprintf()函数：输出函数(格式化输出数据至文件)
```
    FILE *fp;
	errno_t err;
	char * name[] = { "file1", "file2", "file3", "file4", "file5" };
	if((err=fopen_s(&fp,"E://test.txt", "w"))!=0)
	   printf("the file was not opened\n");
       
	for (int i = 0; i < 5; i++)
	{
		fprintf(fp, "%s\n", name[i]);
	}
    
	fclose(fp);
    
```
此处，补充一点知识，在vs里经常遇到：
**fopen的用法是: fp  = fopen(filename, "w")。
而对于fopen_s来说，还得定义另外一个变量errno_t  err，然后err = fopen_s(&fp, filename, "w")。
返回值的话，对于fopen来说，打开文件成功的话返回文件指针（赋值给fp），打开失败则返回NULL值；
对于fopen_s来说，打开文件成功返回0，失败返回非0。**

最终效果如图：
![](http://ols4zt49w.bkt.clouddn.com/51N_X%281B6$EIC$W~ORIV%25%7DT.png)
##### 使用fputs函数逐行写入文本
```
    char *InputStr = "that is ok";
	FILE* fp;
	errno_t err;
	if ((err = fopen_s(&fp, "C://1.txt", "wt")) != 0)
		printf("the file was not opened\n");
	//FILE* fp = fopen("C:\\1.txt", "wt");
	//if (fp != NULL)
	//{
		fseek(fp, 0, SEEK_END);
		fputs(InputStr, fp);
		fputs("\r\n", fp);
		fclose(fp);
	//}

```
最终效果：
![](http://ols4zt49w.bkt.clouddn.com/M_8VG%5BRJM5%7D%5BPA95%7B6%254V%60A.png)
##### 利用fscanf对文件进行格式化输入，读取文件中的数据
```
    int i = 0;
	FILE *fp;
	fopen_s(&fp,"E://test.txt", "r");
	char  name[6][10] = {0};//如果没有初始化，最后没有读取的几位将会乱码。整个字符数组输出，如未初始化，则没有写入确定值的元素是一些不确定值，输出可能就是一堆乱码了。

	while (!feof(fp))
	{
		fscanf_s(fp, "%s", name[i],sizeof(name[i]));
		printf("%s\n", name[i]);
		i++;
	}
	fclose(fp);
```
**注意：整个字符数组输出，如未初始化，则没有写入确定值的元素是一些不确定值，输出可能就是一堆乱码了。**
效果如下：
![](http://ols4zt49w.bkt.clouddn.com/K4FYVZ5$%29OIJ@_%6007%7DJ%60~%5B5.png)
#### C++语言方式
在看具体实例前，我先简要介绍一下：
```
#include <fstream>  
ofstream         //文件写操作 内存写入存储设备   
ifstream         //文件读操作，存储设备读区到内存中  
fstream          //读写操作，对打开的文件可进行读写操作 

```
下面主要列举两个例子，一个是从文件里读取数据，然后控制台输出；另一个是读取一个文件数据，并将数据写入另一个文件，即复制。
##### c++读文件
```
    ifstream in("E://test.txt");
	string filename;
	string line;

	if (in) // 有该文件
	{
		while (getline(in, line)) // line中不包括每行的换行符
		{
			cout << line << endl;
		}
	}
	else // 没有该文件
	{
		cout << "no such file" << endl;
	}

```
效果如下：
![](http://ols4zt49w.bkt.clouddn.com/79%25%25GY~48KKPO%5B3LT%28K%5BO1U.png)
##### c++文件间数据的复制
```
/*实现读取文件file1，写入到文件file2，实现复制*/
void fileCopy(char *file1, char *file2)
{
	// 最好对file1和file2进行判断

	ifstream in(file1);
	ofstream out(file2);
	string filename;
	string line;

	while (getline(in, line))
	{
		out << line << endl;
	}
}
int main()
{
    fileCopy("E://1.txt", "E://2.txt");
	return 0;
}
```
效果如下：
![](http://ols4zt49w.bkt.clouddn.com/ZPH%28KD016GH%25HN%7B%7D~5T%28DJM.png)
**补充**：这里只是简单的文件操作，关于**c++**的文件读写详解可以参考：
**C++**文件读写详解（ofstream,ifstream,fstream） http://blog.csdn.net/kingstar158/article/details/6859379
里面还包含一些二进制文件，缓存与同步等。
**扩展：**
C++的输入输出分为三种：
(1)基于控制台的I/O，头文件iostream
(2)基于文件的I/O, 头文件fstream
(3)基于字符串的I/O,头文件sstream
具体参考博客：http://www.cnblogs.com/gamesky/archive/2013/01/09/2852356.html