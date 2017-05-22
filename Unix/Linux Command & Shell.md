# Linux Command & Shell

---

## 什么是Linux

Linux的组成：

 - Linux内核（重中之重）；

 - GNU工具组件；

 - 图形化桌面环境；

 - 应用软件。

### Linux内核

内核的主要功能：

 - 系统内存管理；

 - 软件程序管理；

 - 硬件设备管理；

 - 文件系统管理。

#### 系统内存管理

内核管理者物理内存、创建和管理虚拟内存（通过磁盘上的存储空间来实现虚拟内存，这块存储空间也被称为交换空）

内核不断在交换空间和实际的物理内存之间交换虚拟内存存储中的内容，变相的扩大内存。

内存存储单元会被分成多个块，这些块被称为页面（page），内核会将每个内存页面放在物理内存或虚拟内存中。未被使用的内存页面会被内核swapping out到虚拟内存中，奖杯使用的内存页面会被内核swapping in到物理内存中，说明白点就是不用的内存数据持久化到磁盘上，用的使用在加载到物理内存中~

可以通过cat /proc/meminfo文件来查看内存的信息

``` cmd
cat /proc/meminfo
MemTotal:        8073300 kB 总物理内存
MemFree:         7277808 kB 空闲物理内存
MemAvailable:    7613620 kB 
Buffers:          151136 kB 文件缓冲区
Cached:           381300 kB 
SwapCached:            0 kB 
Active:           504376 kB
Inactive:         145772 kB
Active(anon):     121504 kB
Inactive(anon):     8524 kB
Active(file):     382872 kB
Inactive(file):   137248 kB
Unevictable:        3656 kB
Mlocked:            3656 kB
SwapTotal:       8286204 kB 交换空间的总大小
SwapFree:        8286204 kB 交换空间可用大小
...
```

一般，Linux系统中的每个进程都会有各自的内存页面，进程不能访问其他进程正在使用的内存页面。

但为了方便的共享数据，可以创建一些共享内存页面，多个进程可在同一共用内存区域进行读取和写入操作，内核就负责管理这块共用内存区，使用ipcs命令来查看系统上的共享内存页面：

``` cmd
# ipcs是Linux下显示进程间通信状态的工具。可以显示消息队列、共享内存和信号量的信息。
ipcs

--------- 消息队列 -----------
键        msqid      拥有者  权限     已用字节数 消息      

------------ 共享内存段 --------------
键        shmid      拥有者  权限     字节     连接数  状态      
0x00000000 0          root       644        80         2                       
0x00000000 32769      root       644        16384      2                       
0x00000000 65538      root       644        280        2                       

--------- 信号量数组 -----------
键        semid      拥有者  权限     nsems     
0x000000a7 65536      root       600        1 
```

#### 软件程序管理

内核管理这运行在Linux中的所有进程，内核会创建init进程用于启动其他进程，内核在创建每一个进程的时候都会在虚拟内存中给新创建的进程分配一块区域用来存储该进程用到的代码和数据。

部分Linux发行版使用一个表来管理系统开机时要自动启动的进程：/etc/inittab表文件。

而另一些发行版（如Ubuntu）则是采用/etc/init.d/目录，将开机时启动或停止应用的脚本放在此目录下。这些脚本通过/etc/rcX.d/目录下的入口启动，这里X代表运行级别：

``` cmd
# /etc/目录下
rc0.d/ rc1.d/ rc2.d/ rc3.d/ rc4.d/ rc5.d/ rc6.d/ rcS.d/
```

运行级别为1时，只有基本的系统进程会启动，同时会启动唯一一个控制台终端进程。我们称之为单用户模式。单用户模式通常用来在系统有问题时进行紧急的文件系统维护。此模式仅有一人才能登陆到系统，也就是系统管理员了~

标准的启动运行级别是3。大多数应用软件，比如网络支持程序等都会启动。

另一种常见的的运行等级是5，在这个运行级别上系统会启动图形化的X Window系统，同时允许用户通过图形化桌面窗口登录系统。

通过ps命令来查看进程信息：

``` cmd
#可以看到init进程的PID是1，也就是系统启动时第一个启动的进程。
#STAT表示当前进程的状态（S表示在睡眠，SW表示在睡眠和等待，R表示在运行中）
#进程名字在最后一列，方括号中的进程是由于不活动而被从内存中换出到磁盘交换空间的进程
ps ax
PID TTY      STAT   TIME COMMAND
1   ?        Ss     0:02 /sbin/init
2   ?        S      0:00 [kthreadd]
3   ?        S      0:00 [ksoftirqd/0]
5   ?        S<     0:00 [kworker/0:0H]
```

#### 硬件设备管理

内核管理着硬件设备，任何要与Linux系统通信的设备，都需要在内核代码中加入其驱动程序代码（driver），驱动相当于应用程序和硬件设备的中间人，Linux加入驱动代码有两种方式：

 - 将代码编译进内核的设备驱动中（低效，需重新编译内核）；

 - 插拔式的设备驱动模块（高效，即插即用，不用就拔除）。

Linux系统会将硬件设备当成特殊文件，称为设备文件，而设备文件又分为三类：

 - 字符型设备文件：每次只能处理一个字符的设备，大多数类型的调制解调器和终端都是该设备文件类型；

 - 块设备文件：每次能处理大块数据的设备，硬盘就是块设备文件；

 - 网络设备文件：采用数据包的发送和接收的设备，各种网卡和特殊的回环设备。

#### 文件系统管理

Linux内核支持的文件系统很多（包含了Windows）：

 - ext：最早的Linux文件系统；

 - ext2：Linux二代文件系统，在一代基础上提供了更多的功能；

 - ext3：Linux三代文件系统，支持日志功能；

 - ext4：Linux时代文件系统，支持高级日志功能；

 - hpfs：OS/2高性能文件系统；

 - jfs：IBM日志文件系统；

 - iso9660：ISO9660文件系统

 - minix：MINIX文件系统

 - msdos：微软的FAT16

 - ncp：Netware文件系统

 - nfs：网络文件系统

 - ntfs：微软NT系统

 - ReiserFS：高级Linux文件系统，提供更好的性能和数据恢复功能

 - smb：支持网络访问的Samba SMB文件系统

 - sysv：较早期的Unix文件系统

 - ufs：BSD文件系统

 - vfat：Win95文件系统

 - XFS：高性能64位日志文件系统

 - 等等..

任何供Linux服务器访问的硬盘都必须格式化成上诉文件系统类型中的一种。

Linux内核采用虚拟文件系统（VFS）作为和每个文件系统交互的接口。

---

## 基本的Bash Shell命令

大多数Linux发行版的默认shell都是GNU Bash Shell

### Bash 手册

通过man + command来展示command的具体作用，通过空格翻页，通过上下键逐行阅读。

在手册中会看到有些附带参数有的是单个-，有的是--，其中单个-的参数可以相互组合，而--参数必须分开书写

### 浏览文件系统

Linux常见的目录名称：

/ 根目录，通常不会在这里存储文件

/bin 二进制目录，存储着GNU用户级的二进制工具

/boot 启动目录，存放启动文件

/dev 设备目录，存储设备节点

/etc 系统配置文件目录

/home 用户的家目录

/lib 库目录

/media 媒体目录

/mnt 挂载目录，移动设备挂载点的地方

/opt 可选目录

/root root的家目录

/sbin 系统二进制目录，存储着GNU管理员的二进制工具

/tmp 临时目录，用于临时文件的创建和删除

/usr 用户安装软件的目录

/var 可变目录，用于存放经常变化的文件，比如日志文件

#### 列表命令

``` shell
# 区分文件和目录，会有明显的/出现
ls -F
av/  data/  opensoft/  resource/  UserXingBook/  xingbook/  xingbookdb.dump  zip/
# ll展示了文件更具体的信息
ll -i
53215233 drwxrwxrwx  9 root  root       4096 5月   2 15:28 ./
       2 drwxr-xr-x 24 root  root       4096 5月  15 14:42 ../
53216627 drwxr-xr-x  2 root  root       4096 5月   2 17:16 av/
53215235 drwxrwxrwx 10 mysql mysql      4096 4月  18 20:32 data/
54527155 drwxr-xr-x  5 root  root       4096 4月  20 15:30 opensoft/
53477380 drwxr-xr-x  3 root  root       4096 4月  20 13:58 resource/
53348060 drwxrwxrwx  4 root  root       4096 4月  19 17:52 UserXingBook/
53216028 drwxr-xr-x  7 root  root       4096 4月  20 14:36 xingbook/
53216023 -rw-r--r--  1 root  root  363250326 4月  18 20:41 xingbookdb.dump
53217968 drwxr-xr-x  2 root  root       4096 4月  20 14:36 zip/

# 
```

第零列的含义：目录和文件的索引值
第一列的含义：d表示目录，-表示文件，c是字符型文件，b为块文件；rwxrwxrwx表示文件的权限（读写执行）
第二列的含义：文件的硬链接总数
第三列的含义：文件所属的用户
第四列的含义：文件所属的用户组
第五列的含义：文件的大小，以字节为单位
第六列的含义：文件最近的修改时间
第七列的含义：文件或目录的名称

#### 链接文件

Linux中有两种不同类型的文件类型：

 - 符号链接，软链接；

 - 硬链接。

两者的区别：

![软硬链接的区别][1]

  [1]: https://i.stack.imgur.com/f7Ijz.jpg

更通俗易懂的链接：http://blog.csdn.net/mahao1107/article/details/46851969

#### 查看文件内容

``` shell
#查看文件统计信息，stat命令展示出的信息十分详细
stat xingbookdb.dump 
文件：'xingbookdb.dump'
大小：363250326 	块：709480     IO 块：4096   普通文件
设备：fc00h/64512d	Inode：53216023    硬链接：1
权限：(0644/-rw-r--r--)  Uid：(    0/    root)   Gid：(    0/    root)
最近访问：2017-04-18 20:49:02.358775234 +0800
最近更改：2017-04-18 20:41:14.000000000 +0800
最近改动：2017-04-18 20:48:18.515268041 +0800
创建时间：-

#查看文件类型，file命令
file hehe
# 软链接类型
hehe: symbolic link to xingbookdb.dump
file xingbookdb.dump 
# 文件类型
xingbookdb.dump: , init=0x454c, stat=0x4b20, dev=0x5945, bas=0x2053
file xingbook
# 目录类型
xingbook: directory

# 查看文件内容，less命令，像cat和more命令就不详细解释了
# -N 显示行号
less -N txt

# 查看文件末尾内容，tail命令
tail -n 200 显示最后两百行信息
tail -f 一直监听着文件的输出信息

# 查看文件头部内容，head命令
```

---

## 高级的Bash Shell命令

### 