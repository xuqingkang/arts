This is 6th ARTS, The Algorithm is still difficult for me again and again.

# 1.Algorithm
### Question: 

### My solution in python3 is:


# 2.Review


# 3.Tips
同事在实施的时候遇到一个大小为60T的磁盘，产品虽说耗存储空间，但之前客户一般都给分布式存储，或者是最大不超过2T的存储盘。看到60T我们都比较感叹说这客户真豪！但因第一次遇到这种还需要自己格式化的盘，所以有点懵逼了，对于这样大存储空间的盘怎么格式化，选择什么类型？都成为问题，后来查询资料。

涉及到两方面，一是分区类型（ＭＢＲ和ＧＰＴ），一种是文件类型（ext4 xfs等）

MBR: MBR只支持最多4个主分区，如果需要更多分区的话，需要在磁盘上新建扩展分区(extended partition)， and only works with disks up to 2 TB in size.

GPT: GPT则支持更多的分区数量，其支持的文件系统大小取决于操作系统和文件类型，如下为以Redhat5~7为例主要文件系统类型对应的size大小。

Filesystem | RHEL 5 | RHEL 6 |  RHEL 7 |
---|---|---|---|
EXT2/3 | 8TiB (5.0), 16TiB (5.1+) | 16TiB |　16TiB
EXT4 | 16TiB [1EiB] (5.6+) | 16TiB |　50TiB [1EiB]
XFS | 100TiB [16EiB] | 300TiB [16EiB] |　500TiB [16EiB]

基于以上分析，60T的硬盘在分区是需要指定为"GPT"类型，并选择"XFS"文件系统类型。

如下为样例：这里为了演示目的，选择了一个1G大小的新增加磁盘进行操作

第一步：parted对/dev/sdb磁盘进行格式化

```
[root@rhel7 ~]# parted /dev/sdb
GNU Parted 3.1
使用 /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel                                                          
新的磁盘标签类型？ gpt                    #这里在mklabel时选择gpt                                
(parted) mkpart                         #开始分区                              
分区名称？  []? gpt1
文件系统类型？  [ext2]? xfs                                               
起始点？ 0                                                                
结束点？ 1G                                                               
警告: The resulting partition is not properly aligned for best performance.
忽略/Ignore/放弃/Cancel? I                                                
(parted) print                          #查看分区信息                                  
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 1074MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system  Name  标志
 1      17.4kB  1074MB  1074MB               gpt1
```

第二步：使用操作系统的mkfs格式化文件为xfs类型

```
[root@rhel7 ~]# mkfs -t xfs /dev/sdb1
meta-data=/dev/sdb1              isize=256    agcount=4, agsize=65534 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0        finobt=0
data     =                       bsize=4096   blocks=262135, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal log           bsize=4096   blocks=853, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

```
查看格式化的结果，可以看到新增加的sdb

```
[root@rhel7 ~]# fdisk -l

磁盘 /dev/sda：64.4 GB, 64424509440 字节，125829120 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000dfcd6

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048   125829119    62401536   8e  Linux LVM
WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

磁盘 /dev/sdb：1073 MB, 1073741824 字节，2097152 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：gpt
```

新建一个文件目录如/gptdisk，然后把其与新建的xfs类型的磁盘分区/dev/sdb1进行mount，便可以进行文件操作。

```
[root@rhel7 ~]# mkdir /gptdisk
[root@rhel7 ~]# mount /dev/sdb1 /gptdisk/
[root@rhel7 ~]# cd /
[root@rhel7 /]# ls
app  bin  boot  dev  etc  gptdisk  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@rhel7 /]# cd gptdisk/
[root@rhel7 gptdisk]# ls
[root@rhel7 gptdisk]# touch aaa.txt
```



# 4.Share

