# deb 包

`deb` 软件包本着简单的原则设计，他可以直接被 `ar`、`tar`、`gzip` 这类应用处理，也就意味着，`deb` 实际上只是一种单纯的打包格式。

这意味着，一旦你不小心删除了系统的 `dpkg` 程序，或者为没有 `dpkg` 包管理工具的 Linux 系统拿到了一个 `deb` 软件包，也可以进行安装或修复系统。

> 当然在非面向平台强行使用包这种行为是极不推荐的

也就是说，可以通过 `ar` 打包与解包 `deb` 文件，并且查看内部结构。

一个 `deb` 软件包包含这个包的元信息 ( `metadata` )，以及实际的二进制或源码，加上其他所需文件。

元信息中包含了当前软件包的版本号，依赖，冲突，不兼容等，他的格式类似于电子邮件头 ( 定义于 [RFC 2822](https://tools.ietf.org/html/rfc2822)，) 可以通过 `apt-cache` 查看。

例如，以下为 `apt` 软件包的元信息：

```console
debian@debian:~$ apt-cache show apt
Package: apt
Version: 1.8.2.2
Installed-Size: 4064
Maintainer: APT Development Team <deity@lists.debian.org>
Architecture: amd64
Replaces: apt-transport-https (<< 1.5~alpha4~), apt-utils (<< 1.3~exp2~)
Provides: apt-transport-https (= 1.8.2.2)
Depends: adduser, gpgv | gpgv2 | gpgv1, debian-archive-keyring, libapt-pkg5.0 (>= 1.7.0~alpha3~), libc6 (>= 2.15), libgcc1 (>= 1:3.0), libgnutls30 (>= 3.6.6), libseccomp2 (>= 1.0.1), libstdc++6 (>= 5.2)
Suggests: apt-doc, aptitude | synaptic | wajig, dpkg-dev (>= 1.17.2), gnupg | gnupg2 | gnupg1, powermgmt-base
Breaks: apt-transport-https (<< 1.5~alpha4~), apt-utils (<< 1.3~exp2~), aptitude (<< 0.8.10)
Description-en: commandline package manager
 This package provides commandline tools for searching and
 managing as well as querying information about packages
 as a low-level access to all features of the libapt-pkg library.
 .
 These include:
  * apt-get for retrieval of packages and information about them
    from authenticated sources and for installation, upgrade and
    removal of packages together with their dependencies
  * apt-cache for querying available information about installed
    as well as installable packages
  * apt-cdrom to use removable media as a source for packages
  * apt-config as an interface to the configuration settings
  * apt-key as an interface to manage authentication keys
Description-md5: 9fb97a88cb7383934ef963352b53b4a7
Recommends: ca-certificates
Section: admin
Priority: important
Filename: pool/updates/main/a/apt/apt_1.8.2.2_amd64.deb
Size: 1418564
MD5sum: 03e41cd6be84c762a3c255a02efa9285
SHA256: 89d691610de7c717abffd2f85932f8e48da71212a1e2e05681855f1f99d1c4f9
```

通常管理 `deb` 软件包使用 `dpkg` 与 `apt` 工具包，前者是更加面向管理员的，而后者是更多面向用户的。但这两者各有各的用途，需要视情况而定使用何者。

Red Hat 发行版使用的 RPM 软件包也可通过转换的方式运行在 Debian 上，但这会带来潜在的兼容性问题，但幸好 `deb` 格式的软件包非常丰富，通常想要的都能找得到。

deb 打包并不在此教程的范围内。
