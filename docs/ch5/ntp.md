在配置集群等服务时，需要保持多台主机的时间统一。

本文不涉及配置 NTP 服务器对外服务，而是如何配置 NTP 客户端。

对于简单的时间同步， `systemd` 自带的 NTP 客户端就足以满足要求，但如果有多一点的需求 ( 例如你需要连接一个硬件来提供时钟 ) ，则需要安装 `ntpd` 服务。

# systemd-timesyncd

当系统安装后，只要安装时网络畅通，则 Debian 会自动配置 `systemd-timesyncd` 并与 Debian 的官方时间服务器同步。

要增加 NTP 服务器，则需要修改 `/etc/systemd/timesyncd.conf`

```bash
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
NTP=pool.ntp.org 0.asia.pool.ntp.org 1.asia.pool.ntp.org 2.asia.pool.ntp.org
```

多服务器间用空格分割。

保存更改后，重启 `systemd-timesyncd` 服务

```sh
$ systemctl restart systemd-timesyncd
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

要启用并运行：

```sh
$ timedatectl set-ntp true
```

同步需要一点时间，可能会卡住。

查看状态使用：

```sh
$ timedatectl timesync-status
```

# ntpd

如果需要更多的特性，或需要同时部署 NTP 服务器，则需要安装 `ntpd` 软件包。

`ntpd` 与 系统自带的 `systemd-timesyncd` 有冲突，需要停止自带服务。

## 作为客户端接收时间

systemd 自带的 timedatectl 只能控制系统自带的 systemd-timesyncd，使用 `timedatectl set-ntp true` 会停止 ntpd 服务。

```sh
$ timedatectl set-ntp false
```

安装 `ntpd` 服务

```sh
$ apt-get install ntp
```

修改配置文件 `/etc/ntp.conf`

在 23 行处可修改原始服务器地址为所需的服务器地址：

```bash
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
server ntp1.aliyun.com
server ntp2.aliyun.com
```

重启服务生效：

```sh
$ systemctl restart ntpd
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

## 作为服务器下发时间

对 ntpd 进行服务器配置需要对配置文件进行修改

在 33-55 行带有配置示例：

```sh
# By default, exchange time with everybody, but don't allow configuration.
restrict -4 default kod notrap nomodify nopeer noquery limited
restrict -6 default kod notrap nomodify nopeer noquery limited

# Local users may interrogate the ntp server more closely.
restrict 127.0.0.1
restrict ::1

# Needed for adding pool entries
restrict source notrap nomodify noquery

# Clients from this (example!) subnet have unlimited access, but only if
# cryptographically authenticated.
#restrict 192.168.123.0 mask 255.255.255.0 notrust


# If you want to provide time to your local subnet, change the next line.
# (Again, the address is an example only.)
#broadcast 192.168.123.255
```

为确保失去网络也能提供同步服务，需要增加一个本地时间作为时间源：

```sh
server 127.0.0.1
fudge 127.0.0.1 stratum 10
```

接下来新增策略，允许局域网内的主机与本机同步时间，例如本例中我的局域网 IP 为 `192.168.50.0/24`,则可以写为

```sh
restrict 192.168.50.0 mask 255.255.255.0 nomodify
```

此处的 `nomodify` 意为不允许客户端使用 `ntpq` 或 `ntpdc` 修改服务器配置。

所有的选项可以通过 `ntp.conf(5)` 查看。

重启服务后通过 `ss -nutlp | grep ntp` 查看服务是否正常监听

```console
root@debian:~# vim /etc/ntp.conf
root@debian:~# systemctl restart ntp
root@debian:~# ss -nutlp | grep ntp
udp   UNCONN 0      0                        192.168.50.157:123         0.0.0.0:*                        users:(("ntpd",pid=3783,fd=19))
udp   UNCONN 0      0                             127.0.0.1:123         0.0.0.0:*                        users:(("ntpd",pid=3783,fd=18))
udp   UNCONN 0      0                               0.0.0.0:123         0.0.0.0:*                        users:(("ntpd",pid=3783,fd=17))
udp   UNCONN 0      0      [fe80::20c:29ff:fe70:453]%ens192:123            [::]:*                        users:(("ntpd",pid=3783,fd=21))
udp   UNCONN 0      0                                 [::1]:123            [::]:*                        users:(("ntpd",pid=3783,fd=20))
udp   UNCONN 0      0                                  [::]:123            [::]:*                        users:(("ntpd",pid=3783,fd=16))
```

如果具有 `123` 端口上的 udp 报文监听则说明服务正常运行。

接下来使用另一台客户机与此主机进行同步测试。这台主机为另一台服务器，运行 Ubuntu 20.04 LTS

```console
panxiao81@homelinux:~$ sudo vim /etc/systemd/timesyncd.conf
[sudo] password for panxiao81:
panxiao81@homelinux:~$ systemctl restart systemd-timesyncd
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to restart 'systemd-timesyncd.service'.
Authenticating as: panxiao81
Password:
==== AUTHENTICATION COMPLETE ===
panxiao81@homelinux:~$ timedatectl timesync-status
       Server: 192.168.50.157 (192.168.50.157)
Poll interval: 2min 8s (min: 32s; max 34min 8s)
         Leap: normal
      Version: 4
      Stratum: 2
    Reference: CA760182
    Precision: 1us (-24)
Root distance: 30.112ms (max: 5s)
       Offset: +5.737ms
        Delay: 218us
       Jitter: 2.168ms
 Packet count: 2
    Frequency: +224.757ppm
```

也可使用 `chrony` 使用和提供 NTP 服务，此处不再赘述，RHEL 8 中的软件源现在默认自带 `chrony` 配置与 `ntpd` 大同小异。

# chrony

## 作为服务器

chrony 配置对比 ntpd 更为简单

对于 Debian，配置文件存放在 `/etc/chrony/chrony.conf`

仅需要简单的使用 `allow` 即可

```sh
allow 10.10.100.0/24
```

如果再简单一点，允许所有主机与其同步时间，只要

```sh
allow all
```

需要使用本地时间的场合，也只需要简单的加一句 `local` 即可

```sh
local stratum 10
```

## 客户端

正常在配置文件里写入 `server pool.ntp.org iburst`

查看当前时间服务器使用 `chronyc sources`

正常情况下 Chrony 会步进同步时钟，其与 NTPd 的工作方式很不相同，并没有一个真正意义上与 `ntpdate` 作用类似的强制同步方式

但如果需要的话，使用 `chronyc -a makestep`

如果 chronyd 没有运行，正如 `ntpdate` 使用的那种情况，则可以使用 `chronyd -q 'server pool.ntp.org iburst'` 手工同步时间。

> 参考资料：[ArchWiki:Network Time Protocol daemon (简体中文)](https://wiki.archlinux.org/index.php/Network_Time_Protocol_daemon_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
> 
> [Red Hat Documentation:CONFIGURING NTP USING THE CHRONY SUITE](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-configuring_ntp_using_the_chrony_suite) 