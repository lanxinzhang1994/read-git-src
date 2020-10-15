# git 源码阅读1：init-db.c

> > 参考： [git源码](https://github.com/git/git/tree/v0.99/) (commit-id:e83c51633)

**************

## 1. C语言main()函数可以有输入参数      

Linus在写main()函数的声明时是这样写的：

```c 
int main(int argc, char **argv)
```

好吧，看了大神的代码我才意识到原来mian是可以带输入参数的，不过反过来想也没什么奇怪的，毕竟main()是函数，是函数就可以带输入参数，这不是本来就是天经地义的吗？

不过c语言main()函数和普通函数的参数也有不同之处。main函数的第一个输入参数的类型为int型，代表参数个数；  main函数的第二个参数的类型为字符型指针数组，数组的每个元素代表每个参数对应的字符串（数组每个元素代表各个参数对应字符串的首地址），其中要注意的是可执行文件名是第一个数组元素。

举个例子：

```c
//file test.c
int main(int argc, char * argv[])  //linus大神喜欢用指针的指针char ** argv，真的是6啊
{
    //......    
}
```

我们编译链接之后得到test.o，运行时输入```  test.o a1 a2 a3```，则有：

argc为整型，argc==4

argv[0]为字符串```"test.o"```     

argv[1]为字符串```"a1"```    

argv[2]为字符串```"a2"```    

argv[3]为字符串```"a3"```    



## 2. getenv(ENV)获取环境变量路径       

```c
char *sha1_dir = getenv(DB_ENVIRONMENT);
```

函数getenv(char * ENV)，输入参数是一个环境变量字符串，返回值也是一个字符串，表示输入环境变量的路径。这个函数的声明在stdlib.h中。

举个例子：   

```c
#include <stdio.h>
#include <stdio.h>

int main(){
    char * dir ;
    dir = getenv("PATH");
    printf("%s \n", dir);
    return 0 ;    
}
```

执行结果：   

```
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin  
```



## 3.mkdir(PATH，MOD) 创建路径      

在inti-db.c中有以下语句：     

```c
	if (mkdir(".dircache", 0700) < 0) {
		perror("unable to create .dircache");
		exit(1);
	}
```

其中函数mkdir()可以用来创建路径，该函数的定义为mkdir(char * path, mod)：     

- path为字符串，代表创建路径；
- mod为路径的权限，linux中用三位八进制数字表示权限，具体查看linux操作系统的相关资料。    
- 函数的返回值为int型，返回0为路径创建成功，返回-1为路径创建失败。      

mkdir()使用时要包含头文件<sys/stat.h>和<sys/type.h>， <sys/stat.h>内大多是与操作系统文件系统相关的定义，<sys/type.h> 内大多是与操作系统各种类型相关的定义。

不知道为什么，貌似我仅仅include了<sys/stat.h>而未include<sys/type.h>，函数mkdir()依然可以使用。



## 4.函数stat(PATH,STAT)和stat结构体       

stat()函数和stat结构体也声明在头文件<sys/stat.h>中。函数stat()用于返回某一路径的状态 ，而路径的状态信息保存在结构体类型stat中，他们的定义如下：    

```c
//函数stat
int stat(const char *filename, struct stat *buf); 

struct stat  
{  
	dev_t       st_dev;     /* ID of device containing file -文件所在设备的ID*/
 	ino_t       st_ino;     /* inode number -inode节点号*/
  	mode_t      st_mode;    /* protection -保护模式?*/
   	nlink_t     st_nlink;   /* number of hard links -链向此文件的连接数(硬连接)*/
   	uid_t       st_uid;     /* user ID of owner -user id*/
   	gid_t       st_gid;     /* group ID of owner - group id*/
   	dev_t       st_rdev;    /* device ID (if special file) -设备号，针对设备文件*/
   	off_t       st_size;    /* total size, in bytes -文件大小，字节为单位*/
   	blksize_t   st_blksize; /* blocksize for filesystem I/O -系统块的大小*/
   	blkcnt_t    st_blocks;  /* number of blocks allocated -文件所占块数*/
	time_t      st_atime;   /* time of last access -最近存取时间*/
	time_t      st_mtime;   /* time of last modification -最近修改时间*/
	time_t      st_ctime;   /* time of last status change - */
}; 
```

需要注意的是stat()的返回值是一个int类型的数据，返回值为0表示状态获取成功，返回值为-1表示状态获取失败，还有在使用stat()函数和stat结构体时，要包含<sys/stat.h>和<sys/type.h>。



## 5.<sys/stat.h>中的常用宏      

<sys/stat.h>中还有一些常用的宏定义，如下：    

```c
 S_ISLNK(st_mode)  //是否是一个连接.
 S_ISREG(st_mode)  //是否是一个常规文件.
 S_ISDIR(st_mode)  //是否是一个目录
 S_ISCHR(st_mode)  //是否是一个字符设备.
 S_ISBLK(st_mode)  //是否是一个块设备
 S_ISFIFO(st_mode) //是否 是一个FIFO文件.
 S_ISSOCK(st_mode) //是否是一个SOCKET文件
```

其中st_mode就是stat结构体中的成员st_mode，如果宏返回真，则代表是连接/文件/路径/字符设备......，返回假则代表不是连接/文件/路径/字符设备......  

明白了2，3，4，5这三个部分以下代码就应该看的懂了：

```c
sha1_dir = getenv(DB_ENVIRONMENT);
if (sha1_dir) {
	struct stat st;
	if (!stat(sha1_dir, &st) < 0 && S_ISDIR(st.st_mode))
		return;
	fprintf(stderr, "DB_ENVIRONMENT set to bad directory %s: ", sha1_dir);
}

```

## 6.errno        



## 7.perror       







## 附录(init-db.c)      

```c    
#include "cache.h"

int main(int argc, char **argv)
{
	char *sha1_dir = getenv(DB_ENVIRONMENT), *path;
	int len, i, fd;

	if (mkdir(".dircache", 0700) < 0) {
		perror("unable to create .dircache");
		exit(1);
	}

	/*
	 * If you want to, you can share the DB area with any number of branches.
	 * That has advantages: you can save space by sharing all the SHA1 objects.
	 * On the other hand, it might just make lookup slower and messier. You
	 * be the judge.
	 */
	sha1_dir = getenv(DB_ENVIRONMENT);
	if (sha1_dir) {
		struct stat st;
		if (!stat(sha1_dir, &st) < 0 && S_ISDIR(st.st_mode))
			return;
		fprintf(stderr, "DB_ENVIRONMENT set to bad directory %s: ", sha1_dir);
	}

	/*
	 * The default case is to have a DB per managed directory. 
	 */
	sha1_dir = DEFAULT_DB_ENVIRONMENT;
	fprintf(stderr, "defaulting to private storage area\n");
	len = strlen(sha1_dir);
	if (mkdir(sha1_dir, 0700) < 0) {
		if (errno != EEXIST) {
			perror(sha1_dir);
			exit(1);
		}
	}
	path = malloc(len + 40);
	memcpy(path, sha1_dir, len);
	for (i = 0; i < 256; i++) {
		sprintf(path+len, "/%02x", i);
		if (mkdir(path, 0700) < 0) {
			if (errno != EEXIST) {
				perror(path);
				exit(1);
			}
		}
	}
	return 0;
}

```





 