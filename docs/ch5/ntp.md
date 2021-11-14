# 时间同步

在配置集群等服务时，需要保持多台主机的时间统一。

本文不涉及配置 NTP 服务器对外服务，而是如何配置 NTP 客户端。

对于简单的时间同步， `systemd` 自带的 NTP 客户端就足以满足要求，但如果有多一点的需求 ( 例如你需要连接一个硬件来提供时钟 ) ，则需要安装 `ntpd` 或 `chrony` 服务。

## systemd-timesyncd

当系统安装后，只要安装时网络畅通，则 Debian 会自动配置 `systemd-timesyncd` 并与 Debian 的官方时间服务器同步。

要增加 NTP 服务器，则需要修改 `/etc/systemd/timesyncd.conf`

```bash
# /etc/systemd/timesyncd.conf

#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
#
# Entries in this file show the compile time defaults.
# You can change settings by editing this file.
# Defaults can be restored by simply deleting this file.
#
# See timesyncd.conf(5) for details.

[Time]
#NTP=
#FallbackNTP=0.debian.pool.ntp.org 1.debian.pool.ntp.org 2.debian.pool.ntp.org 3.debian.pool.ntp.org
#RootDistanceMaxSec=5
#PollIntervalMinSec=32
#PollIntervalMaxSec=2048
```

添加服务器需要取消 `NTP=` 这一行的注释，并填入 NTP 服务器地址。

例如：

```bash
# /etc/systemd/timesyncd.conf

NTP=pool.ntp.org 0.asia.pool.ntp.org 1.asia.pool.ntp.org 2.asia.pool.ntp.org
```

多服务器间用空格分割。

保存更改后，重启 `systemd-timesyncd` 服务

```console
# systemctl restart systemd-timesyncd
```

之后验证配置

使用 `timedatectl show-timesync --all`

```console
root@debian:~# timedatectl show-timesync --all
LinkNTPServers=
SystemNTPServers=0.asia.pool.ntp.org 1.asia.pool.ntp.org
FallbackNTPServers=0.debian.pool.ntp.org 1.debian.pool.ntp.org 2.debian.pool.ntp.org 3.debian.pool.ntp.org
ServerName=0.asia.pool.ntp.org
ServerAddress=139.59.15.185
RootDistanceMaxUSec=5s
PollIntervalMinUSec=32s
PollIntervalMaxUSec=34min 8s
PollIntervalUSec=1min 4s
NTPMessage={ Leap=0, Version=4, Mode=4, Stratum=4, Precision=-24, RootDelay=86.166ms, RootDispersion=14.190ms, Reference=A29FC801, OriginateTimestamp=Tue 2020-12-22 19:34:01 CST, ReceiveTimestamp=Tue 2020-12-22 19:34:02 CST, TransmitTimestamp=Tue 2020-12-22 19:34:02 CST, DestinationTimestamp=Tue 2020-12-22 19:34:02 CST, Ignored=no PacketCount=1, Jitter=0 }
Frequency=7623778
```

NTP 服务器选择顺序如下：

- `systemd-networkd.service(8)` 中针对每个接口的配置，或者 DHCP 服务优先。
- `/etc/systemd/timesyncd.conf` 中定义的 NTP服务器会在运行时被添加到针对每个接口的服务器列表中，守护进程会轮流连接这些服务器直到某个有应答。
- 如果上两步没有取到 NTP 服务器， 那么就是用 `FallbackNTP=` 中定义的服务器。

要启用,运行：

```console
# timedatectl set-ntp true
```

同步需要一点时间，可能会卡住。

查看状态使用：

```console
# timedatectl timesync-status
```

## ntpd

如果需要更多的特性，或需要同时部署 NTP 服务器，则需要安装 `ntpd` 软件包。

`ntpd` 与 系统自带的 `systemd-timesyncd` 有冲突，需要停止自带服务。

systemd 自带的 timedatectl 只能控制系统自带的 systemd-timesyncd，并且两者同时使用会因为同时占用 123/udp 而产生冲突，因此使用 `timedatectl set-ntp false` 停止 `systemd-timesyncd` 服务。

```console
# timedatectl set-ntp false
```

安装 `ntpd` 服务

```console
# apt-get install ntp
```

修改配置文件 `/etc/ntp.conf`

在 23 行处可修改原始服务器地址为所需的服务器地址：

```bash
# /etc/ntp.conf

# pool.ntp.org maps to about 1000 low-stratum NTP servers.  Your server will
# pick a different set every time it starts up.  Please consider joining the
# pool: <http://www.pool.ntp.org/join.html>
pool 0.debian.pool.ntp.org iburst
pool 1.debian.pool.ntp.org iburst
pool 2.debian.pool.ntp.org iburst
pool 3.debian.pool.ntp.org iburst
```

例如修改为：

```bash
# /etc/ntp.conf

server ntp1.aliyun.com
server ntp2.aliyun.com
```

重启服务生效：

```console
# systemctl restart ntpd
```

使用 `ntpq -p` 查看同步状态

```console
root@debian:~# ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 0.debian.pool.n .POOL.          16 p    -   64    0    0.000    0.000   0.000
 1.debian.pool.n .POOL.          16 p    -   64    0    0.000    0.000   0.000
 2.debian.pool.n .POOL.          16 p    -   64    0    0.000    0.000   0.000
 3.debian.pool.n .POOL.          16 p    -   64    0    0.000    0.000   0.000
-tick.ntp.infoma .GPS.            1 u  146  128  176  225.512  -11.584   0.508
+203.107.6.88    100.107.25.114   2 u   12   64  377   29.612   -9.788   0.910
+119.28.183.184  100.122.36.196   2 u   17   64  337   66.883   -8.777   1.218
*202.118.1.130   .PTP.            1 u   77   64  376   44.633    9.074   0.494
-119.28.206.193  100.122.36.4     2 u   73  128  277   69.470   -8.139   5.240
```

delay, offset 和 jitter 不应该为零，ntpd 同步的服务器前有星号，ntpd 可能等待几分钟后才会进行同步

## chrony

`chrony` 为新一代的 NTP 守护进程实现，配置比起 `ntpd` 来说更加简单，`chrony` 目前为 Red Hat 推荐使用的 NTP 守护进程

对于 Debian，配置文件存放在 `/etc/chrony/chrony.conf`

只需在配置文件里写入 `server pool.ntp.org iburst`

其中 `pool.ntp.org` 替换为要同步使用的服务器 IP 或域名

查看当前时间服务器使用 `chronyc sources`

正常情况下 Chrony 会步进同步时钟，其与 ntpd 的工作方式很不相同，并没有一个真正意义上与 `ntpdate` 命令作用类似的强制同步方式

但如果需要的话，使用 `chronyc -a makestep` 强制步进同步。

如果 chronyd 没有运行，正如 `ntpdate` 使用的那种情况，则可以使用 `chronyd -q 'server pool.ntp.org iburst'` 手工同步时间。

> 参考资料：[ArchWiki:Network Time Protocol daemon (简体中文)](https://wiki.archlinux.org/index.php/Network_Time_Protocol_daemon_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
>
> [Red Hat Documentation:CONFIGURING NTP USING THE CHRONY SUITE](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-configuring_ntp_using_the_chrony_suite) 
