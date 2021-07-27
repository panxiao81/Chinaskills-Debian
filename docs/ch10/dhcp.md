包名：`isc-dhcp-server`

配置文件有：

- `/etc/default/isc-dhcp-server`
- `/etc/dhcp/dhcpd.conf`

`/etc/default/isc-dhcp-server` 需要增加监听网卡名，使用 `ip link` 查看。

`IPv6` 部分用不到也不必注释

`/etc/dhcp/dhcpd.conf` 与 Red Hat 类发行版无区别，文件中也包含配置实例。

起不来服务重启系统再试。