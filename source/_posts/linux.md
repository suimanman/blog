

## 环境配置相关

虚拟机+乌班图

如何配置网络

如果配置 vim



### Linux 快捷键：

ctrl+alt+T 打开 shell

在特定的目录下打开 shell：即shell 的路径就是该目录



## 常用命令

到时候需要把内核源文件放到 usr/src 下

ctrl+alt +T :打开shell

使用 useradd 建立用户，但是需要设置密码。
/etc/passwd
/etc/shadow
/etc/sudoers 谁能切换哪些权限

/etc/fstab 记录了所有的挂载信息

切换用户：
sudo（获得临时权限）/su -（变更为root身份）（跟环境变量有关）、sudu su 直接切换至root
visudo 授权
ps 显示进程信息（与shell有关的）

* -l： 仅查看当前shell 产生的进程/

* -ef：查看系统所有进程信息

**管道**

例如 ps -ef | grep ntp，其中 ‘|’ 是管道，它用于进程间的通信，将前者的输出当成后者的输入。

top： 动态持续检测进程状态（5s）

pstree：查看进场间的相关性

**kill**

> 命令格式：
> 
>     kill -signal PID
> 
>     killall [选项] -signal 命令名称：将以 ”命令名称“ 命令创建的所有程序杀死。

该命令本质是发特定的信号给特定的进程（中断）？

默认为 15 号 SIGTEAM 信号（即正常终止进程）

**mount**（一对一挂载）

> moount /dev/sda/ /    将 sda 设备挂载到根目录，当我们挂载后，该目录可以查看设备的所有信息。

实验：把服务端的某个功能挂载到客户端的挂载点

tip：/mut 常用于挂载临时设备

文件权限：

8进制（3个2进制为分别表示 rwx)

创建目录

mkdir -p firstL/secondL (创建多级目录)

**ls**

列举文件信息

* -l ：显示详细信息。

* -a：显示所有文件（隐藏文件 . 开头）

ll hello*： 查看所有以 hello 开头的文件

**cat**

查看文件

-less 可以翻页

-tail 只看后面几行内容

**cp(copy)** 

cp -a file1 file2 （这里-a 同时拷贝了权限）

cp file1 file2 file3 dir //将 file1 file2 file3 复制到 dir

**mv(move)**

移动ro改名，根据最后一个参数进行判断

**rm(remove)**



**chmod**

改变文件访问权限

chmod [u、g、o（owner）、a] [+、-、=] 8进制数 （‘,’ 隔开不同权限组）



## Linux 下的 C  语言编译概述

1. 编辑器： vim
2. 编译器： gnu cc
3. 调试器：gdb

判断 文本or可执行 文件： file xxx

![image-20240328141445878](https://gitee.com/chaixy98/blog_image_public/raw/master/images/image-20240328141445878.png)



make 对编译进行简化，加速

软件安装文件内包含预检测系统和依赖脚本，用于检测软件环境、软件依赖是否充足



**Ubantu 包依赖管理**： DPKG：（命令：dpkg）（升级：apt/apt-get）

* 劣势：不能从远程的软件源下载软件包活处理软件依赖关系
* 解决：APT(Advanced Package Tool)：从 **/etc/apt/sources.list** 配置文件中定义的软件镜像源里下载软件包，可以在线处理复杂的软件依赖



命令列表：

```shell
apt install package 安装包
apt remove package 删除包
apt update 更新源
apt upgrade 更新已安装的包
apt full-upgrade 升级系统，自动智能的处理包的依赖关系

dpkg -i package  安装包
dpkg -r package 删除包
dpkg -l package 显示该包的版本
dpkg -l 列出当前已安装的包
dpkg –configure package 配置包
```



### Vim 相关

* **如何配置 Vim** ：~/.vimrc 文件配置 Vim（语法高亮、显示行号、智能缩进）（插件：vim-plug、vundle 等插件管理器安装插件、语法补全、自动添加头文件、 更换壁纸背景等功能。

* **Vim 三种模式**：
  * 一般命令模式（初始模式）
  * 编辑模式（按 i）
  * 命令行模式（在编辑模式下按 esc）
    * :wq 保存并退出
    * :q 退出
    * :wq! 强制保存退出
    * :q! 强制退出

### GCC

> 写一个动态链接库（实现一些功能性的函数）（利用 gcc 可以进行交叉平台编译的特性）
>
> 通过编译多个源文件，将目标代码链接成一个可执行文件

```shell
gcc -c [name] hello.c //只编译不链接
gcc -o [name] hello.o //链接
gcc -Wall -c hello.c  //编译时打印编译信息
gcc -O -c hello.c 	  //由编译器对代码进行优化，即指令重排等等优化
```

**编译**：

```shell
gcc sinc -lm -L/lib -L/lib64 //libm.so 链接lib和lib64 (系统库，默认会链接)
gcc sinc -lm -l/Path 		 //链接某个文件
```



**动态库**: **lib**xxx.**so**

* 劣势：无法独立运行
* 优势：包体小，运行时做链接，软件发布新版本时，替换动态库文件即可完成更新（灵活性高，通过函数指针实现，需要静态库做目录）

**静态库**:  **lib**xxx**.a**

* 劣势：包体大，直接将库函数链接到可执行文件中，每次发布新版本都要重新发布
* 有时：可独立运行

**生成方式：**

* 直接命令行
* 通过在 makefile 中实现

**如何生成动态库？**

如果源文件为 hello 那么 动态库名字可以为：libmyhello.so（源文件与库文件名字不做强制于要求）

```shell
1. gcc -fPIC -c hello.c //生成目标文件
2. gcc -shared -o libmyhello.so hello.o //指定生成的动态库
```



思考：程序运行时如果出现因找不到库文件而报错，可采取什么方法，使得系统可以找到库文件并加载至内存？

**为什么使用make？**

因为在开发过程中源文件越多，命令行越长（容易出错、且低效），例如：

```shell
gcc -c main.c
gcc -c haha.c
gcc -c sin_value.c
gcc -c cos_value.c
#链接
gcc -o main.o haha.o sin_value.o cos_value.o -lm
```

### Make 工程管理器

**优势：**

* 根据源文件修改时间戳：未改动源码不编译，直接链接
* 无需重复输入冗长 命令

#### Makefile 结构

```shell
Target: dependency_files
	command: /*该行必须以tab开头*/
```

其中target 即为需要生成的目标文件，dependency_files为生成该目标文件所依赖的文件

第二行需要tab开头，然后写生成上述文件所要执行的命令。

![](https://raw.githubusercontent.com/suimanman/imgs/main/%E6%88%AA%E5%B1%8F2024-04-07%2011.09.09.png)



---

## BASH

查看环境变量：

```shell
env
```

### Bash shell的操作环境

命令别名设置*alias，unalias*

>alias 别名='命令 选项'

```shell
alias lm='ls -al | more'
```

>然后就可以用lm代替设置的命令

```shell
unalias lm
```

>取消别名

### 数据流的重定向

#### >

>可通过输出重定向，将显示到显示器的输出重定向到指定的文件

1. 标准输入（STDIN）：

   文件描述符为0，重定向符号为<或<<

2. 标准输出（STDOUT）：

   文件描述符为1，重定向符号为>（覆盖方式）或>>（累加方式）

3. 标准错误输出（STDERR）：

   文件描述符为2，重定向符号为2>或2>>

`文件描述符`:代表一个给定的进程所打开文件的整数句柄。

无论进程需要操作哪些文件，标准输入，标准输出，标准错误输出这三个文件都必须打开。

<< “eof”：表示键盘输入“eof”时，输入结束。

#### 管道

｜

## Linux内核编译与管理

内核：本质是系统中的一个文件，通常是压缩文件，一台主机可有多个内核文件，启动时选择一个加载。

# 进程控制开发

## 概述

程序运行加载到内存，定义为进程，并给予一个ID，称为PID。

不同用户对程序运行时的权限是不同的。

> Linux的程序调用通常称为`fork-and-exec`流程。在Shell下输入命令可运行一个新程序，其过程：Shell进程在读取用户输入命令后先调用fork，生成一个子进程(与父进程几乎相同)，新的子进程再调用exec执行新的实际要执行的程序。

<img src="/Users/wangmeice/Library/Application Support/typora-user-images/image-20240418141728816.png" alt="image-20240418141728816" style="zoom:50%;" />

>
>
>fork时映射的是同一块物理空间，标记为只读，并采用“写时复制”技术，子进程在进行写操作时，产生缺页异常，并为子进程分配一块新的物理空间，存储新的数据段（执行代码不同时，也会分配新的空间来存储代码段）。

## exec函数族

提供了在一个进程内部启动另一个程序执行的方法。

⚠️注意：一定要加上错误判断

常见原因：

1. 找不到可执行文件或文件路径。
2. 指针数组argv和envp忘记用NULL结束。
3. 没有对应可执行文件的执行权限。

 exec的6个函数中真正的系统调用只有execve，其他5个均为库函数，它们最终都会调用execve系统调用。

## umask

通过设置文件权限掩码，指定用户在建立文件或目录时的权限值。

文件权限掩码：屏蔽掉文件权限的对应位。比如对于rwx，权限掩码为4，则屏蔽掉r，为2，屏蔽掉w。

## 守护进程设置

守护进程是运行在后台的一种用来提供服务的进程，他脱离控制台独立运行，守护进程是一种很有用的进程。

Linux的大多数服务器就是用守护进程实现的。比如，Internet服务器inetd，Web服务器httpd等。

1. fork()创建子进程

   >如果是进程组组长是不能调用setsid函数来开启新的会话的,而在shell下运行一个进程，那么这个进程就是一个新的进程组组长进程，所以我们就通过父进程fork创建子进程，而子进程虽然也是属于父进程这个进程组中，但是子进程并不是组长，所以是可以调用setsid函数来开启新的会话。

2. setsid()创建新会话

3. chdir('/')，设置工作目录为根目录。

4. umask(0)，将权限掩码设为0，不屏蔽任何权限。

5. 关闭文件描述符：close函数，从父进程中继承的已经打开的文件描述符，守护进程可能永远也不会读写，但同样消耗资源，所以全部关闭。

   <img src="/Users/wangmeice/Library/Application Support/typora-user-images/image-20240425143356728.png" alt="image-20240425143356728" style="zoom: 50%;" />

# 文件描述符

简称fd，是内核为了高效管理已被打开的文件所创建的索引。

在应用程序请求内核打开/新建一个文件时，内核会返回一个文件描述符对应于这个文件，类型为非负整数。

所有执行I/O操作的系统调用都通过文件描述符。

`通常一个进程启动时，会自动打开3个文件一标准输入、标准输出和标准出错处理。三个文件分别对应文件描述符为：STDIN_FILENO, STDOUT_FILENO, STDERR_ FILENO, 在POSIX中它们的值分别为0、1、2（定义在 <unistd.h>中)`

# 进程间通信

处于用户态的各进程拥有各自独立的地址空间，彼此之间不能直接访问对方的内存地址。因此进程之间要交换数据必须通过内核.

内核提供的进程之间的信息交换的机制称为进程间通信（IPC,InterProcess Communication）

`不同进程靠内存的公共区域来进行数据的交互。`

<img src="/Users/wangmeice/Library/Application Support/typora-user-images/image-20240425145454920.png" alt="image-20240425145454920" style="zoom:50%;" />

## 管道通信

一种最基本的进程间通信机制，是一个进程连接数据流到另一个进程的通道。系统执行命令时，经常需要讲一个程序的输出交给另一个应用程序的输入，如*ps -ef | grep ntp*

管道本质是一个文件，对应一段内核管理的内存缓冲区，临时存储进程发送的信息。

### 无名管道PIPE----‘｜’

当一个管道建立时，会返回两个文件描述符fd[0]和fd[1]，其中fd[0]固定用于读管道，而fd[1]固定用于写管道。

```shell
int pipefd[2];
pipe(pipefd);//用来创建管道
```



## dup2函数

Int dup2(int oldfd,int newfd)

作用：常用于重定向进程中的STDIN，STDOUT，STDERR.

```shell
int oldfd;
oldfd = open ("app_log", (O_RDWR | O_CREATE), 0644 );
3. dup2（oldfd，1 );
4. close (oldfd);
//“写到标准输出的东西，都将改为写人名为“app_log”的文件中
```

## 标准流管道

函数popen().

作用：调用forkO产生子进程，再从子进程中调用“Jbin/sh-e”将参数command作为完整的命令来执行。此外popen（）会建立管道传递数据给子进程（写），或者通过子进程接收数据（读）

函数pclose( )    关闭由popen打开的I/O流，只在popen启动的进程结束后才返回。	

## 命名管道（FIFO）



