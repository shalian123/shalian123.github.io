---
title: linux并发服务器
date: 2017-04-10 11:53:19
tags: [linux,socket,tcp,fock]
categories: linux

---

我们知道Linux的UDP是面向无连接的，但是TCP是面向连接，所以UDP服务器可以并发处理，但是TCP由于是面向连接，一次通信只能和一个客户端相连，所以要想实现并发处理，可以使用fork()创建子进程来实现，编程模式如下：

服务端：

```
#include <stdlib.h> 
#include <stdio.h> 
#include <errno.h> 
#include <string.h> 
#include <netdb.h> 
#include <sys/types.h> 
#include <netinet/in.h> 
#include <sys/socket.h> 
#define MY_PORT 3333
int main(int argc ,char **argv)
{
 int listen_fd,accept_fd;
 struct sockaddr_in     client_addr;
 int n;
 int sin_size;
 
 if((listen_fd=socket(AF_INET,SOCK_STREAM,0))<0)
  {
        printf("Socket Error:%s/n/a",strerror(errno));
        exit(1);
  }
 
 bzero(&client_addr,sizeof(struct sockaddr_in));
 client_addr.sin_family=AF_INET;
 client_addr.sin_port=htons(MY_PORT);
 client_addr.sin_addr.s_addr=htonl(INADDR_ANY);
 n=1;
 /* 如果服务器终止后,服务器可以第二次快速启动而不用等待一段时间  */
 setsockopt(listen_fd,SOL_SOCKET,SO_REUSEADDR,&n,sizeof(int));
 if(bind(listen_fd,(struct sockaddr *)&client_addr,sizeof(client_addr))<0)
  {
        printf("Bind Error:%s/n/a",strerror(errno));
        exit(1);
  }
  listen(listen_fd,5);
  sin_size=sizeof(struct sockaddr_in);
  while(1)
  {
    accept_fd=accept(listen_fd,(struct sockaddr*)&client_addr,(struct socklen_t*)&sin_size);
    if((accept_fd<0)&&(errno==EINTR))
     continue;
    else if(accept_fd<0)
  {
   printf("Accept Error:%s/n/a",strerror(errno));
   continue;
  }
   if((n=fork())==0)
    {
   /* 子进程处理客户端的连接 */
   char buffer[1024];
   printf("get connect form %s/n",inet_ntoa(client_addr.sin_addr));
  //打印出连接的客户端IP地址，
   n=read(accept_fd,buffer,1024);
  // write(accept_fd,buffer,n);
   buffer[n]='/0';
 //  close(listen_fd);
   close(accept_fd);
   printf("server revieve %s",buffer);
   exit(0);
    }
    else if(n<0)
   printf("Fork Error:%s/n/a",strerror(errno));
    close(accept_fd);
  }
} 
```

客户端：
```
#include <stdlib.h> 
#include <stdio.h> 
#include <errno.h> 
#include <string.h> 
#include <netdb.h> 
#include <sys/types.h> 
#include <netinet/in.h> 
#include <sys/socket.h> 
#define portnumber 3333
int main(int argc, char *argv[]) 
{ 
 int sockfd; 
 char buffer[1024]; 
 struct sockaddr_in server_addr; 
 struct hostent *host; 
        /* 使用hostname查询host 名字 */
 if(argc!=2) 
 { 
  fprintf(stderr,"Usage:%s hostname /a/n",argv[0]); 
  exit(1); 
 } 
 if((host=gethostbyname(argv[1]))==NULL) 
 { 
  fprintf(stderr,"Gethostname error/n"); 
  exit(1); 
 } 


 /* 客户程序填充服务端的资料 */ 
 bzero(&server_addr,sizeof(server_addr)); // 初始化,置0
 server_addr.sin_family=AF_INET;          // IPV4
 server_addr.sin_port=htons(portnumber);  // (将本机器上的short数据转化为网络上的short数据)端口号
 server_addr.sin_addr=*((struct in_addr *)host->h_addr); // IP地址
 /* 客户程序发起连接请求 */ 
  
 while(1)
 { 
  /* 客户程序开始建立 sockfd描述符 */ 
  if((sockfd=socket(AF_INET,SOCK_STREAM,0))==-1) // AF_INET:Internet;SOCK_STREAM:TCP
  { 
   fprintf(stderr,"Socket Error:%s/a/n",strerror(errno)); 
   exit(1); 
  } 
  if(connect(sockfd,(struct sockaddr *)(&server_addr),sizeof(struct sockaddr))==-1) 
  { 
   fprintf(stderr,"Connect Error:%s/a/n",strerror(errno)); 
   exit(1); 
  } 
  /* 连接成功了 */ 
  printf("Please input char:/n");
  
  /* 发送数据 */
  fgets(buffer,1024,stdin); 
  write(sockfd,buffer,strlen(buffer)); 
  /* 结束通讯 */ 
 
  close(sockfd); 
 }
 exit(0); 

} 
```
这样服务端可以实现同时监听多个客户端，和UDP的作用效果一样，这仅仅是监听一个端口，如果想同时监听多个端口，可以使用select（）来实现IO复用。

- - -
转载自：http://blog.csdn.net/seanyxie/article/details/5580162