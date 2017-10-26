---
title: 测试linux-fifo
categories:  "linux"

---

# linux进程间通信之有名管道fifo
本文通过具体的实例带你深入了解进程通信里的有名管道通信
## 一、硬件工具
一块已经烧录最小系统进去的开发板
pc机
U盘或TF卡或NFS
## 二、软件工具
ubuntu及虚拟机
交叉编译器（此处采用的是Arm-2009q3）
编辑器（此处采用的是notepad）
超级终端
vim编辑器
## 三、读本文前得了解的知识
交叉编译，文件函数的操作（open，write，read等），对linux进程的了解，fork函数 mkfifo函数等。具体还有疑问可以在评论中一起交流。
## 四、有名管道 fifo
有名管道可以实现无亲缘关系的通信（无名管道只能用于有亲缘进程之间的通信）。有名管道实现进程间通信的速度很快，同时遵行先进先出原则。
> man 3 mkfifo

我们可以看到如下图mkfifo函数得用法：
![mkfifo](http://ols4zt49w.bkt.clouddn.com/mkfifo.png)

为了更好的体验有名管道得通信，我们首先编写creatc.c测试mkfifo函数。

    #include <stdio.h>
    #include <unistd.h>
    #include <string.h>
	#include <sys/types.h>
	#include <sys/stat.h>
	#include <fcntl.h>

	void filecopy(FILE *,char *);

	int main(void)
	{
	FILE *fp1;
	long int i = 100000;
	char buf[] = "I want to study Linux!\n";
	char *file1 = "data.txt";
	
	printf("begin!\n");
	
	if((fp1 = fopen(file1,"a+")) == NULL ){ //fopen返回值为FILE的指针
	 //“a+”：Open for reading and appending (writing at end  of  file).
	 //The file is created if it does not exist
			printf("can't open %s\n",file1);
	}
	while(i--)
	filecopy(fp1,buf);//调用 filecopy 将数据写入 file1 文件中

	fclose(fp1);
	
	printf("over!\n");
	
	return 0;
	}

	void filecopy(FILE *ifp,char *buf)
	{
	char c;
	int i,j;
	j = 0;
	i = strlen(buf)-1;	
	while(i--){
		putc(buf[j],ifp);//int putc(int c, FILE *stream);
		                 //writes the character c, cast to an unsigned char, to stream
		j++;
	}
	putc('\n',ifp);
	}

紧接着编写简单的writepipe.c测试对有名管道得写
	#include <unistd.h>  
	#include <stdlib.h>  
	#include <fcntl.h>  
	#include <limits.h>  //PIPE_BUF
	#include <sys/types.h>  
	#include <sys/stat.h>  
	#include <stdio.h>  
	#include <string.h>  
  
	int main()  
	{  
    const char *fifo_name = "my_fifo";
	char *file1 = "data.txt";	
    int pipe_fd = -1;  
    int data_fd = -1;  
    int res = 0;  
    const int open_mode = O_WRONLY;  
    int bytes_sent = 0;  
    char buffer[PIPE_BUF + 1];  //最后\0
  
    if(access(fifo_name, F_OK) == -1)  //检验权限函数，看文件是否存在
    {  
        //管道文件不存在  
        //创建命名管道  
        res = mkfifo(fifo_name, 0777);  
        if(res != 0)  
        {  
            fprintf(stderr, "Could not create fifo %s\n", fifo_name);  
            exit(EXIT_FAILURE);  
        }  
    }  
  
    printf("Process %d opening FIFO O_WRONLY\n", getpid()); 
	
    //以只写阻塞方式打开FIFO文件，  
    pipe_fd = open(fifo_name, open_mode);
	
	//以只读方式打开数据文件  
    data_fd = open(file1, O_RDONLY);  
    printf("Process %d result %d\n", getpid(), pipe_fd);  
  
    if(pipe_fd != -1)  //打开FIFO文件成功
    {  
        int bytes_read = 0;  
        //向数据文件读取数据  
        bytes_read = read(data_fd, buffer, PIPE_BUF);  
        buffer[bytes_read] = '\0';  
        while(bytes_read > 0)  
        {  
            //向FIFO文件写数据  
            res = write(pipe_fd, buffer, bytes_read);  
            if(res == -1)  
            {  
                fprintf(stderr, "Write error on pipe\n");  
                exit(EXIT_FAILURE);  
            }  
            //累加写的字节数，并继续读取数据  
            bytes_sent += res;  
            bytes_read = read(data_fd, buffer, PIPE_BUF);  
            buffer[bytes_read] = '\0';  
        }  
        close(pipe_fd);  
        close(data_fd);  
    }  
    else  
        exit(EXIT_FAILURE);  
  
    printf("Process %d finished\n", getpid());  
    exit(EXIT_SUCCESS);  
	} 


最后编写readpipe.c测试对有名管道得读
	#include <unistd.h>  
	#include <stdlib.h>  
	#include <stdio.h>  
	#include <fcntl.h>  
	#include <sys/types.h>  
	#include <sys/stat.h>  
	#include <limits.h>  
	#include <string.h>  
  
	int main()  
	{  
    const char *fifo_name = "my_fifo";  
    int pipe_fd = -1;  
    int data_fd = -1;  
    int res = 0;  
    int open_mode = O_RDONLY;  
    char buffer[PIPE_BUF + 1];  
    int bytes_read = 0;  
    int bytes_write = 0;  
    //清空缓冲数组  
    memset(buffer, '\0', sizeof(buffer));  
  
    printf("Process %d opening FIFO O_RDONLY\n", getpid()); 
	
    //以只读阻塞方式打开管道文件，注意与fifowrite.c文件中的FIFO同名  
    pipe_fd = open(fifo_name, open_mode);  
    //以只写方式创建保存数据的文件  
    data_fd = open("DataFormFIFO.txt", O_WRONLY|O_CREAT, 0644);  
    printf("Process %d result %d\n",getpid(), pipe_fd);  
  
    if(pipe_fd != -1)  
    {  
        do  
        {  
            //读取FIFO中的数据，并把它保存在文件DataFormFIFO.txt文件中  
            res = read(pipe_fd, buffer, PIPE_BUF);  
            bytes_write = write(data_fd, buffer, res);  
            bytes_read += res;  
        }while(res > 0);  
        close(pipe_fd);  
        close(data_fd);  
    }  
    else  
        exit(EXIT_FAILURE);  
  
    printf("Process %d finished, %d bytes read\n", getpid(), bytes_read);  
    exit(EXIT_SUCCESS);  
	} 

待三个测试文件创建完成，接下来便是编译运行测试：
> arm-none-linux-gnueabi-gcc -o creatc creatc.c -static
> arm-none-linux-gnueabi-gcc -o writepipe writepipe.c -static
> arm-none-linux-gnueabi-gcc -o readpipe readpipe.c -static

这里介绍 U 盘拷贝代码的方法，也可以编译进文件系统或者使用 NFS 文件系统等。
> mount /dev/sda1 /mnt/udisk/
加载U盘
> ./mnt/udisk/creatc  
> 运行完会创建文本文件

我们可以在终端里 ls 一下，会发现如下图多了data.txt文本：
![](http://ols4zt49w.bkt.clouddn.com/%E5%88%9B%E5%BB%BA%E6%96%87%E6%9C%AC.png)
打开data.txt文本，里面是重复的“I want to study Linux!”，如下图
![](http://ols4zt49w.bkt.clouddn.com/BC%5D7DRIOK8VT%7BZ2604YM5%7DB.png)
> 使用命令./mnt/udisk/writepipe &
> 后台运行管道写的程序，将data.txt文本中的数据写入管道

我们可以使用linux命令jobs查看程序是否在运行
![](http://ols4zt49w.bkt.clouddn.com/T%5BL%29P3S73%2961X%5D5%2526Y@7JQ.png)
此时ls,可以看到 my_fifo 管道被创立了。
> 最后，运行time ./mnt/udisk/readpipe
> 运行并计时从管道读数据需要多长时间

![](http://ols4zt49w.bkt.clouddn.com/R9IQRG5MMOVD%5D5%5D9BBYT%60W8.png)
如上图，共花费0.26s，jobs一下，程序早已运行结束。
> ls -la


![](http://ols4zt49w.bkt.clouddn.com/1DN8%7BJLX@D%7BQR$%283OLZ6EP2.png)
可见整个数据量接近2.3M，传输速度接近10M/S.
综上，有名管道 fifo 给文件系统提供一个路径，这个路径和管道关联，只要知道这个管道路径，
就可以进行文件访问。且传输速度很快。
## 五、注意点
access(const char *pathname,int mode),检测权限函数，具体用法可man access查询一下。
PIPE_BUF,在linux中，它的大小是4096bytes,可man 7 pipe 查询。
