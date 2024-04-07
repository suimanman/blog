

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

![截屏2024-04-07 11.09.09](/Users/wangmeice/github_imgs/截屏2024-04-07 11.09.09.png)

---

## BASH

