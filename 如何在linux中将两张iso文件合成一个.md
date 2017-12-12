#  如何将两个iso合成一个
### 如何将两张iso镜像文件合成一个iso镜像
- 环境我们拿centos 6.9 版本中的两个镜像合成一个完整和做启动光盘。
- 从centso官网上下载一个光盘合成脚本https://wiki.centos.org/
- 我这已经下载好了我把脚本内容贴出来，可以直接用
```bash
#!/bin/bash

# by Chris Kloiber <ckloiber@redhat.com>
# Mods under CentOS by Phil Schaffner <pschaff2@verizon.net>

# A quick hack that will create a bootable DVD iso of a Red Hat Linux
# Distribution. Feed it either a directory containing the downloaded
# iso files of a distribution, or point it at a directory containing
# the "RedHat", "isolinux", and "images" directories.

# This version only works with "isolinux" based Red Hat Linux versions.

# Lots of disk space required to work, 3X the distribution size at least.

# GPL version 2 applies. No warranties, yadda, yadda. Have fun.

# Modified to add sanity checks and fix CentOS4 syntax errors

# TODO:
#   Add checks for available disk space on devices holding output and
#       temp files.
#   Add optional 3rd parameter to specify location of temp directory.
#   Create .discinfo if not present.

OS_VER=\
$((test -e /etc/fedora-release && rpm -qf /etc/fedora-release --qf "FC%{VERSION}") \
|| (test -e /etc/redhat-release && rpm -qf /etc/redhat-release --qf "EL%{VERSION}") \
|| echo OS_unknown)

case "$OS_VER" in
  EL[45]*|FC?)
        IMPLANT=/usr/lib/anaconda-runtime/implantisomd5
        if [ ! -f $IMPLANT ]; then
            echo "Error: $IMPLANT Not Found!"
            echo "Please install anaconda-runtime and try again."
            exit 1
        fi
        ;;
  EL6*|FC1?)
        IMPLANT=/usr/bin/implantisomd5
        if [ ! -f $IMPLANT ]; then
            echo "Error: $IMPLANT Not Found!"
            echo "Please install isomd5sum and try again."
            exit 1
        fi
        ;;
  OS_unknown)
        echo "Unknown OS."
        exit 1
        ;;
  *)
        echo "Fix this script for $OS_VER"
        exit 1
esac

if [ $# -lt 2 ]; then
        echo "Usage: `basename $0` source /destination/DVD.iso"
        echo ""
        echo "        The 'source' can be either a directory containing a single"
        echo "        set of isos, or an exploded tree like an ftp site."
        exit 1
fi

DVD_DIR=`dirname $2`
DVD_FILE=`basename $2`

echo "DVD directory is $DVD_DIR"
echo "ISO file is $DVD_FILE"

if [ "$DVD_DIR" = "." ]; then
    echo "Destinaton Directory $DVD_DIR does not exist"
    exit 1
else
    if [ ! -d "/$DVD_DIR" ]; then
        echo "Destinaton Directory $DVD_DIR must be an absolute path"
        exit 1
    else
        if [ "$DVD_FILE" = "" ] || [ -d "$DVD_DIR/$DVD_FILE" ]; then
            echo "Null ISO file name."
            exit 1
        fi
    fi
fi

which mkisofs >&/dev/null
if [ "$?" != 0 ]; then
    echo "mkisofs Not Found"
    echo "yum install mkisofs"
fi

which createrepo >&/dev/null
if [ "$?" != 0 ]; then
    echo "createrepo Not Found"
    echo "yum install createrepo"
fi

if [ -f $2 ]; then
    echo "DVD ISO destination $2 already exists. Remove first to recreate."
    exit 1
fi

# Make sure there is enough free space to hold the DVD image on the filesystem
# where the home directory resides, otherwise change ~/mkrhdvd to point to
# a filesystem with sufficient free space.

cleanup() {
    [ ${LOOP:=/tmp/loop} = "/" ] && echo "LOOP mount point = \/, dying!" && exit
    [ -d $LOOP ] && rm -rf $LOOP 
    [ ${DVD:=~/mkrhdvd} = "/" ] && echo "DVD data location is \/, dying!" && exit
    [ -d $DVD ] && rm -rf $DVD 
}

cleanup
mkdir -p $LOOP
mkdir -p $DVD

ls $1/*.iso &>/dev/null
if [ "$?" = 0 ]; then

    echo "Found ISO CD images..."

    CDS=`expr 0`
    DISKS="1"

    [ -w / ] || {   # Very portable, but perhaps not perfect, test for superuser.
        echo "Only 'root' may use this script for loopback mounts" 1>&2
        exit 1
    }

    for f in `ls $1/*.iso`; do
        mount -o loop $f $LOOP
        cp -av $LOOP/* $DVD
        if [ -f $LOOP/.discinfo ]; then
            cp -av $LOOP/.discinfo $DVD
            CDS=`expr $CDS + 1`
            if [ $CDS != 1 ] ; then
                DISKS=`echo ${DISKS},${CDS}`
            fi
        fi
        umount $LOOP
    done
else
    if [ -f $1/isolinux/isolinux.bin ]; then

        echo "Found FTP-like tree..."

        if [ -e $1/.discinfo ]; then
            cp -av $1/.discinfo $DVD
        else
# How does one construct a legal .discinfo file if none is found?
            echo "Error: No .discinfo file found in $1"
            cleanup
            exit 1
        fi
        cp -av $1/* $DVD
    else
        echo "Error: No CD images nor FTP-like tree found in $1"
        cleanup
        exit 1
    fi
fi

if [ -e $DVD/.discinfo ]; then
    awk '{ if ( NR == 4 ) { print disks } else { print ; } }' disks="ALL" $DVD/.discinfo > $DVD/.discinfo.new
    mv $DVD/.discinfo.new $DVD/.discinfo
else
    echo  "Error: No .discinfo file found in $DVD"
    cleanup
    exit 1
fi

rm -rf $DVD/isolinux/boot.cat
find $DVD -name TRANS.TBL | xargs rm -f

cd $DVD
createrepo -g repodata/comps.xml ./
mkisofs -J -R -v -T -o $2 -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 8 -boot-info-table $DVD
if [ "$?" = 0 ]; then

    echo ""
    echo "Image complete, create md5sum..."

#  $IMPLANT --force $2
# Don't like forced mediacheck? Try this instead.
    $IMPLANT --supported-iso --force $2

    echo "Start cleanup..."

    cleanup

    echo ""
    echo "Process Complete!"
    echo "Wrote DVD ISO image to $DVD_DIR/$DVD_FILE"
    echo ""
else
    echo "ERROR: Image creation failed, start cleanup..."

    cleanup

    echo ""
    echo "Failed to create ISO image $DVD_DIR/$DVD_FILE"
    echo ""
fi
```
- 现在把两张centos镜像挂载到/mnt上面
```bash
  mount /dev/cdrom /mnt/ 把iso镜像挂载到mnt上
  mkdir /app/iso1  创建一个文件夹
  cp -rp /mnt/*  /app/iso1 将第一张光盘里面所有的内容全部复制过去
  然后在复制二张光盘的，也是要先把光盘挂载到mnt上，注意一点由于第一张全部复制的，所有第二张盘只要复制包文件
  cp -r /mnt/Packages/* iso1/Packages/
  然后在运行脚本
   ./mkdvdiso.sh --help 但是在查看帮助时让我们安装一个校验光盘的包，给装上就好了
rpm -iv /mnt/Packages/isomd5sum-devel-1.0.6-1.el6.
 ./mkdvdiso.sh /app/iso1 /app/centso6.9.iso 运行脚本

```
- 这样就成功，现在我们用ftp服务器给传到win7上。



