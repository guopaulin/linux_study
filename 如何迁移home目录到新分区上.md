# 如何将现有的/home目迁移新的单独分区上，或者其它目录迁移到别的分区
- 在迁移前请确认是否有其它用户正在登陆，或者使用这个目录，务必在完全没有登陆，或者使用这个目录在迁移，否则会造成正在对这个目录进行编辑的文件丢失数据。
> 创建一个新的分区
```bash
[root@centos6 ~]# fdisk /dev/sda
由于这是一个在使用的硬盘所有创建完新的分区并不会自动同步
[root@centos6 ~]# partx -a /dev/sda 
此条命令是同步分区到内存上。
[root@centos6 ~]# mkfs.ext4 /dev/sda6
创建文件系统
[root@centos6 ~]# mkdir /mnt/home
创建一个挂载目录
[root@centos6 ~]# mount /dev/sda6 /mnt/home/
将这个/mnt/home 挂载到这个分区上
cp -av /home/* /mnt/home/ 
用CP 命令-a 备份到刚刚创建的目录上
du -sh /home/
du -sh /mnt/home/
这两命令是查看复制的文件大小是不是少文件

```
> 现在我们把/home 文件打包放到其它服务器上做备份文件。
```bash
[root@centos6 app]# scp /app/home.tar.gz  root@192.168.27.129:/root
```
- 上传到别的服务器上，然后删除根下的home文件里的内容
```bash
[root@centos6 home]# rm -rf *  删除现有家目录的所有文件
[root@centos6 /]# vim /etc/fstab  新分区添加到挂载表
UUID=155cc08a-a66c-4389-8986-92129b20b26c /home ext4 defaults 0 0  
[root@centos6 /]# mount -a 挂载

```
- 挂载完后要把/mnt/home的解除卸载
- 这样就完成了。
> 测试
- 我们登录一个普通用户看一下
