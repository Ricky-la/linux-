# System -Level 

# UNIX I/O

## Opening Files

```c
int fd; /* file descriptor*/

if((fd=open("/etc/host",O_RDONLY))<0){
    perror("open");
    exit(1);
}
```

each process created by a linux shell begins life with three open files associated with a terminal:

0: standard input (stdin)

1: standard output (stdout)

2: standard error (stderr)

## Closing Files

```c
int fd; /* file descriptor*/
int retval;

if((retval=close(fd))<0){
    perror("close");
    exit(1);
}
```

 Closing an already closed file is recipe for disaster in threaded programs 

 

## Reading Files 

specially, updates file position

```c
char buf[512];
int fd;
int nbytes; /*number of bytes read*/
/*Open fiile fd ...*/
/*Then read up to 512 bytes form file fd*/
if((nbytes=read(fd,buf,sizeof(buf)))<0)
{
    perror("read");
    exit(1);
}
```

在标准输入输出中，read 将被挂起并等待，只到输入一个字符串，然后回车

同样的在网络连接中，被挂起和等待，直到新内容输入

return:  ssize_t   signed long int 

negative :error

0: EOF

positive: 读入的字节数

## Writing files

specially, updates file position

```c
char buf[512];
int fd;
int nbytes;
/*Open fiile fd ...*/
/*Then read up to 512 bytes form file fd*/
if((nbytes=write(fd,buf,sizeof(buf)))<0)
{
    perror("write");
    exit(1);
}
```

##  Simple Unix I/O example

```c
int main(void) //bad code 系统调用频繁
{
    char c;
      
    while(Read(STDIN_FILENO,&c,1)!=0)
        Write(STDOUT_FILENO,&c,1);
    exit(0);
}   
```

## On Short Counts 

occur in this situation:

​	Encountering EOF on reads

​	Reading text lines from a terminal

​	Reading and writing network sockets 



# THE RIO Package	

CMU 封装的健壮i/o包

## Unbuffered RIO Input and Output

```c
/*
*  rio_readn Robustly read n bytes (unbuffed) 解决不足值的问题，在没有遇到EOF时，一直
*  读取直到buf满
*  缺点：频繁的系统调用
*/
ssize_t rio_readn(int fd, void*usrbuf,size_t n)   //n 为buf大小
{
    size_t nleft=n;   
    ssize_t nread;   
    char *bufp=usrbuf;
    
    while(nleft>0)  
    {
        if((nread=read(fd,bufp,nleft))<0)    
        {
            if(errno==EINTR)  /*Interrupt by sig handle return*/
                nread=0;    /*and call read() again*/
            else
                nread=-1;    /*errno  set by read()*/
        }
        else if(nread==0)
            break;     /*EOF*/
        nleft-=nread;
        bufp+=nread;    
    }
    return(n-nleft);    /*Return >=0*/
}
```

## buffered RIO Input and Output

```c
//buf直到满/换行/EOF才会使用系统调用 
typedef struct{
     int rio_fd;  
     int rio_cnt;  /*unread bytes in internal buf*/
     char *rio_bufptr;
     char rio_buf[RIO_BUFSIZE];
 }rio_t;

int main（）
{
    int n;
    rio_t rio;
    char buf[MAXLINE];
    
    Rio_readinitb(&rio,STDIN_FILENO);
    while((n=Rio_readlineb(&rio,buf,MAXLINR))!=0)
        Rio_writen(STDOUT_FILENO,buf,n);
    exit(0);
}
```

# File Metadata

```c
struct stat{
    unsigned long  st_dev;  //devices
	unsigned long  st_ino;  //inode
	unsigned short st_mode; //protection and file type
	unsigned short st_nlink; 
	unsigned short st_uid;
	unsigned short st_gid;
	unsigned long  st_rdev;  //device type
	unsigned long  st_size;
	unsigned long  st_blksize;
	unsigned long  st_blocks;
	unsigned long  st_atime; //time of last access 
	unsigned long  st_mtime; //time of last modification
	unsigned long  st_ctime; //time of last change
}  
```

![image-20220831095119852](F:\typora\image\image-20220831095119852.png)



Descriptor table(one table per process):

​	fd>2  regular file

Open file table:

​	File pos : seek

​	refcnt: cited number

v-node table:

​	info in stat struct  



### File sharing     

![image-20220831095215540](F:\typora\image\image-20220831095215540.png)

在一个进程多次操作文件，文件的共享级别为基于v-node table 级别的共享，有独立的File pos

![image-20220831095243995](F:\typora\image\image-20220831095243995.png)

fork一个子进程后，共享级别为基于Open file table 级别的共享，父子进程公用 File pos,要关闭文件，必须每个进程都关闭  

### I/O  Redirection

### 	dup2(fd1,fd2);

![image-20220831095259314](F:\typora\image\image-20220831095259314.png)



# Standard  I/O function

```c
#include <stdio.h>
int main() {
    printf("1");
    printf("2");
    printf("\n");
    printf("3");
    printf("4");
    fflush(stdout);
    printf("5");
    exit(0);
}
```

```c
write(1, "12\n", 312)
write(1, "34", 234) 
write(1, "5", 15) 

```

标准I/O 函数遇到“\n” ,或者fflush时候，才会使用系统调用

