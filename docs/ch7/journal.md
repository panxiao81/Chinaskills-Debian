# Journal

传统的系统日志通常使用 `Syslog`，Systemd 使用 Journal 统一管理系统日志。

Journal 的日志是二进制形式保存在硬盘上的，这点让很多人反对使用 Journal，尤其是当系统宕机时，无法方便的查看出错日志。但 Journal 仍可以配合 Syslog 使用，可将 Journal 的日志转发至 Syslog，之后用喜欢的方式进行处理。

## 确保采用日志永久存储

默认情况下，Journal 会将日志保存在 `/var/log/journal`，但前提是这个目录存在，如果此目录不存在，则 systemd 不会自动创建该文件夹，并将日志输出至 `/run/systemd/journal`，此目录中的内容会在每次开机时被清空。

### 重建日志保存目录

重建目录应确保目录正确，目录权限正确。

```console
$ sudo mkdir -p /var/log/journal
$ sudo chown -R :systemd-journal /var/log/journal
$ sudo chmod 2775 /var/log/journal
$ sudo systemctl restart systemd-journald.service
```

## 查看日志

查看 Journal 的日志须使用 `journalctl` 命令。

如不加任何参数，则输出所有日志，但一般我们需要对日志进行过滤，`journalctl` 提供了许多过滤日志的选项，但最常用的是输出特定 systemd 单元的日志信息。

```console
$ journalctl -eu ntp
Jan 21 03:32:07 debian ntpd[3783]: Soliciting pool server 193.182.111.142
Jan 21 03:32:18 debian ntpd[3783]: Soliciting pool server 144.76.76.107
Jan 21 03:32:39 debian ntpd[3783]: Soliciting pool server 94.237.64.20
Jan 21 03:32:57 debian ntpd[3783]: Soliciting pool server 120.25.115.20
Jan 21 03:32:58 debian ntpd[3783]: Soliciting pool server 84.16.73.33
Jan 21 03:33:14 debian ntpd[3783]: Soliciting pool server 185.255.55.20
Jan 21 03:33:24 debian ntpd[3783]: Soliciting pool server 193.182.111.14
Jan 21 03:33:43 debian ntpd[3783]: Soliciting pool server 193.182.111.12
Jan 21 03:34:04 debian ntpd[3783]: Soliciting pool server 185.255.55.20
Jan 21 03:34:19 debian ntpd[3783]: Soliciting pool server 193.182.111.142
Jan 21 03:34:28 debian ntpd[3783]: Soliciting pool server 144.76.76.107
Jan 21 03:34:48 debian ntpd[3783]: Soliciting pool server 5.79.108.34
Jan 21 03:35:10 debian ntpd[3783]: Soliciting pool server 162.159.200.123
Jan 21 03:39:09 debian ntpd[3783]: 122.117.253.246 local addr 192.168.50.157 -> <null>
Jan 21 22:25:42 debian ntpd[3783]: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): expired less than 25 days
Jan 22 22:25:42 debian ntpd[3783]: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): expired less than 26 days
Jan 23 22:25:42 debian ntpd[3783]: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): expired less than 27 days
Jan 24 22:25:42 debian ntpd[3783]: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): expired less than 28 days
Jan 25 22:25:42 debian ntpd[3783]: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): expired less than 29 days
Jan 26 22:25:42 debian ntpd[3783]: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): expired less than 30 days
Jan 27 22:25:42 debian ntpd[3783]: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): expired less than 31 days
Jan 28 22:25:42 debian ntpd[3783]: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): expired less than 32 days
Jan 29 22:25:42 debian ntpd[3783]: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): expired less than 33 days
Jan 30 22:25:42 debian ntpd[3783]: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): expired less than 34 days
Jan 31 22:25:42 debian ntpd[3783]: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): expired less than 35 days
Feb 01 22:25:42 debian ntpd[3783]: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): expired less than 36 days
Feb 02 22:25:42 debian ntpd[3783]: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): expired less than 37 days
Feb 03 22:25:42 debian ntpd[3783]: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): expired less than 38 days
Feb 04 22:25:42 debian ntpd[3783]: leapsecond file ('/usr/share/zoneinfo/leap-seconds.list'): expired less than 39 days
```

常用参数表：

- **`-f`**  只显示最近的日志信息，另外，当新的日志被写入时会自动显示 ( 类似于 `tail` )
- **`-e`**  显示信息并跳转到日志结尾，可直接看到最近的几条日志
- **`-r`**  以反向顺序显示日志
- **`-k`**  只显示内核信息
- **`-u`**  只显示指定的 systemd 单元信息

## 配置文件相关

对 Journald 的更多配置须修改 `/etc/systemd/journald.conf`，可查看 `man 5 journald.conf` 获取完整信息，每次修改配置文件须重启服务：

```console
$ sudo systemctl restart systemd-journald
```

### 限制日志保存大小

默认情况下，Journald 将日志最大保存容量设定为整个磁盘分区的 10%，但上限为 4GB ，如需要改变这一项，则取消注释并修改配置文件中的 `SystemMaxUse` 选项，例如要限制最多使用 50MB，则设定为：

```sh
SystemMaxUse=50M
```

### 将日志转发至 Syslog

Journald 可以与 Syslog 共存且同时工作，并且向下兼容。

要将 Journald 收集的日志转发至 Syslog，首先保证系统安装且启动了 `rsyslog`，后修改配置文件。

```sh
# 安装 rsyslog
$ sudo apt install rsyslog
# 启动 rsyslog 服务
$ sudo systemctl enable --now rsyslog
# 修改配置文件
$ sudo vi /etc/systemd/journald.conf
```

取消注释并修改下面一行

```sh
ForwardToSyslog=yes
```

别忘了重启 `systemd-journald.service`

```console
$ sudo systemctl restart systemd-journald
```

> 阅读资料：
> 
> [journalctl：查询 systemd 日记](https://documentation.suse.com/zh-cn/sles/15-SP2/html/SLES-all/cha-journalctl.html)
>
> [systemd (简体中文)/Journal (简体中文)](https://wiki.archlinux.org/index.php/Systemd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)/Journal_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))
