# **yum配置使用**
####  网络yum库服务搭建
- 环境：把centos 7当做服务器端，centos6当做客户端
> 首先用镜像搭建一个本地yum
+ 配置/etc/yum.repo.d/bash.repo 配置文件
```bash
[root@centos7 yum.repos.d]# ls
bak  base.repo
``` 
- 如果没有此文件可以创建一个后缀只要是.repo的就行
- 我们现在编辑base.repo文件
```bash
[base]  库id
name=base 库名字
baseurl=file:///run/media/root/CentOS\ 7\ x86_64/  rpm包路径，必须是repodata.文件的父目录路径
gpgcheck=0 包检查
enabled = 1 是否启用
```
- 现在我们安装vsftpd服务
```bash
[root@centos7 yum.repos.d]# yum install vsftpd
```
- ftp共享的文件路径是/var/ftp/pub ,我们进去创建centos/6/os/x86_64/
```bash
[root@centos7 pub]# mkdir -p centos/6/os/x86_64/
[root@centos7 pub]# ll
total 0
drwxr-xr-x. 3 root root 15 Nov 30 19:27 centos
```
- 上面创建的目录是共享目录，现在我们把所需要的rpm包复制到这个目录里由于我是用虚拟机做的所以就把6的光盘挂载到这个目录里
```bash
[root@centos7 pub]# mount /dev/sr0 centos/6/os/x86_64/
mount: /dev/sr0 is write-protected, mounting read-only
[root@centos7 pub]# 
```
- 现在我们启动ftp服务
```bash
systemctl start vsftpd 开启服务
systemctl enable vsftpd 开机启动vsftpd服务
centos7关防火墙
systemctl stop  firewalld 关闭防火墙
systemctl disable firwalld 设置开启启动中
centos6 关防火墙
chkconfig iptables off
service iptables top
检查是不是清空了：iptables -nVL 
---------------------------------
[root@centos7 ~]# systemctl stop firewalld
[root@centos7 ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
----------------------------------------
我们在把selinux 文件修改一下
路径 /etc/selinux/config
[root@centos7 ~]# sed -i.bak 's@SELINUX=enforcing@SELINUX=permissive@'  /etc/selinux/config    
[root@centos7 ~]# setenforce 0  
这样我们可以在浏览器上访问我们centos7的地址用ftp方式ftp://172.168.1.1 就可以看见光盘里的内容。
```
> Centos 6客户端配置
- 我们只需要配置yum仓库
```bash
[root@centos6 ~]# vim /etc/yum.repos.d/baes.repo
[base]
name=guo
baseurl=ftp://192.168.27.129/pub/centos/$releasever/os/$basearch/
gpgcheck=0
```
- 上面的变量代表含义
      + $releasever :当前OS版本的发行主版本号
      + $basearch : 基础平架构x86_64
- 现在我们用 yum repolist 查一下能看到仓库名不
```bash
[root@centos6 ~]# yum repolist
Loaded plugins: fastestmirror, refresh-packagekit, security
Determining fastest mirrors
base                                                      | 4.0 kB     00:00     
base/primary_db                                           | 4.7 MB     00:00     
repo id                                repo name                           status
base                                   guo                                 6,706
repolist: 6,706
```
- 现在我们用yum装一个软件
```bash
[root@centos6 ~]# yum repolist
这样就成功了。
```
### 关于yum排错失误
- 首先检查电脑之间是否能够互联
- 然后检查yum 配置是否正确 （可以用 yum clean all清除一下缓存）
- 看看防火墙和SELinux等配置是否关闭。


