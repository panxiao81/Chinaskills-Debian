包名: `bind9` 

文档包名: `bind9-doc`

系统内文档: `/usr/share/doc/bind9/README.Debian.gz`

配置文件及 `localhost` 的正向与反向解析保存在 `/etc/bind/` 

当前默认工作目录为 `/var/cache/bind`

官方建议将数据库存放在 `/etc/bind/` 并在 `named.conf` 中使用绝对路径

当前 bind 默认以 `bind` 用户与组运行，注意权限问题。

> [How To Configure BIND as a Private Network DNS Server on Debian 9](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-debian-9)