# sudo

在 Linux 中，许多操作需要我们通过 `root` 账户权限完成，但我们通常不想开放这个账户，也不希望经常切换用户，因此就有了 `sudo` 命令，他提供的功能是：当我们需要 `root` 权限时，将权限临时提升为 `root` 执行当前命令来保障权限安全。

在本章节配置好 `sudo` 之后，本书的其他所有操作都将使用普通用户完成。

## 安装 sudo

首先安装 sudo 软件包：

```console
# apt-get install sudo
```

接下来最简单的操作方法是将允许使用 `sudo` 命令的用户加入 `sudo` 组：

```console
# usermod debian -aG sudo
# id debian
uid=1000(debian) gid=1000(debian) groups=1000(debian),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),109(netdev)
```

接下来切换回 `debian` 用户，尝试使用需要管理员权限的命令，并在前面加上 `sudo`

```console
root@debian:~# su - debian
debian@debian:~$ apt update
Reading package lists... Done
E: Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)
E: Unable to lock directory /var/lib/apt/lists/
W: Problem unlinking the file /var/cache/apt/pkgcache.bin - RemoveCaches (13: Permission denied)
W: Problem unlinking the file /var/cache/apt/srcpkgcache.bin - RemoveCaches (13: Permission denied)
debian@debian:~$ sudo apt update

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for debian:
Get:1 http://ftp.debian.org/debian buster-backports InRelease [46.7 kB]
Hit:2 http://deb.debian.org/debian stable InRelease
Get:3 http://deb.debian.org/debian stable-updates InRelease [51.9 kB]
Get:4 http://ftp.debian.org/debian buster-backports/main Sources.diff/Index [27.8 kB]
Hit:5 http://deb.debian.org/debian-security stable/updates InRelease
Get:6 http://ftp.debian.org/debian buster-backports/main Sources 2021-01-12-1400.20.pdiff [1,057 B]
Get:6 http://ftp.debian.org/debian buster-backports/main Sources 2021-01-12-1400.20.pdiff [1,057 B]
Fetched 127 kB in 2s (76.4 kB/s)
Reading package lists... Done
Building dependency tree
Reading state information... Done
16 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

提示输入密码时，输入当前用户 ( `debian` ) 的密码。

## 配置 sudo

sudo 的配置可以使用 `sudo -ll` 查看，使用 `sudo -lU` 查看特定用户的配置

```console
debian@debian:~$ sudo -ll
Matching Defaults entries for debian on debian:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User debian may run the following commands on debian:

Sudoers entry:
    RunAsUsers: ALL
    RunAsGroups: ALL
    Commands:
        ALL
debian@debian:~$ sudo -lU debian
Matching Defaults entries for debian on debian:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User debian may run the following commands on debian:
    (ALL : ALL) ALL
```

`sudo` 的配置文件为 `/etc/sudoers`，但一旦文件内部出现问题会导致 sudo 无法运行，如此时无法使用 `root` 账户则会发生很严重的事故，因此修改配置文件须使用 `visudo` 命令。

`visudo` 会在一个临时文件里修改 `/etc/sudoers` 文件，并检查语法无误后再写入 `/etc/sudoers`

执行：

```sh
$ sudo visudo
```

在 Debian 中，`visudo` 会默认调用 `nano` 编辑器，如要使用 `vim`，可在 `/etc/sudoers` 中指定：

```sh
# /etc/sudoers

# Set default EDITOR to vim, and do not allow visudo to use EDITOR/VISUAL.
Defaults      editor=/usr/bin/vim, !env_editor
```

为某个用户可以执行所有命令，在配置文件中加入：

`用户名   ALL=(ALL) ALL`

允许以某个主机名登录用户执行命令：

`用户名   主机名=(ALL) ALL`

允许wheel用户组成员无密码使用sudo：

`%wheel      ALL=(ALL) NOPASSWD: ALL`

要不询问某个用户的密码:

`Defaults:USER_NAME      !authenticate`

只为用户启用部分命令的执行权限：

`用户名 主机名=/sbin/halt,/sbin/poweroff,/sbin/reboot,/usr/bin/apt`

## 常见问题

### 执行 visudo 提示 command not found

由于非 root 账户的 `$PATH` 环境变量默认没有 `/sbin` 与 `/usr/sbin` 目录，因此无法直接执行 `sbin` 目录下的文件。

默认情况下，在 Debian 中，`/etc/sudoers` 指定了使用 `sudo` 执行命令时 `$PATH` 环境变量会产生变化，这个变化的环境变量会加入一系列 `sbin` 目录。

因此请使用 `sudo visudo`

对其他类似的情况 ( 如 执行 `usermod` 一类的命令 ) 也同理。

### 禁用 root 用户

在配置好 sudo 后，可以选择禁用 root 用户，执行：

```console
$ sudo passwd -l root
```

如需解锁，执行 `sudo passwd -u root`
