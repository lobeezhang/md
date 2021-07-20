###  Shell ch.

#### $ find 

```markdown
-name ‘* 烫 ? 烫 [123abc] 烫 [1-z] 烫 [1-35-6]’
-type f 烫 bcdpls 烫 
-user
-perm
-size c 烫 b 烫 w

-empty 空文件、空目录
-amin -cmin -mmin
-atime -ctime -mtime

选项间-a 与 -0 或 -！非

```

#### $ cat

```markdown
无参时 识别\n；c-d EOF 终止
-b 标记行号（非空行
-E 用$表示行尾
-n number of lines
-s squeeze repeated

```



#### $ cp

```markdown
-i
-f
-r
```



#### $ wc

```
-c 		bytes
-w
-l
```



#### $ grep(文件放最后)

```
-n 行号
-E 使用拓展规则
-F 不适用规则
-i 大小写不敏感
-c 总共有多少行被匹配
-w 被匹配的只能是单词，而不是单词的某一部分
-C  n：显示匹配到的字符串所在的行及其前后各n行，context
-v reverse

regex:
1、基本：字符
2、基本连接
3、. = 任一字符
4、[]中任一字符
拓展5、基？ 出现0~1次
拓展6、基* 出现0~多次
eg：通配符的*就等于regex的.*
拓展7、(regex)当做基
拓展8、(regex1)|(regex2)
9、^基、基$、基+
10、\<单词左右边界\>
```

#### $ sed

```
$ sed "s/int/float/g" filename

-i 实施修改
-e 多点操作
-n 只显示操作/修改行
$ sed '4 a \ ' filename 在第4行后增加一个空(格)行
$ sed -e '4,$ a \ ' filename 在第4行至末行……
$ sed -e 'a \ ' filename 在每行后加一个空(格)行（省略偏移量）

$ sed -en '1,5 p' filename 显示第1-5行
$ sed -n '/str/ p' filename 显示含str的行
$ sed '/str/d' filename 不显示含str的行
$ sed -e '1,5 d' filename 删除1-5行
$ sed '2,4 c str' filename 将2-4行删除，并插入一行str

```



#### 文件查看

```markdown
$ more 文件单页显示
$ less
		ma 烫 'a   mark和跳转到mark点
		其他操作类似vi
$ Less log2013.log log2014.log
		:p 烫 :n 在文档间切换
$ history | less (通过less分页显示)
$ ps -ef |less

$ tail -n 100
$ head 

$sort
$uniq 连续的空行显示为1行


```



#### 其他

```markdown
$ uname -a 
$ cat /etc/issue  //发型版本
$ cat /etc/passwd
$ cat /etc/shadow //加盐密文

$ cat /etc/crontab //定时任务
$ env

$ rm -f -r

$ df -h （disk fs）
$ du ~ -d 1 -h （disk u）

$ passwd change pw

$ sudo useradd -m wang -s /bin/bash
$ sudo userdel -r wang

$ file 给出file type
$ hitory > file.txt

$ tar {c|r|x}{v|f必备|z} 
$ ln -s

```

#### 组合

```
$ cmd1;cmd2
$ history | tail -n 100
$ find . -name "file" | xargs ls -al
$ find . -name '*' -exec ls -l {} \;
```

#### 替换

```
$ cmd1 `cmd2` 
```



#### 编译

```
$ gcc
-E ->.i
-S ->.s
-c ->.o

as -o	汇编器s->o
ld -o	链接器o->exe
od -h   dump files in octal and other formats
nm 查看o文件符号
objdump -d 对o文件反汇编
```



#### gcc 链接库

##### 1、一般

将所有源代码编译链接即可；

##### 2、静态

- 将部分源代码单独编译得到o文件
- `ar crsv **.o lib**.a`生成静态库文件
- 复制到/usr/lib/
- 主源代码编译后，在链接时：`gcc .o -o exe -l**` 



##### 3、动态

- 将部分源代码 `gcc **.c -o **.o -fpic`编译得到位置无关的o文件

- `gcc **.o -o lib**.so -shared`获得shared object 共享库文件

- 复制到/usr/lib

- 主源代码编译后，在链接时：`gcc .o -o exe -l**`

- 共享库文件在链接时有先后和依赖关系

  

##### 4、其他

- -static 链接静态库文件

- 使用ldd查看exe的依赖文件
- 不同版本文件加本版本记号，使用软链接，更新库文件，需要更新软链接即可





### File ch.

文件类型

```markdown
- 普通文件
d 目录文件
l 符号链接文件
c 字符设备文件(每次传输一个字符)
b 块设备文件
p 管道 虚拟文件
s socket
```



#### FD & redirection

stdin 0、stdout 1、stderr 2

```
0> 或追加模式 0>>
1>
2>
&> 等于 1> 2>
> 默认是将1重定向至文件，等于1>

```



#### fopen与open

```c
#include <stdio.h>
FILE *fopen( const char *fname, const char *mode );
```

<span style ="color:rgba(250,0,0,1)">*mode ："r"和"rb"在Linux中无区别</span>

*mode ："a+" 打开一个用于读/写的文本文件(追加) 

##### fopen series

```c
int fread( void *buf, size_t size, size_t num, FILE *stream );
int fwrite( const void *buf, size_t size, size_t count, FILE *stream );

int fscanf(FILE *, const char *format, ... );
int fprintf(FILE *, const char *format, ... );

long ftell(FILE *);

int fseek( FILE *, long offset, int origin );
 //可以使用fseek()移动超过一个文件,但是不能在开始处之前. 使用fseek()清除关联到流的EOF标记. 
```

| SEEK_SET | 从文件的开始处开始搜索 |
| -------- | ---------------------- |
| SEEK_CUR | 从当前位置开始搜索     |
| SEEK_END | 从文件的结束处开始搜索 |

![image-20210320160736443](E:\workLinux\image\image-20210320160736443.png)



fopen w模式 会清空文件；

open ，flag = WRONLY 不会清空文件，当写时只是从头开始写而已。



##### open series



```c
int open(const char *path, int oflag, ...);
//O_CREAT|O_EXCL|O_WRONLY|O_APPEND|O_RDONLY|O_RDWR|O_TRUNC
#include <unistd.h>
ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t count);
//通过返回值判断读取状况
```

<span style= "color:rgba(250,0,0,1)">**关于open与fopen的速度:</span>

- read/write次数越少越好，取决于俄内核缓冲区的大小
- 文件流的读写次数较少，拷贝次数多一次

```c
int dup(int oldfd);
int dup2(int oldfd, int newfd);
//close old ,dup(new)
```

![image-20210320161613232](E:\workLinux\image\image-20210320161613232.png)



```c
int fileno(FILE *stream);

off_t lseek(int fd, off_t offset, int whence);

```

![image-20210320160300089](E:\workLinux\image\image-20210320160300089.png)

#### File open

##### modify

```c
#include <unistd.h>
#include <sys/types.h>

int truncate(const char *path, off_t length);
int ftruncate(int fd, off_t length);
//The file offset is not changed
//文件在系统中，大小已调整，但磁盘没有分配空间

```

##### mmap

```c
void *mmap(void *addr, size_t lenth, int prot, int flags,int fd, off_t offset);
//void *addr = NULL- 自动分配
//size_t lenth 实际长度或者页大小的整数倍(4096B)
//int prot - PROT_EXEC/ PROT_READ/ PROT_WRITE
//int flags - MAP_SHARED/ MAP_PRIVATE/MAP_大页
//int fd
//offset 偏移，一般没什么用？


int munmap(void *addr, size_t length);
```



> mmap 
>
> 分配在堆区；
>
> 当malloc申请空间较大时，内核会调用mmap实现连续内存的分配；
>
> 当插入U盘时，可以通过mmap创建大页文件系统并挂载到VFS。

##### named pipe

```c
$ mkfifo
open pipe
//读写两端未建立就会阻塞在open（先读后写、先写后读）
//最好是O_RDWR打开
```



#### File status

```c

int chmod (const char * pathname, mode_t mode);
//
char *getcwd (char *buf, size_t size);
//ret NULL when fail
//采用(NULL, 0)时，自动malloc一个区域，存储信息并返回其指针，并会自动free
int chdir (const char *path);
//输入相对dir？
//内核解析路径名，确保这个路径名有效;检查它的文件类型和权限位;替换目录路径名和/或它的索引节点号
```

![image-20210319142735555](E:\workLinux\image\image-20210319142735555.png)

##### dir

```c
int mkdir (const char *pathname, mode_t mode );
//权限受umask影响
int rmdir (const char *pathname);
//仅空
```



```c
DIR *opendir (const char *name);
//创建目录流
struct dirent *readdir(dir* pdir);
//使用返回值
int closedir(DIR *pdir);
struct dirent {
               ino_t          d_ino;       /* Inode number */
               off_t          d_off;       /* Not an offset; see below */
               unsigned short d_reclen;    /* Length of this record */
               unsigned char  d_type;      /* Type of file; not supported
                                              by all filesystem types */
               char           d_name[256]; /* Null-terminated filename */
           };
long telldir(DIR *pdir);
void seekdir(DIR *pdir, long loc);
void rewinddir(DIR *dirp);
//resets the position of the directory stream dirp to the beginning of the directory

```

<span style ="color: rgba(250,0,0,1)">*实现深度优先遍历：cp -r；tree</span>

##### stat

```c

int stat(const char *restrict path, struct stat *restrict buf);
int stat(const char *pathname, struct stat *statbuf);
struct stat {
               dev_t     st_dev;         /* ID of device containing file */
               ino_t     st_ino;         /* Inode number */
               mode_t    st_mode;        /* File type and mode */
               nlink_t   st_nlink;       /* Number of hard links */
               uid_t     st_uid;         /* User ID of owner */
               gid_t     st_gid;         /* Group ID of owner */
               dev_t     st_rdev;        /* Device ID (if special file) */
               off_t     st_size;        /* Total size, in bytes */
               blksize_t st_blksize;     /* Block size for filesystem I/O */
               blkcnt_t  st_blocks;      /* Number of 512B blocks allocated */

               struct timespec st_atim;  /* Time of last access */
               struct timespec st_mtim;  /* Time of last modification */
               struct timespec st_ctim;  /* Time of last status change */

           #define st_atime st_atim.tv_sec      /* Backward compatibility */
           #define st_mtime st_mtim.tv_sec
           #define st_ctime st_ctim.tv_sec
           };

```



> restrict
>
> 只可以用于限定和约束指针，并表明指针是访问一个数据对象的唯一且初始的方式.即它告诉编译器，所有修改该指针所指向内存中内容的操作都必须通过该指针来修改,而不能通过其它途径(其它变量或指针)来修改;这样做的好处是,能帮助编译器进行更好的优化代码,生成更有效率的汇编代码.如 int *restrict ptr, ptr 指向的内存单元只能被 ptr 访问到，任何同样指向这个内存单元的其他指针都是未定义的，直白点就是无效指针。
>
> 类似上面的;
>
> void * memcpy（void * restrict s1, const void * restrict s2, size_t n);
> void * memmove(void * s1, const void * s2, size_t n);



```c
struct passwd *getpwuid(uid_t uid);
//uid-->name
struct passwd {
               char   *pw_name;       /* username */
               char   *pw_passwd;     /* user password */
               uid_t   pw_uid;        /* user ID */
               gid_t   pw_gid;        /* group ID */
               char   *pw_gecos;      /* user information */
               char   *pw_dir;        /* home directory */
               char   *pw_shell;      /* shell program */
           };

struct group *grtgrgid(gid_t gid);
struct group {
               char   *gr_name;        /* group name */
               char   *gr_passwd;      /* group password */
               gid_t   gr_gid;         /* group ID */
               char  **gr_mem;         /* NULL-terminated array of pointers
                                          to names of group members */
           };
```

#### about Time

```c
#include <sys/time.h>
//calender时间
int gettimeofday(struct timeval *tv, struct timezone *tz);
struct timeval {
               time_t      tv_sec;     /* seconds */
               suseconds_t tv_usec;    /* microseconds */微秒
           };
```



```c
#include <time.h>
//calender时间
time_t calender_time = time(NULL);

char *ctime(const time_t *timep);
char *ctime_r(const time_t *timep, char *buf);

//分解时间
struct tm *localtime(const time_t *timep);
struct tm *localtime_r(const time_t *timep, struct tm *result);
struct tm {
               int tm_sec;    /* Seconds (0-60) */
               int tm_min;    /* Minutes (0-59) */
               int tm_hour;   /* Hours (0-23) */
               int tm_mday;   /* Day of the month (1-31) */
               int tm_mon;    /* Month (0-11) */
               int tm_year;   /* Year - 1900 */
               int tm_wday;   /* Day of the week (0-6, Sunday = 0) */
               int tm_yday;   /* Day in the year (0-365, 1 Jan = 0) */
               int tm_isdst;  /* Daylight saving time */
           };
```



> The four functions asctime(), ctime(), gmtime() and localtime() return a pointer to static data and hence are **not thread-safe**.  
>
> The  **thread-safe versions**, asctime_r(), ctime_r(), gmtime_r() and localtime_r(), are specified by SUSv2.



#### Select



```c
int select(int nfds, fd_set *restrict readfds,
           fd_set *restrict writefds, fd_set *restrict errorfds,
           struct timeval *restrict timeout);
```



![image-20210320162555440](E:\workLinux\image\image-20210320162555440.png)

<img src="E:\workLinux\image\image-20210320162627975.png" alt="image-20210320162627975" style="zoom:60%;" />



![image-20210320162808907](E:\workLinux\image\image-20210320162808907.png)

![image-20210320162904953](E:\workLinux\image\image-20210320162904953.png)

![image-20210320162957371](E:\workLinux\image\image-20210320162957371.png)

> restrict
>
> c99标准引入的，它只可以用于限定和约束指针，并表明指针是访问一个数据对象的唯一且初始的方式.即它告诉编译器，所有修改该指针所指向内存中内容的操作都必须通过该指针来修改,而不能通过其它途径(其它变量或指针)来修改;这样做的好处是,能帮助编译器进行更好的优化代码,生成更有效率的汇编代码.如 int *restrict ptr, ptr 指向的内存单元只能被 ptr 访问到，任何同样指向这个内存单元的其他指针都是未定义的，直白点就是无效指针。restrict 的出现是因为 C 语言本身固有的缺陷，C 程序员应当主动地规避这个缺陷，而编译器也会很配合地优化你的代码.
>
> 
>
> C库中有两个函数可以从一个位置把字节复制到另一个位置。在C99标准下，它们的原型如下：
> void * memcpy（void * restrict s1, const void * restrict s2, size_t n);
> void * memmove(void * s1, const void * s2, size_t n);
> 这两个函数均从s2指向的位置复制n字节数据到s1指向的位置，且均返回s1的值。两者之间的差别由关键字restrict造成，即memcpy()可以假定两个内存区域没有重叠。memmove()函数则不做这个假定，因此，复制过程类似于首先将所有字节复制到一个临时缓冲区，然后再复制到最终目的地。如果两个区域存在重叠时使用memcpy()会怎样？其行为是不可预知的，既可以正常工作，也可能失败。在不应该使用memcpy()时，编译器不会禁止使用memcpy()。因此，使用memcpy()时，您必须确保没有重叠区域。这是程序员的任务的一部分。 
> 关键字restrict有两个读者。一个是编译器，它告诉编译器可以自由地做一些有关优化的假定。另一个读者是用户，他告诉用户仅使用满足restrict要求的参数。一般，编译器无法检查您是否遵循了这一限制，如果您蔑视它也就是在让自己冒险。

### Proc ch.

##### Proc instruction

什么是进程（2个角度）？和程序有哪些差异？day08

虚拟CPU：时间片轮转法day08

虚拟空间：

```markdown
进程拥有全部的地址空间 ->通过页表映射 ->页框(实际内存)|
		(虚拟内存)							    | ->久未使用的存入磁盘swap(win称虚拟内存，狭义)
```



虚拟/透明：看s非s/存在看不见

![image-20210320163640701](E:\workLinux\image\image-20210320163640701.png)

![image-20210320163703450](E:\workLinux\image\image-20210320163703450.png)

孤儿进程：父进程先退，子进程由一号进程收养

僵尸进程：子进程退，资源未被父进程回收(因忙等原因)，defunct

进程组：组ID是进程组组长ID；fork得到的是同一个进程组；

同一其前台进程组，都会收到ctrl+c的信号。

会话：是进程的集合

- 可以绑定终端（控制终端）
- 存在一个前台进程组，多个后台进程组
- 终端断开连接，信号会发给所有进程组





`$ cat /proc/cpuinfo`

`$ ps`查看进程识图：

![image-20210320163834316](E:\workLinux\image\image-20210320163834316.png)

standard syntax

![image-20210327165528328](E:\workLinux\image\image-20210327165528328.png)

BSD syntax

![image-20210327165600557](E:\workLinux\image\image-20210327165600557.png)

`ps j`

![image-20210327165710977](E:\workLinux\image\image-20210327165710977.png)

`ps -U root -u lz u`

![image-20210327165849936](E:\workLinux\image\image-20210327165849936.png)





`$ free` 查看内存用量

`$ df -h`查看磁盘用量



进程优先级：140级（由低到高-40~99）

默认权限80，nice为0，通过调整nice的值调整权限；

`$ renice`仅用于当前用户的降权（增值）；可用sudo设置负值升权，

`$ renice -n 10 ./main`以及`$ renice -n 10 -p pid`

`$ sched`调度策略(schedule)：

1. -40~59普通other：按优先级分配
2. 60~99实时RT：FIFO(相同优先级)；RR(相同优先级，分时间片)







进程控制块PCB：内核管理进程的数据结构（用户用pid标记进程）

Linux中采用task_struct:

![image-20210320194449498](E:\workLinux\image\image-20210320194449498.png)

##### BUFFER n CACHE

> buffer缓冲区 FIFO
>
> 缓冲区就是一块内存区，它用在输入输出设备和CPU之间，用来存储数据。高速设备与低速设备的不匹配，势必会让高速设备花时间等待低速设备，它使得低速的输入输出设备和高速的CPU能够协调工作，避免低速的输入输出设备占用CPU，解放出CPU，使其能够高效率工作。
>
> 缓冲区根据其对应的是输入设备还是输出设备，分为输入缓冲区和输出缓冲区。
>
> 缓存区的作用
> 1、 可以解除两者的制约关系，数据可以直接送往缓冲区，高速设备不用再等待低速设备，提高了计算机的效率。
>
> 2、 可以减少数据的读写次数，如果每次数据只传输一点数据，就需要传送很多次，这样会浪费很多时间，因为开始读写与终止读写所需要的时间很长，如果将数据送往缓冲区，待缓冲区满后再进行传送会大大减少读写次数，这样就可以节省很多时间。
>
> cache缓存
>
> Cache就是用来解决CPU与内存之间速度不匹配的问题，避免内存与辅助内存频繁存取数据，这样就提高了系统的执行效率。加快了数据取用的速度。

##### MODE

uid/euid二进制文件特殊权限4664：`$ chmod u+s`

1. 要拥有s权限；

2. 拥有user的x权限；

   ---->在执行程序时，修改euid为文件拥有者。

gid/egid文件g特殊权限2664：`$ chmod g+s `

1. 拥有group_x权限；

   ------->

```c
pid_t getpid(void);
pid_t getppid(void);

uid_t getuid(void);//真实
uid_t geteuid(void);//有效<- 权限:系统分配且权限参考的对象

gid_t getgid(void);
gid_t getegid(void);

pid_t getpgid(pid_t pid);
pid_t getpgrp(void);//当前进程的process group ID

pid_t getsid(void);
//returns the session ID of the calling process. 
//要求自身不是进程组组长
```

![image-20210320185240569](E:\workLinux\image\image-20210320185240569.png)

sticky粘滞位1775：`$ chmod o+t`

1. 目录拥有t；
2. 拥有o有w、x权限。

-------->可以在目录下创建文件，删除自己的文件，不可更改、删除别人的文件



##### system/fork/exec

```c
#include <stdlib.h>
int system(const char *command);
//system->shell->cmd
The  system()  library  function  uses fork(2) to create a child process that executes the shell command specified in command using execl(3) as follows:

           execl("/bin/sh", "sh", "-c", command, (char *) 0);

       system() returns after the command has been completed.
       During execution of the command, SIGCHLD will be blocked, and SIGINT and SIGQUIT will be ignored, in the process  that calls system().

```

```c
#include <sys/types.h>
#include <unistd.h>

pid_t fork(void);
//The child does not inherit its parent's memory locks(mlock(2), mlockall(2)).
//On success, the PID of the child process is returned in the parent, and 0 is returned in the child.
//On failure, -1 is returned in the parent, no child process is created, and errno is set appropriately.

```



![image-20210320214725712](E:\workLinux\image\image-20210320214725712.png)

> fork
>
> 上半部：拷贝task_struct,修改pid、ppid，fork的返回值修改为0，把子进程放入就绪队列
>
> 下半部：拷贝数据，用户态：堆栈，全局，环境，文件缓冲区，计时器未决信号
>
> 此外，fork采用copy on write；
>
> 此此外，对于open得到的fo，硬件设备等的应用，父子共享连接和更改。其实际时下采用dup的方式。
>
> 此外外，shell的内涵：bash-fork-setpgid-exec-a.out



```c
#include <unistd.h>
       extern char **environ;

       int execl(const char *path, const char *arg, ... /* (char  *) NULL */);
       int execlp(const char *file, const char *arg, ... /* (char  *) NULL */);
       int execle(const char *path, const char *arg, ... /*, (char *) NULL, char * const envp[] */);//NULL表示列表的结尾
       int execv(const char *path, char *const argv[]);
       int execvp(const char *file, char *const argv[]);
       int execvpe(const char *file, char *const argv[],char *const envp[]);
//注意：arg是字符串时，为const char*；为字符串数组时，为char *const
//envp[]指向环境变量的数组
```

张子：对于exec，l 和 v 二选一，表示采用直接输入的方式(每个参数代表一个命令行参数的指针)，还是char buf[]传入的方式输入命令；

p表示是否使用PATH，此时可以省略path直接输入filename(若在PATH内)

![image-20210320221734352](E:\workLinux\image\image-20210320221734352.png)

##### wait n exit

```c
#include <sys/wait.h>

pid_t wait(int *stat_loc);
pid_t waitpid(pid_t pid, int *stat_loc, int options);
//pid 小于-1，指定pid的绝对值；
//等于-1，随便等某个子，；
//等于0，等某个子whose process group ID is equal to that of the calling process；
//大于0 ，指定pid

//option:WNOHANG     return immediately if no child has exited

//macro：WIFEXITED(wstatus)；WEXITSTATUS(wstatus)；用于查看进程各种变化信息
```

![image-20210320225221069](E:\workLinux\image\image-20210320225221069.png)

![image-20210320225240552](E:\workLinux\image\image-20210320225240552.png)

![image-20210320225251065](E:\workLinux\image\image-20210320225251065.png)

##### guard proc

instruction: 后台运行，不依赖终端（用户登录设备），daemon

```c
int daemon(int nochdir, int noclose);
//run in the background
//If nochdir is zero, daemon() changes the process's current working directory to the root directory ("/");  otherwise,the current working directory is left unchanged.
//If noclose is zero, daemon() redirects standard input, standard output and standard error to /dev/null; otherwise, no changes are made to these file descriptors.
```

实现：

1. fork();
2. setsid();
3. parent proc exit
4. chdir("\\");
5. umask(0);
6. close(fd)*3;

### interproc

```markdown
1、管道-pipe()-流式
2、共享内存-shmget-效率
3、信号量-semget
4、消息队列-msgget-包式
5、信号

$ ipcs
$ ipcrm -m/-s/-q
```

#### Pipe

```c
FILE *popen(const char *command, const char *type);
int pclose(FILE *stream);
```

![image-20210321195830435](E:\workLinux\image\image-20210321195830435.png)

![image-20210321200253148](E:\workLinux\image\image-20210321200253148.png)

```c
int pipe(int fds[2]);
//先fork，再传入数组名

#include <unistd.h>
int unlink(const char *pathname);
```

![image-20210321200923275](E:\workLinux\image\image-20210321200923275.png)

![image-20210321201159252](E:\workLinux\image\image-20210321201159252.png)

rename参数不是路径，而是文件名。

#### Shm

：建立不同虚拟空间映射同一片物理地址

```c
key_t ftok(const char *pathname, int proj_id);
//proj_id随便输
//uses  the  identity  of the file named by the given pathname (which must refer to an existing, accessible file) and the least significant 8 bits of proj_id (which must be nonzero) to generate a key_t type  System V IPC key, suitable for use with msgget(2), semget(2), or shmget(2).

int shmget(key_t key, size_t size, int shmflg);

void *shmat(int shmid, const void *shmaddr, int shmflg);
//建立虚拟地址和物理地址的映射（即共享内存的地址）
int shmdt(const void *shmaddr);
//解除映射
int shmctl(int shmid, int cmd, struct shmid_ds *buf);
//cmd ： IPC_STAT获取信息/IPC_SET设置/IPC_RMID删除（延迟）
```

![image-20210321202843213](E:\workLinux\image\image-20210321202843213.png)

![image-20210321204226534](E:\workLinux\image\image-20210321204226534.png)

共享内存的使用涉及到竞态条件的问题（对临界区的<span style ="color:rgba(250,0,0,1)">并发访问</span>）



#### day10 内存叙述



#### Sem

：描述可用资源的个数

信号量是一个计数器，用于为多个进程提供对共享对象的访问。信号量的**测试**及**加减1操作**应当是原子操作，为此，信号量通常是在内核中实现的。

原子操作(原语)：不可分割，不可中断的操作（中断需要回撤）

二元信号量：P/V

同步：初始化为0

互斥：初始化为1

![image-20210321204759104](E:\workLinux\image\image-20210321204759104.png)

![image-20210321204820266](E:\workLinux\image\image-20210321204820266.png)

```c
int semget(key_t key, int semnum, int semflg);
//semnum 集合中信号的数量，设置好后就不可以更改
int semctl(int semid, int semnum, int cmd, ...);
//cmd ： GETVAL给对应下标的信号量赋值/SETVAL/GETALL/SETALL
//IPC_GET/IPC_SET/IPC_RMID(立即删除)
```

![image-20210321204958856](E:\workLinux\image\image-20210321204958856.png)

![image-20210321205341044](E:\workLinux\image\image-20210321205341044.png)

![image-20210321210418220](E:\workLinux\image\image-20210321210418220.png)

```c
int semop(int semArrid, struct sembuf *buf, size_t nsemops);
//buf传入传出；nsemops要修改的信号量的个数
```

![image-20210321210623932](E:\workLinux\image\image-20210321210623932.png)

![image-20210321210728542](E:\workLinux\image\image-20210321210728542.png)

![image-20210321224005530](E:\workLinux\image\image-20210321224005530.png)





#### Msg

```c
struct mymsg{
    long mtype;
    char mtext[64];
}//自行定义
int msgget(key_t key,int msgflg);

int msgsnd(int msgid, const void *msgp, size_t msgsz, int msgflg);
ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);

//IPC_NOWAIT Return  immediately if no message of the requested type is in the queue.  The system call fails with errno set to ENOMSG.
//所以使用msgrcv时最好搭配perror，当未读取到信号时，会报错
```





#### Sig

产生：

外部硬件 - 键盘信号

内部硬件 - 除零时，协处理器发送；访问不可读内存时；

外部软件 - kill

内部软件 - wirte一个读端关闭的管道



处理：

- 默认term终止、core终止、stop暂停、cont继续、ign忽略

- 指定忽略（丢弃
- 指定处理流程：捕捉

<span style ="color:rgba(250,0,0,1)">**9）SIGKILL和SIGSTOP不可捕捉和丢弃</span>

信号实现机制：

略（几个时序图，自己看）

![image-20210327174423249](E:\workLinux\image\image-20210327174423249.png)



#### Producer & Ponsumer

![image-20210321211112406](E:\workLinux\image\image-20210321211112406.png)

```c
semget();semctl();
struct sembuf;
P,V;

```



### Other

用到引用计数的一些地方：

- 智能管理指针
- 硬链接
- fd对fo的指向
- pthread_mutex



几个返回值

- int atoi()
- uint sizeof()
- ld strlen()
- off_t stat.size ，实际是ld

很多函数都是传入指针

- 例如：

- 但两个退出是例外，waitpid (pid，…)和pthread_join (thid， NULL)；



三个函数：

- 信号回调        void     sigfunc       (int signum)
- 线程入口函数void * threadfunc (void *p)
- 线程清理函数void    cleanfunc    (void *)



一些“打开”

open

fopen

popen



c通过联合体实现可变参数：

semctl()信号量；

sockpairs()；





const：

```c
const char *p; // 声明一个指向字符或字符串常量的指针(p所指向的内容不可修改)

char const *p;// 同上

char * const p;//声明一个指向字符或字符串的指针常量，即不可以修改p的值，也就是地址无法修改。
```

![]()