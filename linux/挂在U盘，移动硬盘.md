# 移动硬盘U盘挂在

## 命令 `mount`

**语法:**
`mount   -t  type   device   dir`

其中type表示要挂载设备文件系统的类型，device表示要挂载的设备，dir表示设备在系统上的挂载点。

linux常用的文件系统类型有磁盘文件系统、网络文件系统、专有/虚拟文件系统。
需要注意的是：__linux系统中只能在root权限用户下挂载设备__

**磁盘文件系统:** 包括硬盘、U盘、磁盘阵列、CDROM、DVD等。常见文件系统有autofs、coda、Ext2、Ext3、Ext4、VFAT、ISO9660（光盘或者光盘镜像）、UFS（Unix File System，Unix文件系统）、FAT（File Allocation Table，文件分配表）、FAT16、FAT32、NTFS（New TechnologyFile System）等。

**网络文件系统：** 可以远程访问的文件系统，这种文件系统在服务器端仍是本地的磁盘文件系统，客户机通过网络远程访问数据。常见文件系统格式有：NFS（Network File System，网络文件系统）、Samba（SMB/CIFS）、AFP（Apple FillingProtocol，Apple文件归档协议）和WebDAV等。

**专有/虚拟文件系统:** 不驻留在磁盘上的文件系统。常见格式有：TMPFS（临时文件系统）、PROCFS（Process FileSystem，进程文件系统）和LOOPBACKFS（Loopback File System，回送文件系统）。

## 1、U盘的挂载

USB接口的U盘对于linux系统而言是当作SCSI设备对待的。首先切换到root权限用户下。插入U盘前先用“fdisk -l”命令查看系统的磁盘列表。如下图所示，系统共有一个硬盘/dev/sda，大小64.4GB，和创建虚拟系统时分配的磁盘大小一致。Linux系统自动磁盘分为三个分区/dev/sda1、/dev/sda2、/dev/sda5。

![avatar](../imgs/20170312231832550.png)

插入U盘后，再次使用“fdisk -l”命令，发现磁盘列表中多了一个硬盘/dev/sdb，容量大小2G，和插入的U盘容量大小一致。同时系统多了一个/dev/sdb1的磁盘分区。这个磁盘分区就是要挂载的U盘。

![alt](../imgs/20170312231849730.png)

 新建一个目录作为U盘的挂接点。比如说要把U盘挂载到 /mnt/usb，那么需要采用下列命令新建 /mnt/usb。

       mkdir /mnt/usb

       然后就可以采用mount命令把U盘挂载在/mnt/usb。

       mount /dev/sdb1 /mnt/usb

       输入命令 cd /mnt/usb进入目录/mnt/usb，然后输入 ls命令就可以查看U盘里的内容了。

U盘使用完毕后，为了避免损坏U盘或者丢失数据，可以采用 umount命令解挂U盘，类似于windows下的弹出U盘操作。

       umount /mnt/usb

## 2、移动硬盘的挂载

移动硬盘的挂载其实和U盘的挂载很类似。在linux中移动硬盘同样当作SCSI设备对待。首先切换到root权限用户下，然后用 fdisk-l 命令查看系统硬盘及分区情况。结果如下：

![avater](../imgs/20170312233042064.png)

插入移动硬盘后，再次输入 fdisk-l命令，可以看出系统中多了一个/dev/sdb的磁盘，大小为1T，同时这个磁盘分成/dev/sdb1、/dev/sdb2、/dev/sdb3、/dev/sdb4共4个分区。这和插入的移动硬盘的情况完全一致。

![avater](https://img-blog.csdn.net/20170312232229572?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZWxlY3Ryb2NyYXp5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

        我们可以把4个分区看成4个U盘。新建/mnt/hddisk1、/mnt/hddisk2、/mnt/hddisk3、/mnt/hddisk4共4个挂载点。

        mkdir /mnt/hddisk1

        mkdir /mnt/hddisk2

        mkdir /mnt/hddisk3

        mkdir /mnt/hddisk4

        也可以采用简化命令完成。

        mkdir   /mnt/hddisk1  /mnt/hddisk2   /mnt/hddisk3   /mnt/hddisk4

        然后分别采用挂载命令挂载4个分区。

        mount  /dev/sdb1   /mnt/hddisk1

        mount  /dev/sdb2   /mnt/hddisk2

        mount  /dev/sdb3   /mnt/hddisk3

        mount  /dev/sdb4   /mnt/hddisk4

       挂载完成之后，就可以进入到/mnt/hddisk1、/mnt/hddisk2、/mnt/hddisk3、/mnt/hddisk4中查看硬盘各分区里的内容了。

       同样如果硬盘不使用了，需要采用umount命令进行解挂。

**备注：**

       最近在虚拟机里使用ubuntu系统时，已经能够自动挂载U盘、硬盘和光盘了。自动挂载点在 /media 目录下，可以直接对存储外设里的文件进行访问和操作。如果插入存储外设时，没有自动挂载，需要在虚拟机里修改设置。以U盘的自动挂载为例，虚拟机版本：VMware Workstation 12 Player（12.5.2 build-4638234），ubuntu系统版本：ubuntu 12.04 LTS。如下图所示，依次点击虚拟机的“Playe(P)”——“可移动设备（R）”——“Silicon Motion USB Flash Disk”——“连接（与主机断开连接）(C)”即可。这里的主机指的电脑主机，在这里指windows主机。U盘在同一时刻只能与主机或者虚拟机二者之一连接， 与虚拟机连接就意味着要和主机断开连接。如果不希望ubuntu自动挂载存储外设，和上述的操作类似，进行断开连接设置即可。如果在ubuntu系统中手动挂载存储外设失败，要考虑系统是不是已经自动挂载。如果自动挂载了就不能也无需手动挂载了，如果一定要手动挂载，需要先解除自动挂载，再重新插拔外设之后重试。

![avater](https://img-blog.csdn.net/20170315090456820?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZWxlY3Ryb2NyYXp5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)