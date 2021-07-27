`sshd` 服务的配置文件为 `/etc/ssh/sshd_config`

其中常用到的配置项目为：

- Ports 配置端口
- ListenAddress 配置在哪个网络监听
- PermitRootLogin 是否允许 `root` 账户登录
- AllowUsers 允许登录的用户与主机

还有一些功能是否开放的项目不再一一介绍

ListenAddress 请配置本机网卡接口，注意 **监听** 地址与客户端地址的区别

PermitRootLogin 默认设定为不允许 `root` 用户使用密码登录，如需要使用密码登录请改为 no

AllowUsers 可填入主机，形如 `root@10.10.10.51`，意为允许 root 账户从 10.10.10.51 登录，也可只写用户，意为允许用户从任何地方登录。

类似的，也存在 `DenyUsers` 的设置项目，但二者只能用一个。

如想禁止一部分地址登录，除了使用 `sshd` 配置文件外，也可配合使用 `/etc/hosts.allow` 与 `/etc/hosts.deny` 他们分别代表允许的主机与禁止的主机。

`hosts.allow` 与 `hosts.deny` 使用的是系统底层的 TCP wrapper，类似于防火墙，因此他不但能对 SSH 进行限制，理论上可以限制很多服务。

语法规则比较特殊，贴上 `man` 手册：

- man 5 hosts_options
- man 5 hosts_access

规则基本格式为：

```
daemon, daemon, ...: client, client, ...: option
```

- daemon 代表要监控的服务，可用逗号隔开，或用 ALL 代表所有
- client 代表要登录的客户端主机，可为主机名，IP/CIDR，单独的 IP 或域名，也可用 ALL 代表所有任意客户端
- 有 `allow`，`deny`，`except` 代表允许，拒绝以及除去，`except` 像这么写：

```
sshd:ALL except 10.10.10.51：deny
```

意为：除了 10.10.10.51 以外，所有其他主机禁止访问 sshd 服务。

