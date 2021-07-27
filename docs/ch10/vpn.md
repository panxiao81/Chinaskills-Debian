只用到 OpenVPN，不多讲。

别一说到 VPN 就想到某逃脱网络审查手段，好东西就是被你们这些人玩坏的。

分 Server 端和 Client 端。

官网文档放这：[2x How To](https://openvpn.net/community-resources/how-to/)

以及: [Static Key Mini-HOWTO](https://openvpn.net/community-resources/static-key-mini-howto/)

访问这个需要科学上网手段，自求多福。

还有 Debian 的官方文档 [虚拟专用网络](https://www.debian.org/doc/manuals/debian-handbook/sect.virtual-private-network.zh-cn.html)

本例将使用最简单的，使用 Static Key 验证的 Site-To-Site VPN 隧道。

在服务器端，生成 Static Key

```sh
$ openvpn --genkey --secret static.key
$ cp static.key /etc/openvpn
```

配置文件只有三行，所以没必要使用示例配置文件了，直接写就好

```sh
$ vim /etc/openvpn/server.conf
```

```ini
dev tun
ifconfig 10.8.0.1 10.8.0.2
secret static.key
```

启动服务：

```sh
$ systemctl start openvpn@server
```

客户端这一侧，需要将服务器端的 Static Key 拷贝过来

```sh
$ scp root@192.168.10.3:/root/secret.key /etc/openvpn/static.key
$ vim /etc/openvpn/SERVER03.conf
```

```ini
remote 192.168.10.3 # 填远端地址或 FQDN
dev tun
ifconfig 10.0.8.2 10.0.8.1
secret static.key
```

```sh
$ systemctl start openvpn@SERVER03
```

ping 对端检查 VPN 隧道是否建立正常

```sh
# 在 Client 上
$ ping 10.0.8.1
```
