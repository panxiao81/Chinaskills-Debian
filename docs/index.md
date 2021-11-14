# 本书简介

本书为应学校所需，以全国职业技能大赛为主要范围，所写的一本 Debian 系 Linux 快速入门与参考书。

本书从基本系统安装讲起，并着重讲述网络服务配置与系统维护，适合 Linux 学习和运维学习者阅读。

本书将不再重复讲述 Linux 通用知识，尤其前几章，更适合具有一些 Linux 使用经验的人阅读。

本书主要参考 [Debian 管理员手册](https://www.debian.org/doc/manuals/debian-handbook/)写成，并按照原文需求使用 [GPL](https://www.gnu.org/licenses/) 与 [CC-BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/) 方式放出。

由于笔者能力有限，书中有误的地方在所难免，如有读者发现问题，可到 [GitHub 项目](https://github.com/panxiao81/Chinaskills-Debian) 中提 `Issue`，我会在力所能及的范围内尽力改正。

## 一些约定

在本书中，会出现大量的代码块来书写命令与配置文件。

为不造成阅读时的困扰，特约定如下：

在代码块中使用 `$` 前缀情况，代表当前使用普通用户，`$` 后的为输入的命令，如：

```console
$ uname
Linux
```

在代码块中使用 `#` 代表当前使用 `root` 用户，如

```console
# systemctl restart ssh
```

在代码块中使用 `$ #` 的形式代表 `#` 后面部分为注释，如

```console
$ # 查看 hosts 文件
$ cat /etc/hosts
```

如果内容为配置文件，则会在第一行用注释注明是什么文件，并且与正文内容空出一行，如：

```sh
# /etc/network/interfaces

# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug ens33
iface ens33 inet dhcp
```

有一部分为实验时完整复制的显示内容，不做另外说明。
