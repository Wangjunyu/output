Title: 编译部署案例-mutt开发版本
Date: 2019-08-08
Category: deploy

编译部署是系统工程师的基本功，在虚拟化环境还没有流行起来的年代，在没有docker、ansible的时候，编译部署是最主要的安装方式了。即使在今天，作为一名系统工程师，也应该可以不借助工具进行编译部署工作。
今天以mutt的开发版本部署为例，通过其中的排查问题过程，实战讲解一下编译部署的坑。

mutt 是一款经典的命令行邮件客户端工具，neomutt是基于mutt的分支其中增加了一些方便应用的增强的功能，但是由于开发者人力有限，现在的稳定版本发布工作已经远远落后于mutt。（具体的情况参见另一篇博客）

需要部署的源码：https://github.com/neomutt/neomutt
编译安装说明：https://github.com/neomutt/neomutt/blob/master/INSTALL.md

## ./configure
下载源码之后，一定一定要看一下INSTALL.md 文件，其中会有对应的参数配置说明。也可以用`./configure --help`查看全量的说明文件。你会发现有大量的配置参数，这里的参数会对最终的功能产生具体的影响。要用哪些呢？建议先以INSTALL文件中出现的参数做尝试。这个过程中如果出现报错，在解决的过程中有可能会反复查看INSTALL文件和网络搜索，一般排错的过程就是学习和强化的过程，渐渐地就会对各个参数有更明确的认识了。

以INSTALL中出现的命令为例进行安装，执行如下代码 `./configure --gnutls --gpgme --gss --sasl --tokyocabinet`,出现的报错为`Error: Unable to find GnuTLS`说明系统找不到这个包对应的目录。有可能是包没有安装，也有可能是包的路径或者需要引用的内容找不到。

Tips:
如果你有用过brew，可以搜索一下`brew neomutt formula`，找到 https://formulae.brew.sh/formula/neomutt 之后打开 https://github.com/Homebrew/homebrew-core/blob/master/Formula/neomutt.rb 会发现一段ruby代码，这是brew安装时系统执行的命令，其中也能找到configure 对应的参数，可以用作参考

这时候可以执行`yum search GnuTLS`看一下结果，如果结果如下，就比较顺利，可以从中间挑选自己需要的包直接安装。
```
gnutls-c++.i686 : The C++ interface to GnuTLS
gnutls-c++.x86_64 : The C++ interface to GnuTLS
gnutls-dane.i686 : A DANE protocol implementation for GnuTLS
gnutls-dane.x86_64 : A DANE protocol implementation for GnuTLS
gnutls-devel.i686 : Development files for the gnutls package
gnutls-devel.x86_64 : Development files for the gnutls package
xmlsec1-gnutls.i686 : GNUTls crypto plugin for XML Security Library
xmlsec1-gnutls.x86_64 : GNUTls crypto plugin for XML Security Library
xmlsec1-gnutls-devel.i686 : GNUTls crypto plugin for XML Security Library
xmlsec1-gnutls-devel.x86_64 : GNUTls crypto plugin for XML Security Library
gnutls.i686 : A TLS protocol implementation
gnutls.x86_64 : A TLS protocol implementation
gnutls-utils.x86_64 : Command line tools for TLS protocol
libtasn1.i686 : The ASN.1 library used in GNUTLS
libtasn1.x86_64 : The ASN.1 library used in GNUTLS
rsyslog-gnutls.x86_64 : TLS protocol support for rsyslog
```
注意，上面提到的包之间是有依赖关系的，gnutls.x86_64 是基础包，gnutls-devel.x86_64 是开发工具包，依赖于gnutls.x86_64,所以一般先装基础包，再安装开发包。`sudo yum install gnutls.x86_64` 执行后显示的内容如下
```
Package gnutls-3.3.29-9.el7_6.x86_64 already installed and latest version
Nothing to do
```
这说明刚才我们说的得到了验证，服务器上已经安装了gnutls.x86_64，configure的时候找不到，试一下`sudo yum install gnutls-devel.x86_64`,解决了这个问题。顺利执行后得到如下的内容。否则还是会出现error需要继续排查和安装，直到相关的包都安装完毕。

在这个过程里还可能遇到的问题
* 安装了对应的包之后报错提示版本太旧，至少要xxx版本。这种情况比较麻烦，一般yum已经不好用了，需要自己去网上找对应的包来安装。这时候要注意你的linux的发行版，找对对应的版本。尽量找二进制的可以直接安装的文件，如果找到的是src的源码文件，自己需要把那个包的编译过程再跑一遍...找到了二进制源码文件也有可能是需要其他的依赖，还需要继续去寻找，如果安装成功后，有可能还需要devel的版本。

这里补充我遇到过和解决掉的问题,其实这些并不是问题的全部...只是在理解了编译安装套路之后遇到的问题，之前还有一些匪夷所思的问题都已经没有记录了。

```
Error: Unable to find GPGME
# GPGME是gpg make easy的缩写，这个包yum下1.3.2版本太旧，必须要1.4以上。最新是1.13，但是很难找到对应的包，并且..安装后还需要gpgme-devel的包，下面的链接供参考
# https://cbs.centos.org/koji/buildinfo?buildID=19862

Error: Trying to remove "yum", which is protected
# yum 删除的时候遇到，调整为rpm用下面的参数删除，并且不删除对应的依赖
# rpm -e --nodeps
# https://stackoverflow.com/questions/15799047/trying-to-remove-yum-which-is-protected-in-centos

Error: Unable to find SASL on centos
# yum安装

Error: Unable to find Notmuch
# yum 安装

liblmdb.so.0.0.0()(64bit) is needed by lmdb-0.9.22-2.el7.x86_64
# 在找到了lmdb（yum没有，网上找的rpm源）rpm安装时缺少依赖，又去找前面的依赖
# https://pkgs.org/download/liblmdb.so.0.0.0()(64bit)
# https://centos.pkgs.org/7/epel-x86_64/lmdb-libs-0.9.22-2.el7.x86_64.rpm.html

Error: Unable to find TokyoCabinet
# yum 安装

Error: no ID for constraint linkend: header-cache.
# 之前出现的，理顺之后就没有再出现
```




```
Summary of build options:

  Version:           20180716
  Host OS:           linux-gnu
  Install prefix:    /usr
  Compiler:           cc
  CFlags:            -g -O2 -std=c99 -fno-delete-null-pointer-checks -D_ALL_SOURCE=1 -D_GNU_SOURCE=1 -D__EXTENSIONS__ -I/usr/include -DNCURSES_WIDECHAR
  LDFlags:           -L/usr/lib -lgssapi_krb5 -lkrb5 -lk5crypto -lcom_err
  Libs:              -ltokyocabinet -lidn -lgnutls -lncursesw -ltinfo -lsasl2 -lanl  -L/usr/lib64 -lgpgme -lgpg-error

  GPGME:             yes
  PGP:               yes
  SMIME:             yes
  Notmuch:           no
  Header Cache(s):   tokyocabinet
  Lua:               no
```
其中标明为no的功能，未来需要的时候可以通过configure做配置再编译安装即可。过程中很可能还会出现刚才的情况反复出现，需要安装不同的依赖关系。

另外需要注意的是，如果这里的参数选择的不够，后面的make 和 make install的过程还有可能出错。

## make clean;make
如果自己之前有过失败的make，建议先执行make clean把对应的临时内容处理完再make

## make install
如果出现如下的报错，需要加上sudo
```
install: cannot remove ‘/usr/share/locale/bg/LC_MESSAGES/neomutt.mo’: Permission denied
make: *** [install-po] Error 1
```
```
Error: no ID for constraint linkend: mix-entry-format.
# 一直忽略了这个问题，貌似和doc的生成有关，对应用无影响
```

到这时，所有的安装过程已经结束，可以试着执行命令看是否安装完成





## 编译安装

neomutt的安装是经典的编译过程。
 * 通过 ./configure 和其中的参数变量设置进行编译前的检查工作
 * make 生成需要的中间文件
 * make install 编译，完成安装

### 背景知识

其实编译安装的背后有大量的背景知识，建议查看「鸟哥私房菜」中的第五部分：Linux 系统管理员中的软件安装： 原始码与 Tarball 以及 软件安装：RPM, SRPM 与 YUM 功能 部分的内容，这里讲的非常清楚，以后有时间也会专门来写一写这部分的笔记。

### 编译安装的好处
在编译的过程中的参数配置，能够更好的了解相关功能的配置关系。试想一下你在macOS下brew安装的时候会去考虑这个问题或者去看fomula么？macOS偏向于个人应用，可以忽略这个过程。但如果是商用的服务器上安装的应用程序，这个过程还是有必要了解清楚的，为以后的配置修改做准备。

### 常用命令
这部分写给自己，有一些命令可能日常不用不记得，来查一下就好。(后面备注的是自己的卡片系统中的定位)

yum search xxx
yum list installed |grep yyy
yum remote zzz
yum install -y xxx

rpm -i xxx.rpm

## Reference

* https://github.com/neomutt
* https://neomutt.org
* http://cn.linux.vbird.org/linux_basic/linux_basic.php
* http://cn.linux.vbird.org/linux_basic/0520source_code_and_tarball.php
* http://cn.linux.vbird.org/linux_basic/0520rpm_and_srpm.php

## ChangeLog

* 2019-08-08 新建文件
* blog、github、quip、du、ow 发布
* LeonTalk 备份（后续版本以wiki的更新为准）

