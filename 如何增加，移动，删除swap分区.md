#  如何增加swap分区，和删除，移动swap
> 由于我们在机器上添加了一条新内存，但是我按照swap容量是内存的1.5到2倍，但是现在不够这个条件所以需要对swap进行扩容
1. 第一步创建一个适合的量的新的分区，或者文件
    + 需要注意一点swap分区和普通分区不同，在创建的时候要知道一下swap属性
```bash
    [root@centos6 home]# fdisk /dev/sdc

WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
         switch off the mode (command 'c') and change display units to
         sectors (command 'u').

Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p
Partition number (1-4): 3
First cylinder (265-6527, default 265): 
Using default value 265
Last cylinder, +cylinders or +size{K,M,G} (265-6527, default 6527): +2G

Command (m for help): h
h: unknown command
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id  更改系统分区ID
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)

Command (m for help): t 这个选项是更改系统分区ID
Partition number (1-4): 3
Hex code (type L to list codes):     82 sawp分区ID是82，也可用L查看所有分区ID号
Changed system type of partition 3 to 82 (Linux swap / Solaris)

Command (m for help): p

Disk /dev/sdc: 53.7 GB, 53687091200 bytes
255 heads, 63 sectors/track, 6527 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x63783858

   Device Boot      Start         End      Blocks   Id  System
/dev/sdc1               1         132     1060258+  83  Linux
/dev/sdc2             133         264     1060290   83  Linux
/dev/sdc3             265         526     2104515   82  Linux swap / Solaris

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
[root@centos6 home]# 
[root@centos6 home]# 
[root@centos6 home]# echo $? 这个可以判断是否有错误
0
[root@centos6 home]# 
```
- 这样我们的分区就创建好了，如果是老硬盘需要手动同步一下，6版本的系统用命令partx -a /dev/sdc,7版本的系统用partprobe 命令同步。创建完一定要检查是否同步成功。
2. 用mkswap 对刚刚创建的分区进行格式化和创建swap文件系统
```bash
[root@centos6 home]# mkswap /dev/sdc3
Setting up swapspace version 1, size = 2104508 KiB
no label, UUID=d0d47511-9cbb-4769-9430-6de164c74c62
[root@centos6 home]# blkid
/dev/sdb1: UUID="5fccd8fb-8c38-43ec-8d90-ff39938a8a9c" TYPE="ext2" 
/dev/sda1: UUID="bbfd63dd-da18-4ac9-affd-4724c571db21" TYPE="ext4" 
/dev/sda2: UUID="03352018-7cef-4ee3-9a05-b0833b67da19" TYPE="ext4" 
/dev/sda3: UUID="0bd51fa6-f577-427a-b0b2-1cb8a631ac16" TYPE="ext4" 
/dev/sda5: UUID="a8c19a79-3c21-4aa4-a648-27a069f25bde" TYPE="swap" 
/dev/sdb2: UUID="aabdedfc-beb9-4c9c-83ab-4a18cd6b24d4" TYPE="ext4" 
/dev/sdb3: LABEL="guo" UUID="581409f1-1db6-4fb5-b999-6ce074d1cec2" TYPE="ext4" 
/ddc: UUID="7a786bc8-c075-4c9a-acad-f042415a6b04" TYPE="ext4" 
/dev/sdc3: UUID="d0d47511-9cbb-4769-9430-6de164c74c62" TYPE="swap" 
```
3. 在/etc/fstab 配置文件中添加对应的条目
```bash
UUID=d0d47511-9cbb-4769-9430-6de164c74c62       swap    swap    defaults        0       0
```
4. 激活swap分区
```bash
查看一下现在的swap信息
[root@centos6 home]# free
             total       used       free     shared    buffers     cached
Mem:       2052688     424516    1628172       1284      24900     179200
-/+ buffers/cache:     220416    1832272
Swap:      2097148          0    2097148
从上面看到内存2G ，swap分区也2G.
现在激活刚刚创建的swap分区
[root@centos6 home]# swapon -a
[root@centos6 home]# free -h
             total       used       free     shared    buffers     cached
Mem:          2.0G       416M       1.6G       1.3M        24M       175M
-/+ buffers/cache:       216M       1.7G
Swap:         4.0G         0B       4.0G
```
5. 这样就扩容成功，现在查看一下swap分区详细信息
```bash
[root@centos6 home]# swapon -s
Filename                                Type            Size    Used    Priority
/dev/sda5                               partition       2097148 0       -1
/dev/sdc3                               partition       2104508 0       -2
```
- priority 这是优先级，越优先级越大就优先使用,想要指定优先级一定要先禁用swap 分区
```bash
[root@centos6 home]# swapoff /dev/sdc3
[root@centos6 home]# swapon -p 10 /dev/sdc3
[root@centos6 home]# swapon -a
[root@centos6 home]# swapon  -s
Filename                                Type            Size    Used    Priority
/dev/sda5                               partition       2097148 0       -1
/dev/sdc3                               partition       2104508 0       10
```
我们要在/etc/fstab 中defaults位置上添加pri=value
```bash
UUID=d0d47511-9cbb-4769-9430-6de164c74c62       swap    swap    pri=10  0       0
```
- 如何生效，先把swap分区禁用，然后在启用就生效了
- 虽然swapon -p 命令是可以设定优先值的，但是这是临时的机器重启会消失。
```bash
[root@centos6 ~]# swapoff /dev/sdc3
[root@centos6 ~]# swapon -a
[root@centos6 ~]# swapon -s        
Filename                                Type            Size    Used    Priority
/dev/sda5                               partition       2097148 0       -1
/dev/sdc3                               partition       2104508 0       10
```

> 如何删除swap分区
1.首先确认没有swap分区没有被使用，如何被使用我们删除分区会导致内存里面的数据会丢失切记
```bash
[root@centos6 home]# free -h
             total       used       free     shared    buffers     cached
Mem:          2.0G       416M       1.6G       1.3M        24M       175M
-/+ buffers/cache:       216M       1.7G
Swap:         4.0G         0B       4.0G
```
2. 禁用要删除的swap分区
```bash
[root@centos6 ~]# swapoff /dev/sdc3
[root@centos6 ~]# free
             total       used       free     shared    buffers     cached
Mem:       2038348     360052    1678296       1284      23780     128384
-/+ buffers/cache:     207888    1830460
Swap:      2097148          0    2097148
```
3. 删除/etc/fstab 对应行
4. 删除分区就好了
### 如果让文件变成swap分区，然后把它移动到别的磁盘
1. **用dd命令创建一个文件**
```bash
[root@centos6 app]# dd if=/dev/zero of=swapfile bs=1M count=2048
2048+0 records in
2048+0 records out
2147483648 bytes (2.1 GB) copied, 20.6218 s, 104 MB/s
[root@centos6 app]# ls
4  456  lost+found  mnt  oo  swapfile  we 
```
- 这一步相对于创建分区了
2. **用mkswap 命令格式变成swap分区**
```bash
[root@centos6 app]# mkswap swapfile 
mkswap: swapfile: warning: don't erase bootbits sectors
        on whole disk. Use -f to force.
Setting up swapspace version 1, size = 2097148 KiB
no label, UUID=695dea75-3394-4932-8ddb-61806004aeea
[root@centos6 app]# blkid swapfile 
swapfile: UUID="695dea75-3394-4932-8ddb-61806004aeea" TYPE="swap"
```
3. **写入/etc/fstab**
```bash
/app/swapfile      swap    swap    defaults        0       0
```
4. **激活swap**
```bash
[root@centos6 app]# swapon -a
[root@centos6 app]# free
             total       used       free     shared    buffers     cached
Mem:       2038348    1968472      69876       1288       6756    1703632
-/+ buffers/cache:     258084    1780264
Swap:      4194296          0    4194296
[root@centos6 app]# swapon -s
Filename                                Type            Size    Used    Priority
/dev/sda5                               partition       2097148 0       -1
/app/swapfile                           file            2097148 0       -2
```
- 由于我们用的文件当swap ，然而文件的性能没有真正磁盘分区性能好所以我们不要设优先级。
4. **迁移文件swap分区到别的磁盘中**
- 确认现在没有在使用swap，然后禁用swap分区
```bash
[root@centos6 app]# free
             total       used       free     shared    buffers     cached
Mem:       2038348    1968596      69752       1288       6804    1703672
-/+ buffers/cache:     258120    1780228
Swap:      4194296          0    4194296
[root@centos6 app]# swapoff swapfile 
[root@centos6 app]# free
             total       used       free     shared    buffers     cached
Mem:       2038348    1967372      70976       1288       6812    1703672
-/+ buffers/cache:     256888    1781460
Swap:      2097148          0    2097148
```
- 把swapfile文件用cp命令复制别的磁盘
```bash
[root@centos6 app]# cp -p swapfile /
[root@centos6 app]# cd /
[root@centos6 /]# ls
app  boot  dev  home  lib64       media  mnt  opt   root  selinux  swapfile  tmp  var
bin  ddc   etc  lib   lost+found  misc   net  proc  sbin  srv      sys       usr
```
- 改配置文件/etc/fstab
```bash
/swapfile       swap    swap    defaults        0       0
```
- 然后在把原来文件删除就成功了。
