iproute2 为现代 Linux 发行版自带的软件包，他代替了传统的 net-tools 软件包，后者已经不再维护。

| Legacy utility                                                           | Replacement command     | Note                                |
| ------------------------------------------------------------------------ | ----------------------- | ----------------------------------- |
| [ifconfig](https://en.wikipedia.org/wiki/Ifconfig "Ifconfig")            | ip addr, ip link, ip -s | Address and link configuration      |
| [route](https://en.wikipedia.org/wiki/Route_(command) "Route (command)") | ip route                | Routing tables                      |
| arp                                                                      | ip neigh                | Neighbors                           |
| iptunnel                                                                 | ip tunnel               | Tunnels                             |
| nameif, ifrename                                                         | ip link set name        | Rename network interfaces           |
| ipmaddr                                                                  | ip maddr                | Multicast                           |
| [netstat](https://en.wikipedia.org/wiki/Netstat "Netstat")               | ip -s, ss, ip route     | Show various networking statistics  |
| brctl                                                                    | bridge                  | Handle bridge addresses and devices |

由于各种原因，不应再使用旧的 net-tools 包，但 netstat 依然通过其他软件包提供。

Linux 网络栈由内核完成，Linux 社区为用户空间提供了 iproute2 工具管理 Linux 网络内核功能。

iproute2 提供了一系列的命令，用于管理 Linux 网络栈的不同方面，包括网络接口，IP 地址，路由表，网络策略路由，网络命名空间等。

## ip link 命令

`ip link` 命令将列出主机上所有可用的网络接口

```text
[root@localhost ~]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:01:5e:e2 brd ff:ff:ff:ff:ff:ff
```

根据输出显示，该主机上有两个网络接口，分别为环回接口 `lo`，以太网接口 `ens33`。

`ip link` 命令的输出包含了网络接口的名称，状态，最大传输单元（MTU），队列长度，硬件地址等信息。

有时，你可能需要得知接口的 MAC 地址，以便区分具体的接口，上述输出中 `link/ether` 后面的地址即为 MAC 地址。

若要临时开启或关闭网络接口，可使用 `ip link set` 命令，如下所示：

```text
[root@localhost ~]# ip link set ens33 down
[root@localhost ~]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:01:5e:e2 brd ff:ff:ff:ff:ff:ff
```

上述命令将 `ens33` 接口关闭，`ip link` 命令的输出显示 `ens33` 接口的状态为 `DOWN`。

若要开启该接口，可使用 `ip link set ens33 up` 命令。

## ip addr 命令

`ip addr` 命令用于显示和管理网络接口的 IP 地址。

```text
[root@localhost ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:01:5e:e2 brd ff:ff:ff:ff:ff:ff
    inet 172.16.84.137/24 brd 172.16.84.255 scope global noprefixroute dynamic ens33
       valid_lft 1595sec preferred_lft 1595sec
    inet6 fe80::8ef7:1b18:75a0:e3f4/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

`ip addr` 命令的输出除去 `ip link` 命令的输出之外，还包含了网络接口的 IP 地址，包括 IPv4 地址和 IPv6 地址。

若要为网络接口添加 IP 地址，可使用 `ip addr add` 命令，如下所示：

```text
[root@localhost ~]# ip addr add 172.16.84.250/24 dev ens33
[root@localhost ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
         valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:01:5e:e2 brd ff:ff:ff:ff:ff:ff
    inet 172.16.84.137/24 brd 172.16.84.255 scope global noprefixroute dynamic ens33
       valid_lft 1595sec preferred_lft 1595sec
    inet 172.16.84.250/24 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::8ef7:1b18:75a0:e3f4/64 scope link noprefixroute
         valid_lft forever preferred_lft forever
```

上述命令为 `ens33` 接口添加了一个 IPv4 地址 `172.16.84.250/24`。

若要删除网络接口的 IP 地址，可使用 `ip addr del` 命令，如下所示：

```text
[root@localhost ~]# ip addr del 172.16.84.250/24 dev ens33
[root@localhost ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:01:5e:e2 brd ff:ff:ff:ff:ff:ff
    inet 172.16.84.137/24 brd 172.16.84.255 scope global noprefixroute dynamic ens33
       valid_lft 1595sec preferred_lft 1595sec
    inet6 fe80::8ef7:1b18:75a0:e3f4/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

上述命令删除了 `ens33` 接口的 IPv4 地址 `172.16.84.250/24`。

## ip route 命令

`ip route` 命令用于显示和管理路由表。

```text
[root@localhost ~]# ip route
default via 172.16.84.2 dev ens33 proto dhcp metric 100
172.16.84.0/24 dev ens33 proto kernel scope link src 172.16.84.137 metric 100
```

`ip route` 命令的输出包含了路由表中的路由信息。

若要添加路由，可使用 `ip route add` 命令，如下所示：

```text
[root@localhost ~]# ip route add 172.16.172.0/24 via 172.16.84.1
[root@localhost ~]# ip route
default via 172.16.84.2 dev ens33 proto dhcp metric 100
172.16.84.0/24 dev ens33 proto kernel scope link src 172.16.84.137 metric 100
172.16.172.0/24 via 172.16.84.1 dev ens33
```

上述命令为 `ens33` 接口添加了一个路由，该路由的目的网络为 `172.16.172.0/24`，下一跳地址为 `172.16.84.1`。

若要删除路由，可使用 `ip route del` 命令，如下所示：

```text
[root@localhost ~]# ip route del 172.16.172.0/24 via 172.16.84.1
[root@localhost ~]# ip route
default via 172.16.84.2 dev ens33 proto dhcp metric 100
172.16.84.0/24 dev ens33 proto kernel scope link src 172.16.84.137 metric 100
```

上述命令删除了 `ens33` 接口的路由 `172.16.172.0/24`。

## ss 命令

`ss` 命令用于显示和管理套接字。他替换了 `netstat` 命令，`ss` 命令的输出更加简洁，更易于阅读，且速度比 `netstat` 命令快得多。

```text
[root@localhost ~]# ss -nutlp
Netid  State      Recv-Q Send-Q                                                                      Local Address:Port                                                                                     Peer Address:Port
udp    UNCONN     0      0                                                                                       *:68                                                                                                  *:*                   users:(("dhclient",pid=7037,fd=6))
tcp    LISTEN     0      128                                                                                     *:22                                                                                                  *:*                   users:(("sshd",pid=1041,fd=3))
tcp    LISTEN     0      100                                                                             127.0.0.1:25                                                                                                  *:*                   users:(("master",pid=1535,fd=13))
tcp    LISTEN     0      128                                                                                  [::]:22                                                                                               [::]:*                   users:(("sshd",pid=1041,fd=4))
tcp    LISTEN     0      100                                                                                 [::1]:25                                                                                               [::]:*                   users:(("master",pid=1535,fd=14))
```

上述命令显示了当前系统中所有的 TCP 和 UDP 套接字。

