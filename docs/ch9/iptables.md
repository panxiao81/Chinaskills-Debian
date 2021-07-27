时间关系，仅整理要点

实际生产建议学习 nftables 与 firewalld

> 参考资料：
> [鸟哥](https://linux.vbird.org/linux_server/centos6/0250simple_firewall.php#netfilter)
> [Debian Wiki:iptables](https://wiki.debian.org/iptables)

iptables 的规则是有顺序的！ 数据包会按规则的排列顺序依次判断通过。

iptables 默认分 3 张 table ( 表 ) 而规则作为 chain 存储在对应的表中。

默认的表为：

- filter 影响进入本机的数据包
- nat 进行 NAT 时用到的表
- mangle 与打标记的包有关，本例不讲述

# iptables 语法

预设规则

预设规则指：当数据包不满足设定的规则时，则执行的默认操作。

如：设定 INPUT 的默认规则为 DROP

```sh
$ iptables -P INPUT DROP
```

添加规则

仅针对网卡 ( 源与目标 IP 地址 )

```sh
$ iptables [-A keychain] [-i interfaces] [-s ip/cidr] -j [ACCEPT | REJECT | DROP]
```

- keychain: 键名
- interfaces: 网卡名
- ip/cidr 源 IP 地址 ( 支持使用 IP/CIDR 格式 )
- ACCEPT/REJECT/DROP  数据包动作：接受/拒绝/丢弃

例：放行来自本机回环地址的所有网络数据包：

```sh
$ iptables -A INPUT -i lo -j ACCEPT
```

针对端口 ( TCP 与 UDP 协议 )

```sh
$ iptables [-A 键名] [-i 名] [-p tcp,udp] \
> [-s 源 IP 地址/CIDR] [--sport 端口范围] \
> [-d 目标 IP 地址/CIDR] [--dport 端口范围] -j [ACCEPT|DROP|REJECT]
```

按状态放行数据包：

```sh
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
```

## NAT

一切的一切不要忘了打开 IP Forwarding!

修改 `/etc/sysctl.conf`

把 `net.ipv4.ip_forward` 改为 `1` 并重启。

结果应为：

```sh
$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```

iptable NAT 路径

- 先经过 NAT Table 的 PREROUTING Key
- 如果包不进入本机，则继续处理
- 再经过 Filter Table 的 FORWARD Key
- 最后经过 NAT Table 的 POSTROUTING Key，再发送出去

要将内网流量通过 NAT 转发至 WAN，即将内网的 RFC1918 私有地址转换为公网地址，需要使用 Source NAT

Source NAT 使用 POSTROUTING Key

```sh
$ iptables -t nat -A POSTROUTING -s $INNET -o $EXTIF -j MASQUERADE
```

其中 `$INNET` 为 NAT 后的内部 IP 地址，`$EXTIF` 为 NAT 后的出口 IP 地址或网卡名。

将内网的服务映射到 WAN，即做端口映射，需要配置 Destination NAT。

DNAT 使用 PREROUTING Key

例如，将 Web 服务的 80 端口转发到内网的 192.168.100.10 的 80 端口上：

```sh
$ iptables -t nat -A PREROUTING -i $EXTIF -p tcp --dport 80 -j DNAT --to-destination 192.168.100.10:80
```

查看防火墙规则使用 `iptables -L`

清除防火墙规则使用 `iptables -F`

不要忘记适当时候放行 ICMP 协议数据包！

如果想禁 ping 的话，最好不要禁掉所有 ICMP 数据包，而仅 DROP 掉 ICMP type 8.

例如：

```sh
$ iptables -A INPUT -i ens33 -p icmp --icmp-type 8 -j DROP
```

由于 iptables 需要按顺序写入，为减小重复调试的工作量，强烈建议写一个 Shell 脚本，在脚本中修改 iptables 规则。

虽然完全可以插入规则，但是删掉重写比较。。。少记一点点东西。

```sh
#!/bin/bash

# A Shell Script for editing iptables
# Written by Xiao Pan 2021

# Clear iptables
# Delete all User-defined roles,chains and zones
iptables -F
iptables -X
iptables -Z

# Set default policy
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT

# Set Rules
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -i ens33 -m state --state RELATED,ESTABLISHED -j ACCEPT
# 我知道你们这些中文玩家大概率记不住这单词，如果忘掉请 "man iptables-extensions"
# Uncommunt this if you want to block ping
# iptables -A INPUT -i ens33 -p icmp --icmp-type 8 -j DROP

iptables -A INPUT -i ens33 -p icmp -j ACCEPT
# Add Accept rules
# iptables -A INPUT -i ens33 -s 192.168.1.0/24 -j ACCEPT

# Set NAT

EXTIF="ens192"
INIFONE="ens224"
INNETONE="192.168.100.0/24"
INITTWO="ens256"
INNETTWO="192.168.200.0/24"

# Clear NAT table rules

iptables -F -t nat
iptables -X -t nat
iptables -Z -t nat
iptables -t nat -P PREROUTING ACCEPT
iptables -t nat -P POSTROUTING ACCEPT
iptables -t nat -P OUTPUT ACCEPT

# Enable Source NAT
if ["$INIFONE" != "" ]; then
    iptables -t nat -A POSTROUTING -s $INNETONE -o $EXTIF -j MASQUERADE
fi
# Like this
# 我知道这个你们大概率还是记不住，请 "man iptables-extensions" 后搜索 “SNAT” 全大写，或者搜索 “TARGET” 在 TARGET EXTENSIONS 中你们也会找得到的。
# 注意：这个只适合用在动态 IP 地址的场合，其他情况依然推荐使用传统的 SNAT 即：

iptables -t nat -A POSTROUTING -s $INNETONE -o $EXTIF -j SNAT --to-source 123.186.228.222

# Add DNAT Rules
# Such as Web service on 192.168.100.10:80

iptables -t nat -A PREROUTING -p tcp -i $EXTIF --dport 80 -j DNAT --to-destination 192.168.100.10:80
# Like this

# Save this, and run "bash ./firewall.sh"
# or "chmod +x ./firewall.sh && ./firewall.sh
# Run As ROOT!
```

# 保存 iptables 配置

## 方法一

导出配置使用 `iptables-save`，直接运行即可将配置输出至标准输出

将文件导出并保存后，编辑 `/etc/network/if-pre-up.d/iptables`

加入以下内容

```sh
#!/bin/sh
# EDIT THIS!!!
RULES="/path/to/rules/file"
/sbin/iptables-restore < "$RULES"
```

给这个脚本加上运行权限：

```sh
$ chmod +x /etc/network/if-pre-up.d/iptables
```
  
## 方法二

安装 `iptables-persistent`，后运行：

```sh
$ iptables-save > /etc/iptables/rules.v4
```


这个也可以稍微看下：

> (iptables防火墙的应用和SNAT/DNAT策略)[https://zhuanlan.zhihu.com/p/26325389]