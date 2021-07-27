使用最小安装 ( 未安装 GUI 情况下 ) 的 Debian 将使用 `network` 网络管理器。

而安装 GUI 的情况下则会安装 `NetworkManager`

配置网络的方式与其他 Linux 系统并没有太多区别。

即便是使用最小方式安装的 Debian，也可配置使用 `NetworkManager` 来管理网络。

# network 网络管理器

`network` 网络管理器是一直以来 Debian 默认使用的网络管理器。

与早期版本的 Red Hat 发行版类似，`network` 把网络配置写在一个文件内，并从文件内加载配置。

但与 Red Hat 系发行版不同的是，Red Hat 发行版将网卡配置文件保存在 `/etc/sysconfig/network-scripts/ifcfg-*name*` 文件中，而 Debian 将网络配置文件统一写在 `/etc/network/interfaces` 文件中。

默认情况下，在一个新安装的 Debian 中，网络是这样配置的，但 Debian 给出的最佳实现是将网络配置分网卡保存到 `/etc/network/interface.d/` 下。

在我的安装好的系统中，网卡默认配置为 DHCP 方式获取 IP 地址，`/etc/network/interfaces` 内容如下：

```bash
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

当下，使用 systemd 的系统默认会使用 `biosdevname` 方案来对网卡命名，他的好处是网卡名不会因重启或更换物理硬件等其他情况而改变，但结果会导致网卡名不再像过去那种 `eth0` 一类的名字易于推断。

因此配置网卡前，可能我们需要使用 `ip addr` 命令 ( [来自 iproute2 工具包](/ch10/iproute2) ), 在我的例子中，运行如下：

```console
root@debian:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:f7:01:7f brd ff:ff:ff:ff:ff:ff
    inet 192.168.28.128/24 brd 192.168.28.255 scope global dynamic ens33
       valid_lft 1498sec preferred_lft 1498sec
    inet6 fe80::20c:29ff:fef7:17f/64 scope link
       valid_lft forever preferred_lft forever
```

如此可见，本例的网卡名为 `ens33`。

在主配置文件中，或在 `/etc/network/interface.d/` 中，创建配置文件 ( 文件名不限 )，对网卡进行配置。

如需要 DHCP，写作如下：

```bash
auto ens33
iface ens33 inet dhcp
```

如配置静态 IPv4 地址，则写作如下：

```bash
auto ens33
iface ens33 inet static
  address 192.168.0.3/24  # IP/CIDR 形式的 IP 地址
  gateway 192.168.0.1  # 网关地址
```

DNS 信息填入 `/etc/resolv.conf`

```bash
$ nameserver 114.114.114.114
```

重启网卡生效

```sh
# 重启网卡
$ ifdown ens33
$ ifup ens33
```
> 文档： [man interface(5)](http://man.he.net/?topic=interfaces&section=all)

# NetworkManager

NetworkManager 是一个由 GNOME 项目开发的，使得 Linux 的网络配置尽可能简单，开箱即用而开发的软件包。

目前 Red Hat 发行版已经自带并默认使用 NetworkManager 来管理网络。

如何查看当前的 Debian 安装是否使用 NetworkManager？

- 使用 `nmcli` 或 `nmtui` 命令，如果运行了 NetworkManager 的管理工具，则当前系统可能在使用 NetworkManager 管理网络
- 使用 `systemctl status network-manager` 查看服务状态，如果具有该服务，并且服务正在运行，则说明当前系统正在使用 NetworkManager 管理网络

NetworkManager 与传统的 network 网络管理器不可同时使用，如果使用两者同时管理同一网卡会造成冲突。

## 安装

最小化安装的 Debian 不会安装 NetworkManager，若要使用 NetworkManager，需要安装软件包。

```sh
$ apt install network-manager
```

屏幕输出

```console
root@debian:~# apt install network-manager
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  crda dns-root-data dnsmasq-base iw libbluetooth3 libgudev-1.0-0 libjansson4
  libjim0.77 libmbim-glib4 libmbim-proxy libmm-glib0 libndp0 libnl-3-200
  libnl-genl-3-200 libnl-route-3-200 libnm0 libpcap0.8 libpcsclite1
  libpolkit-agent-1-0 libpolkit-backend-1-0 libpolkit-gobject-1-0 libqmi-glib5
  libqmi-proxy libteamdctl0 modemmanager policykit-1 ppp usb-modeswitch
  usb-modeswitch-data wireless-regdb wpasupplicant
Suggested packages:
  pcscd libteam-utils comgt wvdial wpagui libengine-pkcs11-openssl
The following NEW packages will be installed:
  crda dns-root-data dnsmasq-base iw libbluetooth3 libgudev-1.0-0 libjansson4
  libjim0.77 libmbim-glib4 libmbim-proxy libmm-glib0 libndp0 libnl-3-200
  libnl-genl-3-200 libnl-route-3-200 libnm0 libpcap0.8 libpcsclite1
  libpolkit-agent-1-0 libpolkit-backend-1-0 libpolkit-gobject-1-0 libqmi-glib5
  libqmi-proxy libteamdctl0 modemmanager network-manager policykit-1 ppp
  usb-modeswitch usb-modeswitch-data wireless-regdb wpasupplicant
0 upgraded, 32 newly installed, 0 to remove and 0 not upgraded.
Need to get 9,152 kB of archives.
After this operation, 31.4 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://mirrors.163.com/debian buster/main amd64 libbluetooth3 amd64 5.50-1.2~deb10u1 [100 kB]
Get:2 http://mirrors.163.com/debian buster/main amd64 libjansson4 amd64 2.12-1 [38.0 kB]
Get:3 http://mirrors.163.com/debian buster/main amd64 libmm-glib0 amd64 1.10.0-1 [942 kB]

--- # 省略部分输出

Setting up libqmi-proxy (1.22.0-1.2) ...
Setting up network-manager (1.14.6-2+deb10u1) ...

The following network interfaces were found in /etc/network/interfaces
which means they are currently configured by ifupdown:
- ens33
If you want to manage those interfaces with NetworkManager instead
remove their configuration from /etc/network/interfaces.

Created symlink /etc/systemd/system/dbus-org.freedesktop.nm-dispatcher.service →
 /lib/systemd/system/NetworkManager-dispatcher.service.
Created symlink /etc/systemd/system/network-online.target.wants/NetworkManager-w
ait-online.service → /lib/systemd/system/NetworkManager-wait-online.service.
Created symlink /etc/systemd/system/multi-user.target.wants/NetworkManager.servi
ce → /lib/systemd/system/NetworkManager.service.
Setting up modemmanager (1.10.0-1) ...
Created symlink /etc/systemd/system/dbus-org.freedesktop.ModemManager1.service →
 /lib/systemd/system/ModemManager.service.
Created symlink /etc/systemd/system/multi-user.target.wants/ModemManager.service
 → /lib/systemd/system/ModemManager.service.
Processing triggers for systemd (241-7~deb10u5) ...
Processing triggers for man-db (2.8.5-2) ...
Processing triggers for dbus (1.12.20-0+deb10u1) ...
Processing triggers for libc-bin (2.28-10) ...
root@debian:~# 
```

安装 NetworkManager 后，安装过程提示在原有 `/etc/network/interfaces` 中的网卡不会被自动由 NetworkManager 配置。

## 配置

### 迁移

NetworkManager 会带来两个新的网络配置工具，`nmcli` 与 `nmtui` 。

但 NetworkManager 会覆盖已有的所有网络配置，包括 DNS 在内，因此所有配置要使用 NetworkManager 配套工具进行。

如果要整体对网络管理器进行迁移，需要进行一番操作。

在继续使用之前，我们将原配置文件的网卡配置注释。

```console
$ root@debian:~# vi /etc/network/interfaces

```

```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface

#allow-hotplug ens33  # 在源配置前加 # 注释
#iface ens33 inet dhcp

```

接下来重启 `networking` 与 `network-manager` 服务

```shell
$ systemctl restart networking
$ systemctl restart network-manager
```

### nmtui

`nmtui` 是一个图形化方式配置网络的工具，易于上手。

在终端中输入

```sh
$ nmtui
```

即可打开 `nmtui` 工具。

```console






                           ┌─┤ NetworkManager TUI ├──┐
                           │                         │ 
                           │ Please select an option │ 
                           │                         │ 
                           │ Edit a connection       │ 
                           │ Activate a connection   │ 
                           │ Set system hostname     │ 
                           │                         │ 
                           │ Quit                    │ 
                           │                         │ 
                           │                    <OK> │ 
                           │                         │ 
                           └─────────────────────────┘ 
                                                       





```

三个选项分别为：
- 编辑连接
- 启用连接
- 设置系统主机名
- 退出

选择 `Edit a connection` 进入编辑

```console

                    ┌───────────────────────────────────────┐
                    │                                       │ 
                    │ ┌─────────────────────────┐           │ 
                    │ │ Ethernet              ↑ │ <Add>     │ 
                    │ │   ens33               ▒ │           │ 
                    │ │   Wired connection 1  ▒ │ <Edit...> │ 
                    │ │                       ▒ │           │ 
                    │ │                       ▒ │ <Delete>  │ 
                    │ │                       ▒ │           │ 
                    │ │                       ▒ │           │ 
                    │ │                       ▒ │           │ 
                    │ │                       ▒ │           │ 
                    │ │                       ▮ │           │ 
                    │ │                       ▒ │           │ 
                    │ │                       ▒ │           │ 
                    │ │                       ▒ │           │ 
                    │ │                       ▒ │           │ 
                    │ │                       ▒ │           │ 
                    │ │                       ↓ │ <Back>    │ 
                    │ └─────────────────────────┘           │ 
                    │                                       │ 
                    └───────────────────────────────────────┘ 
                                                              
```

迁移网络后，出现了两个配置，实际上他们配置的是一张网卡，可以选择其中一个删除。

选中一个连接按 `Enter` 键后，可以很轻松的修改配置。

修改好配置后，回到第一个界面，进入 `Activate a connection`，选择修改的连接，按两下 `Enter` 键重新激活即可。

> 扩展阅读： [Red Hat Enterprise Linux 7 联网指南](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/networking_guide/index)

 