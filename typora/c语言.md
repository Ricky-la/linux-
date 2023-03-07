# contain of 宏

[contain of 宏](https://blog.csdn.net/u011037593/article/details/79071251?spm=1001.2014.3001.5502)

# do{} while(0); 宏

[理解do{} while(0)语句的作用](https://blog.csdn.net/weibo1230123/article/details/81904498)

为了防止出现有多条语句的宏写在条件判断之后

```c
#define SAFE_FREE(p)  free(p); p=NULL；

if(NULL!=p)
   SAFE_FREE(p)
else
{
    ...
}

/*被宏替换后变为*/

if(NULL!=p)
   free(p); p=NULL；
else
{
    ...
}
```

## const * 问题

```c
//const和谁近修饰谁
const int * grape; 
int const * grape; 
int const * const grape_jam; 
const int * const grape_jam; 
 char * const *(*next)(); /*“next是一个指针，它指向一个函数，该函数返回另一个指针，该指针指向一个类型为char的常量指针(二级指针)”*/
```

### __must_check

\#define __must_check      __attribute__((warn_unused_result))

__must_check函数是指调用函数一定要处理该函数的返回值，否则[编译器](https://so.csdn.net/so/search?q=编译器&spm=1001.2101.3001.7020)会给出警告



# 枚举类型作为函数参数

```c
#include<stdio.h>
enum MODE
{
    b，
    bg，
    bgn mix
}
int test(enum MODE m） 
{
    
}
int main(）
{
enum MODE m=bg：
test（m）：
}
```

###  LINUX VERSION CODE  

```c
/usr/include/linux/version.h
#define LINUX VERSION CODE 263213
#define KERNEL VERSION(a，b，C） (((a)<<16)+((b)<8)+(C))
```

# 多个源文件共享全局变量的问题

```c
/*.c*/
#include <stdio.h> 
#define EXPORT_GLOBALS  
#include "define_variable_in_header.h"
int main(int argc,char *argv[])
{
    g_flag = 5; 
    printf("%d\r\n",g_flag); 
    return 0;
}
编译后 unsigned int g_flag; 相当于这个宏是在此.c文件定义的


/*define_variable_in_header.h*/
#ifdef EXPORT_GLOBALS
#define EXTERN
#else
#define EXTERN extern
#endif 
EXTERN unsigned int g_flag; 
编译后 extern unsigned int g_flag;  相当于声明了一个外部变量
```

