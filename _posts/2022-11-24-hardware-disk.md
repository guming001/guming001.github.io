# 存储结构与磁盘划分

### Linux系统中的文件存储结构
*Linux系统中常见的目录名称以及相应内容*

| 目录名称 | 应放置文件的内容 |
| :---: | :--- |
| /boot | 开机所需文件——内核、开机菜单以及所需配置文件等 |
| /dev | 以文件形式存放任何设备与接口 |
| /etc | 配置文件 |
| /home | 用户家目录 |
| /bin | 存放单用户模式下还可以操作的命令 |
| /lib | 开机时用到的函数库，以及/bin与/sbin下面的命令要调用的函数 |
| /sbin | 开机过程中需要的命令 |
| /media | 用于挂载设备文件的目录 |
| /opt | 放置第三方的软件 |
| /root | 系统管理员的家目录 |
| /srv | 一些网络服务的数据文件目录 |
| /tmp | 任何人均可使用的“共享”临时目录 |
| /proc | 虚拟文件系统，例如系统内核、进程、外部设备及网络状态等 |
| /usr/local | 用户自行安装的软件 |
| /usr/sbin | Linux系统开机时不会使用到的软件/命令/脚本 |
| /usr/share | 帮助与说明文件，也可放置共享文件 |
| /var | 主要存放经常变化的文件，如日志 |
| /lost+found | 当文件系统发生错误时，将一些丢失的文件片段存放在这里 |

### 物理设备的命名规则
*常见的硬件设备及其文件名称*

| 硬件设备 | 文件名称 |
| :---: | :--- |
| IDE设备 | /dev/hd[a-d] |
| SCSI/SATA/U盘 | /dev/sd[a-p] |
| 软驱 | /dev/fd[0-1] |
| 打印机 | /dev/lp[0-15] |
| 光驱 | /dev/cdrom |
| 鼠标 | /dev/mouse |
| 磁带机 | /dev/st0或/dev/ht0 |

硬盘设备是由大量的扇区组成的，每个扇区的容量为512字节。其中第一个扇区最重要，它里面保存着主引导记录与分区表信息。就第一个扇区来讲，主引导记录需要占用446字节，分区表为64字节，结束符占用2字节；其中分区表中每记录一个分区信息就需要16字节，这样一来最多只有4个分区信息可以写到第一个扇区中，这4个分区就是4个主分区。

![image00571](https://user-images.githubusercontent.com/2433557/203793957-1a304773-dc07-4a95-b479-2001815e005c.png)

现在，问题来了——第一个扇区最多只能创建出4个分区？于是为了解决分区个数不够的问题，可以将第一个扇区的分区表中16字节（原本要写入主分区信息）的空间（称之为扩展分区）拿出来指向另外一个分区。也就是说，扩展分区其实并不是一个真正的分区，而更像是一个占用16字节分区表空间的指针——一个指向另外一个分区的指针。这样一来，用户一般会选择使用3个主分区加1个扩展分区的方法，然后在扩展分区中创建出数个逻辑分区，从而来满足多分区（大于4个）的需求。当然，就目前来讲大家只要明白为什么主分区不能超过4个就足够了。主分区、扩展分区、逻辑分区可以像下图那样来规划。

![image00421](https://user-images.githubusercontent.com/2433557/203794148-2f359694-568d-439b-9e32-8e8d25d0775b.png)

### 文件系统与数据资料
- **Ext3**：是一款日志文件系统，能够在系统异常宕机时避免文件系统资料丢失，并能自动修复数据的不一致与错误。然而，当硬盘容量较大时，所需的修复时间也会很长，而且也不能百分之百地保证资料不会丢失。它会把整个磁盘的每个写入动作的细节都预先记录下来，以便在发生异常宕机后能回溯追踪到被中断的部分，然后尝试进行修复。
- **Ext4**：Ext3的改进版本，作为RHEL 6系统中的默认文件管理系统，它支持的存储容量高达1EB（1EB=1,073,741,824GB），且能够有无限多的子目录。另外，Ext4文件系统能够批量分配block块，从而极大地提高了读写效率。
- **XFS**：是一种高性能的日志文件系统，而且是RHEL 7中默认的文件管理系统，它的优势在发生意外宕机后尤其明显，即可以快速地恢复可能被破坏的文件，而且强大的日志功能只用花费极低的计算和存储性能。并且它最大可支持的存储容量为18EB，这几乎满足了所有需求。

Linux只是把每个文件的权限与属性记录在inode中，而且每个文件占用一个独立的inode表格，该表格的大小默认为128字节，里面记录着如下信息：

- 该文件的访问权限（read、write、execute）
- 该文件的所有者与所属组（owner、group）
- 该文件的大小（size）
- 该文件的创建或内容修改时间（ctime）
- 该文件的最后一次访问时间（atime）
- 该文件的修改时间（mtime）
- 文件的特殊权限（SUID、SGID、SBIT）
- 该文件的真实数据地址（point）

而文件的实际内容则保存在block块中（大小可以是1KB、2KB或4KB，一般连续八个扇区组成一个block，即4KB），一个inode的默认大小仅为128B（Ext3），记录一个block则消耗4B。当文件的inode被写满后，Linux系统会自动分配出一个block块，专门用于像inode那样记录其他block块的信息，这样把各个block块的内容串到一起，就能够让用户读到完整的文件内容了。

![image](https://user-images.githubusercontent.com/2433557/203795783-b2366265-4806-4cd1-b8ba-1704ca52b204.png)

![image](https://user-images.githubusercontent.com/2433557/203795835-f0acc1b4-e607-400f-9942-96c6995bfc7e.png)

**删除乱码文件**

```
 方法一：
 touch a.txt
 ls -i a.txt
 145326742727878 a.txt
 find . -inum 145326742727878 -exec rm -i {} \;
 解释：意思就是find找到内容作为后面rm删除对象
 
 方法二：
 ls -i at.txt
 10663364 at.txt
 find . -inum 100663364 | xargs rm -f //xargs这个参数就是强力的意思，如 果前面的输出结果包含空格或制表符也会被强力执行）
```

**inode耗尽故障处理**

```
过程：
正常对一块磁盘分区，格式化，挂载（此处是实验所以尽量将分区设置的小一点）
df -i / 挂载点 查看该挂载点的inode点的可用数量
vi 一个shell文件，具体内容如下：
    vi kill.sh
    #! /bin/bash
    i=l
    while [ i
    let i++
    done
然后df -i 查询下inode可用数量还有没有
确认inode数量没有后，再touch一个新文件，看看是什么结果
```

计算机系统在发展过程中产生了众多的文件系统，为了使用户在读取或写入文件时不用关心底层的硬盘结构，Linux内核中的软件层为用户程序提供了一个VFS（Virtual File System，虚拟文件系统）接口，这样用户实际上在操作文件时就是统一对这个虚拟文件系统进行操作了。下所示为VFS的架构示意图。从中可见，实际文件系统在VFS下隐藏了自己的特性和细节，这样用户在日常使用时会觉得“文件系统都是一样的”，也就可以随意使用各种命令在任何文件系统中进行各种操作了（比如使用cp命令来复制文件）。

![image00529](https://user-images.githubusercontent.com/2433557/203796633-a786d5bc-fb43-4420-8ba5-42764b526274.png)

### 挂载硬件设备
#### mount命令
mount命令用于挂载文件系统，格式为“mount 文件系统 挂载目录”。

*mount命令中的参数以及作用*

| 参数 | 作用 |
| :---: | :--- |
| -a | 挂载所有在/etc/fstab中定义的文件系统 |
| -t | 指定文件系统的类型 |

`[root@linuxprobe ~]# mount /dev/sdb2 /backup`

如果想让硬件设备和目录永久地进行自动关联，就必须把挂载信息按照指定的填写格式“设备文件 挂载目录 格式类型 权限选项 自检 优先级”写入到/etc/fstab文件中。

*用于挂载信息的指定填写格式中，各字段所表示的意义*

| 字段 | 意义 |
| :---: | :--- |
| 设备文件 | 一般为设备的路径+设备名称，也可以写唯一识别码（UUID，Universally Unique Identifier） |
| 挂载目录 | 指定要挂载到的目录，需在挂载前创建好 |
| 格式类型 | 指定文件系统的格式，比如Ext3、Ext4、XFS、SWAP、iso9660（此为光盘设备）等 |
| 权限选项 | 若设置为defaults，则默认权限为：rw, suid, dev, exec, auto, nouser, async |
| 自检 | 若为1则开机后进行磁盘自检，为0则不自检 |
| 优先级 | 若“自检”字段为1，则可对多块硬盘进行自检优先级设置 |

#### umount命令
umount命令用于撤销已经挂载的设备文件，格式为“umount [挂载点/设备文件]”。

`[root@linuxprobe ~]# umount /dev/sdb2`

### 添加硬盘设备
首先需要在虚拟机中模拟添加入一块新的硬盘存储设备，然后再进行分区、格式化、挂载等操作，最后通过检查系统的挂载状态并真实地使用硬盘来验证硬盘设备是否成功添加。

#### fdisk命令
在Linux系统中，管理硬盘设备最常用的方法就当属fdisk命令了。fdisk命令用于管理磁盘分区，格式为“fdisk [磁盘名称]”，它提供了集添加、删除、转换分区等功能于一身的“一站式分区服务”。

*fdisk命令中的参数以及作用*

| 参数 | 作用 |
| :---: | :--- |
| m | 查看全部可用的参数 |
| n | 添加新的分区 |
| d | 删除某个分区信息 |
| l | 列出所有可用的分区类型 |
| t | 改变某个分区的类型 |
| p | 查看分区信息 |
| w | 保存并退出 |
| q | 不保存直接退出 |

我们首先使用fdisk命令来尝试管理/dev/sdb硬盘设备。在看到提示信息后输入参数p来查看硬盘设备内已有的分区信息，其中包括了硬盘的容量大小、扇区个数等信息：

```
[root@linuxprobe ~]# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x47d24a34.
Command (m for help): p
Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x47d24a34
Device Boot Start End Blocks Id System
```

输入参数n尝试添加新的分区。系统会要求您是选择继续输入参数p来创建主分区，还是输入参数e来创建扩展分区。这里输入参数p来创建一个主分区：

```
Command (m for help): n
Partition type:
p primary (0 primary, 0 extended, 4 free)
e extended
Select (default p): p
```

在确认创建一个主分区后，系统要求您先输入主分区的编号。我们在前文得知，主分区的编号范围是1～4，因此这里输入默认的1就可以了。接下来系统会提示定义起始的扇区位置，这不需要改动，我们敲击回车键保留默认设置即可，系统会自动计算出最靠前的空闲扇区的位置。最后，系统会要求定义分区的结束扇区位置，这其实就是要去定义整个分区的大小是多少。我们不用去计算扇区的个数，只需要输入+2G即可创建出一个容量为2GB的硬盘分区。

```
Partition number (1-4, default 1): 1
First sector (2048-41943039, default 2048):此处敲击回车
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-41943039, default 41943039): +2G
Partition 1 of type Linux and of size 2 GiB is set
```

再次使用参数p来查看硬盘设备中的分区信息。果然就能看到一个名称为/dev/sdb1、起始扇区位置为2048、结束扇区位置为4196351的主分区了。这时候千万不要直接关闭窗口，而应该敲击参数w后回车，这样分区信息才是真正的写入成功啦。

```
Command (m for help): p
Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x47d24a34
Device Boot Start End Blocks Id System
/dev/sdb1 2048 4196351 2097152 83 Linux
Command (m for help): w
The partition table has been altered!
Calling ioctl() to re-read partition table.
Syncing disks.
```

在上述步骤执行完毕之后，Linux系统会自动把这个硬盘主分区抽象成/dev/sdb1设备文件。我们可以使用file命令查看该文件的属性，但是刘遄老师在讲课和工作中发现，有些时候系统并没有自动把分区信息同步给Linux内核，而且这种情况似乎还比较常见（但不能算作是严重的bug）。我们可以输入***partprobe***命令手动将分区信息同步到内核，而且一般推荐连续两次执行该命令，效果会更好。如果使用这个命令都无法解决问题，那么就重启计算机吧，这个杀手锏百试百灵，一定会有用的。

```
[root@linuxprobe ]# file /dev/sdb1
/dev/sdb1: cannot open (No such file or directory)
[root@linuxprobe ]# partprobe
[root@linuxprobe ]# partprobe
[root@linuxprobe ]# file /dev/sdb1
/dev/sdb1: block special
```

如果硬件存储设备没有进行格式化，则Linux系统无法得知怎么在其上写入数据。因此，在对存储设备进行分区后还需要进行格式化操作。在Linux系统中用于格式化操作的命令是mkfs。这条命令很有意思，因为在Shell终端中输入mkfs名后再敲击两下用于补齐命令的Tab键，会有如下所示的效果：

```
[root@linuxprobe ~]# mkfs
mkfs         mkfs.cramfs   mkfs.ext3   mkfs.fat     mkfs.msdos   mkfs.xfs
mkfs.btrfs   mkfs.ext2     mkfs.ext4   mkfs.minix   mkfs.vfat
```

```
[root@linuxprobe ~]# mkfs.xfs /dev/sdb1
meta-data=/dev/sdb1 isize=256 agcount=4, agsize=131072 blks
= sectsz=512 attr=2, projid32bit=1
= crc=0
data = bsize=4096 blocks=524288, imaxpct=25
= sunit=0 swidth=0 blks
naming =version 2 bsize=4096 ascii-ci=0 ftype=0
log =internal log bsize=4096 blocks=2560, version=2
= sectsz=512 sunit=0 blks, lazy-count=1
realtime =none extsz=4096 blocks=0, rtextents=0
```

终于完成了存储设备的分区和格式化操作，接下来就是要来挂载并使用存储设备了。与之相关的步骤也非常简单：首先是创建一个用于挂载设备的挂载点目录；然后使用mount命令将存储设备与挂载点进行关联；最后使用df -h命令来查看挂载状态和硬盘使用量信息。

```
[root@linuxprobe ~]# mkdir /newFS
[root@linuxprobe ~]# mount /dev/sdb1 /newFS/
[root@linuxprobe ~]# df -h
Filesystem            Size  Used Avail   Use%  Mounted on
/dev/mapper/rhel-root  18G  3.5G   15G    20%  /
devtmpfs              905M     0  905M     0%  /dev
tmpfs                 914M  140K  914M     1%  /dev/shm
tmpfs                 914M  8.8M  905M     1%  /run
tmpfs                 914M     0  914M     0%  /sys/fs/cgroup
/dev/sr0              3.5G  3.5G     0   100%  /media/cdrom
/dev/sda1             497M  119M  379M    24%  /boot
/dev/sdb1             2.0G   33M  2.0G     2%  /newFS
```

#### du命令
既然存储设备已经顺利挂载，接下来就可以尝试通过挂载点目录向存储设备中写入文件了。在写入文件之前，先介绍一个用于查看文件数据占用量的du命令，其格式为“du [选项] [文件]”。简单来说，该命令就是用来查看一个或多个文件占用了多大的硬盘空间。我们还可以使用du -sh /\*命令来查看在Linux系统根目录下所有一级目录分别占用的空间大小。

### 添加交换分区
SWAP（交换）分区是一种通过在硬盘中预先划分一定的空间，然后将把内存中暂时不常用的数据临时存放到硬盘中，以便腾出物理内存空间让更活跃的程序服务来使用的技术，其设计目的是为了解决真实物理内存不足的问题。但由于交换分区毕竟是通过硬盘设备读写数据的，速度肯定要比物理内存慢，所以只有当真实的物理内存耗尽后才会调用交换分区的资源。

使用SWAP分区专用的格式化命令mkswap，对新建的主分区进行格式化操作（省略fdish分区操作）：
```
[root@linuxprobe ~]# mkswap /dev/sdb2
Setting up swapspace version 1, size = 5242876 KiB
no label, UUID=2972f9cb-17f0-4113-84c6-c64b97c40c75
```

使用swapon命令把准备好的SWAP分区设备正式挂载到系统中。我们可以使用free -m命令查看交换分区的大小变化（由2047MB增加到7167MB）：

```
[root@linuxprobe ~]# free -m
             total     used     free     shared     buffers     cached
Mem:          1483      782      701          9           0        254
-/+ buffers/cache:      526      957
Swap:         2047        0     2047
[root@linuxprobe ~]# swapon /dev/sdb2
[root@linuxprobe ~]# free -m
             total     used     free     shared     buffers    cached
Mem:          1483      785      697          9           0       254
-/+ buffers/cache:      530      953
Swap:         7167        0      7167
```

为了能够让新的交换分区设备在重启后依然生效，需要按照下面的格式将相关信息写入到配置文件中，并记得保存：

```
[root@linuxprobe ~]# vim /etc/fstab
#
# /etc/fstab
# Created by anaconda on Wed May 4 19:26:23 2017
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/rhel-root                     /            xfs       defaults   1 1
UUID=812b1f7c-8b5b-43da-8c06-b9999e0fe48b /boot        xfs       defaults   1 2
/dev/mapper                               /rhel-swap   swap swap defaults   0 0
/dev/cdrom                                /media/cdrom iso9660   defaults   0 0
/dev/sdb1                                 /newFS       xfs       defaults   0 0
/dev/sdb2                                 swap         swap      defaults   0 0
```

### 磁盘容量配额
Linux系统的设计初衷就是让许多人一起使用并执行各自的任务，从而成为多用户、多任务的操作系统。但是，硬件资源是固定且有限的，如果某些用户不断地在Linux系统上创建文件或者存放电影，硬盘空间总有一天会被占满。针对这种情况，root管理员就需要使用磁盘容量配额服务来限制某位用户或某个用户组针对特定文件夹可以使用的最大硬盘空间或最大文件个数，一旦达到这个最大值就不再允许继续使用。可以使用quota命令进行磁盘容量配额管理，从而限制用户的硬盘可用容量或所能创建的最大文件个数。quota命令还有软限制和硬限制的功能。

- 软限制：当达到软限制时会提示用户，但仍允许用户在限定的额度内继续使用。
- 硬限制：当达到硬限制时会提示用户，且强制终止用户的操作。

RHEL 7系统中已经安装了quota磁盘容量配额服务程序包，但存储设备却默认没有开启对quota的支持，此时需要手动编辑配置文件，让RHEL 7系统中的/boot目录能够支持quota磁盘配额技术。另外，对于学习过早期的Linux系统，或者具有RHEL 6系统使用经验的读者来说，这里需要特别注意。早期的Linux系统要想让硬盘设备支持quota磁盘容量配额服务，使用的是usrquota参数，而RHEL 7系统使用的则是uquota参数。在重启系统后使用mount命令查看，即可发现/boot目录已经支持quota磁盘配额技术了：

```
[root@linuxprobe ~]# vim /etc/fstab
#
# /etc/fstab
# Created by anaconda on Wed May 4 19:26:23 2017
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/rhel-root                  /            xfs       defaults        1 1
UUID=812b1f7c-8b5b-43da-8c06-b9999e0fe48b /boot         xfs       defaults,uquota  1 2
/dev/mapper                            /rhel-swap   swap swap defaults         0 0
/dev/cdrom                             /media/cdrom iso9660   defaults         0 0
/dev/sdb1                              /newFS       xfs       defaults         0 0
/dev/sdb2                              swap         swap      defaults         0 0
[root@linuxprobe ~]# reboot
[root@linuxprobe ~]# mount | grep boot
/dev/sda1 on /boot type xfs (rw,relatime,seclabel,attr2,inode6r2,inode64,usrquota)
```

接下来创建一个用于检查quota磁盘容量配额效果的用户tom，并针对/boot目录增加其他人的写权限，保证用户能够正常写入数据：

```
[root@linuxprobe ~]# useradd tom
[root@linuxprobe ~]# chmod -Rf o+w /boot
```

### xfs_quota命令
xfs_quota命令是一个专门针对XFS文件系统来管理quota磁盘容量配额服务而设计的命令，格式为“quota [参数] 配额 文件系统”。其中，-c参数用于以参数的形式设置要执行的命令；-x参数是专家模式，让运维人员能够对quota服务进行更多复杂的配置。接下来我们使用xfs_quota命令来设置用户tom对/boot目录的quota磁盘容量配额。具体的限额控制包括：硬盘使用量的软限制和硬限制分别为3MB和6MB；创建文件数量的软限制和硬限制分别为3个和6个。

```
[root@linuxprobe ~]# xfs_quota -x -c 'limit bsoft=3m bhard=6m isoft=3 ihard=6 
tom' /boot
[root@linuxprobe ~]# xfs_quota -x -c report /boot
User quota on /boot (/dev/sda1)   Blocks
User ID Used Soft Hard Warn/Grace
---------- --------------------------------------------------
root 95084    0    0    00 [--------]
tom  0        3072 6144 00 [--------]
```

当配置好上述的各种软硬限制后，尝试切换到这个普通用户，然后分别尝试创建一个体积为5MB和8MB的文件。可以发现，在创建8MB的文件时受到了系统限制：

```
[root@linuxprobe ~]# su - tom
[tom@linuxprobe ~]$ dd if=/dev/zero of=/boot/tom bs=5M count=1
1+0 records in
1+0 records out
5242880 bytes (5.2 MB) copied, 0.123966 s, 42.3 MB/s
[tom@linuxprobe ~]$ dd if=/dev/zero of=/boot/tom bs=8M count=1
dd: error writing ‘/boot/tom’: Disk quota exceeded
1+0 records in
0+0 records out
6291456 bytes (6.3 MB) copied, 0.0201593 s, 312 MB/s
```

#### edquota命令
edquota命令用于编辑用户的quota配额限制，格式为“edquota [参数] [用户] ”。在为用户设置了quota磁盘容量配额限制后，可以使用edquota命令按需修改限额的数值。其中，-u参数表示要针对哪个用户进行设置；-g参数表示要针对哪个用户组进行设置。edquota命令会调用Vi或Vim编辑器来让root管理员修改要限制的具体细节。下面把用户tom的硬盘使用量的硬限额从5MB提升到8MB：

```
[root@linuxprobe ~]# edquota -u tom
Disk quotas for user tom (uid 1001):
 Filesystem blocks  soft   hard   inodes   soft   hard
 /dev/sda   6144    3072   8192   1        3      6
[root@linuxprobe ~]# su - tom
Last login: Mon Sep 7 16:43:12 CST 2017 on pts/0
[tom@linuxprobe ~]$ dd if=/dev/zero of=/boot/tom bs=8M count=1
1+0 records in
1+0 records out
8388608 bytes (8.4 MB) copied, 0.0268044 s, 313 MB/s
[tom@linuxprobe ~]$ dd if=/dev/zero of=/boot/tom bs=10M count=1
dd: error writing ‘/boot/tom’: Disk quota exceeded
1+0 records in
0+0 records out
8388608 bytes (8.4 MB) copied, 0.167529 s, 50.1 MB/s
```

### 软硬方式链接
- 硬链接（hard link）：可以将它理解为一个“指向原始文件inode的指针”，系统不为它分配独立的inode和文件。所以，硬链接文件与原始文件其实是同一个文件，只是名字不同。我们每添加一个硬链接，该文件的inode连接数就会增加1；而且只有当该文件的inode连接数为0时，才算彻底将它删除。换言之，由于硬链接实际上是指向原文件inode的指针，因此即便原始文件被删除，依然可以通过硬链接文件来访问。需要注意的是，由于技术的局限性，我们不能跨分区对目录文件进行链接。
- 软链接（也称为符号链接[symbolic link]）：仅仅包含所链接文件的路径名，因此能链接目录文件，也可以跨越文件系统进行链接。但是，当原始文件被删除后，链接文件也将失效，从这一点上来说与Windows系统中的“快捷方式”具有一样的性质。

#### ln命令
ln命令用于创建链接文件，格式为“ln [选项] 目标”

*ln命令中可用的参数以及作用*

| 参数 | 作用 |
| :---: | :--- |
| -s | 创建“符号链接”（如果不带-s参数，则默认创建硬链接）|
| -f | 强制创建文件或目录的链接 |
| -i | 覆盖前先询问 |
| -v | 显示创建链接的过程 |

为了更好地理解软链接、硬链接的不同性质，接下来创建一个类似于Windows系统中快捷方式的软链接。这样，当原始文件被删除后，就无法读取新建的链接文件了。

```
[root@linuxprobe ~]# echo "Welcome to linuxprobe.com" > readme.txt
[root@linuxprobe ~]# ln -s readme.txt readit.txt
[root@linuxprobe ~]# cat readme.txt 
Welcome to linuxprobe.com
[root@linuxprobe ~]# cat readit.txt 
Welcome to linuxprobe.com
[root@linuxprobe ~]# ls -l readme.txt 
-rw-r--r-- 1 root root 26 Jan 11 00:08 readme.txt
[root@linuxprobe ~]# rm -f readme.txt 
[root@linuxprobe ~]# cat readit.txt 
cat: readit.txt: No such file or directory
```

接下来针对一个原始文件创建一个硬链接，即相当于针对原始文件的硬盘存储位置创建了一个指针，这样一来，新创建的这个硬链接就不再依赖于原始文件的名称等信息，也不会因为原始文件的删除而导致无法读取。同时可以看到创建硬链接后，原始文件的硬盘链接数量增加到了2。

```
[root@linuxprobe ~]# echo "Welcome to linuxprobe.com" > readme.txt
[root@linuxprobe ~]# ln readme.txt readit.txt
[root@linuxprobe ~]# cat readme.txt 
Welcome to linuxprobe.com
[root@linuxprobe ~]# cat readit.txt 
Welcome to linuxprobe.com
[root@linuxprobe ~]# ls -l readme.txt 
-rw-r--r-- 2 root root 26 Jan 11 00:13 readme.txt
[root@linuxprobe ~]# rm -f readme.txt 
[root@linuxprobe ~]# cat readit.txt 
Welcome to linuxprobe.com
```
