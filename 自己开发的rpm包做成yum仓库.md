## 如果开发rpm包也想配置yum仓库方案
> 我们知道要想配置yum仓库需要写baseurl rmp包路径
- 而baseurl的路径要写repodate目录的父目录，单独开发rpm包是没有这个文件的所有我们需要用命令生产一下。
```bash
[root@centos7 app]# ll
total 40
-r--r--r--. 1 root root 36884 Nov 30 21:02 tree-1.5.3-3.el6.x86_64.rpm
现在我们在app下有一个单独的rpm包，我们需要生产repodata的目录就好了。
```
```bash
[root@centos7 app]# createrepo /var/ftp/pub/app/
Spawning worker 0 with 1 pkgs
Spawning worker 1 with 0 pkgs
Workers Finished
Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Sqlite DBs complete
[root@centos7 app]# ls
repodata  tree-1.5.3-3.el6.x86_64.rpm
[root@centos7 app]# 
这样就完成了，如果后续还有rpm包放进去，那就在执行一次。
```
createrepo  /rpm包的路径
