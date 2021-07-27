`dpkg` 是处理 `deb` 软件包的工具，他被所有基于 Debian 的发行版默认附带。

`dpkg` 与 Red Hat 发行版的 `rpm` 工具相似，属于一类管理工具。

`dpkg` 在使用时需要已经存在某个软件包。软件包可以从 Debian 的安装光盘获取，也可以从网络上下载得到。

例如我当前有一个软件包为 `containerd.io_1.4.3-1_amd64.deb` ( Docker 的容器运行时 )，保存在 `/home/debian` 目录中。

该软件包可从 Docker 官方源中下载得到，地址为

http://download.docker.com/linux/debian/dists/buster/pool/stable/amd64/containerd.io_1.4.3-1_amd64.deb 

如果 Linux 实验环境连通互联网，可直接使用 `curl` 或 `wget` 工具下载。

由于 `wget` 为 GNU 软件包附带的软件，Debian 会默认安装，因此可以使用 `wget` 工具下载：

```console
debian@debian:~$ pwd
/home/debian
debian@debian:~$ wget http://download.docker.com/linux/debian/dists/buster/pool/stable/amd64/containerd.io_1.4.3-1_amd64
.deb
--2021-01-13 16:07:06--  http://download.docker.com/linux/debian/dists/buster/pool/stable/amd64/containerd.io_1.4.3-1_amd64.deb
Resolving download.docker.com (download.docker.com)... 54.239.169.41, 54.239.169.76, 54.239.169.15, ...
Connecting to download.docker.com (download.docker.com)|54.239.169.41|:80... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: https://download.docker.com/linux/debian/dists/buster/pool/stable/amd64/containerd.io_1.4.3-1_amd64.deb [following]
--2021-01-13 16:07:07--  https://download.docker.com/linux/debian/dists/buster/pool/stable/amd64/containerd.io_1.4.3-1_amd64.deb
Connecting to download.docker.com (download.docker.com)|54.239.169.41|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 28103678 (27M) [binary/octet-stream]
Saving to: ‘containerd.io_1.4.3-1_amd64.deb’

containerd.io_1.4.3-1_amd64.d 100%[=================================================>]  26.80M  12.9MB/s    in 2.1s

2021-01-13 16:07:11 (12.9 MB/s) - ‘containerd.io_1.4.3-1_amd64.deb’ saved [28103678/28103678]
debian@debian:~$ ls -l
total 27448
-rw-r--r-- 1 debian debian 28103678 Dec 29 00:38 containerd.io_1.4.3-1_amd64.deb
```

对其管理方法如下：

# 安装软件包

使用 `dpkg -i`:

> 即 `--install`

> 参数都有 “长” 版 (一个或多个字，前置两个链接号) 与 “短” 版 (一个字母，通常是长版字的第一个字母，前置一个链接号) 两种写法。这是 POSIX 标准常见的用法。

```sh
$ sudo dpkg -i containerd.io_1.4.3-1_amd64.deb
[sudo] password for debian:
Selecting previously unselected package containerd.io.
(Reading database ... 36792 files and directories currently installed.)
Preparing to unpack containerd.io_1.4.3-1_amd64.deb ...
Unpacking containerd.io (1.4.3-1) ...
Setting up containerd.io (1.4.3-1) ...
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /lib/systemd/system/containerd.service.
Processing triggers for man-db (2.8.5-2) ...
```

有时会遇到特定软件包所包含的文件已经在另一个软件包中被安装，如果能够完全确定这不会造成问题的话，可使用 `--force-overwrite` 参数进行强制覆盖。

虽然存在许多 `--force-*` 参数，但通常都不推荐使用，这也是为什么通常推荐使用 `apt` 从软件源中安装软件包。

> 当然，如果你真的知道你在做什么的话当然可以忽略这句话

# 卸载软件包

有两个卸载选项，分别为 `-r` ( `--remove` ) 与 `-P` ( `--purge` )，前者为卸载软件包但保留应用配置，后者为完全清除，不保留配置。

除去安装需要完整指定要安装的软件包的文件名以外，其他操作只需指定包名。

```console
debian@debian:~$ sudo dpkg -r containerd.io
(Reading database ... 36808 files and directories currently installed.)
Removing containerd.io (1.4.3-1) ...
Processing triggers for man-db (2.8.5-2) ...
debian@debian:~$ sudo dpkg -P containerd.io
(Reading database ... 36794 files and directories currently installed.)
Purging configuration files for containerd.io (1.4.3-1) ...
```

# 查询 `dpkg` 数据库，检查 `deb` 文件

一些常用参数整理如下：

| 短参数 | 长参数 | 功能 |
|-|-|-|
| `-L` | `--listfiles` | 列出软件包会安装的文件列表 |
| `-S` | `--search` | 列出哪一个包包含指定的文件 |
| `-l` | `--list` | 列出本机上已知的包及安装状态 |
| `-s` | `--status` | 列出包头 ( 即包的详细信息 ) |
| `-c` | `--contents` | 列出安装包包含的文件 |
| `-I` | `--info` | 指定文件查看包信息 |

一些使用例：

```console
debian@debian:~$ dpkg -c containerd.io_1.4.3-1_amd64.deb
drwxr-xr-x root/root         0 2020-12-02 22:33 ./
drwxr-xr-x root/root         0 2020-12-02 22:33 ./etc/
drwxr-xr-x root/root         0 2020-12-02 22:33 ./etc/containerd/
-rw-r--r-- root/root       886 2020-12-02 22:33 ./etc/containerd/config.toml
drwxr-xr-x root/root         0 2020-12-02 22:33 ./lib/
drwxr-xr-x root/root         0 2020-12-02 22:33 ./lib/systemd/
drwxr-xr-x root/root         0 2020-12-02 22:33 ./lib/systemd/system/
-rw-r--r-- root/root      1263 2020-12-02 22:33 ./lib/systemd/system/containerd.service
drwxr-xr-x root/root         0 2020-12-02 22:33 ./usr/
drwxr-xr-x root/root         0 2020-12-02 22:33 ./usr/bin/
-rwxr-xr-x root/root  57193712 2020-12-02 22:33 ./usr/bin/containerd
-rwxr-xr-x root/root   7264728 2020-12-02 22:33 ./usr/bin/containerd-shim
-rwxr-xr-x root/root   9945400 2020-12-02 22:33 ./usr/bin/containerd-shim-runc-v1
-rwxr-xr-x root/root   9966072 2020-12-02 22:33 ./usr/bin/containerd-shim-runc-v2
-rwxr-xr-x root/root  30444912 2020-12-02 22:33 ./usr/bin/ctr
-rwxr-xr-x root/root  14437328 2020-12-02 22:33 ./usr/bin/runc
drwxr-xr-x root/root         0 2020-12-02 22:33 ./usr/share/
drwxr-xr-x root/root         0 2020-12-02 22:33 ./usr/share/doc/
drwxr-xr-x root/root         0 2020-12-02 22:33 ./usr/share/doc/containerd.io/
-rw-r--r-- root/root      2168 2020-12-02 22:33 ./usr/share/doc/containerd.io/changelog.Debian.gz
-rw-r--r-- root/root      1055 2020-12-02 22:33 ./usr/share/doc/containerd.io/copyright
drwxr-xr-x root/root         0 2020-12-02 22:33 ./usr/share/man/
drwxr-xr-x root/root         0 2020-12-02 22:33 ./usr/share/man/man5/
-rw-r--r-- root/root      1991 2020-12-02 22:33 ./usr/share/man/man5/containerd-config.toml.5.gz
drwxr-xr-x root/root         0 2020-12-02 22:33 ./usr/share/man/man8/
-rw-r--r-- root/root       544 2020-12-02 22:33 ./usr/share/man/man8/containerd-config.8.gz
-rw-r--r-- root/root       786 2020-12-02 22:33 ./usr/share/man/man8/containerd.8.gz
-rw-r--r-- root/root      4501 2020-12-02 22:33 ./usr/share/man/man8/ctr.8.gz
debian@debian:~$ dpkg -L vim-common
/.
/etc
/etc/vim
/etc/vim/vimrc
/usr
/usr/bin
/usr/bin/helpztags
/usr/lib
/usr/lib/mime
/usr/lib/mime/packages
/usr/lib/mime/packages/vim-common
/usr/share
/usr/share/applications
/usr/share/applications/vim.desktop
/usr/share/doc
/usr/share/doc/vim-common
/usr/share/doc/vim-common/NEWS.Debian.gz
/usr/share/doc/vim-common/README.Debian
/usr/share/doc/vim-common/changelog.Debian.gz
/usr/share/doc/vim-common/changelog.gz
/usr/share/doc/vim-common/copyright
/usr/share/icons
/usr/share/icons/hicolor
/usr/share/icons/hicolor/48x48
/usr/share/icons/hicolor/48x48/apps
/usr/share/icons/hicolor/48x48/apps/gvim.png
/usr/share/icons/hicolor/scalable
/usr/share/icons/hicolor/scalable/apps
/usr/share/icons/hicolor/scalable/apps/gvim.svg
/usr/share/icons/locolor
/usr/share/icons/locolor/16x16
/usr/share/icons/locolor/16x16/apps
/usr/share/icons/locolor/16x16/apps/gvim.png
/usr/share/icons/locolor/32x32
/usr/share/icons/locolor/32x32/apps
/usr/share/icons/locolor/32x32/apps/gvim.png
/usr/share/lintian
/usr/share/lintian/overrides
/usr/share/lintian/overrides/vim-common
/usr/share/man
/usr/share/man/da
/usr/share/man/da/man1
/usr/share/man/da/man1/vim.1.gz
/usr/share/man/da/man1/vimdiff.1.gz
/usr/share/man/de
/usr/share/man/de/man1
/usr/share/man/de/man1/vim.1.gz
/usr/share/man/fr
/usr/share/man/fr/man1
/usr/share/man/fr/man1/vim.1.gz
/usr/share/man/fr/man1/vimdiff.1.gz
/usr/share/man/it
/usr/share/man/it/man1
/usr/share/man/it/man1/vim.1.gz
/usr/share/man/it/man1/vimdiff.1.gz
/usr/share/man/ja
/usr/share/man/ja/man1
/usr/share/man/ja/man1/vim.1.gz
/usr/share/man/ja/man1/vimdiff.1.gz
/usr/share/man/man1
/usr/share/man/man1/helpztags.1.gz
/usr/share/man/man1/vim.1.gz
/usr/share/man/man1/vimdiff.1.gz
/usr/share/man/pl
/usr/share/man/pl/man1
/usr/share/man/pl/man1/vim.1.gz
/usr/share/man/pl/man1/vimdiff.1.gz
/usr/share/man/ru
/usr/share/man/ru/man1
/usr/share/man/ru/man1/vim.1.gz
/usr/share/man/ru/man1/vimdiff.1.gz
/usr/share/pixmaps
/usr/share/pixmaps/gvim.svg
/usr/share/pixmaps/vim-16.xpm
/usr/share/pixmaps/vim-32.xpm
/usr/share/pixmaps/vim-48.xpm
/usr/share/vim
/usr/share/vim/vim81
/usr/share/vim/vim81/debian.vim
/var
/var/lib
/var/lib/vim
/var/lib/vim/addons
/usr/share/man/da/man1/rview.1.gz
/usr/share/man/da/man1/rvim.1.gz
/usr/share/man/de/man1/rview.1.gz
/usr/share/man/de/man1/rvim.1.gz
/usr/share/man/fr/man1/rview.1.gz
/usr/share/man/fr/man1/rvim.1.gz
/usr/share/man/it/man1/rview.1.gz
/usr/share/man/it/man1/rvim.1.gz
/usr/share/man/ja/man1/rview.1.gz
/usr/share/man/ja/man1/rvim.1.gz
/usr/share/man/man1/rview.1.gz
/usr/share/man/man1/rvim.1.gz
/usr/share/man/pl/man1/rview.1.gz
/usr/share/man/pl/man1/rvim.1.gz
/usr/share/man/ru/man1/rview.1.gz
/usr/share/man/ru/man1/rvim.1.gz
/usr/share/vim/vimrc
debian@debian:~$ dpkg -S /bin/busybox
busybox: /bin/busybox
debian@debian:~$ dpkg -s coreutils
Package: coreutils
Essential: yes
Status: install ok installed
Priority: required
Section: utils
Installed-Size: 15719
Maintainer: Michael Stone <mstone@debian.org>
Architecture: amd64
Multi-Arch: foreign
Version: 8.30-3
Pre-Depends: libacl1 (>= 2.2.23), libattr1 (>= 1:2.4.44), libc6 (>= 2.28), libselinux1 (>= 2.1.13)
Description: GNU core utilities
 This package contains the basic file, shell and text manipulation
 utilities which are expected to exist on every operating system.
 .
 Specifically, this package includes:
 arch base64 basename cat chcon chgrp chmod chown chroot cksum comm cp
 csplit cut date dd df dir dircolors dirname du echo env expand expr
 factor false flock fmt fold groups head hostid id install join link ln
 logname ls md5sum mkdir mkfifo mknod mktemp mv nice nl nohup nproc numfmt
 od paste pathchk pinky pr printenv printf ptx pwd readlink realpath rm
 rmdir runcon sha*sum seq shred sleep sort split stat stty sum sync tac
 tail tee test timeout touch tr true truncate tsort tty uname unexpand
 uniq unlink users vdir wc who whoami yes
Homepage: http://gnu.org/software/coreutils
debian@debian:~$ dpkg -l "base*"
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name           Version      Architecture Description
+++-==============-============-============-==================================================
un  base           <none>       <none>       (no description available)
un  base-config    <none>       <none>       (no description available)
ii  base-files     10.3+deb10u6 amd64        Debian base system miscellaneous files
ii  base-passwd    3.5.46       amd64        Debian base system master password and group files
debian@debian:~$ dpkg -I containerd.io_1.4.3-1_amd64.deb
 new Debian package, version 2.0.
 size 28103678 bytes: control archive=1520 bytes.
      28 bytes,     1 lines      conffiles
     381 bytes,    13 lines      control
     850 bytes,    13 lines      md5sums
    1345 bytes,    32 lines   *  postinst             #!/bin/sh
     643 bytes,    21 lines   *  postrm               #!/bin/sh
     225 bytes,     7 lines   *  prerm                #!/bin/sh
 Package: containerd.io
 Version: 1.4.3-1
 Architecture: amd64
 Maintainer: Containerd team <help@containerd.io>
 Installed-Size: 126263
 Depends: libc6 (>= 2.14), libseccomp2 (>= 2.3.0)
 Conflicts: containerd, runc
 Replaces: containerd, runc
 Provides: containerd, runc
 Section: devel
 Priority: optional
 Homepage: https://containerd.io
 Description: An open and reliable container runtime
```

