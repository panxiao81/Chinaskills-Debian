# NTP 服务器

NTP 服务器可用于为其他主机下发时间，使用 UDP 123 端口通信。

通常使用的 NTP 服务器端程序有两种，分别为 `ntpd` 与 `chrony`。

本篇继承前文配置 NTP 客户端，讲述如何配置 NTP 服务器。

## ntpd

对 ntpd 进行服务器配置需要对配置文件进行修改

在 33-55 行带有配置示例：

```sh
# /etc/ntp.conf

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
server 127.127.1.0
fudge 127.127.1.0 stratum 10
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

## chrony

chrony 配置对比 ntpd 更为简单

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

> 参考资料：[ArchWiki:Network Time Protocol daemon (简体中文)](https://wiki.archlinux.org/index.php/Network_Time_Protocol_daemon_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
>
> [Red Hat Documentation:CONFIGURING NTP USING THE CHRONY SUITE](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-configuring_ntp_using_the_chrony_suite) 