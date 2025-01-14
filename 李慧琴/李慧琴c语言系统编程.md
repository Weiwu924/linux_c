# c语言系统开发

## 李慧琴linux嵌入式C语言编程

- [x] 谁打开谁关闭
- [x] 谁申请谁释放
- [x] 是资源就一定有上限



> 共用体

同一个存储空间可以存储多种数据类型，但是同一时刻只能一个成员存在，空间大小由最大的数据类型所占空间大小

> 位域

> ### makefile文件编写 (make工程管理器)

对于多级的项目结构，运行前的编译工作，通过多级makefile文件的方式进行编译工作,全过程相当于从叶子往根进行遍历

`对于makefile文件的详细解释：`
现在我有几个文件：main.c tool1.c tool2.c tool1.h tool2.h，其中main.c文件依赖于tool1.c和tool2.c，将它们通过gcc编译成为可执行文件tool

最一般的makfile写法：

```makefile
tool:main.o	tool1.o	tool2.o
	gcc main.o	tool1.o	tool2.o	-o	tool

main.o:main.c
	gcc main.c	-c -g -Wall	-o	main.o

tool1.o:tool1.c
	gcc	tool1.c	-c -g -Wall	-o	tool1.o

tool2.o:tool2.c
	gcc tool2.c	-c -g -Wall	-o	tool2.o

clean:
	rm -rf *.o tool
```

初步改进：我们使用代替的方式替换编译过程中出现的资源或是命令，其中比较特殊的是$^指代在上一句命令中所需要的资源文件，$@指代需要生成的目标文件

```makefile
OBJS=main.o	tool1.o	tool2.o
CC=gcc
CFLAGS+=-c -g -Wall

tool:$(OBJS)
	$(CC) $^	-o	$@

main.o:main.c
	$(CC) $^	$(CFLAGS)	-o	$@

tool1.o:tool1.c
	$(CC)	$^	$(CFLAGS)	-o	$@

tool2.o:tool2.c
	$(CC) $^	$(CFLAGS)	-o	$@

clean:
	rm -rf *.o tool
```

再进一步改进：我们使用%通配符的形式将类似的语句形成一个公式，表示我们需要的tool可执行文件依赖于所有OBJS所包含的文件，而这些文件中所有的.o文件都需要通过%.o:%.c生成

```makefile
OBJS=main.o	tool1.o	tool2.o
CC=gcc
CFLAGS+=-c -g -Wall

tool:$(OBJS)
	$(CC) $^	-o	$@

%.o:%.c
	$(CC) $^	$(CFLAGS)	-o	$@

clean:
	rm -rf *.o tool
```

我们也可以直接使用makefile来运行可执行文件，其中@./$(target)的@代表不打印命令执行记录

```makefile
OBJS=main.o	tool1.o	tool2.o
CC=gcc
CFLAGS+=-c -g -Wall
target=tool

$(target):$(OBJS)
	$(CC) $^	-o	$@

%.o:%.c
	$(CC) $^	$(CFLAGS)	-o	$@

clean:
	rm -rf *.o tool

run:$(target)
	@./$(target)
```



> **俄罗斯方块实现**

**1、碰到的问题**

- 图形（画图问题）
- 对输入设备的设置问题（不要让用户输入的WASD显示到终端）
- 并发问题（下降同时接收键盘输入）



> ## UNIX环境高级编程

### **全书结构拆解**  3~16章(9章)

> I/O：

3、5章 14章

> 文件系统：

4、6、7章

> 并发：

信号 10章

多进程、多线程  10、11章

> IPC:进程间通信

8章 进程基础（涉及到多进程）

13章 守护进程

15、16章



注意事项：

1、弃用root用户

2、重构之前的代码

3、基于：项目、课堂代码、面试题、实验性题目、推荐书籍的课后题



> ## 时间表

| 时间 |       内容       |
| :--: | :--------------: |
| 8/11 |      标准IO      |
| 8/12 |    系统调用IO    |
| 8/13 |     文件系统     |
| 8/14 |  目录和用户操作  |
| 8/15 |     进程专题     |
| 8/16 |     信号专题     |
| 8/17 |     线程专题     |
| 8/18 |      高级IO      |
| 8/19 |     管道实例     |
| 8/20 | 进、线程通信专题 |
| 8/21 |   网络编程专题   |

# 一、标准I/O操作

**io操作是一切实现的基础**，对于数据可以存储，转存进文件中，需要时从文件中拿出，所以是一切操作基础。

系统调用io（sysio），标准IO(stdio)，不同的操作系统环境下，同一个函数依赖的是不同的系统调用，所以才需要标准IO，不管是什么操作系统，底层的输入输出到底是怎么实现的，所有的标准IO在任何操作系统下，参数相同，返回值以及调用方法完全一致，所以就解决了不同系统下输入输出方法不相匹配的问题，比如不论是什么系统，printf函数用法完全一致。标准IO移植性好。所以在任何的操作系统下，标准IO依赖于系统调用IO实现。

<img src="李慧琴c语言系统编程/image-20230829204427241.png" alt="image-20230829204427241" style="zoom:67%;" />



**标准IO（stdio）：FILE 类型贯穿始终**

`FILE实际上是一个结构体`

> 查看man手册:man ls

常见文件操作函数：

## 1、打开关闭函数

`fopen()`

```c
FILE *fopen(const char *pathname, const char *mode);
```

第一个参数是需要打开的文件名称，第二个参数是打开的模式。

`errno`->相当于一个全局变量，如果当前有一个操作出现错误，那么计算机会将错误原因放在errno里，所以需要及时将errno中的信息打印出来，如果没有及时打印，当前的errno里面的信息可能会被其他的错误信息覆盖掉。

文件打开方式：
`r r+ w w+ a a+`

r：以只读方式打开文件，流被定位到文件首部

r+: 以读写的方式打开文件，流被定位到文件首部

w：截断文件到0长度，或者以只写的方式创建一个文件，有则清空，无则创建

w+：以读写方式打开文件，有则清空，无则创建

a：在文件尾追加内容，流指针定位在文件尾(文件最后一个字节的后一个位置)

a+：在文件尾追加内容，流指针定位在文件尾(文件最后一个字节的后一个位置)，有则清空，无则创建

r和r+使用的时候，要求文件必须存在，如果文件不存在，直接返回错误值。

> 示例程序fopen.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

int main()
{
    FILE *fp;

    fp = fopen("/tmp/out","r");

    if(fp == NULL)
    {
        fprintf(stderr,"fopen() failed!,errno = %d\n",errno);
        exit(1);
    }   

    puts("OK!");

    exit(0);
}
```

`函数perror()会主动关联errno，可以直接打印errno信息。`

> 使用实例

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

int main()
{
    FILE *fp;

    fp = fopen("/tmp/out","r");

    if(fp == NULL)
    {
        perror("fopen()");
        exit(1);
    }   

    puts("OK!");

    exit(0);
}
```

`另外还有一个函数strerror()，同样可以打印出错误信息，返回一个串，该串会记录errno错误值`

> 使用实例

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>

int main()
{
    FILE *fp;

    fp = fopen("/tmp/out","r");

    if(fp == NULL)
    {
		fprintf(stderr,"fopen():%s\n",strerror(errno));
        exit(1);
    } 

    puts("OK!");

    exit(0);
}
```

:star2:因为fopen的返回值是FILE的指针变量，本质上FILE是一个结构体，也就是说，函数fopen()返回一个结构体的第一个元素的地址。

`那么这个FILE的结构体存在什么位置呢？`

- 栈 :x:

分析->如果存在于栈上，那么在函数fopen()内部必定存在FILE tmp这样的语句，最后因为返回值是FILE型数据，所以返回tmp，但是tmp在fopen函数调用结束之后就被销毁了，所以不应该是栈。

```c
FILE *fopen(const char *path, const char *mode)
{
    FILE tmp;
    
    //赋值操作
    tmp.xx = xxx;
    .....
        
    return &tmp;
}
```

- 静态区 :x:

```c
FILE *fopen(const char *path, const char *mode)
{
    static FILE tmp;//使用static修饰会将变量放置在静态区
    
    //赋值操作
    tmp.xx = xxx;
    .....
        
    return &tmp;
}
```

因为static修饰，被放置在静态区的变量，被重复调用的时候，只会声明一次，那么tmp就只会被声明一次，不管是操作什么文件，所有的地址完全相同，那么就意味着在操作所有的文件的时候，使用的是同一个指针，那么最终会导致，不管是操作什么文件，都在操作上一份文件，会冲掉上一份文件。

- 堆 :arrow_up_small:正确

```c
FILE *fopen(const char *path, const char *mode)
{
    FILE *tmp = NULL;//堆上即代表着动态空间
    
    tmp = malloc(sizeof(FILE));
    //赋值操作
    tmp->xx = xxx;
    .....
        
    return tmp;
}
```

所以可见，如果一个函数存在一个逆操作时，那么函数返回值指针一定是存在堆上面，反之不一定成立。

`fclose()`

```c
int fclose(FILE *fp);
```

> 使用实例

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

int main()
{
    FILE *fp;

    fp = fopen("/tmp/out","r");

    if(fp == NULL)
    {
        perror("fopen()");
        exit(1);
    }   

    puts("OK!");
    
	fclose(fp);

    exit(0);
}
```

在linux系统中文件打开的个数是存在上限，其上限存在于`ulimit -a`命令下面的open files参数中

> 验证程序maxfopen

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

int main()
{
    int count = 0;
    FILE *fp;

    while(1)
    {
        fp = fopen("/tmp/out","r");
        if(fp == NULL)
        {
            perror("fopen()");
            break;
        }   
        count++;
    }
    
    printf("count = %d\n",count);
    fclose(fp);

    exit(0);
}
```

## 2、输入输出函数

### (1)字符相关

`fgetc()`

`fputc()`

```c
int fgetc(FILE *stream);
```

```  c
int fputc(int c,FILE *stream);  //c设定的输出项
```

> 实例：使用现有文件构建程序mycopy.c，
> `while` 循环使用 `fgetc` 从源文件（`fps`）读取字符，并使用 `fputc` 将它们写入目标文件（`fpd`）。随着读取字符，`fps` 中的文件位置指示器会自动向前移动，因此下一次调用 `fgetc` 将读取文件中的下一个字符。同样，使用 `fputc` 写入字符时，`fpd` 中的文件位置指示器也会自动向前移动。
>
> 用法：./mycopy srcfile destfile

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

int main(int argc, char **argv)
{

    FILE *fps, *fpd;
    int ch;
    
    if (argc < 3)
    {
        fprintf(stderr,"Usage...\n");
        exit(1);
    }

    fps = fopen(argv[1], "r");
    if (fps == NULL)
    {
        perror("fopen()");
        exit(1);
    }

    fpd = fopen(argv[2], "w");
    if (fpd == NULL)
    {
        fclose(fps);
        perror("fopen()");
        exit(1);
    }

    while (1)
    {
        ch = fgetc(fps);
        if (ch == EOF)
            break;

        fputc(ch, fpd);
    }

    fclose(fpd);
    fclose(fps);

    exit(0);
}
```

> 实例：获取文件中有效字符数的函数fgetc.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

/**
 * 获取一个文件中具有多少个有效字符
 * 用法：./fgetc srcfile
 */
int main(int argc, char **argv)
{
    FILE *fp;
    int count;

    if (argc < 2)
    {
        fprintf(stderr, "Usage:%s <srcfile>\n", argv[0]);
        exit(1);
    }

    fp = fopen(argv[1], "r");
    if (fp == NULL)
    {
        perror("fopen()");
        exit(1);
    }

    while (fgetc(fp) != EOF)
    {
        count++;
    }

    printf("count = %d\n", count);
    fclose(fp);

    exit(0);
}
```



====================================================================================================

### （2)字符串相关

`fgets()`读取一整行的内容

`fputs()`

```c
char *fgets(char *s, int size, FILE *stream);
int fputs(const char *s, FILE *stream);
```

有两种情况可以导致fgets函数的正常结束

1. 读到size-1字节的数据
2. 读到换行符号“\n”

> 实例：重写mycopy.c为mycopy_fgets.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

#define BUFSIZE 1024

int main(int argc, char **argv)
{

    FILE *fps, *fpd;
    char buf[BUFSIZE];

    if (argc < 3)
    {
        fprintf(stderr, "Usage...\n");
        exit(1);
    }

    fps = fopen(argv[1], "r");
    if (fps == NULL)
    {
        perror("fopen()");
        exit(1);
    }

    fpd = fopen(argv[2], "w");
    if (fpd == NULL)
    {
        perror("fopen()");
        exit(1);
    }

    while (fgets(buf, BUFSIZE, fps) != NULL)
    {
        fputs(buf, fpd);
    }

    fclose(fpd);
    fclose(fps);

    exit(0);
}
```



====================================================================================================

`fread()`存在使用风险，建议使用的时候就是单字节实现

`fwrite()`

```c
size_t fread(void *ptr,size_t size,size_t nmemb,FILE *stream);
size_t fwrite(const void *ptr, size_t size,size_t nmemb,FILE *stream);
```

> 实例：重写mycopy.c为mycopy_fread.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

#define BUFSIZE 1024

int main(int argc, char **argv)
{

    FILE *fps, *fpd;
    char buf[BUFSIZE];
    int n;

    if (argc < 3)
    {
        fprintf(stderr, "Usage...\n");
        exit(1);
    }

    fps = fopen(argv[1], "r");
    if (fps == NULL)
    {
        perror("fopen()");
        exit(1);
    }

    fpd = fopen(argv[2], "w");
    if (fpd == NULL)
    {
        perror("fopen()");
        exit(1);
    }

    while ((n = fread(buf, 1, BUFSIZE, fps)) > 0)
    {
        fwrite(buf, 1, n, fpd);
    }

    fclose(fpd);
    fclose(fps);

    exit(0);
}
```

====================================================================================================

`printf() scanf()`不谈

输出：

`fprintf()`

`sprintf()`可以将多类型的数据转化成一个整体的串

```c
int fprintf(FILE *stream, const char *format, ...);
int sprintf(char *str, const char *format, ...);
```

> 实例程序sprintf.c

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    char buf[1024];
    int year = 2023,month = 8,day = 30;
    
    sprintf(buf,"%d-%d-%d",year,month,day);
    
    puts(buf);
    exit(0);
}
```

`atoi`->将一个字符串转换成整数，相当于是sprintf的反向函数

> 实例atoi.c

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    char str[] = "123456";

    printf("%d\n",atoi(str));
    exit(0);
}
```

`snprintf`:因为sprintf不检查缓冲区达大小，为了防止越界，提出snprintf

```c
int snprintf(char *str, size_t size, const char *format, ...);
```

输入：

` fscanf()` 

`sscanf()`

```c
int fscanf(FILE *stream, const char *format, ...);
```

====================================================================================================



## 3、寻找文件位置函数（操作文件位置指针）

> 文件位置指针解决的问题
>
> fp = fopen();
>
> fputc(c,fp) * 10 向fp写入10个字符
>
> fgetc(fp) * 10   从fp中拿取10个字符
>
> 这样是不能够拿到对应输入的功能，因为在执行fputs函数时，文件位置指针已经向后移动了，所以函数fgetc读取得到的字符是上面输入字符之后的字符，此时就需要用到操作文件位置指针的函数

========================================================================================

`fseek()`寻找文件位置，定位offset是偏移量，whence是指相对位置，开始处(SEEK_SET)，当前位置(SEEK_CUR)，尾部(SEEK_END)

```c
int fseek(FILE *stream, long offset, int whence);
```

那么上面的问题可以这样解决

```c
fp = fopen();

fputc(c,fp) * 10; // 向fp写入10个字符

fseek(fp,-10,SEEK_CUR);

fgetc(fp) * 10 ;  //从fp中拿取10个字符
```

========================================================================================



`ftell()`报告当前文件指针位置

```c
long ftell(FILE *stream);
```

因为long型的数据大小在不同的系统下有着不同的定义，如果在32位操作系统下，那么long型的数据就是-2G~2G-1大小，那么理论上，fseek函数可以从当前位置whence前后2G的位置，所以可以接收的文件大小为4G,但是在文件系统中找不到负值的文件位置,所以ftell函数只能够适用于2G的文件大小，所以当fseek和ftell配合使用的时候，实际可操作的文件大小为2G。

所以出现了fseeko()和ftello(),解决了文件大小问题`但是不适用于c99`

```c
int fseeko(FILE *stream, off_t offset, int whence);

off_t ftello(FILE *stream);
```

> 利用fseek和ftell函数来重构获取文件大小的函数flen.c
>
> 使用方法：./flen srcfile

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

/**
 * 获取一个文件中具有多少个有效字符
 * 用法：./flen <srcfile>
 */
int main(int argc, char **argv)
{
    FILE *fp;

    if (argc < 2)
    {
        fprintf(stderr, "Usage:%s <srcfile>\n", argv[0]);
        exit(1);
    }

    fp = fopen(argv[1],"r");
    if(fp == NULL)
    {
        perror("fopen()");
        exit(1);
    }

    fseek(fp,0,SEEK_END); //寻找到文件尾部
    printf("%ld\n",ftell(fp));

    fclose(fp);

    exit(0);
}
```

========================================================================================



`rewind()`不管当前位置在什么地方，直接寻找到文件开始处，相当于fseek(fp,0L,SEEK_SET);

```c
void rewind(FILE *stream);
```



fseek通常还有一个作用是用来产生空洞文件，空洞文件可以提前占据一定量的内存空间，比如说在下载文件的时候，在开始的时候会直接拉开一个原文件大小的空洞文件。

```c
FILE *fp;
fseek(fp,size,SEEK_SET); //会占据size大小内存的位置
```

## 4、强制刷新输出缓冲区:star2:

`fflush()`

```c
int fflush(FILE *stream);
```

> 实例程序fflush.c

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    printf("Before while");

    while (1);

    printf("After while");
    exit(0);
}
```

上段程序猜想是能够打印Before while，然后程序卡死，但是终端实际上连Before while都无法输出。因为标准输出是行缓冲的模式，只有当行缓冲区满了之后，才会刷新缓冲区，意味着输出。所以修改上述程序

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    printf("Before while");

    fflush(stdout);  //刷新所有打开的输出流，行缓冲模式
    while (1);

    printf("After while");
    exit(0);
}
```

或者在printf中添加"\n"

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    printf("Before while\n");
    
    while (1);

    printf("After while\n");
    exit(0);
}
```

:star:缓冲区的作用

大多数情况下是好事，合并系统调用

- [x] 行缓冲 1、换行的时候刷新 2、行满了的时候刷新 3、强制刷新（标准输出是这样的，因为是终端设备）
- [x] 全缓冲 1、行满了的时候刷新 2、强制刷新（只要不是终端设备）
- [x] 无缓冲 需要立即输出内容 如stderr

==========================================================================================

`如何完整获得一行文件内容(有多少拿多少)`

`getline()`这个函数的主要作用是帮助你以更安全和方便的方式读取变长的文本行，而不需要手动管理缓冲区的大小。函数的声明如下：

```c
ssize_t getline(char **lineptr, size_t *n, FILE *stream);
```

`getline` 函数的主要优点是它可以自动处理分配和重新分配缓冲区的问题，以适应变长的文本行，从而简化了文件读取的代码。只需要提供一个指针来接收文本行，`getline` 会负责分配、重新分配和管理内存。`它还返回读取的字符数，或者在出现错误时返回 -1`，以帮助检测是否成功读取了一行文本。这使得处理不定长度的文本行变得更加方便。

> 输出每一行有多少给有效字符getline.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char **argv)
{
    FILE *fp;
    char *linebuf;
    size_t linesize;
    size_t ret;

    if(argc < 2)
    {
        perror("Usage...");
        exit(1);
    }

    fp = fopen(argv[1],"r");
    if(fp == NULL)
    {
        perror("fopen()");
        exit(1);
    }

while (1)
{
    ret = getline(&linebuf,&linesize,fp);  //当读取失败时，返回-1
    if(ret < 0)  //不希望出现不能够控制的分支，直接写<0
        break;
   	
    printf("%d\n",strlen(linebuf));
}

    fclose(fp);

    exit(0);
}
```

============================================================================================

`临时文件`：对用户体提交的数据进行保存，进行后续操作

- [x] 如何不冲突地创建临时文件
- [x] 及时销毁临时文件



`tmpnam()`为一个临时文件创建一个可用的名字，因为该函数创建一个文件需要两步，首先是获取一个名称，再去为这个文件分配内存，因为不是`原子化操作`，所以在高并发的情况下，有可能前面一个文件先获得文件名后，还没来得及去申请内存，这时候第二个tmpnam操作来了，那么会获取同样一个名字，这就会导致文件创建冲突。所以该函数使用存在风险

`tmpfile()`创建一个临时文件，直接创建，创建的是匿名文件，ls寻找不到，全程操作FILE*指针，因为没有名字，所以不会冲突。

```c
char *tmpnam(char *s);
FILE *tmpfile(void); 
```



# 二、文件IO/系统调用IO

**文件描述符是文件IO中贯穿始终的类型**

> 文件描述符fd的概念（整型数、数组下标、优先使用当前数组可用范围内最小的下标）

在**标准IO**中，FILE贯穿始终，当打开一个文件时会产生一个结构体中，其中必定有一个指针为pos，指定当前的位置，**标准IO是依赖文件IO实现的**。在**系统调用IO**中，每一个文件一定会有自己的唯一标识inode，也会产生一个结构体，该结构体会包含其对应的文件所有的信息，结构体会包含pos指针。文件描述符的本质是整型数，根据这个整型数去获得指针，通过指针找到文件操作结构体，从而操作文件。每次打开一个文件都会产生一个结构体，**无论是不是open同一个文件**。在结构体当中还存在一个计数器，记录着文件被引用的次数（该结构体本身被依赖的次数），当有多个指针指向该结构体时，会通过判断计数器的数值来决定要不要在释放指针的时候释放掉结构体内存。**在一个系统中能够打开的文件最多的个数其实就是这个文件描述符数组的大小，可以通过ulimit -a指令查看open files数值。**

> 文件IO(系统调用IO)操作：open,close,read,write,lseek（文件描述符的创建和对其的操作）

**open/close**获取一个文件描述符和释放一个文件描述符号

```c
int open(const char *pathname, int flags)
int open(const char *pathname, int flags, mode_t mode)
int close(int fd)
```

当参数flags中包含CREATE时候，需要使用第二种open实现方式，反之，使用第一种。两个open的实现是由变参函数完成，而不是函数重载。**变参和重载**两者的区分是通过向它输入多个参数，如果编译能够通过，没有报错，就是变参实现的，反之是重载，因为重载的参数个数是固定的。

pathname是文件地址，flags是位图（当前使用权限问题），open成功会返回一个文件描述符，失败会返回-1，并且输出errno信息。

buf理解成写的缓冲区，cache理解成读的缓冲区。

文件的权限问题

标准IO   ---->    系统调用IO

r -> O_RDONLY

r+ -> O_RDWR

w -> O_WRONLY | O_CREAT | O_TRUNC

w+ -> O_RDWR | O_CREAT | O_TRUNC

> 读写操作

**read/write**

> read()成功的时候会返回成功读取的字符数，失败会返回-1，记录错误到errno中
>
> write()成功的时候会返回成功写入的字节数，返回值是0的时候代表没有写进去，失败会返回-1，记录错误到errno中
>
> lseek()文件定位操作，offset是偏移量，whence是相对偏移位置，开始处(SEEK_SET)，当前位置(SEEK_CUR)，尾部(SEEK_END)相当于是fseek和fseeko的综合使用。

```c
ssize_t read(int fd, void *buf ,size_t count)
ssize_t write(int fd, const void *buf ,size_t count)
off_t lseek(int fd, off_t offset, int whence)
```

> **实例：**重构获拷贝文件的程序

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

#define BUFSIZE 1024

int main(int argc, char **argv)
{
    
    int sfd,dfd,len,ret,pos;
    
    char buf[BUFSIZE];
    
    if(argc < 3)
    {
        fprintf(stderr,"Usage...\n");
        exit(1);
    }
    
    sfd = open(argc[1],O_RDONLY);
    if(sfd < 0)
    {
        perror("open() 出错");
        exit(1);
    }
    
    dfd = open(argv[2],O_WRONLY | O_CREAT, O_TRUNC, 0600);
    if(dfd < 0)
    {
        close(sfd);
        perror("打开失败");
        exit(1);
    }
    
    while(1)
    {
       len = read(sfd,buf,BUFSIZE); //成功读取的文件字节数
       
       if(len < 0)
       {
           perror("read()出错");
           break;
       }
       
        if(len == 0)  //已经读完，或者由于阻塞进程时间过长，被中断程序打断读取
        {
            break;
        }
        
        pos = 0;
        
        while(len > 0)  //防止写入的字节数小于读取的字节数，处理意外情况
        {
            ret = write(dfd,buf+pos,len); //成功写入的字节数
            if(ret < 0)
              {
                 perror("write()出错");
                 exit(1);
              }
            pos += ret;
            len -= ret;
        }
    }

    close(dfd);
    close(sfd);  //最后关闭被依赖的文件描述符
   
    exit(0);
}
```



> 文件IO和标准IO的区别

举例：**传达室老大爷跑邮局**

从标准IO的角度来讲的话，传达室的老大爷具有20封信的缓冲区，当老大爷收到的信个数达到缓冲区上限或者有紧急的信需要处理时，老大爷回去跑一趟邮局，对应于文件的flush操作，刷新缓冲区。

从系统调用IO的角度来看的话，传达室的老大爷是拿到一封信就跑一趟邮局（从user态切换成kernel态再回到user态）。

**标准IO具有缓冲区，吞吐量大，系统调用IO是即时处理，响应速度快。**所以使一个程序变快可以分为吞吐量和响应速度两个角度来看。

**标准IO和系统调用IO不能够混用！！！！**标准IO依赖于系统IO实现。

原因是：标准IO是具有缓冲区的,写操作中标准IO结构体的pos参数和系统调用IO结构体里面的pos参数是不一致的。

```c
int main()
{
    putchar('a');
    write(1,"b",1);
    
    putchar('a');
    write(1,"b",1);
 
    putchar('a');
    write(1,"b",1);
    
    exit(0);
}
```

输出是：bbbaaa  并非是：ababab，因为write()是系统调用，立即执行，putchar()是标准IO操作，会先放置在缓冲区中，再调用系统调用IO一次性执行。

**两者间的转换：**

fileno()函数可以将标准IO转换成系统调用IO

fdopen()函数可以将系统调用IO转换成标准IO

```c
int fileno(FILE *stream);
FILE *fdopen(int fd, const char *mode);
```



> IO的效率问题

time ./task

time命令计算后面的任务执行时间

real记录的是真正的用户体验时间，大小是user＋sys时间还大出一点点，因为还要加上一些任务调度时间。

> 文件共享：多个任务共同操作一个文件，或者协同完成一个文件任务

面试题：写程序删除一个文件的第十行

补充函数：truncate()/ftruncate()  -> 可以将一个文件截断到多长

**分析**：应该找到第十行的行首和第十一行的行首，从第十一行开始不断覆盖前面的行内容，达到覆盖第十行的效果。

**不同的思路**：

```c
伪代码：
/***************在一个程序中打开同一个文件两次**********************/
1 -> open r -> fd1 -> lseek 11        首先以只读的形式打开文件，找到文件的第11行
2 -> open r+ -> fd2 -> lseek 10       再以读写的方式（不要用只写的方式打开文件，因为要先保证文件存在），得到文件描述符fd2，找到文件的第十行
    
while()
{
    1 -> fd1 -> read
    2 -> fd2 -> write    通过fd1进行读，fd2进行写，循环完成整个文档的删除第十行的操作
}
/**************************************************************/
//通过两个进程完成，一个进程负责读，一个进程负责写
process1 -> open -> r
process2 -> open -> r+
（需要进程间通信，同样可以通过线程实现）
p1 -> read -> p2 -> write   进程1读的时候会将读到的内容保存在cache中，传递给进程2，让它重新写到文件中。
/***************************************************************/
```

> 原子操作:

原子：不可分割的最小单位操作，解决竞争和冲突。

比如tmpnam的操作就不是原子操作，会产生一系列问题

> 程序中的重定向：dup,dup2

通过操作文件描述符来完成将puts()输出的文字输出在文件中。

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

#define FNAME "/tmp/out"

int main()
{
    
    close(1);  //关闭文件描述符1，代表着文件标准输出的位置
    int fd = open(FNAME,O_WRONLY | O_CREAT | OTRUNC,0600);   //此时打开的是FNAME文件，文件描述符是优先使用最小的可用位置，因为前面关闭了fd=1的位置，所以下次打开的是fd=1的位置，即标准输出的位置。
    if(fd < 0)
    {
        perror("open()失败");
        exit(1);
    }
/*******************************/
    puts("hello!");   //要求不要输出在中断，输出在指定的文件内
    
    exit(0);
}
```

使用dup函数完成相同的功能（dup将复制一个老的文件描述符fd，拷贝到当前可用范围内最小的的一个文件描述符）

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

#define FNAME "/tmp/out"

int main()
{
    int fd = open(FNAME,O_WRONLY | O_CREAT | OTRUNC,0600);
    if(fd < 0)
    {
        perror("open()失败");
        exit(1);
    }
    
    close(1);
    dup(fd);  //此时fd指向1，因为老fd在上面已经定义，可用范围内最小的文件描述符是上面关闭的1
    
    close(fd);
/*******************************/
    puts("hello!");   //要求不要输出在中断，输出在指定的文件内
    
    exit(0);
}
```

但是上面的程序存在问题，因为并不能保证每个文件刚开始文件描述符数组位置1的位置是启用的，当某个文件的文件描述符fd=1的位置一开始就是关的话，那么int fd = open(FNAME,O_WRONLY | O_CREAT | OTRUNC,0600);这里取到的就是文件描述1，下面又将文件描述符1关闭了，所以dup失败。还有另外一个情况，在多进程的情况下，就不能保证一定复制到的位置就是想要的位置。根本原因就是 close(1);和 dup(fd);这两个操作不是原子性的。

**采用dup2来解决上面的问题**。

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

#define FNAME "/tmp/out"

int main()
{
    int fd = open(FNAME,O_WRONLY | O_CREAT | OTRUNC,0600);
    
    if(fd < 0)
    {
        perror("open()失败");
        exit(1);
    }
    
    dup2(fd,1); //这里就是先关闭文件描述符1，再将fd的内容复制到1的位置
    
    if(fd != 1)
    {
         close(fd);
    }
/*******************************/
    puts("hello!");   //要求不要输出在终端，输出在指定的文件内
    
    exit(0);
}
```

采用dup2()这样解决了`原子性操作`的问题，但是fd本身就是1的情况还没有被解决。采用上面的if(fd != 1)解决。因为dup2()在发现新fd=旧fd时将不会做任何事情，直接返回新的fd。

最后还应该将输出设备改成到终端（要保持写的是小模块而不是main函数的思想去写代码，尽量保持程序进到你的函数是什么状态出去之后还是什么状态）

> 同步：sync,fsync,fdatasync

sync(): 同步内核层面的数据，在解除设备挂载或者说是关机时候，需要将buf和cache中的数据同步到磁盘中

fsync()：指定同步一个文件的buf和cache

fdatasync(): 只同步数据不同步亚数据，亚数据指的是文件修改时间属性之类的，不是数据内容本身

```c
void sync(void);
int fsync(int fd);
int fdatasync(int fd);
```



> fcntl():管家级别的函数

文件描述符所变的魔术几乎都来源于该函数，管理文件描述符

```c
int fcntl(int fd,int cmd,.../*arg/)
```

fcntl()的返回值会跟随cmd的不同而改变

> ioctl():设备相关的内容,管理设备

> /dev/fd目录：是一个虚目录，显示的是当前进程文件描述符的信息，谁打开看就显示谁的文件描述符信息
>
> 比如 ls -l /dev/fd/  显示的就是ls命令进程的文件描述符信息

# 三、文件系统

> 一、目录和文件

1、**获取文件属性**

类似于ls

一个命令：cmd --长格式 -短格式  非选项的传参

在任何命令之后加上--都表示之后的选项都属于非选项的传参

比如：touch -a 不成功，但是touch -- -a能够创建名为-a的文件

或者也可以 touch ./-a 指定当前路径

**函数**：stat结构体、stat()、fstat()、lstat()

```c
int stat(const char *pathname, struct stat *statbuf);
int fstat(int fd, struct stat *statbuf);
int lstat(const char *pathname, struct stat *statbuf);
```

`stat结构体`

```c
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

               /* Since Linux 2.6, the kernel supports nanosecond
                  precision for the following timestamp fields.
                  For the details before Linux 2.6, see NOTES. */
           	   struct timespec st_atim;  /* Time of last access */
               struct timespec st_mtim;  /* Time of last modification */
               struct timespec st_ctim;  /* Time of last status change */

           #define st_atime st_atim.tv_sec      /* Backward compatibility */
           #define st_mtime st_mtim.tv_sec
           #define st_ctime st_ctim.tv_sec
}

```

stat:通过文件路径获取属性，面对符号链接文件时获取的是所指向的目标文件的属性，将获得的文件属性信息填入到buf地址中。

fstat:通过文件描述符获取属性

lstat:面对符号链接文件时获取的是符号链接文件的属性

> 实例：通过以上函数实现获取文件长度的函数flen.c
>

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

static off_t flen(const char *fname)
{
    struct  stat statres;
    
    if(stat(fname,&statres) < 0)
    {
        perror("stat()");
        exit(1);
    }

    return statres.st_size;
}

int main(int argc,char **argv)
{

    if(argc <  2)
    {
        fprintf(stderr,"Usage...\n");
        exit(1);
    }

    printf("%lld\n",(long long)flen(argv[1]));

    exit(0);
}
```

**空洞文件**

在文件的stat()结构体中，st_size并不等于st_blksize * st_blocks，一个文件的实际大小是st_blksize * st_blocks

> 实例: 生成一个超级大的文件，但是大多数空间都是空的

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <fcntl.h>

int main(int argc,char **argv)
{
    int fd;

    if(argc <  2)  //输入参数个数不正确
    {
        fprintf(stderr,"Usage...\n");
        exit(1);
    }

    fd = open(argv[1],O_WRONLY|O_CREAT|O_TRUNC,0600);
    if(fd < 0)
    {
        perror("open()");8
        exit(1);
    }

    lseek(fd,5LL*1024LL*1024LL*1024LL-1LL,SEEK_SET);  //一下子拉取5G空间，会发生整数溢出
    write(fd,"",1);  //只写入一个字节

    close(fd);

    exit(0);
}

```

linux下的size值并不是文件实际所占的大小，只是文件的一个属性而已，blocks存储的是具体的文件所占block分区的个数，将它乘以block大小就是实际的文件大小。在windows中文件的size值代表的就是实际的文件所占磁盘大小，但是linux环境下，就只是文件的属性，不代表实际值。



2、**文件访问权限**

一般上，文件的权限信息格式为：-rwxrwxrwx ,分别代表着（文件属性）1位（文件所有者的权限）3位（同组的用户权限）3位（其他用户的权限）3位

**文件类型**

> 测试文件类型

在linux系统下，一共包含七中文件类型：dcb-lsp,依次是目录，字符设备，块设备，常规文件，链接文件，socket文件，命名管道

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <fcntl.h>

static int ftype(const char*fname){
    
    struct stat statres;
    if(stat(fname,&statres) < 0 )
    {
        perror("stat()");
        exit(1);    
    }

    if(S_ISREG(statres.st_mode))
        return '_';
            else if(S_ISDIR(statres.st_mode))
                return 'd';
                else if(S_ISSOCK(statres.st_mode))
                    return 's';
                    else
                        return '?';

}

int main(int argc,char **argv)
{
    int fd;

    if(argc <  2)  //输入参数个数不正确
    {
        fprintf(stderr,"Usage...\n");
        exit(1);
    }

    printf("%c",ftype(argv[1]));

    exit(0);
}

```

st_mode是一个16位的位图，用于表示文件类型，文件访问权限，以及特殊权限位，文件所有的属性信息都在stat.st_mode之上



3、**umask**

`umask` 是一个重要的概念，用于设置默认的文件和目录权限掩码。它决定了在创建新文件或目录时，系统会自动为其设置的默认权限。`umask` 值中的每一位都代表了一种权限，包括读取、写入和执行权限。

`umask` 的值是一个三位八进制数，通常以四位数表示（前面的 0 也是八进制的一部分），它表示了权限的掩码。这个掩码会从默认的权限中去掉相应的权限位。

具体来说，每个权限位对应的数字表示如下：

- 读取权限：4
- 写入权限：2
- 执行权限：1

例如，如果 `umask` 值是 `022`，那么在创建新文件时，系统会将权限 `666`（八进制）减去 `022`，得到最终的权限 `644`（八进制），即用户拥有读写权限，而组和其他用户只有读权限。

同样地，如果 `umask` 值是 `002`，在创建新目录时，系统会将权限 `777`（八进制）减去 `002`，得到最终的权限 `775`（八进制），这将赋予用户和组读写执行权限，而其他用户只有读和执行权限。

`umask` 的设置是在用户登录时通过 shell 配置文件（如 `.bashrc` 或 `.bash_profile`）或全局配置文件（如 `/etc/profile` 或 `/etc/login.defs`）中完成的。可以使用 `umask` 命令来查看或设置当前的 `umask` 值。

总之，`umask` 在系统中起到重要的作用，确保了新创建的文件和目录不会默认具有过于宽松的权限，从而提升了系统的安全性。



4、**文件权限更改、管理**

函数chmod() fchmod()

```c
int chmod(const char *pathname, mode_t mode);
int fchmod(int fd, mode_t mode);
```

修改文件的权限

- [x] 通过数字来配置，chmod 666 a.c 那么a.c的权限就会变成rw_rw_rw_
- [x] 通过字母来配置，chmod a+x a.c 那么a.c的所有用户将会获得执行它的权限，要是仅仅需要某些用户获得权限，可以 u+x  g+x o+x，分别代表着用户获得，同组用户获得，其他用户获得。



5、**粘住位**

t位，给某一个二进制可执行命令设置当前t位，将某一个命令的使用痕迹进行保留，为的是再次装载此模块的速度比较快。

常用于给目录设置t位，根目录上面的tmp具有t位，当文件具有t位时，每个用户对于目录文件的操作就会变得特殊化。



6、**文件系统**：**FAT**,**UFS**

文件系统: 文件或者数据的存储格式问题(存储或管理)，在不同的文件系统中，文件的存储形式就会不相同。

两种不同文件系统的差别：

**FAT16/32**：实际上是`静态单链表`存储,缺点是由于单链表存储，文件走向只能是单向的，承载能力有限。

**UFS文件系统**：inode是一个结构体，有一个文件几乎所有的内容，首先是12个指针，内部还包含有三级指针结构，256K，256*256k,256 * 256 * 256k，所以UFS文件系统不怕大文件。文件名存在于目录文件中，并不是inode中。目录文件包含inode和fname信息。inode位图会记录各个inode的使用情况，用或是空闲，以1/0标识



7、**硬链接，符号链接（软链接）**

文件硬链接：ln src des ，硬链接文件和源文件的inode相同，链接之后，源文件的硬连接数Links会增长，有点像两个指针指向同一个文件数据块，当源文件被删除，硬链接文件依然存在。

符号链接（软链接）：ln -s src des,像是windows里面的快捷方式文件，源文件和符号链接文件是独立的两个文件，两者的inode是不相同的。

硬链接特点：与目录项是同义词，且建立硬链接有限制:不能给分区建立，不能给目录建立，符号链接都可以

8、**utime**

更改的是最后一次读写的时间，atime,mtime

9、**目录创建和销毁**

命令：mkdir，rmdir（必须空）

10、**更改当前工作路径**

cd（chdir）、chroot(假根安全机制)、getcwd()获取当前工作的路径

11、**分析目录**、**读取目录内容**

glob()进行目录解析,可以实现下面所有函数的功能，解析模式/通配符

> **glob函数应用**:查看/etc目录下面有多少a*.conf文件

```c
#include <stdio.h>
#include <stdlib.h>
#include <glob.h>

#define PAT "/etc/a*.conf"

static int errfunc_(const char *errpath,int eerrno)
{
    puts(errpath);
    fprintf(stderr,"ERROR MSG:%s\n",stderror(eerror));
    return 0;
}

int main()
{
    glob_t globres;
    int i;
    
    int err = glob(PAT,0,NULL,errfunc_,&globres);
    
    if(err)
    {
        printf("Error code = %d\n",err);
        exit(1);
    }
    
    for(i = 0;i < globres.gl_pathc; i++)
    {
        puts(globres.gl_pathv[i]);
    }
    
    globfree(&globres);
    
    exit(0);
}
```

opendir()

closedir()

readdir()

> 实例：得到./etc目录下所有的文件（和上面的glob实现相同的功能）
> 其中：`struct dirent` 是用于表示目录项的数据结构，通常在Unix和Linux系统中使用。这个结构通常包含有关文件和目录的信息，例如文件名、文件类型和文件的inode号等。`struct dirent` 是 POSIX 标准中定义的，用于在C程序中进行目录遍历和操作。
>
> struct dirent {
>     ino_t d_ino;       // inode number
>     off_t d_off;       // offset to the next dirent
>     unsigned short d_reclen; // length of this record
>     unsigned char d_type;   // type of file
>     char d_name[];     // filename
> };

```c
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>

#define PAT "/etc"

int main()
{
   DIR *dp;
   struct dirent *cur;
   
   dp = opendir(PAT);
   if(dp == NULL)
   {
       perror("opendir()");
       exit(1);
   }
    
   while(cur = readdir(dp) != NULL)
   {
       puts(cur -> d_name);
   }
   
   closedir(dp);
   
   exit(0); 
}
```

rewinddir()

seek()

telldir()

**du命令**可以查看一个目录或者文件所占磁盘的大小，默认以k为单位。

> 实例：实现一个mydu程序，实现和du一样的功能(递归实现获得对于文件或文件夹所占数据块的大小数值)

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>
#include <glob.h>
#include <string.h>

#define PATHSIZE 1024

static int path_noloop(const char* path)
{
    char * position;
    position = strrchr(path,'/');
    if(position == NULL)//没有找到该字符
    {
        exit(1);
    }

    if(strcmp(position+1,".") == 0 || strcmp(position+1),".." == 0)
        return 0;
    
    return 1;
}

static int64_t mydu(const char *path)
{
    struct stat statres;
    char nextpath[PATHSIZE];
    glob_t globres;
    int err;
    int i;
    int sum = 0;

    if(lstat(path,&statres) < 0)  //利用lstat来做属性分析，因为对于链接文件，stat函数处理有出入
    {
        perror("lstat()");
        exit(1);
    }

//非目录文件
   if(!S_ISDIR(statres.st_mode)) 
   {
        return statres.st_blocks;
   }

//目录文件
   strncpy(nextpath,path,PATHSIZE);  //复制文件路径
   strncat(nextpath,"/*",PATHSIZE);  //追加/*

   err = glob(nextpath,0,NULL,&globres);  //获得nextpath路径下所有的文件
    if(err)
    {
        printf("Error code = %d\n",err);
        exit(1);
    }


    strncpy(nextpath,path,PATHSIZE);
    strncat(nextpath,"/.*",PATHSIZE);
    
    err = glob(nextpath,GLOB_APPEND,NULL,&globres);  //获得nextpath路径下所有的文件,GLOB_APPEND将所有隐藏的结果追加到globres中
    if(err)
    {
        printf("Error code = %d\n",err);
        exit(1);
    }
 
    //递归解析目录下所有文件
    for(i = 0; i < globres.gl_pathc; i++)
    {
        if(path_noloop(globres.gl_pathv[i]))  //保证当前文件一定没有回路，因为文件结构不一定是典型的树形结构
        {
            sum += mydu(globres.gl_pathv[i]);
        }
    }

    sum += statres.st_blocks;

    return sum;

}

int main(int argc,char **argv)
{
    if(argc < 2)
    {
        fprintf(stderr,"Usage...\n");
        exit(1);
    }

    printf("%lld\n",mydu(argv[1])/2);

    exit(0);
} 
```



> 二、系统数据文件和信息

1、/etc/passwd

FreeBSD ->BDB轻量级数据库存储用户信息

HP UNIX -> fs

passwd包含字段:

用户名，加密口令、用户ID、组ID、注释字段、登陆项

查询函数：getpwuid()、getpwnam()

> 实例：通过用户名id查找用户其他相关信息

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <pwd.h>

int main(int argc, char **argv)
{

    struct passwd *pwdline;

    if(argc < 2)
    {
        fprintf(stderr,"Usag.....\n");
        exit(1);
    }

    pwdline = getpwuid(atoi(argv[1]));

    puts(pwdline->pw_name);

    exit(0);
}
```

2、/etc/group

getgrgid()、getgrgrnam()



3、/etc/shadow阴影文件

作为普通用户，权限不够。

hash 混淆 不可逆 如果原先的串相同，所得串也相同，防备管理员监守自盗

加密（安全）：攻击成本大于收益

安全？穷举：口令随机校验

shadow文件格式:**$**加密方式**$**原串或上杂子串**$**加密之后的密码

getspnam()通过用户名获得shadowpasswd所有信息

crypt()加密函数



4、时间戳

time_t

time()从内核中取时间

gmtime()将时间转换成年月日结构体

localtime()将时间转换成年月日结构体

mktime()由结构体转换成时间time大整数

strftime()格式化字符串->格式化时间和日期

asctime()将结构体转换成字符串

ctime()将time大整数转换成字符串



> 三、进程环境

1、**main函数**

int main(int argc,char **argv)

2、**进程终止方式**

`正常终止`：

1. 从main函数返回，进程的return返回值是给其父进程的
2. 调用exit()
3. 调用__exit()或者是_ _Exit()，这两个是系统调用，exit()依赖于它们实现
4. 最后一个线程从其启动例程返回
5. 最后一个线程调用pthread_exit()

在正常情况下使用 `exit()`，因为它允许程序做一些必要的清理工作。但在某些情况下，如果需要立即终止进程，而不关心清理操作，可以使用 `_exit()`。

`异常终止`：

1. 调用abort()：`abort()` 是一个C标准库函数，用于立即终止程序的执行。它的主要目的是在程序发生严重错误或不可恢复的问题时，提供一种可靠的方法来终止程序的执行。
2. 接到一个`信号`并终止
3. 最后一个线程对其取消请求作出响应

`atexit()：钩子函数`

> 实例：实际的运行效果是先打印Begin、End,再在即将调用exit()之前再调用钩子函数，以**逆序**的方式调用，输出是f3/f2/f1

```c
#include <stdio.h>
#include <stdlib.h>

static void f1(void)
{
    puts("f1() is working!");
}

static void f2(void)
{
    puts("f2() is working!");
}

static void f3(void)
{
    puts("f3() is working!");
}

int main(int argc, char **argv)
{

    puts("Begin!");

    atexit(f1);  //钩子函数
    atexit(f2);
    atexit(f3);

    puts("End!");

    exit(0);
}
```

3、**命令行参数分析**

getopt()

getopt_long()

> 实例：通过配置命令行参数去获得信息
>
> `localtime()` 是C标准库函数，用于将时间戳（表示自 1970 年 1 月 1 日以来的秒数）转换为本地时间表示。它将一个时间戳作为参数，并返回一个指向 `struct tm` 结构的指针，该结构包含了本地时间的各个组成部分，如年、月、日、时、分、秒等。原型如下：
>
> ```c
> struct tm *localtime(const time_t *timer);
> ```

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <getopt.h>
#include <unistd.h>
#include <string.h>

/**
 * -y: year
 * -m: month
 * -d: day
 * -H: hour
 * -M: minute
 * -S: second
 */

#define TIMESTRSIZE 1024
#define FMTSTRSIZE 1024

int main(int argc, char **argv)
{
    int c;
    char fmtstr[FMTSTRSIZE];
    time_t stamp; //时间戳，这是一长串数字，代表从某个时间已经经过了多少时间
    struct tm *tm; //本地时间结构体。具有年月日时分秒等内容
    char timestr[TIMESTRSIZE];

    fmtstr[0] = '\0';

    stamp = time(NULL);
    tm = localtime(&stamp);

    while (1)
    {
        c = getopt(argc, argv, "H:MSy:md");
        if (c < 0)
        {
            break;
        }

        switch (c)
        {
        case 'H':
            if(strcmp(optarg,'12') == 0)
            {
                strncat(fmtstr,"%I(%P)",FMTSTRSIZE);
            }else if(strcmp(optarg,'24') == 0)
            {
                strncat(fmtstr,"%H",FMTSTRSIZE);
            }else
            {
                fprintf(stderr,"Invalid argument of -H\n");
            }
            break; 
        case 'M':
            strNcat(fmtstr, "%M ", FMTSTRSIZE);
            break;
        case 'S':
            strNcat(fmtstr, "%S ", FMTSTRSIZE);
            break;
        case 'y':
            if(strcmp(optarg,'2') == 0)
            {
                strncat(fmtstr,"%y ",FMTSTRSIZE);
            }else if(strcmp(optarg,'4') == 0)
            {
                strncat(fmtstr,"%Y ",FMTSTRSIZE);
            }else
            {
                fprintf(stderr,"Invalid argument of -y\n");
            }
            break;
        case 'm':
            strNcat(fmtstr, "%m ", FMTSTRSIZE);
            break;
        case 'd':
            strNcat(fmtstr, "%d ", FMTSTRSIZE);
            break;
        default:
            break;
        }
    }

    strftime(timestr, TIMESTRSIZE, fmtstr, tm);   //将时间信息格式化为字符串
    puts(timestr);

    exit(0);
}
```

4、**环境变量**

环境变量：key = value

存储大量的键值对，包括一些默认的设置

getenv()获取环境变量值

setenv()改变或者添加环境变量值

unsetenv()删除一个环境变量



5、**c程序的存储空间布局**

pmap()



6、**库**

动态库

静态库 

手工装载库

dlopen()

dlclose()

dlerror()

dlsym()查找符号

```txt
在C语言中，静态库（Static Library）和动态库（Dynamic Library）是两种用于组织和共享代码的不同方式，它们具有以下主要区别：

链接时机：

静态库：在编译时，静态库的代码会被完全复制到可执行文件中。这意味着可执行文件包含了静态库的所有代码，无需在运行时加载库。
动态库：在编译时，只包含了动态库的链接信息，而实际的库代码在运行时从外部加载。
文件大小：

静态库：可执行文件的大小较大，因为它包含了静态库的所有代码。每个使用该库的可执行文件都会包含一份库的拷贝。
动态库：可执行文件较小，因为它只包含了动态库的链接信息。实际的库代码只需在系统中保存一份，多个应用程序可以共享这一份库。
更新和维护：

静态库：如果需要更新静态库中的代码，所有依赖该库的应用程序都需要重新编译以使用新版本的库。
动态库：如果需要更新动态库中的代码，只需替换库的实际文件即可，不需要重新编译依赖库的应用程序。
内存使用：

静态库：每个应用程序都加载一份库的代码，因此可能会浪费内存，尤其是在有多个应用程序使用相同库的情况下。
动态库：库的代码在内存中只有一份，多个应用程序可以共享，从而节省内存。
兼容性：

静态库：可执行文件与特定版本的静态库绑定，如果库的API发生更改，可能需要修改和重新编译应用程序。
动态库：可执行文件与库的链接信息绑定，不需要重新编译，只需确保库的API兼容性。
```

7、**函数跳转**

压栈进行函数控制，goto不能够进行跨函数的跳转

下面的两个函数可以跨函数跳转(**安全跳转**):

setjmp()设置跳转点

longjmp()从某位置跳回到跳转点



8、**资源获取和控制**

getrlimit() ：获取资源限制

setrlimit() ：设置资源限制

rlimit结构体：包含软限制和硬限制，普通用户不能够增加自己的硬限制，但能够减少。root可任意增加或减少硬限制



# 四、进程

**1、进程标识符pid**

类型pid_t，有符号的十六位整型数

命令ps

进程号是顺次向下使用

`getpid()`获取当前进程的标识符

`getppid()`获取当前进程的父进程的标识符



**2、进程产生**

fork()复制当前进程创建子进程，副本和原本基本上一模一样，fork之后，在父进程中返回的是子进程的pid，在子进程中返回的是0，如果子进程失败，返回值是-1。

fork后父子进程的区别：fork的返回值不一样，pid不同，ppid不同，未决信号和文件锁不继承，资源利用量归零。

init进程：1号进程，是所有进程的祖先进程

在fork函数使用过程中刷新缓冲区fflush()的重要性 

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main()
{
    pid_t pid;
    printf("[%d]:Begin!\n",getpid());

    fflush(NULL);   //在fork之前必须刷新该刷新的流，不然会导致输出错误
    
    pid = fork();
    if(pid < 0)
    {
        perror("创建进程失败\n");
        exit(1);
    }

    if(pid == 0)   //子进程
    {
        printf("[%d]:Child is working!\n",getpid());
    }else      //父进程
    {
        printf("[%d]:Parent is working!\n",getpid());
    }

    printf("[%d]:End!\n",getpid());

    exit(0);
}
```

永远不要猜测父子进程谁先被调度，是由调度器的调度策略来决定父子进程谁先执行。

> 实例：使用多进程去寻找30000000到30000200之间的质数

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define LEFT 30000000
#define RIGHT 30000200

int main()
{
    pid_t pid;

    int i, j, mark;

    for (i = LEFT; i <= RIGHT; i++)
    {

        pid = fork();

        if (pid < 0)
        {
            perror("子进程创建失败");
            exit(1);
        }
 
        if (pid == 0)   //子进程
        {
            mark = 1;
            
            for(j = 2; j < i / 2; j++)
            {
                if(i % j == 0)
                {
                    mark = 0;
                    break;
                }
            }

            if (mark)
            {
                printf("%d is a primer\n", i);
            }

            exit(0);  //确保子进程正常结束，不进入到后续的循环中，创建更多的进程
        }
    }

    exit(0);
}

```

**父子进程关系：**当父进程先于子进程结束时，他所创建的子进程将变成僵尸进程，所有的僵尸进程将由init进程接管，虽然僵尸进程占据不了太多内存，但是他会占据pid，这是需要解决的问题,所以需要收尸僵尸进程。

vfork()：在fork的时候，当复制数据的时候，原始的fork会使得子进程也复制一份同样的数据，会造成存储空间的浪费，但是vfork会使得子进程和父进程使用相同的存储空间。但是现在的fork已经可以处理复制数据的问题，fork时，父子进程对于数据块是只读共享的，当需要进行数据块的更改时，以谁改谁拷贝的原则进行处理，只能修改拷贝的数据，所以vfork基本要被废弃。



**3、进程的消亡及释放资源 **

等待进程状态发生变化

wait()

waitpid()具有等待指定pid和配置是否要阻塞选项的操作

常见的进程分配方式有`分块、交叉分配法、池类算法`

为了使得各项资源或者计算任务分配平均，使用**交叉分配法**

较好的分配方法是**池类方法**

> 交叉分配法实现

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>
#include <sys/wait.h>

#define LEFT 30000000
#define RIGHT 30000200
#define N 3

int main()
{

    int i, j, mark, n;
    pid_t pid;

    for(n = 0; n < N; n++)
    {
        pid = fork();

        if (pid < 0) //创建子进程失败
        {
            perror("fork()");
            for(int m = 0; m < n - 1; m++)
            {
                wait(NULL);
            }
            exit(1);
        }

        if (pid == 0)
        {
            for(i = LEFT + n; i <= RIGHT; i += N)
            {
                mark = 1;
                for(j = 2; j < i / 2; j++)
                {
                    if (i % j == 0)
                    {
                        mark = 0;
                        break;
                    }
                }

                if (mark)
                {
                    printf("%d is a primer\n", i);
                }
            }

            exit(0); // 确保子进程正常结束，不进入到后续的循环中，创建更多的进程
        }
    }

    for (n = 0; n < N; n++)
    {
        wait(NULL); 
    }

    exit(0);
}
```

**4、exec函数族**

```c
execl()  execplp()  execle()  execv()  execvp()  执行一个文件
```

`exec函数族会使用一个新的进程映像替换当前进程的映像`，所以在shell下，创建进程会有一个新的名称而不是复制shell进程。

这些`exec`函数族在成功执行时不会返回，因为它们会替代当前进程的内存映像。如果执行失败，它们将返回-1，并设置全局变量`errno`来指示错误的类型。

`exec`函数族通常用于创建新进程，例如在`fork`之后，子进程可以使用`exec`来加载不同的程序。这允许进程在不启动新的进程的情况下更改自己的代码和数据，实现了进程的动态替换和程序调用。

```c
#include <unistd.h>

    extern char **environ;

    int execl(const char *pathname, const char *arg, .../* (char  *) NULL */);   
    int execlp(const char *file, const char *arg, ... /* (char  *) NULL */);
    int execle(const char *pathname, const char *arg, ... /*, (char *) NULL, char *const envp[] */);
    int execv(const char *pathname, char *const argv[]);//arg可以携带多个参数，最后以NULL来结束
    int execvp(const char *file, char *const argv[]);

```

> 实例：打印时间戳,在终端里面的命令是：date + %s

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main()
{
    puts("Begin!");
 
    fflush(NULL);
    
    execl("/bin/date","date","+%s",NULL);
    perror("execl()");
    exit(1);
    
    puts("End!");
    
    exec(0);
}
```

换种形式实现

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>
#include <sys/wait.h>

int main()
{
    pid_t pid;

    puts("Begin!");
 
    fflush(NULL);
    
    pid = fork();
    if(pid < 0)
    {
        perror("fork()");
        exit(1);
    }
    
    if(pid == 0)
    {
        execl("/bin/date","date","+%s",NULL);  //替换子进程镜像
        perror("execl()");
        exit(1);
    }
    
    wait(NULL);

    puts("End!");
    
    exit(0);
}
```

因为子进程是通过复制父进程的方式来创建的，子进程的文件描述符同父进程相同，因为父进程的标准输出是在命令行，所以子进程在输出的时候，同样也是通过命令行输出，其他的文件描述符子进程和父进程也相同。

> 实现mysh，解析由命令行输入的命令

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <glob.h>

#define DELIMS " \t\n"  //分隔符包含空格、制表符、换行

struct cmd_st
{
    glob_t globres;  //用来存储解析完成的命令行字符串，已经被拆开的字符串
};

static void prompt(void)
{
    printf("mysh-0.1$ ");
}

static void parse(const char *line, struct cmd_st *res)  //解析通过终端输入的命令字符串
{
    char *tok;  //在放入glob之前暂时存储命令行字符串

    int flag = 0;  //对于解析到的命令行内容，只有第一个不追加，后面的命令都追加到glob

    while (1)
    {
        tok = strsep(&line, DELIMS);  //用于将字符串分割成多个子字符串，类似于 strtok() 函数。
        if (tok == NULL)
            break;
        if (tok[0] == '\0')  //一个数组的第0位空间为0，说明该数组是空，表示当前解析的命令行字符串中有多个分隔符，需要继续接收
            continue;

        glob(tok, GLOB_NOCHECK | GLOB_APPEND * flag, NULL, &res->globres);  //用于根据指定的文件路径模式查找匹配的文件
        flag = 1;
    }
}

int main()
{
    pid_t pid;
    char *linebuf = NULL;
    size_t linebuf_size = 0;
    struct cmd_st cmd;

    while (1)
    {
        prompt();  //打印提示符

        //获取命令行所有的内容，获取一整行的内容，getline是一个相当强大的工具函数
        if (getline(&linebuf, &linebuf_size, stdin) < 0)  //从标准输入中获取字符串
            break;

        parse(linebuf, &cmd);  //解析上面拿到的命令

        if (0) // 内部命令
        {
            /*do something*/
        }
        else // 外部命令，在linux环境下，大部分命令都是外部命令
        {
            pid = fork();
            if (pid < 0)
            {
                perror("fork()");
                exit(1);
            }

            if (pid == 0)  //子进程
            {
                //变参就选择execvp
                execvp(cmd.globres.gl_pathv[0],cmd.globres.gl_pathv);
                perror("execvp()");
                exit(1);
            }
            else
            {
                wait(NULL);
            }
        }
    }
    exit(0);
}
```

strtok()：将一个字符串分割成多个子字符串，通常根据特定的分隔符进行分割。

```c
char *strtok(char *str, const char *delim);   //接收到的字符串，设置的分割符
```

- `str` 是要分割的字符串，第一次调用时传入待分割的字符串，之后调用时传入 NULL，表示继续使用上一次的字符串继续分割。
- `delim` 是分隔符字符串，其中包含一个或多个字符，用于指定在哪些字符处分割字符串。

strsep()

**5、用户权限及组权限**

uid : real uid、effective uid、save uid

gid ：real gid、effective gid、save gid

u+s -> 指的是当某一个可执行文件具有u+s权限的时候，当别的用户使用该可执行文件的时候，它的身份会切换成当前文件的user身份来运行

g+s -> 当某一个文件具有g+s的权限时，当别的用户使用该文件时，将会切换自己的身份为同组身份。

所以当普通用户在执行passwd命令时，其实是在用root的身份在运行。

`函数 `

getuid()返回用户实际id

geteuid()返回当前进程的effective uid

getgid()

getegid()

setuid()设置用户的effective uid，鉴定权限的时候使用的是effective uid

setgid()设置group的effective uid

`原子化交换`

setreuid() 交换用户的real uid和effective uid

setregid() 交换组的real uid和effective uid

`设置用户和组id`

seteuid()

setguid()



> 实例：用户权限切换

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc,char **argv)
{
    pid_t pid;

    if(argc < 3)
    {
        fprintf("Usage...");
        exit(1);
    }
    
    pid = fork();
    if(pid < 0)
    {
        perror("fork()..");
        exit(1);
    }

    if(pid == 0)
    {
        setuid(atoi(argv[1]));
        execvp(argv[2],argv+2);
        perror("execvp()..");
        exit(1);
    }

    wait(NULL);

    exit(0);

}
```



**6、观摩：解释器文件**

脚本文件

```c
#！/bin/bash

ls
whoami
cat /etc/shadow
ps
```

**7、system()**执行一个shell命令，其实是fork/exec/wait的封装

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{

    system("date +%s > /tmp/out");

    exit(0);
}
```



**8、进程会计**

acct()不可移植，了不了解无所谓



**9、进程时间**

times() : time的命令就是通过它封装的

```c
#include <sys/times.h>

clock_t times(struct tms *buf);

"clock_t"
//其中clock_t是滴答数，1秒钟有多少个滴答数构成，是更为精确的计时方式

"结构体tms"
struct tms {
    
    clock_t tms_utime;  /* user time */
    clock_t tms_stime;  /* system time */
    clock_t tms_cutime; /* user time of children */
    clock_t tms_cstime; /* system time of children */
};

```



**10、守护进程**(精灵进程)，所有的守护进程全部脱离控制终端tty,因为控制终端的输入输出操作会影响到守护进程的工作，一直运行在后台。

> 守护进程的概念

`会话session`

一个shell里面会包含多个进程组，进程组内包含多个进程，标识sid

`终端`

setsid()，创建一个会话并且设置进程组的id，父进程不能够调用。所有的守护进程将由init进程管理，不需要它的父进程wait()

setsid()的主要用途：

1. **创建新会话**：
   - `setsid` 会创建一个新的会话，并将调用它的进程设置为该会话的领导者。
   - 新会话不会与终端关联，这意味着进程不再受终端的控制，即使终端被关闭，进程也会继续运行。
   - 由于脱离了终端控制，它可以在后台运行，独立于用户登录会话。
2. **断开与父进程的联系**：
   - 通常，`setsid` 被守护进程使用，以确保它不会因为父进程的退出而受到影响。
   - 调用 `setsid` 后，进程会脱离父进程的控制，成为一个孤儿进程组的领导者。
3. **改变工作目录**：
   - `setsid` 会改变进程的工作目录为根目录，这有助于避免进程在后续运行中受到目录的卸载或删除的影响。

```c
#include <unistd.h>
pid_t setsid(void);
```

getpgrp()：返回当前进程组的id或者查看一个指定的进程所在的进程组的id

getpgid(): 获取某一个进程的组id

setpgid()：指定某一个进程放到某一个进程组中

> 实现一个守护进程mydaemon.c

```c
#include <stdio.h>
#include <stdlib.h>

#define FNAME "/tmp/out"

static int daemmonize(void)
{
   
    int fd;
    
    pid_t pid;
    pid = fork();
    if(pid < 0)
    {
        perror("fork()");
        return -1;
    }
    
    if(pid > 0) //父进程
    {
        return 0;
    }
    
    fd = open("/dev/null",O_RDWR); //终端
    
    if(fd < 0)
    {
        perror("open()");
        return -1;
    }
	
    //将fd重定向到0、1、2上
    dup2(fd,0);
    dup2(fd,1);
    dup2(fd,2);
    if(fd > 2)
        close(fd);
    
    setsid(); //该函数，父进程不能够调用，此时daemanize成为该会话的领导者
    
    //将当前工作路径设置为根路径
    chdir("/");
    return 0;
}

int main()
{
    FILE *fp;
    
    if(daemmonize())   //创建守护进程
    {
        exit(1);
    }
    //该守护进程的任务，只要该守护进程一直在后台运行着，就打开一个文件，往里面写入数字
    fp = fopen(FNAME,"w");
    if(fp == NULL)
    {
        perror("fopen()");
        exit(1);
    }
    
    for(int i = 0;i < 100;i++)
    {
        fprintf(fp,"%d\n",i);
        fflush(fp);
        slep(1);
    }
    
    exit(0);
}
```

**11、系统日志**

每个应用程序都需要具有一个系统日志

syslogd服务，当有进程需要写系统日志时，将自己需要的日志格式规定传递给这个服务，由它来统一进行系统日志的书写，只要syslogd有权利写系统日志。（权限分离）

下面的三个函数构成linux环境下的系统日志架构

openlog() 将本今进程建立与syslogd的连接

syslog()  传递信息给syslogd

closelog()  断开当前进程和syslogd的连接

```c
//示范程序
#include <syslog.h>
#include <stdio.h>

int main() {
    // 初始化日志系统
    openlog("my_program", LOG_PID | LOG_CONS, LOG_USER);

    // 使用 syslog() 输出日志消息
    syslog(LOG_INFO, "This is an informational message.");
    syslog(LOG_ERR, "An error occurred!");

    // 输出到标准输出，演示不同的输出方式
    printf("Printed to standard output.\n");

    // 关闭日志系统
    closelog();

    return 0;
}
```



# 五、信号

`异步事件处理的两种方法`：查询法（当前异步事件发生比较密集），通知法（当前异步事件发生比较稀疏），严格意义上没有实际的通知法，同样需要一定机制去保证通知法正常执行。异步事件什么时候到来不知道，出现什么结果不清楚。

**1、信号**

软件层面的中断，信号的响应依赖于中断，core文件保存程序出错的现场，最典型的就是`段错误`

**2、signal() **

```c
void (* signal(int signum,void (*func)(int))) (int);
```

> 实例1 : 忽略进程外部中断信号

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>

int main()
{
    int i;
  
    signal(SIGINT,SIG_IGN);    //忽略中断信号，该进程将不会被其他的中断所打断，将一直在中断输出字符

    for (i = 0; i < 10; i++)
    {  
        write(1, '*', 1);
        sleep(1);
    }

    exit(0);
}

```

> 实例2：中断信号发生时，处理指定任务

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

//中断处理函数
static void int_handler(int s)
{
    write(1,'!',1);
}

int main()
{

    int i;

    signal(SIGINT,int_handler);   //在该程序未结束的时候，收到一个中断信号就去执行int_handler函数内部的动作

    for (i = 0; i < 10; i++)
    {
        write(1, '*', 1);
        sleep(1);
 
    }

    exit(0);
}
```

`信号会打断阻塞的系统调用`

对于读写类似的系统调用操作，如果等待时间较长，都有几率发生被外部信号打断的情况出现，所以在代码中，必须有意识地加上对于出错类型地判断

```c
if(... < 0)  //表示某操作未成功

{
	if(errno == EINTR) //代表由中断信号引发的出错
    {
        //重新进行一遍操作
    }
    
    //正常的出错情况
    perror("某操作出错");
    exit(1);
}
```



> :a:`3、信号不可靠`

不可靠指的是`信号的行为`不是信号的丢失

因为在中断中，中断处理函数的执行环境是内核进行布置的，当一个中断信号发生时，同时相同的一个信号也发生了，那么就有可能会导致前面一个中断函数的运行环境出错。



**4、可重入函数**：首次调用后，还没结束，出现第二次调用

所有的系统调用都是可重入的，一部分库函数也是可重入的。

如：memcpy()、rand_r()、localtime_r()。。。带上_r的函数，会有一个版本不支持重入



**5、信号的响应过程**

信号从收到到响应有一个不可避免的延时。

   某个时刻，进程接收到一个中断，此时进程将运行环境保存进入kernel，从kernel态转换成user态，在这期间，会进行mask和pending的按位与操作，以此来判断已经接收到了什么形式的信号，没有中断将不会判断是否接收到了信号，所有信号依赖于中断机制，按位与之后，如果有接收到信号，将会把对应的pending位和mask位全部置为0，再去将kernel里面的函数返回地址替换成信号中断处理函数的地址，执行完之后，再次进入kernel态，将原本的地址替换回来，将对应的mask位置为1，重新从kernel态进入User态，并按位与mask和pending，直到没有信号，最后返回原本的进程继续执行。

`思考`：

如何忽略掉一个信号？不能阻止信号到来，将信号屏蔽字mask永置为0

标准信号为什么要丢失？因为pending是位图，所以不论来多少个同样的信号，其对应的pending永远是1，所以会造成信号丢失。

标准信号的响应没有严格顺序



**6、信号常用函数**

**kill()、raise()、alarm()、pause()、abort()、system()、sleep()**

kill(pid_t pid,int sig)发送信号给一个进程,pid值从正数，0，-1，< -1中拿，代表不同的发送模式

raise(int sig)给当前进程或线程发送一个信号，就是给自己发送信号

alarm(unsigned int seconds)给当前进程发送一个计时信号SIGALRM，以seconds秒为单位。

> 实例：将会以5秒为单位发送闹钟信号，当程序中有多个alarm时，执行最后一个

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

int main()
{
    alarm(5);

    while(1)
        pause(); //等待alarm信号来打断pause()
    
    exit(0);
}

```

> 定时循环:1、使用time函数实现 2、使用alarm函数实现，从结果上来看，alarm的精度要比time的精度要高

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

int main()
{
    time_t end;
    __int64 count = 0;

    end = time(NULL) + 5;

    while (time(NULL) <= end)
        count++;

    printf("%lld", count);
    exit(0);
}
```



```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>

static volatile int loop = 1;  //volatile关键字使程序直接去往数据所存储的实际物理位置拿数据

static void alrm_handler(int s)
{
    loop = 0;
}

int main()
{
    __int64 count = 0;

    alarm(5);
    signal(SIGALRM, alrm_handler);

    while (loop)
    {
        count++;
    }

    printf("%d",count);
    exit(0);
}

```

pause()等待一个信号来打断它

abort()

system()

sleep()不要在代码出现sleep()

> :apple: 实例：实现缓慢cat，漏桶实例，在等待的时候就只是等待，每秒去看是否读到了数据，永远就是每秒十个字节的传输数据

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <signal.h>
#include <errno.h>
#include <fcntl.h>

#define CPS 10
#define BUFSIZE CPS

static volatile int loop = 0;

static void alrm_handler(int s)   //中断信号处理函数
{
    alarm(1);
    loop = 1;
}

int main(int argc, char **argv)
{
    int sfd, dfd = 1;
    char buf[BUFSIZE];
    int len, ret, pos;

    if (argc < 2)
    {
        fprintf(stderr, "Usage...");
        exit(1);
    }

    signal(SIGALRM, alrm_handler);

    do
    {
        sfd = open(argv[1], O_RDONLY);
        if (sfd < 0)
        {
            if (errno != EINTR)
            {
                perror("open()");
                exit(1);
            }
        }

    } while (sfd < 0);

    while (1)
    {
        while (!loop)
            pause();  //pause() 函数会使进程挂起，直到接收到一个信号为止
        loop = 0;

        while((len = read(sfd, buf, BUFSIZE)) < 0)
        {
            if (errno == EINTR)  //判断是否是由于中断导致的错误
            {
                continue;
            }
            perror("read()");
            break;
        }

        if (len == 0)
            break;
        pos = 0;

        while (len > 0)  //坚持写入len个字节
        {
            ret = write(dfd, buf + pos, len);
            if (ret < 0)
            {
                if (errno == EINTR)
                {
                    continue;
                }
                perror("write()");
                exit(1);
            }
            pos += ret;
            len -= ret;
        }
    }

    close(sfd);

    exit(0);
}

```



> 实例：实现缓慢cat，令牌桶实例，在没有数据的时候，会进行累计计数，假如等待了三秒，会存30个字节的传输权限，用完之后，还原成每秒10个字节的传输速度

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <signal.h>
#include <errno.h>
#include <fcntl.h>

#define CPS 10
#define BUFSIZE CPS
#define BURST 100 // token上限值

static volatile int token = 0;  //用来积攒读取操作的权限个数

static void alrm_handler(int s)
{
    alarm(1);
    token++;
    if (token > BURST)
        token = BURST;
}

int main(int argc, char **argv)
{
    int sfd, dfd = 1;
    char buf[BUFSIZE];
    int len, ret, pos;

    if (argc < 2)
    {
        fprintf(stderr, "Usage...");
        exit(1);
    }

    signal(SIGALRM, alrm_handler);

    do
    {
        sfd = open(argv[1], O_RDONLY);
        if (sfd < 0)
        {
            if (errno != EINTR)
            {
                perror("open()");
                exit(1);
            }
        }

    } while (sfd < 0);

    while (1)
    {
        while (token <= 0)
            pause();
        token--;  //读一次就会消耗一个token个数

        while ((len = read(sfd, buf, BUFSIZE)) < 0)
        {
            if (errno == EINTR)
            {
                continue;
            }
            perror("read()");
            break;
        }

        if (len == 0)
            break;
        pos = 0;

        while (len > 0)
        {
            ret = write(dfd, buf + pos, len);
            if (ret < 0)
            {
                if (errno == EINTR)
                {
                    continue;
                }
                perror("write()");
                exit(1);
            }
            pos += ret;
            len -= ret;
        }
    }

    close(sfd);

    exit(0);
}

```

> 将令牌桶函数打包成库

库三要素：主函数main、具体实现c文件、c文件的头文件

`main.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <signal.h>
#include <errno.h>
#include <fcntl.h>
#include <string.h>
#include "mytbf.h"

#define CPS 10
#define BUFSIZE 1024
#define BURST 100 // token上限值

static volatile sig_atomic_t token = 0;  //用来积攒读取操作的权限个数，sig_atomic_t保证对于token的赋值是原子化的，因为对于token的赋值可能是基于多条机器指令完成，那么在这之间可能会对其有自增或者是自减的操作导致token得不到正确的值。

int main(int argc, char **argv)
{
    int sfd, dfd = 1;
    char buf[BUFSIZE];
    int len, ret, pos;
    int size;

    mytbf_t *tbf;

    if (argc < 2)
    {
        fprintf(stderr, "Usage...");
        exit(1);
    }

    tbf = mytbf_init(CPS,BURST);
    if(tbf == NULL)
    {
        fprintf(stderr,"mytbf_init()");
        exit(1);
    }

    do
    {
        sfd = open(argv[1], O_RDONLY);
        if (sfd < 0)
        {
            if (errno != EINTR)
            {
                perror("open()");
                exit(1);
            }
        }

    } while (sfd < 0);

    while (1)
    {
        size = mytbf_fetchtoken(tbf,BUFSIZE);

        if(size < 0)
        {
            fprintf(stderr,"mytbf_fetchtoken():%s",strerror(-size));
            exit(1);
        }

        while ((len = read(sfd, buf, BUFSIZE)) < 0)
        {
            if (errno == EINTR)
            {
                continue;
            }
            perror("read()");
            break;
        }

        if (len == 0)
            break;
        pos = 0;

        while (len > 0)
        {
            ret = write(dfd, buf + pos, len);
            if (ret < 0)
            {
                if (errno == EINTR)
                {
                    continue;
                }
                perror("write()");
                exit(1);
            }
            pos += ret;
            len -= ret;
        }
    }

    close(sfd);

    exit(0);
}

```

`mytbf.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <signal.h>
#include <unistd.h>
#include <string.h>
#include "mytbf.h"

typedef void (*sighandler_t)(int);

static struct mytbf_st *job[MYTBF_MAX];
static int inited = 0;
static sighandler_t alarm_handler_save;

struct mytbf_st
{
    int cps;
    int burst;
    int token;
    int pos;
};

static void alarm_handler(int s)
{
    int i;

    alarm(1);

    for (i = 0; i < MYTBF_MAX; i++)
    {
        if (job[i] != NULL)
        {
            job[i]->token += job[i]->cps;
            if (job[i]->token > job[i]->burst)
                job[i]->token = job[i]->burst;
        }
    }
}

static void module_unload(void) // 模块卸载，或者由于意外中断
{
    int i;

    signal(SIGALRM, alarm_handler_save);

    alarm(0); // 关闭信号
    for (i = 0; i < MYTBF_MAX; i++)
        free(job[i]);
}

static void module_load(void)
{
    alarm_handler_save = signal(SIGALRM, alarm_handler);
    alarm(1);

    atexit(module_unload); // 钩子函数，当此模块卸载的时候，调用
}

static int min(int a, int b)
{
    return (a < b) ? a : b;
}

static int get_free_pos(void)
{
    int i;

    for (i = 0; i < MYTBF_MAX; i++)
    {
        if (job[i] == NULL)
            return i;
    }
    return -1;
}

mytbf_t *mytbf_init(int cps, int burst) // 初始化令牌桶
{
    struct mytbf_st *me;
    int pos;

    if (!inited)
    {
        module_load();
        inited = 1;
    }

    pos = get_free_pos();
    if (pos < 0)
        return NULL;

    me = malloc(sizeof(*me));

    if (me == NULL)
        return NULL;

    me->token = 0;
    me->cps = cps;
    me->burst = burst;
    me->pos = pos;

    job[pos] = me;

    return me;
}

int mytbf_fetchtoken(mytbf_t *ptr, int size) // 申请多少个令牌桶，实际返回申请到的令牌桶个数
{
    int n;
    struct mytbf_st *me = ptr;

    if (size <= 0)
        return -EINVAL;

    while (me->token <= 0)
        pause();

    n = min(me->token, size);

    me->token -= n;

    return n;
}

int mytbf_returntokrn(mytbf_t *ptr, int size) // 将没有使用完的令牌桶归还，返回实际规划成功的令牌桶个数
{

    struct mytbf_st *me = ptr;

    if (size <= 0)
    {
        return -EINVAL;
    }

    me->token += size;

    if (me->token > me->burst)
        me->token = me->burst;

    return size;
}

int mytbf_destroy(mytbf_t *ptr) // 释放申请的令牌桶结构体
{
    struct mytbf_st *me = ptr;
    job[me->pos] = NULL;
    free(me);
    return 0;
}
```

`mytbf.h`

```c
#ifndef __MYTBF_H__
#define __MYTBF_H__

#define MYTBF_MAX 1024  //存储令牌的数组上线

typedef void mytbf_t;

mytbf_t * mytbf_init(int cps,int burst);  //初始化令牌桶

int mytbf_fetchtoken(mytbf_t*,int);   //申请多少个令牌桶，实际返回申请到的令牌桶个数

int mytbf_returntoken(mytbf_t*,int);  //将没有使用完的令牌桶归还，返回实际规划成功的令牌桶个数

int mytbf_destroy(mytbf_t *);  //释放申请的令牌桶结构体

#endif
```

alarm()

> 使用单一计时器，构造一组函数，实现任意数量计数器

```c

```

> **7、信号集**

信号集类型：sigset_t

sigemptyset();置空信号集

sigfillset();置满信号集

sigaddset();向信号集中添加特定信号

sigdelset();从信号集中删除特定信号

sigismember();判断是否是集合当中的成员

> 函数原型

```c
int sigemptyset(sigset_t *set);
int sigfillset(sigset_t *set);
int sigaddset(sigset_t *set, int signum);
int sigdelset(sigset_t *set, int signum);
int sigismember(const sigset_t *set, int signum);
```

> **8、信号屏蔽字、pending集的处理**

`sigprocmask()` 是一个用于操作进程的信号屏蔽集的函数。它可以用来设置、修改或获取当前进程的信号屏蔽集。

```c
int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
```

函数参数说明：

- how

  ：表示操作信号屏蔽集的方式，可以采用以下值之一：

  - `SIG_BLOCK`：将 `set` 中的信号集添加到当前信号屏蔽集中。
  - `SIG_UNBLOCK`：从当前信号屏蔽集中移除 `set` 中的信号集。
  - `SIG_SETMASK`：用 `set` 中的信号集替代当前信号屏蔽集。

- `set`：一个指向 `sigset_t` 类型的指针，它指定要执行操作的信号集。

- `oldset`：一个指向 `sigset_t` 类型的指针，如果不为 NULL，则会将当前信号屏蔽集的值存储在 `oldset` 中，以便查询以前的信号屏蔽集状态。

函数返回值：

- 如果成功，返回 0。
- 如果出现错误，返回 -1，并设置 `errno` 来指示错误的类型。

`sigprocmask()` 函数通常用于控制进程中哪些信号被屏蔽，以确保在关键部分的代码中不会被中断。它在多线程和信号处理的上下文中经常使用，以确保信号的可靠处理。

> 实例程序：

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

//中断处理函数
static void int_handler(int s)
{
    write(1,"!",1);
    write(1,"\n",1);
}

int main()
{
    int i,j;
    sigset_t set,saveset;

    signal(SIGINT,int_handler);   //在该程序未结束的时候，收到一个中断信号就去执行int_handler函数内部的动作
    sigemptyset(&set);
    sigaddset(&set,SIGINT);
    //1、保存集合中信号的状态
	sigprocmask(SIG_UNBLOCK,&set,&saveset);
    for(j = 0; j < 100; j++)
    {
        //要求在下面的循环执行期间不受信号的影响，也就是说在此处将信号值阻塞住
        sigprocmask(SIG_BLOCK,&set,NULL);
        for (i = 0; i < 5; i++)
        {
            write(1, "*", 1);
            sleep(1);
        }
        write(1,"\n",1);  //在打印完毕换行符之后再响应信号值
        sigprocmask(SIG_UNBLOCK,&set,NULL);
    }
    //2、恢复到旧集合的状态
    sigprocmask(SIG_UNBLOCK,&saveset,NULL);  //1和2的操作集合在进入和出去这个函数的时候它的状态是不发生改变的
    exit(0);
}
```

`输出的效果：`

```txt
root@b6508848aada:/workspace/linux_c/celery/11_9# ./sig 
*****^C
!
**^C***
!
*^C^C*^C^C^C^C^C*** //此时标准信号就出现了丢失的情况，因为现在mask位已经被视为1，不管后续来多少同样的信号都只会记录一次，所以只会被响应一次。
!
**^\Quit (core dumped)
```

可以看到在终端上首先会进行BLOCK内的操作，在执行完毕打印操作之后才会响应信号，同时不论打出多少个中断信号都只会响应一次。

`可以依靠恢复旧集合的方式来达到同样的效果`

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

//中断处理函数
static void int_handler(int s)
{
    write(1,"!",1);
    write(1,"\n",1);
}

int main()
{
    int i,j;
    sigset_t set,oset;

    signal(SIGINT,int_handler);   //在该程序未结束的时候，收到一个中断信号就去执行int_handler函数内部的动作
    sigemptyset(&set);
    sigaddset(&set,SIGINT);
    
    for(j = 0; j < 100; j++)
    {
        //要求在下面的循环执行期间不受信号的影响，也就是说在此处将信号值阻塞住
        sigprocmask(SIG_BLOCK,&set,&oset);
        for (i = 0; i < 5; i++)
        {
            write(1, "*", 1);
            sleep(1);
        }
        write(1,"\n",1);  //在打印完毕换行符之后再响应信号值
        sigprocmask(SIG_SETMASK,&oset,NULL);
    }
    exit(0);
}
```

`sigpending()函数：`扎进内核，取出pending集合的信息，会告诉使用者当前内核取到了什么样的信号

> **9、函数替换**
>
> **sigsuspend()、sigaction()->替换signal()**
>
> **setitimer()->替换alarm()**

`setitimer` 函数用于设置定时器，它允许程序在经过指定的时间后触发一个信号，优点是误差不累计。

`sigaction()` 是一个用于设置信号处理函数的系统调用。

```c
int setitimer(int which, const struct itimerval *new_value, struct itimerval *old_value);

int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
```

```txt
setitimer函数参数说明：

which：指定要设置的定时器类型，可以取以下值之一：
ITIMER_REAL：设置真实时间定时器，以指定的时间间隔触发 SIGALRM 信号。
ITIMER_VIRTUAL：设置虚拟时间定时器，以指定的时间间隔触发 SIGVTALRM 信号。
ITIMER_PROF：设置CPU时间定时器，以指定的时间间隔触发 SIGPROF 信号。
new_value：一个指向 struct itimerval 结构的指针，包含了新的定时器设置。
old_value：一个指向 struct itimerval 结构的指针，如果不为 NULL，则会将旧的定时器设置存储在其中。
函数返回值：

如果成功，返回 0。
如果出现错误，返回 -1，并设置 errno 来指示错误的类型。

sigaction函数参数说明：
signum：要捕获的信号的编号。
act：指向 struct sigaction 结构的指针，该结构包含有关信号处理的信息。
oldact：可选参数，指向 struct sigaction 结构的指针，用于存储先前的信号处理信息。
struct sigaction 结构的定义如下：

struct sigaction {
    void (*sa_handler)(int);
    void (*sa_sigaction)(int, siginfo_t *, void *);
    sigset_t sa_mask;
    int sa_flags;
    void (*sa_restorer)(void);
};
sa_handler：指定了信号处理函数的地址。当 sa_flags 不包含 SA_SIGINFO 时，使用 sa_handler。
sa_sigaction：指定了信号处理函数的地址。当 sa_flags 包含 SA_SIGINFO 时，使用 sa_sigaction。
sa_mask：定义在信号处理期间要阻塞的信号集。
sa_flags：指定了一些标志，例如 SA_RESTART，它表示系统调用应在信号处理完成后自动重新启动。
sa_restorer：已弃用，用于设置一个恢复函数，目前不再使用。
函数返回值为 0 表示成功，-1 表示错误，错误信息存储在 errno 中。
```

> 示例程序

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <sys/time.h>

void timer_handler(int signo) {
    printf("Timer expired! Signo: %d\n", signo);
}

int main() {
    // 设置 SIGALRM 信号的处理函数
    struct sigaction sa;
    sa.sa_handler = timer_handler;
    sa.sa_flags = 0;
    sigaction(SIGALRM, &sa, NULL);

    // 设置定时器
    struct itimerval timer;
    timer.it_value.tv_sec = 1;   // 初始延迟1秒
    timer.it_value.tv_usec = 0;
    timer.it_interval.tv_sec = 1; // 重复每1秒
    timer.it_interval.tv_usec = 0;

    if (setitimer(ITIMER_REAL, &timer, NULL) == -1) {
        perror("setitimer");
        exit(1);
    }

    // 让程序运行，等待定时器触发
    while (1) {
        // 此处可以执行其他操作
    }

    return 0;
}

```

因为在不同的系统当中，sleep函数可能是由alarm()函数和pause()函数封装的，当在程序当中出现多个sleep()函数时，意味着会出现多个alarm()函数，当出现多个alarm()函数，系统只会生效最后一个alarm()函数，从而导致程序运行出错。

`sigaction函数实例程序：`

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

// 自定义的信号处理函数，通过ctrl+c来触发中断信号
void sigint_handler(int signo) {
    printf("Caught SIGINT (Ctrl+C) signal. Exiting.\n");
    exit(EXIT_SUCCESS);
}

int main() {
    // 设置信号处理结构体
    struct sigaction sa;
    sa.sa_handler = sigint_handler;
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;

    // 使用 sigaction() 设置信号处理函数
    if (sigaction(SIGINT, &sa, NULL) == -1) {
        perror("sigaction");
        exit(EXIT_FAILURE);
    }

    // 主程序，等待信号
    printf("Waiting for SIGINT (Ctrl+C) signal...\n");
    while (1) {
        sleep(1);  // 在等待信号时可以执行其他操作
    }

    return 0;
}

```

在此示例中，程序设置了一个信号处理函数 `sigint_handler`，用于捕获 `SIGINT` 信号。当用户按下 Ctrl+C 时，会触发 `SIGINT` 信号，然后调用 `sigint_handler` 函数。函数输出一条消息并正常退出程序。

`sigsuspend()函数：`在解除对一个信号集的阻塞状态之后马上进入等待信号的阶段，会使得这时间不会因为外来信号的干扰导致程序错过应该等待的信号。

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

//中断处理函数
static void int_handler(int s)
{
    write(1,"!",1);
    write(1,"\n",1);
}

int main()
{
    int i,j;
    sigset_t set,saveset,oset;

    signal(SIGINT,int_handler);   //在该程序未结束的时候，收到一个中断信号就去执行int_handler函数内部的动作
    sigemptyset(&set);
    sigaddset(&set,SIGINT);
    //1、保存集合中信号的状态
	sigprocmask(SIG_UNBLOCK,&set,&saveset);
    for(j = 0; j < 100; j++)
    {
        //要求在下面的循环执行期间不受信号的影响，也就是说在此处将信号值阻塞住
        sigprocmask(SIG_BLOCK,&set,&oset);
        for (i = 0; i < 5; i++)
        {
            write(1, "*", 1);
            sleep(1);
        }
        write(1,"\n",1);  //在打印完毕换行符之后再响应信号值
        
        sigsuspend(&oset); //相当于下面四步的原子化操作
        /**        
        sigset_t tmpset;
        sigprocmask(SIG_SETMASK,&oset,&tmpset); //在这之间可能会有新的信号来到
        pause(); //因为和上一步的操作是非原子化的，所以在这之间可能会因为外来的信号已经被响应，导致程序在pause()的位置被阻塞住。
        sigprocmask(SIG_SETMASK,&tmpset,NULL); 
        */

    }
    //2、恢复到旧集合的状态
    sigprocmask(SIG_UNBLOCK,&saveset,NULL);  //1和2的操作集合在进入和出去这个函数的时候它的状态是不发生改变的
    exit(0);
}
```

单单利用pause()函数绝对完成不了一个信号驱动程序，原因在于如果不加上别的机制的话，他一定完成不了一个原子操作。

> **10、实时信号**

如果同时接收标准信号和实时信号，那么首先一定是先响应标准信号，实时信号是介于SIGRTMIN到SIGRTMAX之间，没有自己的一个默认动作，但是标准信号是存在自己特殊的动作表述，比如SIGIO、SIGPWR...除了SIGUSR1和SIGUSR2。

`实时信号范例：`

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

#define MYRTSIG (SIGRTMIN+6)

//中断处理函数
static void mysig_handler(int s)
{
    write(1,"!",1);
    write(1,"\n",1);
}

int main()
{
    int i,j;
    sigset_t set,saveset,oset;

    signal(MYRTSIG,mysig_handler);   //在该程序未结束的时候，收到一个中断信号就去执行int_handler函数内部的动作
    sigemptyset(&set);
    sigaddset(&set,MYRTSIG);
    //1、保存集合中信号的状态
	sigprocmask(SIG_UNBLOCK,&set,&saveset);
    for(j = 0; j < 100; j++)
    {
        //要求在下面的循环执行期间不受信号的影响，也就是说在此处将信号值阻塞住
        sigprocmask(SIG_BLOCK,&set,&oset);
        for (i = 0; i < 5; i++)
        {
            write(1, "*", 1);
            sleep(1);
        }
        write(1,"\n",1);  //在打印完毕换行符之后再响应信号值
        
        sigsuspend(&oset); //相当于下面四步的原子化操作
    }
    //2、恢复到旧集合的状态
    sigprocmask(SIG_UNBLOCK,&saveset,NULL);  //1和2的操作集合在进入和出去这个函数的时候它的状态是不发生改变的
    exit(0);
}
```

在以上的程序当中，`实时信号是不存在丢失`的情况，但是标准信号是会被丢失的，除了信号响应是否存在顺序以及是否丢失，在别的方面，实时信号和标准信号是没有区别的。kill是发送信号的指令

<img src="李慧琴c语言系统编程/image-20231110103931191.png" alt="image-20231110103931191" style="zoom:67%;" />

实时信号的排队是存在上限的，使用ulimit -a可以查看上限数量：可以修改

<img src="李慧琴c语言系统编程/image-20231110104237974.png" alt="image-20231110104237974" style="zoom:67%;" />

> `多线程产生并发要比信号简单的多`

# 六、线程

### 1、线程的概念

   一个正在运行的函数，一个进程空间内最少有一个函数在运行， 其实我们可以不以main函数作为程序入口，main线程也不必被成为主线程，因为它和其他线程是兄弟关系，没有上下级的关系。线程间通信比进程间通信要简单，因为在同一个空间内运行，多线程并发比多进程并发要简单。`posix线程是一套标准，而不是实现`，pthread_t类型是posix类型下的线程类型。一个进程下存在多个线程在工作，进程相当于线程的容器，在同一个进程内的线程以进程号（容器号）为起始来消耗进程号，`进程就是容器，用于承载多线程`。

> `线程和信号不要混用！！！`（除非是小范围内）

线程的安全实现要比信号的安全实现要简单。

 `pthread_equal()`函数是用于比较两个线程标识符（`pthread_t` 类型）是否相等的函数。

```c
int pthread_equal(pthread_t thread1, pthread_t thread2);
```

`实例程序`:在编译链接的时候需要加上-pthread

```c
#include <pthread.h>
#include <stdio.h>

int main() {
    pthread_t thread1, thread2;

    // 创建线程并获取线程标识符
    pthread_create(&thread1, NULL, myThreadFunction, NULL);
    pthread_create(&thread2, NULL, myThreadFunction, NULL);

    // 比较线程标识符
    if (pthread_equal(thread1, thread2)) {
        printf("Thread identifiers are equal.\n");
    } else {
        printf("Thread identifiers are not equal.\n");
    }

    return 0;
}

```

makefile文件：

```makefile
target=primer
CC=gcc
CFLAGS+=-c	-Wall	-pthread
LDFLAGS+=-pthread
OBJS=main.o

$(target):$(OBJS)
	CC	$^	-o	$@

%.o:%.c
	CC	$^	$(CFLAGS)	-o	$@
```

`pthread_self()`函数获取当前线程的线程标识符

```c
pthread_t pthread_self(void);
```

`实例程序`

```c
#include <pthread.h>
#include <stdio.h>

void* myThreadFunction(void* arg) {
    // 获取当前线程的线程标识符
    pthread_t self = pthread_self();
    printf("Thread ID: %lu\n", (unsigned long)self);

    // 线程的其他操作...

    return NULL;
}

int main() {
    pthread_t thread1, thread2;

    // 创建线程并启动
    pthread_create(&thread1, NULL, myThreadFunction, NULL);
    pthread_create(&thread2, NULL, myThreadFunction, NULL);

    // 等待线程结束
    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    return 0;
}
```

### 2、线程的创建：

`pthread_create()`函数用于创建一个新的线程。以下是该函数的原型：

```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                   void *(*start_routine) (void *), void *arg);
```

**说明：**

- 线程创建成功后，新线程将立即执行 `start_routine` 函数。
- `pthread_create` 函数创建的线程是可结合的，可以使用 `pthread_join` 函数等待线程的结束。
- 如果不再需要等待新线程结束，可以分离线程，使用 `pthread_detach` 函数，使得线程自己消亡。
- 线程的属性可以通过 `pthread_attr_init` 和相关函数进行配置。

`使用示例：`

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>

void* myThreadFunction(void* arg) {
    // 线程的工作内容
    printf("Hello from the new thread!\n");

    pthread_exit(NULL);//退出线程
}

int main() {
    pthread_t myThread;

    // 创建新线程
    int result = pthread_create(&myThread, NULL, myThreadFunction, NULL);
    
    if (result != 0) {
        fprintf(stderr,"pthread_create():%s\n",strerror(result));
        exit(1);
    }

    // 主线程继续执行其他操作...

    // 等待新线程结束
    result = pthread_join(myThread, NULL); //NULL标识不关心返回值状态

    if (result != 0) {
        perror("Thread join failed");
        exit(0);
    }

    exit(0);
}

```

`线程的调度取决于调度器的策略`，有可能线程还没来得及被调度，作为它容器的进程已经结束了。

### 线程的终止：

`3种终止方式`

1. 线程从启动例程中返回，返回值就是线程的退出码
2. 线程可以被同一进程中的其他线程取消
3. 线程调用pthread_exit()函数（正常结束），相当于进程中的exit()函数，线程收尸函数pthread_join()相当于进程中的wait()操作。

`pthread_join` 函数用于等待一个指定的线程结束。以下是该函数的原型：

```c
int pthread_join(pthread_t thread, void **retval);
```

`使用实例`

```c
#include <pthread.h>
#include <stdio.h>

void* myThreadFunction(void* arg) {
    // 线程的工作内容
    printf("Hello from the new thread!\n");
    
    // 返回值示例
    int *result = malloc(sizeof(int));
    *result = 42;
    pthread_exit(result);
}

int main() {
    pthread_t myThread;
    void* threadResult;

    // 创建新线程
    int result = pthread_create(&myThread, NULL, myThreadFunction, NULL);
    
    if (result != 0) {
        perror("Thread creation failed");
        return 1;
    }

    // 等待新线程结束，并获取返回值
    result = pthread_join(myThread, &threadResult); 

    if (result != 0) {
        perror("Thread join failed");
        return 1;
    }

    printf("Thread returned: %d\n", *(int*)threadResult);

    // 释放返回值的内存
    free(threadResult);

    return 0;
}

```

### 栈清理：

以下两个函数相当于进程中的钩子函数`atexit()`

pthread_cleanup_push()相当于挂到钩子上

pthread_cleanup_pop()相当于从钩子上取下来执行

函数原型：

```c
void pthread_cleanup_push(void (*routine)(void*), void *arg);
```

**参数：**

- `routine`：指定的清理处理程序函数的指针。
- `arg`：传递给清理处理程序函数的参数。

**说明：**

- `pthread_cleanup_push` 用于将清理处理程序推入线程的清理处理程序堆栈。
- 在 `pthread_cleanup_push` 和相应的 `pthread_cleanup_pop` 之间的代码块中，可以多次调用 `pthread_cleanup_push` 来添加多个清理处理程序。

```c
void pthread_cleanup_pop(int execute);
```

**参数：**

- `execute`：一个整数，如果为非零，表示执行清理处理程序；如果为零，则表示不执行。

**说明：**

- `pthread_cleanup_pop` 用于从线程的清理处理程序堆栈中弹出一个清理处理程序，并决定是否执行它。
- 在调用 `pthread_cleanup_pop` 之前，必须通过 `pthread_cleanup_push` 添加了一个或多个清理处理程序。

**注意：**

- 在实际编程中，`pthread_cleanup_push` 和 `pthread_cleanup_pop` 主要用于确保在线程退出时执行一些清理操作，而不是作为一种常规的编程实践。
- 在现代的多线程编程中，更推荐使用 RAII（资源获取即初始化）和智能指针等技术，而不是依赖于 `pthread_cleanup_push` 和 `pthread_cleanup_pop`。
- `pthread_cleanup_push` 和 `pthread_cleanup_pop`应该成对出现

`实例程序`

```c
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>

// 清理处理程序的参数结构
struct CleanUpArgs {
    char *message;
    int *counter;
};

// 清理处理程序函数
void cleanUpHandler(void *arg) {
    struct CleanUpArgs *cleanupArgs = (struct CleanUpArgs *)arg;
    printf("Cleaning up: %s\n", cleanupArgs->message);

    // 在这里执行一些清理操作，例如释放资源
    free(cleanupArgs->message);
    free(cleanupArgs);
}

// 线程函数
void* myThreadFunction(void* arg) {
    // 创建清理处理程序参数
    struct CleanUpArgs *cleanupArgs = (struct CleanUpArgs *)malloc(sizeof(struct CleanUpArgs));
    cleanupArgs->message = "Thread cleanup";
    cleanupArgs->counter = (int *)arg;

    // 将清理处理程序推入堆栈
    pthread_cleanup_push(cleanUpHandler, cleanupArgs);

    // 线程的工作内容
    for (int i = 0; i < 5; ++i) {
        printf("Thread working... (%d)\n", *cleanupArgs->counter);
        (*cleanupArgs->counter)++;
        sleep(1);
    }

    // 如果线程正常退出，清理处理程序将被执行
    // 如果线程通过pthread_exit()退出，清理处理程序也会执行
    // 如果线程因为取消而退出，清理处理程序不会执行

    // 弹出清理处理程序
    pthread_cleanup_pop(1);

    // 线程的其他操作...

    return NULL;
}

int main() {
    pthread_t myThread;
    int counter = 0;

    // 创建新线程
    int result = pthread_create(&myThread, NULL, myThreadFunction, &counter);

    if (result != 0) {
        perror("Thread creation failed");
        return 1;
    }

    // 主线程继续执行其他操作...

    // 等待新线程结束
    result = pthread_join(myThread, NULL);

    if (result != 0) {
        perror("Thread join failed");
        return 1;
    }

    printf("Final counter value: %d\n", counter);

    return 0;
}

```

### 线程的取消选项：

`pthread_cancel` 函数用于请求取消指定的线程。以下是该函数的原型：

```c
int pthread_cancel(pthread_t thread);
```

可以先取消再对线程进行收尸操作。

**说明：**

- `pthread_cancel` 函数用于向指定线程发送取消请求，请求线程退出。
- 实际上，`pthread_cancel` 只是向线程发送一个取消请求，线程是否真正取消取决于线程是否响应取消请求。
- 取消请求的处理取决于线程的取消状态和取消点设置。
- 如果线程成功取消，它将执行清理处理程序（如果有），然后退出。

`示范程序：`

```c
#include <pthread.h>
#include <stdio.h>

void* myThreadFunction(void* arg) {
    // 线程的工作内容
    while (1) {
        printf("Thread working...\n");
        // 模拟长时间运行的任务
        usleep(500000);
    }

    return NULL;
}

int main() {
    pthread_t myThread;

    // 创建新线程
    int result = pthread_create(&myThread, NULL, myThreadFunction, NULL);

    if (result != 0) {
        perror("Thread creation failed");
        return 1;
    }

    // 主线程继续执行其他操作...

    // 发送取消请求
    result = pthread_cancel(myThread);

    if (result != 0) {
        perror("Thread cancel failed");
        return 1;
    }

    // 等待新线程结束
    result = pthread_join(myThread, NULL);

    if (result != 0) {
        perror("Thread join failed");
        return 1;
    }

    printf("Thread canceled.\n");

    return 0;
}
```

`pthread_setcancelstate()->设置取消选项：`允许和不允许。

`pthread_setcanceltype()——>设置取消的类型`。如果是允许取消的话，又分为异步取消和推迟取消（推迟到cancel点再响应）。cancel点是：都是可能引发阻塞的系统调用。

`pthread_testcancel()->就是设置一个取消点`，因为在一些程序中是没有取消点（系统调用）的。

线程分离：

pthread_detach()函数用于将指定的线程标识符所代表的线程标记为分离状态，从而允许系统在线程退出后立即回收其资源，无需等待其他线程调用 `pthread_join`。以下是该函数的原型：

```c
int pthread_detach(pthread_t thread);
```

**说明：**

- 将线程标记为分离状态后，线程退出时会自动释放其资源，无需其他线程调用 `pthread_join`。
- 分离状态的线程不会产生僵尸线程，它的退出状态将被系统自动处理。
- 如果线程已经处于分离状态，再次调用 `pthread_detach` 将返回错误，但不会产生其他影响。

`实例程序`

```c
#include <pthread.h>
#include <stdio.h>

void* myThreadFunction(void* arg) {
    // 线程的工作内容...

    return NULL;
}

int main() {
    pthread_t myThread;

    // 创建新线程
    int result = pthread_create(&myThread, NULL, myThreadFunction, NULL);

    if (result != 0) {
        perror("Thread creation failed");
        return 1;
    }

    // 将线程标记为分离状态
    result = pthread_detach(myThread);

    if (result != 0) {
        perror("Thread detach failed");
        return 1;
    }

    // 主线程继续执行其他操作...

    return 0;
}
```

### 3、线程竞争和同步：

线程竞争问题

> 使用201个线程筛选质数（线程竞争实例）

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <string.h>

#define LEFT 30000000
#define RIGHT 30000200
#define THRNUM (RIGHT - LEFT + 1)

static void *thr_prime(void *p);

int main()
{
    int i,err;
    pthread_t tid[THRNUM];

    for (i = LEFT; i <= RIGHT; i++)
    {
        err = pthread_create(tid + (i - LEFT), NULL, thr_prime, &i);
        if (err)
        {
            fprintf(stderr, "pthread_create():%s\n", strerror(err));
            exit(1);
        }
    }

    for (i = LEFT; i <= RIGHT; i++)
    {
        pthread_join(tid[i - LEFT], NULL);
    }

    exit(0);
}

static void *thr_prime(void *p)
{
    int i, mark, j;
    i = *(int *)p; //在这里使用的是地址传参，所有线程的指针全部指向同一个i的地址，造成竞争问题。
    mark = 1;

    for (j = 2; j < i / 2; j++)
    {
        if (i % j == 0)
        {
            mark = 0;
            break;
        }
    }

    if (mark)
        printf("%d is a primer.\n", i);

    pthread_exit(NULL);
}
```

> 对上面的程序进行改进

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <string.h>

#define LEFT 30000000
#define RIGHT 30000200
#define THRNUM (RIGHT - LEFT + 1)

struct thr_arg_st
{
    int n;
};

static void *thr_prime(void *p);

int main()
{
    int i, err;
    pthread_t tid[THRNUM];
    struct thr_arg_st *p;
    void* ptr;

    for (i = LEFT; i <= RIGHT; i++)
    {

        p = malloc(sizeof(*p));  //使得各个线程指向的是不同的空间
        if (p == NULL)
        {
            perror("malloc()");
            exit(1);
        }
        
        p->n = i;
        
        err = pthread_create(tid + (i - LEFT), NULL, thr_prime, p);
        if (err)
        {
            fprintf(stderr, "pthread_create():%s\n", strerror(err));
            exit(1);
        }
    }

    for (i = LEFT; i <= RIGHT; i++)
    {
        pthread_join(tid[i - LEFT], &ptr);
        free(ptr); //值已经取走，必须马上释放空间
    }

    exit(0);
}

static void *thr_prime(void *p)
{
    int i, mark, j;
    i = ((struct thr_arg_st *)p)->n;
    
    mark = 1;

    for (j = 2; j < i / 2; j++)
    {
        if (i % j == 0)
        {
            mark = 0;
            break;
        }
    }

    if (mark)
        printf("%d is a primer.\n", i);

    pthread_exit(p);  //拿回p指针,在join函数中进行拿取并free
}
```

`当前空间下能够创建线程的个数取决于外在的资源量`，比如栈空间大小以及用户态能够使用的总空间，还有线程标识符的个数。

所以上述代码还需要修改

![image-20231114102935889](李慧琴c语言系统编程/image-20231114102935889.png)

上面的THRNUM代表着需要创建多少个线程来完成筛选质数的任务，当LEFT、RIGHT的间距增大的时候，肯定会有超过能够创建线程上限的问题。

`所以需要采取方法进行对任务进行分配`

1. 池类算法
2. 交叉分配法
3. 分块

> 线程同步

`互斥量`

互斥锁：

静态初始化`pthread_mutex_t`

```c
pthread_mutex_t my_mutex = PTHREAD_MUTEX_INITIALIZER;
```

动态初始化`pthread_mutex_init`函数原型如下：

```c
//初始化某个互斥量
int pthread_mutex_init(pthread_mutex_t *mutex, const pthread_mutexattr_t *attr);
//销毁某个互斥量
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```

```c
pthread_mutex_t my_mutex;
pthread_mutex_init(&my_mutex, NULL);
```

`限制和解除限制操作某个资源`

```c
//阻塞式抢锁
pthread_mutex_lock(pthread_mutex_t *mutex)
//非阻塞式抢锁
pthread_mutex_trylock(pthread_mutex_t *mutex)
//对某个锁住的资源进行解锁
pthread_mutex_unlock(pthread_mutex_t *mutex)
```

`临界区：同一时间只允许一个用户操作的文件`，对于互斥锁，实际上它所限制的是某一段代码是否能够执行来去限制某个资源是否能够被使用。

> `实例代码1：`使用互斥锁控制读写同一份文件

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <string.h>
#include <unistd.h>

#define THRNUM 20
#define FNAME "/workspace/linux_c/celery/11_15/out"
#define LINESIZE 1024

static pthread_mutex_t mut = PTHREAD_MUTEX_INITIALIZER;

static void *thr_add(void *p)
{
    FILE *fp;
    char linebuf[LINESIZE];

    fp = fopen(FNAME, "r+");
    if (fp == NULL)
    {
        perror("fopen");
        exit(1);
    }

    //以下是需要限制访问的临界区资源（对文件的读写操作），需要上锁
    pthread_mutex_lock(&mut);
    fgets(linebuf, LINESIZE, fp);
    fseek(fp, 0, SEEK_SET);
    sleep(1);
    fprintf(fp, "%d\n", atoi(linebuf) + 1);
    fclose(fp);
    pthread_mutex_unlock(&mut);

    pthread_exit(NULL);
}

int main()
{
    int i, err;
    pthread_t tid[THRNUM];

    for (i = 0; i < THRNUM; i++)
    {
        err = pthread_create(tid + 1, NULL, thr_add, NULL);
        if (err)
        {
            fprintf(stderr, "pthread_create():%s\n", strerror(err));
            exit(1);
        }
    }

    for (i = 0; i < THRNUM; i++)
        pthread_join(tid[i], NULL);
        

    pthread_mutex_destroy(&mut);

    exit(0);
}

```

> `实例代码2：`有四个线程分别往终端顺序输出abcd (使用互斥锁链)

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <string.h>
#include <unistd.h>

#define THRNUM 4

static pthread_mutex_t mutex[THRNUM];

static int next(int n)
{
    if ((n + 1) == THRNUM)
        return 0;
    else
        return (n + 1);
}

static void *thr_func(void *p)
{
    int n = (int)p;
    int c = 'a' + n;
    while (1)
    {
        pthread_mutex_lock(mutex + n); //锁上自己
        write(1, &c, 1);   //打印自己
        pthread_mutex_unlock(mutex + next(n)); //解开下一个打印的锁
    }

    pthread_exit(NULL);
}

int main()
{
    int i, err;
    pthread_t tid[THRNUM];

    for (i = 0; i < THRNUM; i++)
    {
        pthread_mutex_init(mutex + i, NULL);
        pthread_mutex_lock(mutex + i);
        err = pthread_create(tid + i, NULL, thr_func, (void *)i);
        if (err)
        {
            fprintf(stderr, "pthread_create:%s\n", strerror(err));
            exit(1);
        }
    }

    pthread_mutex_unlock(mutex + 0);  //解开打印a的互斥锁

    alarm(3); // 3秒后强制杀死进程

    for (i = 0; i < THRNUM; i++)
        pthread_join(tid[i], NULL);

    exit(0);
}
```

读锁->相当于是共享锁   写锁->相当于是互斥锁

### 4、线程属性：

`pthread_attr_init` 函数是用于初始化线程属性对象的 POSIX 线程库函数，

`pthread_attr_destroy` 函数是用于销毁线程属性对象的 POSIX 线程库函数。其原型如下：

```c
#include <pthread.h>

int pthread_attr_init(pthread_attr_t *attr);
int pthread_attr_destroy(pthread_attr_t *attr);
```

> 检查一个机器中能够创建的线程最大数：

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

void *func(void *p)
{
    while(1)
        pause();
    pthread_exit(NULL);
}

int main()
{
    int i;
    int err;
    pthread_t tid[1024];

    for (i = 0; ;i++)
    {
        err = pthread_create(&tid[i], NULL, func, NULL);
        if (err)
        {
            fprintf(stdout, "pthread_create()。。。");
            break;
        }
    }

    printf("%d\n", i);

    for (int j = 0; j <= i; j++)
    {
        pthread_join(&tid[j],NULL);
    }

    return 0;
}
```

设置线程的属性：

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

void *func(void *p)
{
    while (1)
        pause();
    pthread_exit(NULL);
}

int main()
{
    int i;
    int err;
    pthread_t tid;
    pthread_attr_t attr;
    pthread_attr_init(&attr);
    pthread_attr_setstacksize(&attr, 1024 * 1024);

    err = pthread_create(&tid, &attr, func, NULL);
    if (err)
    {
        fprintf(stdout, "pthread_create()。。。");
    }

    pthread_join(&tid, NULL);

    pthread_attr_destroy(&attr);

    return 0;
}
```

更多有关线程的属性设置函数，在man手册的pthread_attr_init的see also方面

### 线程同步的属性：

互斥量属性：
pthread_mutexattr_init();

pthread_mutexattr_destory();

 //是否是跨进程起作用

pthread_mutexattr_getpshared();

pthread_mutexattr_setpshared(); 

创建一个子进程

clone();

pthread_mutexattr_gettype();

pthread_mutexattr_settype();

![image-20231205102740599](李慧琴c语言系统编程/image-20231205102740599.png)

条件变量:

`pthread_cond_init` 函数是 POSIX 线程库中用于初始化条件变量（condition variable）的函数。条件变量通常与互斥锁一起使用，用于线程间的同步。其原型如下：

```c
#include <pthread.h>

int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr);
```

这个函数用于初始化条件变量对象，以便后续的使用。条件变量是在多线程编程中用于线程间通信的一种机制，它允许一个线程在某个条件不满足时进入休眠状态，等待其他线程发出信号（通过 `pthread_cond_signal` 或 `pthread_cond_broadcast` 函数）来唤醒它。

- `cond`：指向要初始化的条件变量对象的指针。
- `attr`：指向条件变量属性对象的指针，通常可以设置为 `NULL`，表示使用默认属性。条件变量的属性对象目前很少使用，因此一般情况下会将其设置为 `NULL`。

```c
#include <pthread.h>
#include <stdio.h>

int main() {
    pthread_cond_t cond;

    // 初始化条件变量对象
    if (pthread_cond_init(&cond, NULL) != 0) {
        perror("pthread_cond_init");
        return 1;
    }

    // 在这里可以使用 cond 对象进行线程同步操作

    // 销毁条件变量对象，释放资源
    if (pthread_cond_destroy(&cond) != 0) {
        perror("pthread_cond_destroy");
        return 1;
    }

    // 在这之后，cond 对象不再可用

    return 0;
}
```

### 5、重入

`多线程中的IO操作，目前使用的IO操作都已经实现了多线程的并发，即他们在使用时候会先lock住，再进行后续的操作。`

当然与之匹配的unlock的函数，也已经定义：
比如getchar_unlock(void);这种函数在尾部会加上unlock标识，但是在多进程中使用是不安全的。

### 线程和信号关系：

`不建议混用`

pthread_sigmask() 设置mask位

pthread_kill()  以线程为单位，发送一个信号

### 线程与fork的关系：


​	线程和`fork`是两种不同的并发机制，分别用于创建并发执行的进程和线程，线程适合在同一进程中共享数据和并发执行任务，而`fork`适合创建独立的进程，每个进程执行不同的任务。选择线程还是`fork`取决于具体的应用需求和设计考虑。下面是关于线程和`fork`之间的一些主要区别：

1. **创建方式**：
   - **线程：** 线程是在同一进程中创建的轻量级执行单元。线程共享进程的地址空间，文件描述符等资源，可以访问相同的数据。线程是由调用线程库的程序创建和管理的。
   - **`fork`：** `fork` 是创建一个新的进程的系统调用。新的进程是原始进程（父进程）的副本，包括它的地址空间、文件描述符等。这两个进程在执行时是完全独立的，它们之间不共享内存。
2. **并发性**：
   - **线程：** 线程是在同一进程中并发执行的。线程之间可以共享相同的数据，通过共享内存来进行通信。因为它们共享进程的资源，线程之间的通信相对容易。
   - **`fork`：** `fork` 创建的两个进程是独立的，它们之间的通信相对复杂，通常需要使用进程间通信（IPC）机制，如管道、消息队列、共享内存等。
3. **开销**：
   - **线程：** 线程通常比进程轻量级，因为它们共享相同的地址空间和其他资源。线程的创建、切换和销毁开销相对较小。
   - **`fork`：** `fork` 创建新进程的开销较大，因为它需要复制父进程的整个地址空间。
4. **共享状态**：
   - **线程：** 线程共享相同的地址空间，因此它们可以直接访问相同的数据。这使得线程之间的通信更为方便。
   - **`fork`：** `fork` 创建的两个进程有各自独立的地址空间，它们之间的通信需要额外的机制。
5. **数据同步**：
   - **线程：** 线程之间需要进行显式的同步来避免竞态条件和确保数据的一致性。通常使用互斥锁、条件变量等机制。
   - **`fork`：** `fork` 创建的进程之间不需要显式同步，因为它们有独立的地址空间。但是，如果它们需要通信，仍然需要使用 IPC 机制进行同步。

# 七、高级IO

## 1、非阻塞IO

`为什么需要非阻塞？`

```txt
在C语言编程中，使用非阻塞I/O（Input/Output）有几个主要的原因：

提高并发性能： 阻塞I/O会导致程序在等待I/O操作完成时被阻塞，这会浪费CPU的处理能力。使用非阻塞I/O可以使程序在等待I/O的同时执行其他任务，提高并发性能，特别是在多线程或多进程的环境中。

响应性： 非阻塞I/O可以使程序更加灵活地响应事件。在阻塞I/O中，当一个I/O操作发生时，程序将被阻塞直到操作完成。而在非阻塞I/O中，程序可以轮询或通过回调等机制来检查I/O状态，从而更快地响应事件。

避免死锁： 在多线程或多进程的程序中，阻塞I/O可能导致死锁。如果一个线程或进程在等待I/O时被阻塞，而其他线程或进程持有了它需要的资源，就可能发生死锁。非阻塞I/O可以一定程度上减少这种情况的发生。

实时性： 在一些实时系统中，对事件的快速响应是至关重要的。非阻塞I/O能够更好地支持实时性需求，因为它允许程序在等待I/O时执行其他任务。

在C语言中，可以通过使用系统调用如fcntl或ioctl来设置文件描述符为非阻塞模式，或者使用专门的库函数如select、poll、epoll等来实现非阻塞I/O。这些机制允许程序轮询或异步地处理I/O事件，从而提高程序的性能和响应性。
```

## 2、有限状态机编程

`简单流程`：如果一个程序的自然流程是结构化的，就是简单流程

`复杂流程`：如果一个程序的自然流程是非结构化的，就是复杂流程

> 使用有限状态机实现数据中继

`先读取后写的有限状态机模型`

<img src="李慧琴c语言系统编程/image-20231206103740565.png" alt="image-20231206103740565" style="zoom: 33%;" />

```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <errno.h>
#include <unistd.h>

#define TTY1 "../dev/tty11"
#define TTY2 "../dev/tty12"
#define BUFSIZE 1024


// 定义状态机具有的各个状态
enum
{
    STATE_R = 1,
    STATE_W,
    STATE_Ex,
    STATE_T
};

// 定义状态机模型结构体
struct fsm_st
{
    int state;
    int sfd;
    int dfd;
    int len;
    int pos;
    char *errstring;
    char buf[BUFSIZE];
};

static void fsm_driver(struct fsm_st *fsm)
{

    int ret;

    switch (fsm->state)
    {
    case STATE_R:
        fsm->len = read(fsm->sfd, fsm->buf, BUFSIZE);
        if (fsm->len == 0)
            fsm->state = STATE_T;
        else if (fsm->len < 0)
        {
            if (errno == EAGAIN)
                fsm->state = STATE_R;
            else
            {
                fsm->errstring = "read()";
                fsm->state = STATE_Ex;
            }
        }
        else
        {
            fsm->pos = 0;
            fsm->state = STATE_W;
        }
        break;
    case STATE_W:
        ret = write(fsm->dfd, fsm->buf + fsm->pos, fsm->len);
        if (ret < 0)
        {
            if (errno == EAGAIN)
                fsm->state = STATE_W;
            else
            {
                fsm->errstring = "write()";
                fsm->state = STATE_Ex;
            }
        }
        else
        {
            fsm->pos += ret;
            fsm->len -= ret;
            if (fsm->len == 0)
                fsm->state = STATE_R;
            else
                fsm->state = STATE_W;
        }

        break;
    case STATE_Ex:
        perror(fsm->errstring);
        fsm->state = STATE_T;
        break;
    case STATE_T:
        /* code */
        // 进程结束
        break;
    default:
        // 进程结束
        abort();
        break;
    }
}

void relay(int fd1, int fd2)
{
    struct fsm_st fsm_12, fsm_21; // 定义两个状态机，一个读左写右，一个读右写左

    int fd1_save = fcntl(fd1, F_GETFL);         // 获取文件描述符并保存
    fcntl(fd1, F_SETFL, fd1_save | O_NONBLOCK); // 设置文件描述符

    int fd2_save = fcntl(fd2, F_GETFL);
    fcntl(fd2, F_SETFL, fd2_save | O_NONBLOCK);

    fsm_12.state = STATE_R;
    fsm_12.sfd = fd1;
    fsm_12.dfd = fd2;

    fsm_21.state = STATE_R;
    fsm_21.sfd = fd2;
    fsm_21.dfd = fd1;

    while (fsm_21.state != STATE_T || fsm_12.state != STATE_T)
    {
        fsm_driver(&fsm_12);
        fsm_driver(&fsm_21);
    }

    // 对于文件进入的状态要做保留，并且在出去的时候需要恢复
    fcntl(fd1, F_SETFL, fd1_save);
    fcntl(fd2, F_SETFL, fd2_save);
}

int main()
{
    int fd1, fd2;
    fd1 = open(TTY1, O_RDWR);
    if (fd1 < 0)
    {
        perror("open()");
        exit(1);
    }

    fd2 = open(TTY2, O_RDWR | O_NONBLOCK);
    if (fd2 < 0)
    {
        perror("open()");
        exit(1);
    }

    relay(fd1, fd2);

    close(fd1);
    close(fd2);

    exit(0);
}
```

> 中继引擎实现

主要涉及到的文件为，main.c relayer.c relayer.h makefile

main.c

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <fcntl.h>
#include <errno.h>
#include <unistd.h>
#include "relayer.h"


#define TTY1 "../dev/tty9"
#define TTY2 "../dev/tty10"
#define TTY3 "../dev/tty11"
#define TTY4 "../dev/tty12"

int main()
{
    int fd1, fd2, fd3, fd4;
    int job1, job2;

    fd1 = open(TTY1, O_RDWR);
    if (fd1 < 0)
    {
        perror("open()");
        exit(1);
    }

    fd2 = open(TTY2, O_RDWR | O_NONBLOCK);
    if (fd2 < 0)
    {
        perror("open()");
        exit(1);
    }

    job1 = rel_addjob(fd1, fd2);
    if(job1 < 0)
    {
        fprintf(stderr,"rel_addjob ,err is %s\n",strerror(-job1));
        exit(1);
    }

    fd3 = open(TTY3, O_RDWR);
    if (fd3 < 0)
    {
        perror("open()");
        exit(1);
    }

    fd4 = open(TTY2, O_RDWR | O_NONBLOCK);
    if (fd4 < 0)
    {
        perror("open()");
        exit(1);
    }

    job2 = rel_addjob(fd3, fd4);
    if(job2 < 0)
    {
        fprintf(stderr,"rel_addjob ,err is %s\n",strerror(-job2));
        exit(1);
    }

    while(1)
        pause();

    close(fd1);
    close(fd2);
    close(fd3);
    close(fd4);

    exit(0);
}

```

relayer.h

```c
#ifndef __RELAYER_H__
#define __RELAYER_H__

#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <errno.h>
#include <unistd.h>
#include <string.h>

#define BUFSIZE 1024
#define REL_JOBMAX  10000

enum
{
    STATE_RUNNING=1,
    STATE_CANCEL,
    STATE_OVER
};

struct rel_stat_st
{
    int state;
    int fd1;
    int fd2;
    int64_t count21,count12;
    
};

//添加任务
int rel_addjob(int fd1,int fd2);
/**
 * return >= 0  成功，返回当前任务id
 *        == -EINVAL  失败，参数非法
 *        == -ENOSPC  失败，任务数组满
 *        == -ENOMEM  失败，内存分配有误
*/

//取消任务
int rel_canceljob(int fd);
/**
 * return == 0  成功，指定任务成功取消
 *        == -EINVAL  失败，参数非法
 *        == -EBUSY  失败，任务已经被取消
*/

//给任务收尸
int rel_waitjob(int fd,struct rel_stat_st *);
/**
 * return       == 0 成功，指定任务已经停止，并且返回状态
 *              == -EINVAL  失败，参数非法
*/

//获取任务状态
int rel_statjob(int fd,struct rel_stat_st *);
/**
 * return       == 0 成功，指定任务任务状态已经返回
 *              == -EINVAL  失败，参数非法
*/

#endif
```

relayer.c

```c
#include "relayer.h"
#include <pthread.h>

// 定义状态机具有的各个状态
enum
{
    STATE_R = 1,
    STATE_W,
    STATE_Ex,
    STATE_T
};

// 定义状态机模型结构体
struct rel_fsm_st
{
    int state;
    int sfd;
    int dfd;
    int len;
    int pos;
    char *errstring;
    char buf[BUFSIZE];
    __int64_t count;
};

//定义单个任务结构体
struct rel_job_st
{
    int fd1;
    int fd2;
    int job_state;
    struct rel_fsm_st fsm12, fsm21;
    int fd1_save;
    int fd2_save;
};

// 定义存储job的数组，该数组是临界资源，每次需要改变其中的值时，必须上锁操作
static struct rel_job_st *rel_job[REL_JOBMAX];
static pthread_mutex_t mut_rel_job = PTHREAD_MUTEX_INITIALIZER;
static pthread_once_t init_once = PTHREAD_ONCE_INIT;

static void fsm_driver(struct rel_fsm_st *fsm)
{
    int ret;

    switch (fsm->state)
    {
    case STATE_R:
        fsm->len = read(fsm->sfd, fsm->buf, BUFSIZE);
        if (fsm->len == 0)
            fsm->state = STATE_T;
        else if (fsm->len < 0)
        {
            if (errno == EAGAIN)
                fsm->state = STATE_R;
            else
            {
                fsm->errstring = "read()";
                fsm->state = STATE_Ex;
            }
        }
        else
        {
            fsm->pos = 0;
            fsm->state = STATE_W;
        }
        break;
    case STATE_W:
        ret = write(fsm->dfd, fsm->buf + fsm->pos, fsm->len);
        if (ret < 0)
        {
            if (errno == EAGAIN)
                fsm->state = STATE_W;
            else
            {
                fsm->errstring = "write()";
                fsm->state = STATE_Ex;
            }
        }
        else
        {
            fsm->pos += ret;
            fsm->len -= ret;
            if (fsm->len == 0)
                fsm->state = STATE_R;
            else
                fsm->state = STATE_W;
        }

        break;
    case STATE_Ex:
        perror(fsm->errstring);
        fsm->state = STATE_T;
        break;
    case STATE_T:
        /* code */
        // 进程结束
        break;
    default:
        // 进程结束
        abort();
        break;
    }
}

static void *thr_relayer(void *p)	//该线程负责推动job数组内部的所有状态机
{
    while (1)
    {
        pthread_mutex_lock(&mut_rel_job);
        for (int i = 0; i < REL_JOBMAX; i++)
        {
            if (rel_job[i] != NULL)
            {
                if (rel_job[i]->job_state == STATE_RUNNING)
                {
                    fsm_driver(&rel_job[i]->fsm12);
                    fsm_driver(&rel_job[i]->fsm21);
                    if (rel_job[i]->fsm12.state == STATE_T && rel_job[i]->fsm21.state == STATE_T)
                    {
                        rel_job[i]->job_state = STATE_OVER;
                    }
                }
            }
        }
        pthread_mutex_unlock(&mut_rel_job);
    }
}

// 创建线程推状态机
static void module_load(void)
{
    int err;
    pthread_t tid_relayer;

    err = pthread_create(&tid_relayer, NULL, thr_relayer, NULL);
    if (err)
    {
        fprintf(stderr, "pthread_create():%s\n", strerror(err));
        exit(1);
    }
}

static int get_free_pos_unlocked()
{
    for(int i = 0; i < REL_JOBMAX; i++)
    {
        if(rel_job[i] == NULL)
            return i;
    }
    return -1;
}

// 添加任务
int rel_addjob(int fd1, int fd2)
{
    int pos;

    struct rel_job_st *me;

    // 从一开始就推动状态机，只推一次
    pthread_once(&init_once, module_load);

    me = malloc(sizeof(*me));
    if (me == NULL)
    {
        return -ENOMEM;
    }

    me->fd1 = fd1;
    me->fd2 = fd2;
    me->job_state = STATE_RUNNING;

    // 确保文件描述符都是非阻塞形式的
    me->fd1_save = fcntl(me->fd1, F_GETFL);
    fcntl(me->fd1, F_SETFL, me->fd1_save | O_NONBLOCK);
    me->fd2_save = fcntl(me->fd2, F_GETFL);
    fcntl(me->fd2, F_SETFL, me->fd2_save | O_NONBLOCK);

    // 设置初始状态机状态
    me->fsm12.sfd = me->fd1;
    me->fsm12.dfd = me->fd2;
    me->fsm12.state = STATE_R;

    me->fsm21.sfd = me->fd2;
    me->fsm21.dfd = me->fd1;
    me->fsm21.state = STATE_R;

    // 加锁获取当前数组中的空位
    pthread_mutex_lock(&mut_rel_job);
    pos = get_free_pos_unlocked();
    if (pos < 0)
    {
        // 解锁
        pthread_mutex_unlock(&mut_rel_job);
        // 还原文件描述符初始状态
        fcntl(me->fd1, F_SETFL, me->fd1_save);
        fcntl(me->fd2, F_SETFL, me->fd2_save);
        // 释放已经申请的空间
        free(me);
        return -ENOSPC;
    }
    rel_job[pos] = me;
    pthread_mutex_unlock(&mut_rel_job);
}

#if 0
//取消任务
int rel_canceljob(int fd)

//给任务收尸
int rel_waitjob(int fd,struct rel_stat_st *)

//获取任务状态
int rel_statjob(int fd,struct rel_stat_st *)

#endif
```

makefile

```makefile
target=relayer
CC=gcc
CFLAGS+=-c	-Wall	-pthread
LFLAGS+=-pthread	-o
OBJS=	main.c	relayer.c	

$(target):$(OBJS)
	$(CC)	$^	$(LFLAGS)	$@

%.o:%.c
	$(CC)	$^	$(CFLAGS)	$@

clean:
	rm	-rf	*.o	relayer
```

## 3、IO多路转接

监视文件描述符的行为，只有在发生程序感兴趣的文件操作的时候，才会去做后续的操作。发生了现象才去做对应的事情，解决盲等的问题。

`select()`

`poll()`

`epoll()`

以上三个函数实现io多路转接，实现监视文件描述符的行为。

select()传参部位具有问题，poll()以文件描述符为单位来组织事件，epoll()是linux引申过来的，简化用户在poll上面的操作内容。

必须掌握poll的使用

三个函数的原型为：

`select`

```c
#include <sys/select.h>

int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

- `nfds`: 所有文件描述符的范围（最大文件描述符 + 1）。

- `readfds`: 用于检查是否有数据可读的文件描述符集合。

- `writefds`: 用于检查是否可以写入的文件描述符集合。

- `exceptfds`: 用于检查是否发生异常情况的文件描述符集合。

- `timeout`: 是一个 `struct timeval` 结构，用于指定超时时间，即 `select()` 在等待可操作的文件描述符时最多等待的时间。

  函数的返回值表示发生了程序感兴趣的行为的文件描述符的数量，如果发生错误则返回 -1

该函数的使用：readfds，writefds，exceptfds中存储着用户感兴趣的文件描述符，当有文件描述符出现以上集合中的任意一个，select函数都会返回。

> 示例程序：该程序演示了如何同时监视标准输入和标准输出是否准备好读取或写入：

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/select.h>

int main() {
    fd_set readfds, writefds;
    struct timeval timeout;
    int result;

    while (1) {
        // 清空文件描述符集合
        FD_ZERO(&readfds);
        FD_ZERO(&writefds);

        // 将标准输入（0）添加到readfds集合
        FD_SET(0, &readfds);

        // 将标准输出（1）添加到writefds集合
        FD_SET(1, &writefds);

        // 设置超时时间为5秒
        timeout.tv_sec = 5;
        timeout.tv_usec = 0;

        // 使用select函数检查文件描述符状态
        result = select(2, &readfds, &writefds, NULL, &timeout);

        if (result == -1) {
            perror("select");
            exit(EXIT_FAILURE);
        } else if (result == 0) {
            // 超时
            printf("Timeout\n");
        } else {
            // 有文件描述符准备好
            if (FD_ISSET(0, &readfds)) {
                printf("Read from stdin is ready\n");
                // 在这里可以读取标准输入的数据
            }

            if (FD_ISSET(1, &writefds)) {
                printf("Write to stdout is ready\n");
                // 在这里可以写入标准输出的数据
            }
        }
    }

    return 0;
}

```

*select缺点：监视的事件太过单一，事件的存放位置和结果的存放位置是一致的。*

`poll`

函数原型：

```c
#include <poll.h>

int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

- `fds`: 一个指向 `struct pollfd` 结构体数组的指针，每个结构体描述一个待监视的文件描述符。
- `nfds`: 待监视的文件描述符数量。
- `timeout`: 指定超时时间，单位是毫秒；如果设为负数，`poll()` 将一直等待，阻塞，如果为0，`poll()` 将立即返回，非阻塞。

> 示例程序：该程序监视标准输入是否准备好读取：

```c
#include <stdio.h>
#include <stdlib.h>
#include <poll.h>

int main() {
    struct pollfd fds[1];
    int result;

    // 设置要监视的文件描述符
    fds[0].fd = 0;      // 标准输入
    fds[0].events = POLLIN;  // 监视可读事件

    while (1) {
        // 使用poll函数检查文件描述符状态，阻塞形式
        result = poll(fds, 1, -1);

        if (result == -1) {
            perror("poll");
            exit(EXIT_FAILURE);
        } else if (result == 0) {
            // 永远不会发生，因为超时设置为-1（等待时间无限）
        } else {
            // 有文件描述符准备好
            if (fds[0].revents & POLLIN) {
                printf("Read from stdin is ready\n");
                // 在这里可以读取标准输入的数据
            }
        }
    }

    return 0;
}
```

它设置了一个 `struct pollfd` 数组，描述了要监视的文件描述符以及监视的事件类型。在循环中，`poll()` 函数会阻塞程序，直到有文件描述符准备好或超时。请注意，该示例程序中的超时设置为-1，表示一直等待，直到有事件发生。

# 八、管道实例

> 池类算法

<img src="李慧琴c语言系统编程/image-20231211140627996.png" alt="image-20231211140627996" style="zoom: 50%;" />

使用循环队列实现，引申到管道实现，但是这里是在同一个进程空间内，使用管道实现该功能有点小题大做，是使用循环队列实现比较合适。

`mypipe.h`

```c
#ifndef __MYPIPE_H__
#define __MYPIPE_H__

#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define PIPESIZE        1024
#define MYPIPE_READ     0x00000001UL
#define MYPIPE_WRITE    0x00000002UL

typedef void mypipe_t;

struct mypipe_st
{
    int head;
    int tail;
    char data[PIPESIZE];
    int datasize;
    int count_rd;
    int count_wr;
    pthread_mutex_t mut;
    pthread_cond_t cond;
};

//管道初始化
mypipe_t* mypipe_init(void);

//管道注册
int mypipe_register(mypipe_t *, int opmap);

//管道注销
int mypipe_unregister(mypipe_t *, int opmap);

//读取单个字节
static int mypipe_readbyte_unlocked(struct mypipe_st*,char *);

//读取管道
int mypipe_read(mypipe_t*, void *buf, size_t size);

//写入管道
int mypipe_write(mypipe_t*,const void *buf, size_t size);

//管道销毁
int mypipe_destory(mypipe_t*);

#endif
```

mypipe.c

```c
#include "mypipe.h"

// 管道初始化
mypipe_t *mypipe_init(void)
{
    struct mypipe_st *me;
    me = malloc(sizeof(*me));
    if (me == NULL)
    {
        return NULL;
    }

    me->head = 0;
    me->tail = 0;
    me->datasize = 0;
    me->count_rd = 0;
    me->count_wr = 0;
    pthread_mutex_init(&me->mut, NULL);
    pthread_cond_init(&me->cond, NULL);

    return me;
}

int next(int p)
{
    if ((p + 1) != PIPESIZE)
        return p + 1;
    else
        return 0;
}

// 管道注册
int mypipe_register(mypipe_t *ptr, int opmap)
{

    struct mypipe_st *me = ptr;

    pthread_mutex_lock(&me->mut);

    if (opmap & MYPIPE_READ)
        me->count_rd++;
    if (opmap & MYPIPE_WRITE)
        me->count_wr++;

    pthread_cond_broadcast(&me->cond);

    // 管道必须要具有读写双方才能工作
    while (me->count_rd <= 0 || me->count_wr <= 0)
        pthread_cond_wait(&me->cond, &me->mut);

    pthread_mutex_unlock(&me->mut);

    return 0;
}

// 管道注销
int mypipe_unregister(mypipe_t *ptr, int opmap)
{
    struct mypipe_st *me = ptr;

    pthread_mutex_lock(&me->mut);

    if (opmap & MYPIPE_READ)
        me->count_rd--;
    if (opmap & MYPIPE_WRITE)
        me->count_wr--;
    
    pthread_cond_broadcast(&me->cond);

    pthread_mutex_unlock(&me->mut);

    return 0;
}

// 读取单个字节
static int mypipe_readbyte_unlocked(struct mypipe_st *me, char *datap)
{
    if (me->datasize <= 0)
        return -1;
    *datap = me->data[me->head];
    me->head = next(me->head);
    me->datasize--;
    return 0;
}

// 读取管道
int mypipe_read(mypipe_t *ptr, void *buf, size_t size)
{
    struct mypipe_st *me = ptr;
    int i;

    pthread_mutex_lock(&me->mut);

    while (me->datasize <= 0 && me->count_wr > 0)
    {
        pthread_cond_wait(&me->cond, &me->mut);
    }

    if(me->datasize <= 0 && me->count_wr <= 0)
    {
        pthread_mutex_unlock(&me->mut);
        return 0;
    }    

    for (i = 0; i < size; i++)
    {
        if (mypipe_readbyte_unlocked(me, buf + i) != 0)
            break;
    }

    pthread_cond_broadcast(&me->cond);
    pthread_mutex_unlock(&me->mut);

    return i;
}

// 写入管道
int mypipe_write(mypipe_t *, const void *buf, size_t size)
{

}

// 管道销毁
int mypipe_destory(mypipe_t *ptr)
{
    struct mypipe_st *me = ptr;

    pthread_mutex_destroy(&me->mut);
    pthread_cond_destroy(&me->cond);

    free(ptr);

    return 0;
}
```

# 九、进程间通信



> 1、管道

内核提供，单工，自同步机制（管道永远迁就慢一点的一方）

- [x] `匿名管道`（不能用于没有亲缘关系的进程间通信）

pipe()  创建一个管道，在一个管道当中，写端和读端都是具有两个的，一端同时具有读写功能

`示例1：父进程往管道中写，子进程从管道当中读`，那么父进程不需要读端，可以将读端关闭，子进程关闭写端

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFSIZE 1024

int main()
{
    int pd[2];
    pid_t pid;
    char buf[BUFSIZE];

    if (pipe(pd) < 0)
    {
        perror("pipe()");
        exit(1);
    }

    pid = fork();
    if (pid < 0)    
    {
        perror("fork()");
        exit(1);
    }
    else if (pid == 0)   //子进程
    {
        close(pd[1]);
        int len = read(pd[0],&buf,BUFSIZE);
        write(1,buf,len);
        close(pd[0]);
        exit(0);
    }
    else                //父进程
    {
        close(pd[0]);
        write(pd[1],"Hello!",6);
        close(pd[1]);
        wait(NULL);
        exit(0);
    }

}
```

`示例2：`和以上是相同功能

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <fcntl.h>

#define BUFSIZE 1024

int main()
{
    int pd[2];
    pid_t pid;
    char buf[BUFSIZE];

    if (pipe(pd) < 0)
    {
        perror("pipe()");
        exit(1);
    }

    pid = fork();
    if (pid < 0)
    {
        perror("fork()");
        exit(1);
    }
    else if (pid == 0) // 子进程
    {
        close(pd[1]);
        dup2(pd[0], 0);
        close(pd[0]);
        int fd = open("/dev/null", O_RDWR);
        dup2(fd, 1);                               // 关闭标准写
        dup2(fd, 2);                               // 关闭标准报错
        execl("/usr/mpg123", "mpg123", "-", NULL); // 当前子进程变成了mpg123
        perror("execl()");
        exit(1);
    }
    else // 父进程
    {
        close(pd[0]);
        // 父进程从网上收取数据，往管道中写
        close(pd[1]);
        wait(NULL);
        exit(0);
    }
}
```

- [x] `命名管道`

mkfifo 是一个用于创建命名管道（FIFO）的函数，其原型定义通常在 `<sys/types.h>` 和 `<sys/stat.h>` 头文件中。以下是 `mkfifo` 函数的一般原型：

```c
#include <sys/types.h>
#include <sys/stat.h>

int mkfifo(const char *pathname, mode_t mode);
```

- `pathname`：包含要创建的FIFO的路径名。
- `mode`：指定创建的FIFO的访问权限，通常以八进制表示（例如，`0666`）。

`mkfifo` 函数返回 `0` 表示成功，返回 `-1` 表示失败，并设置相应的错误号，可以通过 `errno` 获取错误信息。

以下是一个简单的示例程序，演示了如何使用 `mkfifo` 创建一个命名管道：

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

int main() {
    const char *fifoPath = "my_fifo";

    // 创建FIFO，设置访问权限为0666
    if (mkfifo(fifoPath, 0666) == -1) {
        perror("mkfifo");
        exit(EXIT_FAILURE);
    }

    printf("FIFO created successfully at %s\n", fifoPath);

    return 0;
}
```

> 2、XSI -> SysV

key值，确认让通信双方拿到同一个通信机制

key值 -> ftok()

`ftok` 是一个用于生成 System V IPC（Inter-Process Communication，进程间通信）键的函数，其原型如下：

```c
#include <sys/types.h>
#include <sys/ipc.h>

key_t ftok(const char *pathname, int proj_id);
```

- `pathname`：是一个指向字符串的指针，通常是一个存在的文件的路径名。`ftok` 使用这个路径名来生成一个键值。
- `proj_id`：是一个用户定义的项目标识符，用于进一步区分 IPC 对象。在不同的项目中，`proj_id` 的值应该不同。

函数返回一个键值（key_t 类型），该键值可以被用于创建或获取 System V IPC 对象，比如消息队列、信号量、共享内存等。

请注意，使用 `ftok` 时需要注意以下几点：

1. 要确保 `pathname` 参数对应的文件存在，否则可能会出现错误。
2. 不同的文件路径和项目标识符组合可能生成相同的键值，因此在使用 `ftok` 生成键值时要小心避免冲突。
3. `proj_id` 的范围应该在 [0, 255] 之间。

> 消息队列（msg）、信号量数组(sem)、共享内存(shm)的相关函数适用下面的规则

1. `创建：xxxget`
2. `操作：xxxop`
3. `控制：xxxctl`

### ①消息队列（msg）

msgget() 创建一个消息队列

msgsnd() 发送消息

msgrcv() 接收消息

msgctl() 对某一个消息队列进行控制

主动端：先发包的一方

被动端：先收包的一方`（先运行）`

`双方协议`

```c
//通信双方的协议文件
/**
 * ip地址、端口、组播、广播、对齐方式、机器字长、数据格式
*/
#ifndef __PROTO_H__
#define __PROTO_H__

#define   KEYPATH   "/etc/services"
#define   KEYPROJ   'g'
#define   NAMESIZE 32

struct msg_st
{
    long mtype; //标识当前消息的类型
    char name[NAMESIZE];
    int math;
    int chinese;
};

#endif
```

`接收方`

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <string.h>

#include "proto.h"

// 被动端

int main()
{
    key_t key;
    int msgid;
    struct msg_st rbuf;

    key = ftok(KEYPATH, KEYPROJ);
    if (key <= 0)
    {
        perror("ftok()");
        exit(1);
    }

    msgid = msgget(key, IPC_CREAT | 0600); // 消息队列使用的权限
    if (msgid < 0)
    {
        perror("msgget()");
        exit(1);
    }

    while (1)
    {
        if (msgrcv(msgid, &rbuf, sizeof(rbuf) - sizeof(long), 0, 0) < 0)
        {
            perror("msgrcv()");
            exit(1);
        }
        printf("NAME = %s\n",rbuf.name);
        printf("MATH = %d\n",rbuf.math);
        printf("CHINESE = %d\n",rbuf.chinese);
    }

    //销毁消息队列
    msgctl(msgid,IPC_RMID,NULL);

    exit(0);
}
```

`发送方`

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <string.h>

#include "proto.h"

// 主动端

int main()
{
    key_t key;
    int msgid;
    struct msg_st sbuf;

    key = ftok(KEYPATH, KEYPROJ);
    if (key <= 0)
    {
        perror("ftok()");
        exit(1);
    }

    msgid = msgget(key, 0); // 消息队列使用的权限
    if (msgid < 0)
    {
        perror("msgget()");
        exit(1);
    }

    sbuf.mtype = 1;
    strcpy(sbuf.name,"Alan");
    sbuf.math = 100;
    sbuf.chinese = 100;

    if (msgsnd(msgid, &sbuf, sizeof(sbuf) - sizeof(long), 0) < 0)
    {
        perror("msgsnd()");
        exit(1);
    }

    puts("send!\n");

    exit(0);
}
```

消息队列具有一定的数据缓存的效果，可以通过获取key值来实现两个没有亲缘关系的进程间实现通信

### ②信号量数组

`示例：使用信号量数组实现进程间同步`

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <errno.h>

#define PROCNUM 20
#define FNAME "/tmp/out"
#define LINESIZE 1024

int semid;

static void P(void)
{

    struct sembuf op;

    op.sem_num = 0;
    op.sem_op = -1;
    op.sem_flg = 0;

    while (semop(semid, &op, 1) < 0)
    {
        if (errno != EINTR || errno != EAGAIN)
        {
            perror("semop()");
            exit(1);
        }
    }
}

static void V(void)
{
    struct sembuf op;

    op.sem_num = 0;
    op.sem_op = 1;
    op.sem_flg = 0;

    if (semop(semid, &op, 1) < 0)
    {
        perror("semop()");
        exit(1);
    }
}

static void func_add(void)
{
    FILE *fp;
    int fd;
    char linebuf[LINESIZE];

    fp = fopen(FNAME, "r+");
    if (fp == NULL)
    {
        perror("fopen()");
        exit(1);
    }

    P(); // 取资源量
    fgets(linebuf, LINESIZE, fp);
    fseek(fp, 0, SEEK_SET);
    sleep(1);
    fprintf(fp, "%d\n", atoi(linebuf) + 1);
    fflush(fp);
    V(); // 还资源量

    fclose(fp);

    return;
}

int main()
{
    pid_t pid;
    int i;

    semid = semget(IPC_PRIVATE, 1, 0600);
    if (semid < 0)
    {
        perror("semget()");
        exit(1);
    }

    if (semctl(semid, 0, SETVAL, 1) < 0)
    {
        perror("semctl()");
        exit(1);
    }

    for (i = 0; i < PROCNUM; i++)
    {
        pid = fork();
        if (pid < 0)
        {
            perror("fork()");
            exit(1);
        }
        else if (pid == 0)
        {
            func_add();
            exit(0);
        }
    }

    for (i = 0; i < PROCNUM; i++)
        wait(NULL);

    semctl(semid, 0, IPC_RMID);

    exit(0);
}
```

### ③共享内存

实例：

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

#define MEMSIZE 1024

int main()
{
    pid_t pid;
    int shmid;
    void *ptr;

    //具有亲缘关系的进程之间是不需要key值的
    shmid = shmget(IPC_PRIVATE, MEMSIZE, 0600);
    if (shmid < 0)
    {
        perror("shmget()");
        exit(1);
    }

    pid = fork();
    if (pid < 0)
    {
        perror("fork()");
        exit(1);
    }
    else if (pid == 0)
    {
        ptr = shmat(shmid, NULL, 0);
        if (ptr == (void *)-1)
        {
            perror("shmat()");
            exit(1);
        }
        strcpy(ptr, "hello");
        shmdt(ptr);
        exit(0);
    }
    else
    {
        wait(NULL);
        ptr = shmat(shmid, NULL, 0);
        if (ptr == (void *)-1)
        {
            perror("shmat()");
            exit(1);
        }
        put(ptr);
        shmdt(ptr);
        shmctl(shmid,IPC_RMID,NULL);
        exit(0);
    }

}

```

> 3、网络套接字socket

`讨论：跨主机的传输需要注意的问题`

1. 字节序问题：

   大端存储：低地址处放高字节

   小端存储：低地址处放低字节

   <img src="李慧琴c语言系统编程/image-20231212093543993.png" alt="image-20231212093543993" style="zoom:50%;" />

   目前不再考虑大端还是小端，通过主机和字节间的关系进行处理

   主机字节序：host

   网络字节序：network

   _to__:有以下几种组合形式：htons(主机到网络对端的两位信息)、htonl(主机到网络对端的四位信息)、ntohs(网络到主机对端的两位信息)、ntohl(网络到主机对端的四位信息)

2. 对齐：

   ```c
   struct
   {
       int i;
       char ch;
       float f;
   };
   ```

   由于编译器存在`对齐`的功能，以上结构体所占据的字节个数是12个字节，而不是9个字节。

<img src="李慧琴c语言系统编程/image-20231212095317467.png" alt="image-20231212095317467" style="zoom:50%;" />

在此处套接字通信的时候的解决方法是使用宏定义使得编译器不进行对齐

3. 数据类型长度问题：

   int、char、float。。。。等数据类型的大小属于未定义行为。

   解决办法：使用通用的数据类型标识，比如int32_t,uint32_t,int64_t,int8_t,uint8_t

> `SOCKET：统一管理通信以及协议族`

```c
#include <sys/types.h>
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
```

- `domain` 参数指定了通信协议族，常见的有 `AF_INET`（IPv4）和 `AF_INET6`（IPv6）。
- `type` 参数指定了套接字的类型，常见的有 `SOCK_STREAM`（流套接字，用于面向连接的TCP通信）和 `SOCK_DGRAM`（数据报套接字，用于无连接的UDP通信）等等。
- `protocol` 参数指定了使用的具体协议，通常传递0以选择默认协议。

简单实例：

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>

int main() {
    // 创建一个IPv4的TCP套接字
    int sockfd = socket(AF_INET, SOCK_STREAM, 0);

    if (sockfd == -1) {
        perror("Socket creation failed");
        return 1;
    }

    printf("Socket created successfully\n");

    // 关闭套接字
    close(sockfd);

    return 0;
}
```

这只是socket的创建过程，实际上你还需要进行一系列的操作，如绑定地址、监听连接、接受连接等，具体的操作取决于你的应用场景。



## 1、报式套接字

相比较于其他的套接字，他所包含的内容和使用是要丰富一些的

`主动端：`

1、取得socket

2、给socket绑定地址（可以省略）

3、发收消息

4、关闭socket



`被动端`：先运行

1、取得socket

2、给socket绑定地址

3、收发消息

4、关闭socket



`以下示例展示一个完整的报时套接字的通信`：

2、有关通信协议的设置

proto.h

```c
#ifndef __PROTO_H__
#define __PROTO_H__

#define NAMEMAX  (512 - 8 - 8)    //512是udp传输推荐的报长度，8是udp报头长度，8是math和chinese的长度
#define RCVPORT  "1989"

#define IPSTRSIZE 10

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <stdint.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>

struct msg_st
{
    uint32_t math;   //四字节
    uint32_t chinese;   //四字节
    uint8_t name[1];
}__attribute__((packed));  //不对齐

#endif
```

2、接收端

rcver.c

```c
#include "proto.h"

int main()
{

    int sockfd;
    struct msg_st *rcvbufp;
    struct sockaddr_in laddr, raddr;
    socklen_t raddr_len;
    char ipstr[IPSTRSIZE];

    int size = sizeof(struct msg_st) + NAMEMAX - 1;
    rcvbufp = malloc(sizeof(size));

    sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sockfd < 0)
    {
        perror("socket()");
        exit(1);
    }

    laddr.sin_family = AF_INET;
    laddr.sin_port = htons(atoi(RCVPORT));
    //将点分式表示的地址转换成二进制表示的ip地址
    inet_pton(AF_INET, "0.0.0.0", &laddr.sin_addr); //"0.0.0.0"表示能够匹配任何地址

    if (bind(sockfd, (void *)&laddr, sizeof(laddr)) < 0)
    {
        perror("bind()");
        exit(1);
    }

    /**
     * 对raddr_len初始化
    */
   raddr_len = sizeof(raddr);

    while (1)
    {
        recvfrom(sockfd,rcvbufp,size,0,(void*)&raddr,&raddr_len);
        //将二进制表示的地址转换成点分式表示的ip地址
        inet_ntop(AF_INET,&raddr.sin_addr,ipstr,IPSTRSIZE);
        printf("---MESSAGE FROM %s:%d---\n",ipstr,ntohs(raddr.sin_port));
        printf("NAME = %s\n",rcvbufp->name);
        //因为考虑到大端小端存储，所以以下两项需要进行转换，网络到主机
        printf("MATH = %d\n",ntohl(rcvbufp->math));
        printf("CHINESE = %d\n",ntohl(rcvbufp->chinese));
    }

    close(sockfd);

    exit(0);
}
```

3、发送端

snder.c

```c
#include "proto.h"

int main(int argc,char *argv[])
{

    if(argc < 3)
    {
        fprintf(stderr,"Usage.......\n");
        exit(1);
    }

    if(strlen(argv[2]) > NAMEMAX)
    {
        fprintf(stderr,"名字太长\n");
        exit(1);
    }

    int sndfd;
    struct msg_st *sndbufp;
    struct sockaddr_in raddr;

    sndfd = socket(AF_INET,SOCK_DGRAM,0);
    if(sndfd < 0)
    {
        perror("socket()");
        exit(1);
    }

    int size = sizeof(struct msg_st) + strlen(argv[2]);
    sndbufp = malloc(size);
    if(sndbufp < 0)
    {
        perror("malloc()");
        exit(1);
    }

    //bind()省去
    strcpy(sndbufp->name, argv[2]);
    sndbufp->chinese = htonl(100);
    sndbufp->math = htonl(100);

    raddr.sin_family = AF_INET;
    raddr.sin_port = htons(atoi(RCVPORT));
    inet_pton(AF_INET,argv[1],&raddr.sin_addr.s_addr);

    if(sendto(sndfd,sndbufp,size,0,(void *)&raddr,sizeof(raddr))<0)
    {
        perror("sendto()");
        exit(1);
    }

    puts("OK!");

    close(sndfd);

    exit(1);
}
```

`运行结果（先运行接收端）`

<img src="李慧琴c语言系统编程/image-20231212151856265.png" alt="image-20231212151856265" style="zoom: 67%;" />



> `多点通讯`：广播（全网广播，子网广播）、多播/组播

setsockopt()

getsockopt()

> 广播示例：

snder.c

```c
#include "proto.h"

int main(int argc,char *argv[])
{

    if(argc < 2)
    {
        fprintf(stderr,"Usage.......\n");
        exit(1);
    }

    if(strlen(argv[1]) > NAMEMAX)
    {
        fprintf(stderr,"名字太长\n");
        exit(1);
    }

    int sndfd;
    struct msg_st *sndbufp;
    struct sockaddr_in raddr;

    sndfd = socket(AF_INET,SOCK_DGRAM,0);
    if(sndfd < 0)
    {
        perror("socket()");
        exit(1);
    }

    int val = 1;

    if(setsockopt(sndfd,SOL_SOCKET,SO_BROADCAST,&val,sizeof(val)) < 0)
    {
        perror("setsockopt()");
        exit(1);
    }

    int size = sizeof(struct msg_st) + strlen(argv[2]);
    sndbufp = malloc(size);
    if(sndbufp < 0)
    {
        perror("malloc()");
        exit(1);
    }

    //bind()省去
    strcpy(sndbufp->name, argv[1]);
    sndbufp->chinese = htonl(100);
    sndbufp->math = htonl(100);

    raddr.sin_family = AF_INET;
    raddr.sin_port = htons(atoi(RCVPORT));
    inet_pton(AF_INET,"255.255.255.255",&raddr.sin_addr.s_addr);

    if(sendto(sndfd,sndbufp,size,0,(void *)&raddr,sizeof(raddr))<0)
    {
        perror("sendto()");
        exit(1);
    }

    puts("OK!");

    close(sndfd);

    exit(1);
}

```

rcver.c

```c
#include "proto.h"

int main()
{

    int sockfd;
    struct msg_st *rcvbufp;
    struct sockaddr_in laddr, raddr;
    socklen_t raddr_len;
    char ipstr[IPSTRSIZE];

    int size = sizeof(struct msg_st) + NAMEMAX - 1;
    rcvbufp = malloc(sizeof(size));

    sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sockfd < 0)
    {
        perror("socket()");
        exit(1);
    }

    int val = 1;
    if(setsockopt(sockfd,SOL_SOCKET,SO_BROADCAST,&val,sizeof(val)))
    {
        perror("setsockopt()");
        exit(1);
    }

    laddr.sin_family = AF_INET;
    laddr.sin_port = htons(atoi(RCVPORT));
    //将点分式表示的地址转换成二进制表示的ip地址
    inet_pton(AF_INET, "0.0.0.0", &laddr.sin_addr); //"0.0.0.0"表示能够匹配任何地址

    if (bind(sockfd, (void *)&laddr, sizeof(laddr)) < 0)
    {
        perror("bind()");
        exit(1);
    }

    /**
     * 对raddr_len初始化
    */
   raddr_len = sizeof(raddr);

    while (1)
    {
        recvfrom(sockfd,rcvbufp,size,0,(void*)&raddr,&raddr_len);
        //将二进制表示的地址转换成点分式表示的ip地址
        inet_ntop(AF_INET,&raddr.sin_addr,ipstr,IPSTRSIZE);
        printf("---MESSAGE FROM %s:%d---\n",ipstr,ntohs(raddr.sin_port));
        printf("NAME = %s\n",rcvbufp->name);
        //因为考虑到大端小端存储，所以以下两项需要进行转换，网络到主机
        printf("MATH = %d\n",ntohl(rcvbufp->math));
        printf("CHINESE = %d\n",ntohl(rcvbufp->chinese));
    }

    close(sockfd);

    exit(0);
}
```

> 多播实例:

proto.h

```c
#ifndef __PROTO_H__
#define __PROTO_H__

#define NAMEMAX  (512 - 8 - 8)   //512是udp传输推荐的报长度，8是udp报头长度，8是math和chinese的长度
#define RCVPORT  "1989"
#define MGROUP  "224.2.2.2"
#define IPSTRSIZE 10

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <stdint.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>

struct msg_st
{
    uint32_t math;   //四字节
    uint32_t chinese;   //四字节
    uint8_t name[1];
}__attribute__((packed));  //不对齐

#endif

```

snder.c

```c
#include "proto.h"

int main(int argc, char *argv[])
{

    if (argc < 2)
    {
        fprintf(stderr, "Usage.......\n");
        exit(1);
    }

    if (strlen(argv[1]) > NAMEMAX)
    {
        fprintf(stderr, "名字太长\n");
        exit(1);
    }

    int sndfd;
    struct msg_st *sndbufp;
    struct sockaddr_in raddr;

    sndfd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sndfd < 0)
    {
        perror("socket()");
        exit(1);
    }

    struct ip_mreqn mreq;

    inet_pton(AF_INET, MGROUP, &mreq.imr_multiaddr);
    inet_pton(AF_INET, "0.0.0.0", &mreq.imr_address);
    mreq.imr_ifindex = if_nametoindex("eth0");

    // 创建多播组
    if (setsockopt(sndfd, IPPROTO_IP, IP_MULTICAST_IF, &mreq, sizeof(mreq)) < 0)
    {
        perror("setsockopt()");
        exit(1);
    }

    int size = sizeof(struct msg_st) + strlen(argv[2]);
    sndbufp = malloc(size);
    if (sndbufp < 0)
    {
        perror("malloc()");
        exit(1);
    }

    // bind()省去
    strcpy(sndbufp->name, argv[1]);
    sndbufp->chinese = htonl(100);
    sndbufp->math = htonl(100);

    raddr.sin_family = AF_INET;
    raddr.sin_port = htons(atoi(RCVPORT));
    inet_pton(AF_INET, MGROUP, &raddr.sin_addr.s_addr);

    if (sendto(sndfd, sndbufp, size, 0, (void *)&raddr, sizeof(raddr)) < 0)
    {
        perror("sendto()");
        exit(1);
    }

    puts("OK!");

    close(sndfd);

    exit(1);
}

```

rcver.c

```c
#include "proto.h"

int main()
{

    int sockfd;
    struct msg_st *rcvbufp;
    struct sockaddr_in laddr, raddr;
    socklen_t raddr_len;
    char ipstr[IPSTRSIZE];

    int size = sizeof(struct msg_st) + NAMEMAX - 1;
    rcvbufp = malloc(sizeof(size));

    sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sockfd < 0)
    {
        perror("socket()");
        exit(1);
    }

    struct ip_mreqn mreq;
    inet_pton(AF_INET, MGROUP, &mreq.imr_multiaddr); //多播组地址
    inet_pton(AF_INET, "0.0.0.0", &mreq.imr_address);
    mreq.imr_ifindex = if_nametoindex("eth0");

   //加入多播组
    if(setsockopt(sockfd,IPPROTO_IP,IP_ADD_MEMBERSHIP,&mreq,sizeof(mreq)) < 0)
    {
        perror("setsockopt()");
        exit(1);
    }

    laddr.sin_family = AF_INET;
    laddr.sin_port = htons(atoi(RCVPORT));
    //将点分式表示的地址转换成二进制表示的ip地址
    inet_pton(AF_INET, "0.0.0.0", &laddr.sin_addr); //"0.0.0.0"表示能够匹配任何地址

    if (bind(sockfd, (void *)&laddr, sizeof(laddr)) < 0)
    {
        perror("bind()");
        exit(1);
    }

    /**
     * 对raddr_len初始化
    */
   raddr_len = sizeof(raddr);

    while (1)
    {
        recvfrom(sockfd,rcvbufp,size,0,(void*)&raddr,&raddr_len);
        //将二进制表示的地址转换成点分式表示的ip地址
        inet_ntop(AF_INET,&raddr.sin_addr,ipstr,IPSTRSIZE);
        printf("---MESSAGE FROM %s:%d---\n",ipstr,ntohs(raddr.sin_port));
        printf("NAME = %s\n",rcvbufp->name);
        //因为考虑到大端小端存储，所以以下两项需要进行转换，网络到主机
        printf("MATH = %d\n",ntohl(rcvbufp->math));
        printf("CHINESE = %d\n",ntohl(rcvbufp->chinese));
    }

    close(sockfd);

    exit(0);
}

```



> UDP传输分析

UDP现象之一：`丢报`，主要原因是`阻塞`，解决办法是`流控`（停等式流控）

字段TTL，代表着能够跳转的路由的个数，在linux一般定义成64，足够使用。

<img src="李慧琴c语言系统编程/image-20231213110819875.png" alt="image-20231213110819875" style="zoom:50%;" />

停等式流控：
在消息传输过程中，会有一个ack应答信号进行控制传输。

<img src="李慧琴c语言系统编程/image-20231213112352697.png" alt="image-20231213112352697" style="zoom:50%;" />

data以及ack都需要添加编号，以免产生混淆。

> 使用UDP的传输形式重构之前的数据传输程序ftp

有限状态机构造

<img src="李慧琴c语言系统编程/image-20231213113126080.png" alt="image-20231213113126080" style="zoom:50%;" />

## 2、流式套接字

> TCP传输协议分析

开始发很多个data数据包，直到收到一个ack为止，每收到一个ack，就再次发出一个data包。将很多个包发出去，抢占沿途的路由资源，反正每个发出去的包都有自己对应的ack信号，所以只要根据反馈回来的ack包就可以确定哪个数据包丢失了，进行重传。

<img src="李慧琴c语言系统编程/image-20231213152625064.png" alt="image-20231213152625064" style="zoom:50%;" />

​	当上述发一个包接一个ack的过程进行没有错误产生的时候，后续改成接到一个ack就发送两个数据包，扩大发送包的数量。如果检测到丢包的话，就将发送包的数量减半，从丢的那个包开始重新传数据。

​	但是在传输之前，传输两端需要进行编号的约定。（`三次握手`）

<img src="李慧琴c语言系统编程/image-20231213155329674.png" alt="image-20231213155329674" style="zoom:80%;" />

以上基于半连接池的方式，会受到半连接池洪水攻击，即通过大量发送第一次握手而不发送第三次握手，使得发送方的半连接池满，从而接收不到真正需要接收的握手请求。

`为了解决这个问题，提出cookie`

对端ip＋对端端口 + 我端ip + 我端端口 + 两者的通信协议 + sort信息（由内核产生，一秒改变一次），再做hash操作，得到一个内容，称作`cookie`，当第一次握手后，发送方将cookie的内容和ack信息一同发出，当第三次握手的时候，接收方应该带上cookie来，经过验证之后，方才建立连接。

<img src="李慧琴c语言系统编程/image-20231213161418964.png" alt="image-20231213161418964" style="zoom: 80%;" />

> `流式套接字详解`

C端（主动端）接收方

1、获取SOCKET

2、给SOCKET取得地址（可以省略）

3、发送连接

4、收发消息

5、关闭



S端

1、获取SOCKET

2、给SOCKET取得地址

3、将SOCKET置为监听模式

4、接收连接

5、收发消息

6、关闭

> 实例程序

协议

proto.h

```c
#ifndef __PROTO_H__
#define __PROTO_H__

#include <sys/types.h>
#include <sys/socket.h>   // 基本socket函数和数据结构
#include <netinet/in.h>   // Internet地址族相关定义
#include <arpa/inet.h>    // 处理IP地址的函数
#include <unistd.h>       // UNIX系统调用

#define SERVERPORT      "1989"
#define FMT_STAMP       "%lld\r\n"


#endif
```

S端	server.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#include "proto.h"

#define IPSTRSIZE 40
#define BUFSIZE 1024

static void server_job(int sd)
{
    char buf[BUFSIZE];

    int len = sprintf(buf, FMT_STAMP, (long long)time(NULL));

    if (send(sd, buf, len, 0) < 0)
    {
        perror("send()");
        exit(1);
    }
}

int main()
{
    int sockfd;
    int newsockfd;
    struct sockaddr_in laddr, raddr;
    char ipstr[IPSTRSIZE];

    // 获取socket
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0)
    {
        perror("socket()");
        exit(1);
    }

    // 设置socket属性
    int val = 1;
    if(setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, val, sizeof(val)) < 0)  //SO_REUSEADDR可以使得闲置的地址立马可以被使用
    {
        perror("setsockopt()");
        exit(1);
    }

    laddr.sin_family = AF_INET;
    laddr.sin_port = htons(atoi(SERVERPORT));
    inet_pton(AF_INET, "0.0.0.0", &laddr.sin_addr.s_addr);

    // 取得地址
    if (bind(sockfd, (void *)&laddr, sizeof(laddr)) < 0)
    {
        perror("bind()");
        exit(1);
    }

    // 监听
    if (listen(sockfd, 200) < 0)
    {
        perror("listen()");
        exit(1);
    }

    // 接收连接
    socklen_t raddr_len = sizeof(raddr);

    while (1)
    {
        newsockfd = accept(sockfd, (void *)&raddr, &raddr_len);
        if (newsockfd < 0)
        {
            perror("accept()");
            exit(1);
        }

        inet_ntop(AF_INET, &raddr.sin_addr.s_addr, ipstr, IPSTRSIZE);
        printf("CLIENT:%s:%d\n", ipstr, ntohs(raddr.sin_port));

        // 收发消息
        server_job(newsockfd);

        close(newsockfd);
    }

    // 关闭
    close(sockfd);

    exit(0);
}
```

C端	client.c

```c
#include "proto.h"

int main(int argc, char *argv[])
{

    FILE *fp;
    long long stamp;

    if (argc < 2)
    {
        fprintf(stderr, "Usage....\n");
        exit(1);
    }

    int sd;
    struct sockaddr_in laddr;

    //获取socket
    sd = socket(AF_INET, SOCK_STREAM, 0);
    if (sd < 0)
    {
        perror("socket()");
        exit(1);
    }

    //连接
    laddr.sin_family = AF_INET;
    laddr.sin_port = htons(atoi(SERVERPORT));
    inet_pton(AF_INET, argv[1], &laddr.sin_addr.s_addr);

    if (connect(sd, (void *)&laddr, sizeof(laddr)) < 0)
    {
        perror("connect()");
        exit(1);
    }

    fp = fdopen(sd, "r+");
    if (fp == NULL)
    {
        perror("fdopen()");
        exit(1);
    }

    if(fscanf(fp, FMT_STAMP, &stamp) < 1)
    {
        fprintf(stderr,"Bad format\n");
    }else
    {
        fprintf(stdout,"stamp = %lld\n",stamp);
    }

    fclose(fp);
    
    //或者直接
	//recv()
    //close()
    
    exit(0);
}
```

`将以上的程序修改成并发的版本`

主要修改server端

server.c

```c
#include <time.h>

#include "proto.h"

#define IPSTRSIZE 40
#define BUFSIZE 1024

static void server_job(int sd)
{
    char buf[BUFSIZE];

    int len = sprintf(buf, FMT_STAMP, (long long)time(NULL));

    if (send(sd, buf, len, 0) < 0)
    {
        perror("send()");
        exit(1);
    }
}

int main()
{
    int sockfd;
    int newsockfd;
    struct sockaddr_in laddr, raddr;
    char ipstr[IPSTRSIZE];

    pid_t pid;

    // 获取socket
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0)
    {
        perror("socket()");
        exit(1);
    }

    // 设置socket属性
    int val = 1;
    if (setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, val, sizeof(val)) < 0)
    {
        perror("setsockopt()");
        exit(1);
    }

    laddr.sin_family = AF_INET;
    laddr.sin_port = htons(atoi(SERVERPORT));
    inet_pton(AF_INET, "0.0.0.0", &laddr.sin_addr.s_addr);

    // 取得地址
    if (bind(sockfd, (void *)&laddr, sizeof(laddr)) < 0)
    {
        perror("bind()");
        exit(1);
    }

    // 监听
    if (listen(sockfd, 200) < 0)
    {
        perror("listen()");
        exit(1);
    }

    // 接收连接
    socklen_t raddr_len = sizeof(raddr);

    while (1)
    {
        newsockfd = accept(sockfd, (void *)&raddr, &raddr_len);
        if (newsockfd < 0)
        {
            perror("accept()");
            exit(1);
        }

        pid = fork();
        if (pid < 0)
        {
            perror("fork()");
            exit(1);
        }

        if (pid == 0)
        {
            close(sockfd);
            inet_ntop(AF_INET, &raddr.sin_addr.s_addr, ipstr, IPSTRSIZE);
            printf("CLIENT:%s:%d\n", ipstr, ntohs(raddr.sin_port));
            // 收发消息
            server_job(newsockfd);

            close(newsockfd);
            
            exit(0);
        }
        close(newsockfd);
    }
    // 关闭
    close(sockfd);

    exit(0);
}
```











