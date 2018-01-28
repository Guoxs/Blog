---
title: Linux 操作基础
date: 2016-07-07 17:24:29
tags: Linux
---

## Linux 基础概念
Linux实际上指的是Linux内核即Linux Kernel，而平时常说常用的Linux，如CentOS、Red Hat、Ubuntu等都是Linux发行版，即Linux Distribution

Linux Kernel + Software + Tools  → Linux Distribution

**Linux特点**：免费开源，多任务、多用户，稳定，配置要求低

<!--more-->
**常用工具**： 

- VMware
- Putty
>Putty是一个Telnet、SSH、rlogin、纯TCP以及串行接口连接软件。

- SSH Secure Shell Client 
> `ssh secure shell client` 是一个用来替代TELNET、FTP以及R命令的工具包，主要是想解决口令在网上明文传输的问题。SSH是英文 `Secure Shell` 的简写形式。通过使用SSH，可以把所有传输的数据进行加密，这样"中间人"这种攻击方式就不可能实现了，而且也能够防止DNS欺骗和IP欺骗。

**几种用户界面**
**CLI:** command-line interface，命令行界面
**GUI:** Graphical User Interface，图形用户界面
**TUI:** Text-based User Interface，文本用户界面

## Linux文件结构
Linux文件结构图：
![Linux文件结构图][1]
### / 目录
第一层次结构的根、整个文件系统层次结构的根目录。

**/bin/**
需要在单用户模式可用的必要命令（可执行文件）；面向所有用户，例如：cat、ls、cp，和/usr/bin类似。

**/boot/**
引导程序文件，例如：kernel、initrd；时常是一个单独的分区。

**/dev/**
必要设备， 例如：/dev/null。

**/etc/**
特定主机，系统范围内的配置文件。

**/home/**
用户的家目录，包含保存的文件、个人设置等，一般为单独的分区。

**/lib/**
/bin/ and /sbin/ 中二进制文件必要的库文件。


**/media/**
可移除媒体(如CD-ROM)的挂载点 。

**/lost+found**
在ext3文件系统中，当系统意外崩溃或机器意外关机，会产生一些文件碎片在这里。当系统在开机启动的过程中fsck工具会检查这里，并修复已经损坏的文件系统。当系统发生问题。可能会有文件被移动到这个目录中，可能需要用手工的方式来修复，或移到文件到原来的位置上。

**/mnt/**
临时挂载的文件系统。比如cdrom,u盘等，直接插入光驱无法使用，要先挂载后使用。

**/opt/**
可选应用软件包。

**/proc/**
虚拟文件系统，将内核与进程状态归档为文本文件（系统信息都存放这目录下）。例如：uptime、 network。在Linux中，对应Procfs格式挂载。该目录下文件只能看不能改（包括root）。

**/root/**
超级用户的家目录。

**/sbin/**
必要的系统二进制文件，例如： init、 ip、 mount。**sbin目录下的命令，普通用户都执行不**了。

**/srv/**
站点的具体数据，由系统提供。

**/tmp/**
临时文件，**在系统重启时目录中文件不会被保留**。

**/usr/**
默认软件都会存于该目录下。用于存储只读用户数据的第二层次；包含绝大多数的(多)用户工具和应用程序。

**/var/**
**变量文件——在正常运行的系统中其内容不断变化的文件**，如日志，脱机文件和临时电子邮件文件。有时是一个单独的分区。如果不单独分区，有可能会把整个分区充满。如果单独分区，给大给小都不合适。

### /etc/目录
**/etc/sysconfig/network**
IP、掩码、网关、主机名配置。

**/etc/resolv.conf**
DNS服务器配置。

**/etc/fstab**
开机自动挂载系统，所有分区开机都会自动挂载。

**/etc/inittab**
设定系统启动时Init进程将把系统设置成什么样的runlevel及加载相关的启动文件配置。

**/etc/exports**
设置NFS系统用的配置文件路径。

**/etc/init.d**
这个目录来存放系统启动脚本。

**/etc/profile**, /etc/csh.login,  /etc/csh.cshrc
全局系统环境配置变量

**/etc/issue**
认证前的输出信息，默认输出版本内核信息。

**/etc/group**
类似/etc/passwd ，但说明的不是用户而是组。

**/etc/passwd**
用户数据库，其中的域给出了用户名、真实姓名、家目录、加密的口令和用户的其他信息。

**/etc/shadow**
在安装了影子口令软件的系统上的影子口令文件.影子口令文件将 /etc/passwd 文件中的加密口令移动到/etc/shadow 中，而后者只对root可读，这使破译口令更困难。

**/etc/sudoers**
可以sudo命令的配置文件

**/etc/syslog.conf**
系统日志参数配置

**/etc/skel/**
默认创建用户时，把该目录拷贝到家目录下。

### /usr/目录
**/usr/doc**
Linux技术文档。

**/usr/include**
用来存放Linux下开发和编译应用程序所需要的头文件。

**/usr/lib**
存放一些常用的动态链接共享库和静态档案库。

**/usr/man**
帮助文档所在的目录。

**/usr/src**
Linux开放的源代码，就存在这个目录。

**/usr/bin/**
非必要可执行文件 (在单用户模式中不需要)；面向所有用户。

**/usr/local/**
本地数据的第三层次，具体到本台主机。通常而言有进一步的子目录，例如：bin/、lib/、share/。这是提供给一般用户的/usr目录，在这里安装一般的应用软件。

### /var/目录
**/var/log/message**
日志信息，按周自动轮询。

**/var/spool/cron/root**
定时器配置文件目录，默认按用户命名。

**/var/log/secure**
记录登陆系统存取信息的文件，不管认证成功还是认证失败都会记录。

**/var/log/wtmp**
记录登陆者信息的文件，last、who、w 命令信息来源于此。

**/var/spool/clientmqueue/**
当邮件服务未开启时，所有应发给系统管理员的邮件都将堆放在此。

**/var/spool/mail/**
邮件目录。

**/var/log/**
各种程序的Log文件，特别是login   (/var/log/wtmp log所有到系统的登录和注销) 和syslog (/var/log/messages 里存储所有核心和系统程序信息）。 /var/log 里的文件经常不确定地增长，应该定期清除。

**/var/run**  
保存到下次引导前有效的关于系统的信息文件.例如， /var/run/utmp 包含当前登录的用户的信息.

**/var/cache/**
应用程序缓存数据。这些数据是在本地生成的一个耗时的I/O或计算结果。应用程序必须能够再生或恢复数据。缓存的文件可以被删除而不导致数据丢失。

### /proc/目录
**/proc/meminfo**
查看内存信息。

**/proc/cpuinfo**
关于处理器的信息，如类型、厂家、型号和性能等。

**/proc/cmdline**
加载 kernel 时所下达的相关参数。查阅此文件，可了解系统是如何启动的。

**/proc/filesystems**  
目前系统已经加载的文件系统。

**/proc/ioports**
目前系统上面各个装置所配置的 **I/O 位址**。

**/proc/kcore**
内存的大小。

**/proc/modules**
目前我们的 Linux 已经加载的模块列表，也可以想成是驱动程序。

**/proc/mounts**
系统已经挂载的数据。

###/dev/目录
**/dev/hd[a-t]**  IDE设备

**/dev/sd[a-z]**  SCSI设备

**/dev/ram[0-15]** 内存

**/dev/null** 无限数据接收设备,相当于黑洞

**/dev/zero** 无限零资源

**/dev/tty[0-63]** 虚拟终端

**/dev/ttyS[0-3]** 串口

**/dev/lp[0-3]** 并口

**/dev/console** 控制台

## Linux 常用命令

### 基础操作
1、`man [command]` 查看命令帮助。也可以用 `command –h` 或 `–-help` 查看简要帮助。查看帮助文件时，按Enter键下翻行，q键退出

2、`date` 显示时间日期，`uptime` 显示系统运行时间

3、`uname –a` 显示内核版本

4、-a = --all， -r -t = -rt

5、TAB 键自动补全，如有多个可补全项，连按TAB可现实是出所有可补全的文件名

6、`clear` 清屏， Shift + PgUp、PgDn 上下翻屏

7、`history` 查看历史命令，并可加参数以重复历史命令，`history –c` 清空历史命令，`history [n]` 显示最近的n条命令，`![n]` 执行history中的第n条命令

### 电源操作

1、`poweroff` 关机

2、`reboot` 重启

3、`init` 
 init 0 关机
 init 4 安全模式
 init 6 重启

### 用户及用户组操作
1、`su [user]` 切换用户 
`su –` 或 `su = su root`

2、`sudo [command]` 以**root权限**执行命令

3、`passwd` 修改密码

4、`id` 显示 uid 和 gid，root的id是0

5、`useradd [username]` 增加用户
>默认情况下新增用户如果不指定用户组则会新建一个以用户名命名的新用户组

6、`groupadd [groupname]` 增加用户组

7、`usermod -g [groupname] [username]` 修改用户所属组

8、`whoami`，`who，`w` 显示用户信息

9、`exit` 退出登录

### 文件及目录操作

 1、`~ `为用户目录（root的就是/root，user的就是/home/user）
相对路径：`. `为当前目录，`.. `为上一级目录，`-` 为上一目录

2、`pwd` 显示当前工作目录

3、`ls` 列出命令
 >**-a** 列出所有文件（包括文件名以 . 开头的隐藏文件）
 > **-R** 显示子目录结构
 > **-l** 显示详细信息 = `ll`

4、`cd` 切换目录

5、`cat` 查看文件
>` tac `逆向查看文件

6、`more`，`less` 翻页查看文件 （more只能下翻，less可上可下）

7、`head` 查看头几行，`tail` 查看后几行
 >`head +n/-n` 表示显示头n行或不显示后n行
 > `tail +n/-n` 表示不显示头n行或显示后n行

8、`touch` 更新文件修改时间或新建文件
> **-a** 仅修改访问时间
>  **-c** 仅修改时间，如无文件也不新建文件
>  **-m** 仅修改修改时间


 >Linux 的三个时间参数
 > **modification time（mtime）**：文件内容更改的时间，ls默认显示的时间
 > **status time（ctime）**：状态更改（权限或属性更改）的时间
 > **access time（atime）**：文件内容被取用的时间

9、`mkdir` 新建目录，`rmdir` 删除空目录

10、`cp [option] source destination` 复制
>**-a** 相当于`-pdr`，常用于备份
> **-i** 交互模式，目标文件已存在时询问是否覆盖
> **-p** 文件属性也复制（默认情况下，目标文件的所有者是cp命令操作者）
> **-d** 若源文件为连接文件，则只复制连接文件
> **-r** 递归复制，用于目录的复制
> **-l** 创建硬连接
> **-s** 创建软连接

11、`mv [option] source destination` 移动或重命名
 >**-f** 强制，直接覆盖
 > **-i** 询问是否覆盖
 > **-u** 若source比较新才会更新

12、`rm` 删除
 **-f** 强制
 **-r** 递归
 **-i** 交互
  **rm -rf** 强制删除文件夹

13、`file` 查看文件类型

14、`which` 在 **$PATH** 中查找可执行文件

15、`whereis` 在数据库中查找

16、`find [PATH] [option] [action]` 高级的查找命令

### 文件类型
>**d** 目录
>**-** 文件
>**2** 连接文件（hard link，symbolic link），只记录硬链接数
>**b** 可供存储设备
>**c** 串行端口设备
>`file [filename]` 查看文件类型

![文件信息][2]
信息从左到右分别是：
文件类型、权限、连接、用户、用户组、大小、修改时间、文件名

**hard link 和 symbolic link**

硬链接和软连接：
![硬链接和软连接][3]

**硬连接**
指通过**索引节点**来进行连接。在Linux的文件系统中，保存在磁盘分区中的文件不管是什么类型都给它分配一个编号，称为索引节点号 (**Inode Index**)。在Linux中，多个文件名指向同一索引节点是存在的，一般这种连接就是硬连接。

`硬连接的作用`： 允许一个文件拥有多个有效路径名，这样用户就可以建立硬连接到重要文件，以防止“误删”的功能。**文件真正删除的条件是与之相关的所有硬连接文件均被删除**。

**硬连接的2个限制**

- 不允许给目录创建硬链接
- 只有在同一文件系统中的文件之间才能创建链接。
>即不同硬盘分区上的两个文件之间不能够建立硬链接。这是因为硬链接是通过结点指向原始文件的，而文件的`i-结点`在不同的文件系统中可能会不同。

**软连接**

也称为为符号连接（Symbolic）。软链接文件**类似于Windows的快捷方式**。它实际上是一个特殊的文件。在符号连接中，文件实际上是一个**文本文件**，其中包含的有另一文件的位置信息。删除原文件，对硬链接没有影响。

## 文件权限操作
**UGO模式** **user** **group** **others**

**a**（all） = u+g+o

**rwx**  read write execute ，421，7=4+2+1=rwx

 **对于文件**：

- `r`，可读取文件内容
- `w`，可编辑文件内容（不包含删除文件的权限）
- `x`，可执行

 **对于目录**：

- `r`，可查询目录下文件名数据，即可ls
- `w`，新建、删除、重命名、移动目录内文件
- `x`，可进入该目录，即可cd

### 修改权限
1、`chown user file` 改变文件所有者

2、`chgrp group file` 改变文件用户组

3、`chmod`改变文件权限
>chmod u/g/o/a +/- r/w/x file
> chmod mark file  （mark必须用三个数字同时修改，如777）

**默认权限** `umask`
`umask` 查看umask，`umask mark` 设置umask
 文件的默认权限为：666 - umask
 目录的默认权限为：777 - umask

### 文件权限操作
**文件隐藏属性** 
1、`chattr +/-/= [option] file`
>**-A** 访问文件将不修改 atime
> **-S** 对该文件修改马上同步到磁盘（异步写入 VS 同步写入）
> **-a** 文件只能增加数据，不能修改和删除数据，只有 root 可设置
> **-c** 压缩文件
> **-d** 该文件不被 dump 备份
> **-i** 完全无法修改，添加删除文件的内容
> **-s** 该文件被删除时，所有数据将一干二净
> **-u** 与 s 相反

2、`lsattr` 列出文件的隐藏属性

3、**文件特殊权限** SUID，SGID，SBIT

- `SUID`，/usr/bin/password，执行者对该程序有x权限，程序执行时，获得程序所有者 owner 的权限，只能作用于二进制程序
- `SGID`，类似 SUID，针对 group 设置，可作用于文件或目录
- `SBIT`，只作用于目录，用户在目录下创建文件或目录时，只有自己和 root 有权移动删除

>**特殊权限的设置**
> SUID = 4, SGID = 2, SBIT = 1
>权限数字前加上特殊权限的数字即可，如chmod 4777 file，在 `rwx` 中只是把 `x` 改为 `s` 。

## vi,vim 编辑器
`命令模式`，默认进入，任何模式下按 **ESC** 返回命令模式
 `o` 在当前行的下面插入新行
 `dd` 删除整行
 `yy` 将当前行的内容放入缓存区 （复制）
 `n+yy` 复制多行
 `p` 粘贴
`u` 撤销
 `r` 替换
 `/` 查找关键字
 `i` 进入插入 **insert** 模式
 `：` 进入**ex模式**
 `:w` 保存
 `:q` 退出
 `:q!` 强制退出
 `:x` 保存并退出
 `:set number` 显示行号
 `：!` 系统命令
 `:sh` 切换到命令行，`ctrl+d` 切回


[1]: http://static.zybuluo.com/guoxs/pntt6qrd0wmsfz9366dx48u1/o_linux%E7%B3%BB%E7%BB%9F%E7%BB%93%E6%9E%84%E5%9B%BE.jpg
[2]: http://static.zybuluo.com/guoxs/3crpmnmvl82ztl10tf1t8wt4/%E5%9B%BE%E7%89%871.png
[3]: http://static.zybuluo.com/guoxs/3k3zvl4evgn2tuar1t2oq80t/111041250301052.png