# git源码阅读2：read_catche.c        

> > 参考： [git源码](https://github.com/git/git/tree/v0.99/) (commit-id:e83c51633)                  

**********************************

## 1 access() 

有时在使用c语言进行文件操作时，我们常常需要知道文件/路径是否存在，文件是否可读，文件是否可写，文件是否可执行。access()函数可以帮我们获得这些与文件或路径有关的信息（需要引入<unistd.h>，UNIX-Standard），access()函数的声明如下：      

```c
int access(const char *pathname, int mode);
```

输入参数pathname为一个字符串，表示文件路径；mod为我们需要知道的信息种类，在<unistd.h>中，通过宏定义的方式将种类映射为以下四种:

```c
#define F_OK 0   //路径/文件是否存在
#define X_OK 1   //文件是否具有执行权限
#define W_OK 2   //文件是否具有写权限 
#define R_OK 4   //文件是否具有读权限  
```

如果函数的模式有效，返回0，模式无效，返回-1。

举一个简单的例子：    

```c 
#include <stdio.h>   
#include <unistd.h>  

int main()
{
    char * fname = "./test_out/"
    printf("%0d \n",access(fname,F_OK));   
    printf("%0d \n",access(fname,X_OK));
    printf("%0d \n",access(fname,W_OK));
    printf("%0d \n",access(fname,R_OK));
	return 0 ;    
}

```

## 2 fopen()与open()     

在c语言中使用fopen()打开文件并进行文件操作是再常见不过的应用了。但是我发现linus大神在git的源码中更喜欢使用open()函数。实在是才疏学浅，我也不知道与fopen()相比open()函数的优点在哪里。貌似我很少见到有人使用open()。关系两者的比较推荐看着两个文章 [总结open与fopen的区别](https://www.cnblogs.com/NickyYe/p/5497659.html)，[C语言中open与fopen的的解释和区别](https://blog.csdn.net/LEON1741/article/details/78091974) 。

open()函数的定义如下：

```c
int open(const char *path, int flag, int mode);  
```

path肯定就是代表路径的字符串啦，而flag和mode是什么呢，flag可以类比为fopen()的使用方式，而mode仅在建立文件时有用表示新建文件的初始权限。open()需要的头文件有

```
#include <sys/types.h>    
#include <sys/stat.h>    
#include <fcntl.h>
```

### 2.1 flag的配置方法   

 flag的备选值已经在头文件中用宏定义定义过了，可以直接使用。         

**以下为必选项**     

| flag | 含义 |
| ---- | ---- |
| O_RDONLY   |    以只读方式打开文件 |
| O_WRONLY   |   以只写方式打开文件 |
|  O_RDWR          |        以可读写方式打开文件|

**上述三种旗标是互斥的，也就是不可同时使用，但可与下列的旗标利用OR(|)运算符组合。**    

| flag | 含义 |
| ---- | ---- |
|O_CREAT|      若欲打开的文件不存在则自动建立该文件|
|O_EXCL   |       如果O_CREAT也被设置，此指令会去检查文件是否存在，文件若不存在则建立该文件，否则将导致打开文件错误。此外，若O_CREAT与O_EXCL同时设置，并且欲打开的文件为符号连接，则会打开文件失败。|
| O_NOCTTY | 如果欲打开的文件为终端机设备时。则不会将该终端机当成进程控制终端机 |
| O_TRUNC | 若文件存在并且以可写的方式打开时，此旗标会令文件长度清为0。而原来存于该文件的资料也会消失 |
| O_APPEND |  当读写文件时会从文件尾开始移动，也就是所写入的数据会以附加的方式加入到文件后面 |
| O_NONBLOCK  | 以不可阻断的方式打开文件，也就是无论有无数据读取或等待，都会立即返回进程之中 |
| O_NDELAY  | 同O_NONBLOCK |
| O_SYNC | 以同步的方式打开文件 |
| O_NOFOLLOW  | 如果参数pathname所指的文件为一符号连接，则会令打开文件失败 |
| O_DIRECTORY    | 如果参数pathname所指的文件并非为一目录，则会令打开文件失败。注：此为Linux2. 2以后特有的旗标，以避免一些系统安全问题。 |



### 2.2 mode的配置方法      

mod表示新建文件的初始权限   

| mod  | 含义 |
| ---- | ---- |
|  S_IRWXU    |  00700权限，代表该文件所有者具有可读、可写及可执行的权限。    |
| S_IRUSR 或S_IREAD | 00400权限，代表该文件所有者具有可读取的权限。|
| S_IWUSR 或S_IWRITE | 00200权限，代表该文件所有者具有可写入的权限。|
| S_IXUSR 或S_IEXEC | 00100权限，代表该文件所有者具有可执行的权限。|
| S_IRWXG  | 00070权限，代表该文件用户组具有可读、可写及可执行的权限。|
| S_IRGRP | 00040权限，代表该文件用户组具有可读的权限。|
| S_IWGRP | 00020权限，代表该文件用户组具有可写入的权限。|
| S_IXGRP | 00010权限，代表该文件用户组具有可执行的权限。|
| S_IRWXO | 00007权限，代表其他用户具有可读、可写及可执行的权限。|
| S_IROTH | 00004权限，代表其他用户具有可读的权限。|
| S_IWOTH | 00002权限，代表其他用户具有可写入的权限。|
| S_IXOTH | 00001权限，代表其他用户具有可执行的权限。|



### 2.3 函数返回值返回一个文件描述符        

什么是文件描述符，请参考 [每天进步一点点——Linux中的文件描述符与打开文件之间的关系](https://blog.csdn.net/cywosp/article/details/38965239)




## 3 (void*) -1  

[ref](http://blog.sina.com.cn/s/blog_c2f250bd0101heb1.html)  



## 4 mmap()   munmap()      





## 5 fstat()      



## 6 calloc malloc  







