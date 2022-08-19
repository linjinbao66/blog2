---
title: Centos 7 下编译openjdk
date: 2019-10-22
tags:
  - Java
---

## openjdk编译

最近在学习java虚拟机的技术，尝试下手动编译`openjdk`

## 准备工作

1. CentOS7 环境
2. openjdk8 源码
3. bootstrap jdk源码（我用的是openjdk 7）

注意事项：

1. 目标jdk版本号比bootstrap jdk 大1
2. 准备可能需要安装各种编译工具

   \`\`\`code

   yum -y install build-essential gawk m4 libasound2-print-dev binutils libmotif3 libmotif-dev ant

yum install libX\*

```text
## 准备目标`jdk`和`bootstrap jdk`

```code
## 目标jdk，openjdk8
unzip  openjdk-8-src-b132-03_mar_2014.zip
```

解压完成会得到一个`openjdk`文件夹，里面目录如下：

```text
[root@localhost openjdk8]# ll
total 348
-rwxrwxrwx.  1 root root   1503 Mar  4  2014 ASSEMBLY_EXCEPTION
drwxr-xr-x.  3 root root     89 Oct 22 14:03 build
drwxrwxrwx.  6 root root     61 Mar  4  2014 common
-rwxrwxrwx.  1 root root   1235 Mar  4  2014 configure
drwxrwxrwx.  5 root root    157 Mar  4  2014 corba
-rwxrwxrwx.  1 root root   1384 Mar  4  2014 get_source.sh
drwxrwxrwx.  7 root root    182 Mar  4  2014 hotspot
drwxrwxrwx.  6 root root    232 Mar  4  2014 jaxp
drwxrwxrwx.  6 root root    232 Mar  4  2014 jaxws
drwxrwxrwx.  6 root root    169 Mar  4  2014 jdk
drwxrwxrwx.  6 root root    169 Mar  4  2014 langtools
-rwxrwxrwx.  1 root root  19263 Mar  4  2014 LICENSE
drwxrwxrwx.  6 root root    189 Mar  4  2014 make
-rwxrwxrwx.  1 root root   6430 Mar  4  2014 Makefile
drwxrwxrwx. 12 root root    275 Mar  4  2014 nashorn
-rwxrwxrwx.  1 root root   1549 Mar  4  2014 README
-rwxrwxrwx.  1 root root 129333 Mar  4  2014 README-builds.html
drwxrwxrwx.  2 root root     22 Mar  4  2014 test
-rwxrwxrwx.  1 root root 178445 Mar  4  2014 THIRD_PARTY_README
```

我这里为了方便区分，改名字成`openjdk8`

`bootstrap jdk`如下：

```text
## openjdk7
[root@localhost java]# cd jdk1.7.0_80/
[root@localhost jdk1.7.0_80]# ll
total 19780
drwxr-xr-x. 2 10 143     4096 Apr 11  2015 bin
-r--r--r--. 1 10 143     3339 Apr 11  2015 COPYRIGHT
drwxr-xr-x. 4 10 143      122 Apr 11  2015 db
drwxr-xr-x. 3 10 143      132 Apr 11  2015 include
drwxr-xr-x. 5 10 143      185 Apr 11  2015 jre
drwxr-xr-x. 5 10 143      250 Apr 11  2015 lib
-r--r--r--. 1 10 143       40 Apr 11  2015 LICENSE
drwxr-xr-x. 4 10 143       47 Apr 11  2015 man
-r--r--r--. 1 10 143      114 Apr 11  2015 README.html
-rw-r--r--. 1 10 143      499 Apr 11  2015 release
-rw-r--r--. 1 10 143 19947513 Apr 11  2015 src.zip
-rw-r--r--. 1 10 143   110114 Mar 17  2015 THIRDPARTYLICENSEREADME-JAVAFX.txt
-r--r--r--. 1 10 143   173571 Apr 11  2015 THIRDPARTYLICENSEREADME.txt
[root@localhost jdk1.7.0_80]#
```

## 编译检查

进入`openjdk8`文件夹，执行命令`./configure --with-boot-jdk=/root/java/jdk1.7.0_80` 这里要指定编译的bootstrap jdk

进行编译检查

检查结束可能需要安装CCache：

```text
wget https://www.samba.org//ftp/ccache/ccache-3.6.tar.xz
tar -xvf ccache-3.6.tar.xz
 cd ccache-3.6
./configure -prefix=/var/ccache
make -j8
make install
ln -s /var/ccache/bin/ccache /usr/bin/ccache
```

再次执行编译检查

然后执行编译命令`make all`

等待40分钟，结果如下： ![&#x7ED3;&#x679C;](https://raw.githubusercontent.com/linjinbao666/resources/master/amromPicture/jdk%E7%BC%96%E8%AF%91%E7%BB%93%E6%9E%9C.png)

## 测试编译出来的jdk是否可用

编译出来的jdk在目录如下中

```text
/root/java/openjdk8/build/linux-x86_64-normal-server-release/images/j2sdk-image/bin
```

执行测试

```text
[root@localhost bin]# ./java -version
openjdk version "1.8.0-internal"
OpenJDK Runtime Environment (build 1.8.0-internal-root_2019_10_22_14_11-b00)
OpenJDK 64-Bit Server VM (build 25.0-b70, mixed mode)
[root@localhost bin]#
```

## 搞一个`HelloWorld`

```text
public class Hello{

        public static void main(String[] args){

                System.out.println("Hello");


        }


}
```

编译运行

```text
[root@localhost testjdk]# javac Hello.java
[root@localhost testjdk]# java Hello
Hello
[root@localhost testjdk]#
```

## 总结要点

1. _**Building JDK 8 requires use of a version of JDK 7 that is at Update 7 or newer. JDK 8 developers should not use JDK 8 as the boot JDK, to ensure that JDK 8 dependencies are not introduced into the parts of the system that are built with JDK 7.**_
2. 下载源码极度困难
3. 编译耗时长
4. [官方文档](http://hg.openjdk.java.net/jdk8/jdk8/raw-file/tip/README-builds.html)
5. [官方文档](http://cr.openjdk.java.net/~ihse/demo-new-build-readme/common/doc/building.html)

下载链接统一整理：

1. [openjdk7](https://github.com/linjinbao666/resources/tree/master/%E8%BD%AF%E4%BB%B6%E9%93%BE%E6%8E%A5/jdk1.7%20)
2. [openjdk8](https://github.com/linjinbao666/resources/tree/master/%E8%BD%AF%E4%BB%B6%E9%93%BE%E6%8E%A5/jdk1.8)
3. [openjdk8（我编译出来的）](https://github.com/linjinbao666/resources/tree/master/%E8%BD%AF%E4%BB%B6%E9%93%BE%E6%8E%A5/jdk1.8-lin%20)

全部： 链接：[https://pan.baidu.com/s/1V41RPhDKCq7jK8PVxBIVWQ](https://pan.baidu.com/s/1V41RPhDKCq7jK8PVxBIVWQ) 提取码：5ccy
