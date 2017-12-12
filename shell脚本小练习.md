## <font color=red>练习 </font>
> 1、编写脚本/root/bin/systeminfo.sh,显示当前主机系统信息，包括主机名，IPv4地址，操作系统版本，内核版本，CPU型号，内存大小，硬盘大小
```bash
[root@centos7 ~]# vim Hardware_1.sh
#!/bin/bash
#filename:   hardware information
#date:       2017-11-22
#name:       paulin
#version:    0.1 

echo at present system `hostname`
echo at present cpu `lscpu | grep 'Model name'| tr -s ' '  |cut -d: -f2`
echo at present ipv4 `[root@centos7 ~]# ifconfig ens33 |grep netmask |tr -s ' ' ':' |cut -d: -f3`
echo at present kernel `uname -r`
cat /etc/centos-release
echo at present free `cat /proc/meminfo | head -n1 |cut -d: -f2 |tr -d ' '`
echo HHD ` df -h | grep dev/sd`
```
>2、编写脚本/app/backup.sh，可实现每日将/root/目录备份到/root/etcYYYY-mm-dd中  
```bash
[root@centos7 app]# cat backup.sh 
#!/bin/bash
#date : 2017-11-23
#function :backup/etc

cp -a /root  /app/etc`date +%F`
[root@centos7 app]# 
```
> 3、编写脚本/root/bin/disk.sh,显示当前硬盘分区中空间利用率最大的值
```bash
[root@centos7 app]# cat disk.sh 
#!/bin/bash
#Features : disk utilization
#date :2071-11-23
#vs :1 

echo disk utilization `(df -h |grep /dev/sd |tr -s ' '  |sort -t' ' -k5 |cut -d' ' -f1,5 |head -1)` diskut 
```
> 4、编写脚本/root/bin/links.sh,显示正连接本主机的每个远程主机的IPv4地址和连接数，并按连接数从大到小排序
```bash

```
### 练习二
>　1、编写脚本/root/bin/sumid.sh，计算/etc/passwd文件中的第10个用户和第20用户的ID之和　
```bash
[root@centos7 app]# vim sumid.sh 
#!/bin/bash
#Function :
#date  :2017--11--23
#at

uid10=`head -10 /etc/passwd |tail -1 |cut -d: -f3`
uid20=`head -20 /etc/passwd |tail -1 |cut -d: -f3`
echo The sum of them $[uid10+uid20];
```
> 2、编写脚本/root/bin/sumspace.sh，传递两个文件路径作为参数给脚本，计算这两个文件中所有空白行之和
```bash
#!/bin/bash
kong1=`grep '^$' $1 | wc -l`
kong2=`grep '^$' $2 | wc -l`
echo the sum of them $[kong1+kong2]
```
> 3、编写脚本/root/bin/sumfile.sh,统计/etc, /var, /usr目录中共有多少个一级子目录和文件
```bash
#!/bin/bash
etc_1=`ls -A -1 /etc |wc -l`
var_1=`ls -A -1 /var |wc -l`
usr_1=`ls -A -1 /usr |wc -l`
echo the sum of them $[etc_1+var_1+usr_1]
```
> 1、编写脚本/root/bin/argsnum.sh，接受一个文件路径作为参数；如果参数个数小于1，则提示用户“至少应该给一个参数”，并立即退出；如果参数个数不小于1，则显示第一个参数所指向的文件中的空白行数
```bash

