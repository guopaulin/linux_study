## rpm
+ 包的分类和拆包
   1. Application-VERSION-ARCH.rpm: 主包
   2. Application-devel-VERSION-ARCH.rpm 开发子包
   3. Application-utils-VERSION-ARHC.rpm 其它工具子包
   4. Application-libs-VERSION-ARHC.rpm 库子包 
+ 包之间 ：可能存在依赖关系，甚至出现循环依赖
+ 解决依赖包管理工具
   1. yum :rpm包管理器前端管理工具
   2. apt-get :deb包管理器前端管理工具
   3. zypper :suse上的rpm前段管理工具
   4. dnf :Fedora 18+ rpm包管理器前端管理工具

- 所有的二进制程序的运行都是依赖系统上的库文件如何查看一个二进制程序所调用的库程序
   - <font color=red>ldd /path/file </font>
   ```bash 
    [root@centos7 app]# which ls  查看命令的路径
        alias ls='ls --color=auto'  
        /usr/bin/ls  
    [root@centos7 app]# ldd /usr/bin/ls     查看二进制程序所调用的库  
        linux-vdso.so.1 =>  (0x00007ffd46fba000)    
        libselinux.so.1 => /lib64/libselinux.so.(0x00007f60116a2000)    
        libcap.so.2 => /lib64/libcap.so.2 (0x00007f601149d000)    
        libacl.so.1 => /lib64/libacl.so.1 (0x00007f6011293000)  
        libc.so.6 => /lib64/libc.so.6 (0x00007f6010ed0000)  
        libpcre.so.1 => /lib64/libpcre.so.1 (0x00007f6010c6e000)  
        libdl.so.2 => /lib64/libdl.so.2 (0x00007f6010a69000)  
        /lib64/ld-linux-x86-64.so.2 (0x00005616bd174000)  
        libattr.so.1 => /lib64/libattr.so.1 (0x00007f6010864000)  
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f6010648000)  
    ```
- 管理及查看本机装载的库文件
    + ldconfig  加载库文件
    + ldconfig -p 和/sbin/ldconfig -p 一样
    + /sbin/ldconfig -p :显示本机已经缓存的所有可用库文件名及文件路径映射关系
- 配置文件
    + /etc/li.so.conf,    /etc/ld.so.conf.d/*.conf  
- 缓存文件   
    + /etc/ld.so.cache  
- 程序调用的系统库文件很重要，如果库文件损坏可导致很多调用此库的文件无法正常运行。修复系统的库文件就是用光盘进入救援模式修复相应的库文件。
   
#### 包管理理器   
- 功能 ：将编译好的应用程序的各组成文件打包一个或几个程序包文件，从而方便快捷地实现程序包的安装、卸载、查询、升级和效验等管理工具操作。  
- 包文件组（每个包独有）  
     1. RPM包内的文件主程序  
     2. RPM的元数据，如名称，版本，依赖性，描述  
     3. 安装或卸载时运行的脚本  
- 在系统里/var/lib/rpm 有一个公共数据库，里面存储着所安装的rpm包的记录。
     1. 程序包名称及其版本  
     2. 依赖关系  
     3. 功能说明  
     4. 包安装后生成的各种文件路径及其效验码信息。
     5. rpm  {--initdb|--rebuilddb} 
        + initdb  初始化，如果事先不存在数据库，则新建之否则，不执行任何操作
        + rebuilddb : 重建已安装的包头的数据库索引目录

- <font color=red>如果删除库则会导致rpm安装时出现不知道安装那个包 也不知道电脑安装了什么包，就会出现混乱</font>  
- 管理程序包的方式
     1. 使用包管理器：rpm  
     2. 使用前段工具：yum,dnf  

### **如何获取系统镜像，和rpm包**  
> centos 镜像  
https://www.centos.org/download/    
http://mirrors.aliyun.com    
http://mirrors.sohu.com    
http://mirrors.163.com    
- 和项目官方站点下载   
- 第三方组织：  
     + Fedora-EPEL
- 搜索引擎：   
http://pkgs.org  
http://rpmfind.net  
http://rpm.pbone.net  
https://sourceforge.net/  

- 自己制作
- <font color=red>如果是第三方建议要检查其合法性，来源性合法性，程序包的完整性</font>  

### **Centos 使用rpm命令管理程序包**
- 安装、卸载、升级、查询、效验、数据库维护
> 安装
- rpm [option ] rpm包路径
     + -i (-- install)  安装
     + -v  显示详细信息
     + -vv 显示更详细信息
     + -h  以#显示程序安装进度
     + 常用 rpm -ivh /包路径
     + --test : 测试安装，但不是真正的安装。
     + --nodeps :忽略包的依赖
     + --replacepkgs :重新安装程序 是覆盖安装
     + --replacefiles ：两个程序有些相同的包，导致另一程序不能安装，用这个命令可以让这个程序相同的文件覆盖之前安装程序安装的文件
     + --nosignature : 不检查来源合法性
     + --nodigest : 不检查包的完整性
     + --noscripts :不执行程序包内置的脚本
         + %pre: 安装前脚本； --nopre
         + %post: 安装后脚本； --nopost
         + %preun: 卸载前脚本； --nopreun 
         + %postun: 卸载后脚本； --nopostun 
    + --force :强制安装
    + rpm2cpio tree.rpm |cpio -tv 查看rmp包都有什么文件
    + rpm2cpio  包路径.rmp | cpio -id   解压rpm包到当前目录

#### **rpm 包升级**
> rpm {-U|--upgrade} [install-optins] PACKAGE_FILE..  
> rpm {-F|--freshen} [install-optins] PACKAGE_FILE..  
> upgrade :安装有旧版本程序包，则升级，如果没有旧的安装包则‘安装’
> freshen ：安装有旧版本程序包，则升级，如果没有则不执行升级操作
> rpm -Uvh PACKAGE_FILE
> rpm -fvh PACKAGE_FILE
#### **降级**
> --oldpackage:降级
#### **升级注意项**
1. 不要对内核做升级操作，linux支持多内核版本并存，因此对直接安装新版本内核
2. 如果原程序包的配置文件安装后曾被修改，卸载、升级时，新版本的提供的同一个配置文件并不会直接覆盖老版本配置文件，而是把新版本的文件重新命名.rpmnew 后缀保留

### **rpm包查询**
> rpm {-q|--query}[select-options] [query-option]   
[select-options]    
-a :所有包  
-f :查看指定文件由那个程序包安装生成  
-p :rpmfile :针对尚未安装的程序包文件做查询操作  
--whatprovides  CAPABILITY :查询指定的能力功能由那个包所提供  
--whatrequires  CAPABILITY :查询指定的能力被那个包所依赖  
>  rpm2cpio 包文件|cpio –itv 预览包内文件  
>  rpm2cpio 包文件|cpio –id “*.conf” 释放包内文件  

> [query-options]
-- changelog :查询rpm包的更新日志文件
-c :查询程序的配置文件
-d :查询程序的文档
-i :查看程序的信息内容
-l :查看指定的程序包安装后生成的所有文件
--scripts :查看程序自带的脚本
--provides :查看指定程序包所提供的功能和能力
-R :查询指定程序的所依赖的库文件

#### **包卸载**
> rpm {-e|--erase} [--allmatches][--nodeps][--noscripts][--notriggers][--test] PACKAGE_NAME

#### **包校验** 
> rpm {-V|--verify} [select-options][verify-options]


> 导入所需要的公钥
- rpm -K|checksig rpmfile 检查包的完整性和签名
- rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CENTOS-7
- 查看是否导入公钥：rpm -qa “gpg-pubkey*”
## **yum**
- Centos :yum ,dnf
- yum:yellowdog update modifier ,rpm的前端程序，可解决软件包的相关依赖性，可在多个库之间定位软件包，up2date的替代工具
- yum repository :yum repo,存储了众多rpm包，以及包的相关的元数据文件（放置于特定目录repodate下）
- 文件服务器
   + 支持http://
   + https://
   + ftp://
   + file://

#### **yum客户端配置文件**
> /etc/yum.conf : 为所有仓库提供公共配置  
> /etc/yum.repos.d/*.repo : 为仓库的指向提供配置，编辑仓库文件的位置，文件要以.repo后缀  
```bash
[root@centos7 yum.repos.d]# vim base.repo 
[base]
name=base
baseurl=file:///run/media/root/CentOS\ 7\ x86_64/
gpgcheck=0
```
- 上面就是一个简单的仓库配置
- 现在我们说一下这些都代表什么意思
```bash  
[base] 库的ID  
name=base 库名字  
baseurl=file{http;https;ftp} 这里是写rpm包的位置，必须repodate所在为主的父目录  
enabled={0|1}是否启用库 （0代表不启用，1带表启用）  
gpgcheck={0|1} 是否启用包检查（如何不写默认是启用的，这样要把下面的导入公钥路径写上或者手动添加，不然会用yum安装不成功）  
gpgkey=url {如果启用这要把这个公钥自动导入，写上公钥所在的路径}  
```  
#### **在配置yum仓库时可以配置的变量**
1. $releasever :当前OS的发行版本的主版本号
2. $arch :平台，i386,i486,i586,x86_64
3. $basearch :基础平台：i386,x86_64
- 实列：
1. http://server/centos/$releasever/$basearch
2. http://server/centos/7/x86_64

#### **互联网上的源**
- 阿里云
   + http://mirrors.aliyun.com
## **yum命令**
- yum-config-manager --disable "仓库名" 禁用仓库
- yun-config-manager --enable  "仓库名" 启用仓库
- 我们也可以用命令创建yum配置文件
  + yum-config-manage --add-repo=baseurl的路径
```bash
  [root@centos7 ~]# yum-config-manager --add-repo=ftp://192.168.27.13
Loaded plugins: fastestmirror, langpacks
adding repo from: ftp://192.168.27.13
[192.168.27.13]
name=added from: ftp://192.168.27.13
baseurl=ftp://192.168.27.13
enabled=1
```

- 这里少了一个gpgcheck 包检查，如果不写是默认检查一定要注意
- 这个命令一定要进去/etc/yum.repo.d/下面运行，这个是创建当前目录下。  
- 所以我们还是最好用vim 自己创建  
> yum命令的用法：    
> yum [option] [command] [包名]       
> 显示仓库列表： yum repolist [all|enabled|disabled]       
> 显示程序包           
   1. yum list   
   2. yum list [all|glob_exp1] [glob_exp2] [...]  
   3. yum list [available|installed|updates] [glob_exp1][...]
> 安装程序包：     
   1. yum install package1 package2 [......]
   2. yum reinstall package1[...] (重新安装)
> 升级或者降级程序包：      
   1. yum update [package1] [package2] [...]
   2. yum downgrade package1 [...] (降级)

> 检查可用升级： yum check-update    

> 卸载程序包： yum remove |erase package ..    
> 查看程序包详细内容 ：yum info [...]   
> 查看指定特性（可以是某文件）是由哪个程序包所提供      
      >  yum provides |whatprovides feature1[ feature2] 

> 清理本地缓存：    
     > 清除/var/cache/yum/$basearch/$releasever 缓存
     > yum clean [all |packages |metafata |expire-cacherpmdb|plugins]
     > 一把我们都用all就好了。  
> 构建缓存  
> yum makecache   
> 我们在用yum repolist 和安装时都会自动缓存。  

> 搜索：yum search [特征或者关键字] 会搜索整个yum库带这个字或者特性的包，包括summary信息。

> 查看指定包所依赖的capabilities （功能）  
> yum deplist package1 [package20] [..]

> **查看yum事务历史**
> yum history 列出安装历史目录  
> yum history info [#] 查看第#号安装软件的内容  
> yum history packages-list 包名  列出安装包列表  
> yum history undo # 撤销第#号所做的步骤，如果#号是安装则这条命令是卸载相反。  

> 日志 ：/var/log/yum.log

> 包组管理的相关命令：
> yum groupinstall group1 [group2] [...] 安装组  
> yum groupupdate  group1 [group2] [...] 更新组  
> yum grouplist 列出组  
> yum groupremove group1 [group2] [...] 卸载组  
> yum groupinfo group1 [group2] [...] 查看组详细信息  

> yum 的命令行选项  
> --nogpgchech :禁止包检查  
> -y ： 自动yes  
> -q : 静默模式  
> --disablerepo=repiidglob :临时禁用此处指定的repo  
> --disablerepo=repoidglob : 临时启用此处指定的repo  
> --noplugins :禁用所有插件  

> 系统安装光盘作为本地yum仓库  
1. 挂载光盘至某目录 ：列如：/mnt/cdrom  
   - mount /dev/cd /mnt/cdrom  
2. 创建yum 配置文件  
```bash
[base]
name=name
baseurl=路径
gpgcheck=包检查
enabled=是否启用
```
如果是开发的单独rpm包创建yum库：
> createrepo[ options] /目录  
> 会生成repodata的目录这是yum必须要的。

### **编译安装方式**
> 安装环境
- 从官方网站上下载软件包到服务器上，就以apache 程序
- http://httpd.apache.org/ 下载2.4版本  
- 用我们刚刚搭建好的yum库来安装编译所需要的环境，我们直接安装yum groupinstall "Development Tools" 这个是开发包组。
> 编译安装  
+ **第一步**
 1. 把下载的程序包解压
 2. 进入目录运行 ./configure --help 看一下帮助文档可以有效的了解参数
 3. 运行 ./configure ----prefix=/app/httpd24  
    --sysconfdir=/etc/httpd24 --enable-ssl  
        --enable-proxy-fcgi  
    选项：指定安装位置和启用的特性  
    安装路径的设定：--prefix=/PATH ：指定默认安装位置，如果不指定默认为/usr/local/  
    --sysconfdir=/PATH :配置文件安装位置  
    可选特性开启或者禁用某些功能  
    --disable-feature  --enable=feature  
    可选用的包  
    --with-PACKAGE 包依赖  --without-PACKAGE 禁用依赖关系
 4. 在运行这个脚本的时候会检查依赖到的外部环境，如依赖的软件包
- <font color=red>注意：通常被编译操作依赖的程序包，需要安装此程序包的“开</font>
发”组件，其包名一般类似于name-devel-VERSION  
 通过选项传递的参数开启功能和安装路径等，执行时会参考用户指定以及Makefile.in 文件生成Makefile  。在这个过程中会报错，错误数没有装一些包我们只要按照提示装就好，一般都是包名-devel-版本
- **第二步make**
  + ./configure 运行完用 echo $?看看是否返回为0，如果为0则这步没有问题，若不是0则需要从新执行./configure 来看报什么错误然后在进行./configure 。
  + 我们直接在当前目录下运行mark 生成二进制文件

- **第三步mark install**
  + 我们要执行以下echo $?来确保上步命令是否有错误。
  + 在当前目录运行命令 mark install  
  + 这样编译就算完成了。
- **安装后的配置**
  + 配置运行环境 在/etc/profile.d/创建一个以.sh后缀的文件  
    1）PATH修改  在软件生成的bin里有一些命令，我们加入环境变量里可以 不用输入全路径运行，当然最好把路径加在最前面，  
    避免和系统的混了导致出错，
    ```bash 
     vim /etc/profile.d/httpd24.sh  
     PATH=/app/httpd24/bin:$PATH  
     . /etc/profile.d/httpd24.sh   
    ```
    + 启动服务 apachectl
    + 我们在看一下man帮助是否显示正常 
     ```bash
     导入帮助手册
     编辑/etc/man.config|man_db.conf文件
     添加一个MANPATH
     ```
     这样就完成了。






        

![notice1.png](0)