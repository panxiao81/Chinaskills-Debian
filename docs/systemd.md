Systemd 是当前主流的 Linux 系统管理工具，他负责启动其他系统程序，启动系统守护进程，挂载磁盘，监视进程，系统日志，维护登录用户列表，可以简单的管理网络，时间同步，日志转发等。

过去，Linux 继承 Unix，使用 `init`，启动与管理守护进程类似如下：

```sh
$ service smb start
# 或者
$ sudo /etc/init.d/apache2 start
```

`init` 脚本非常复杂，且 `init` 只能进行串行启动，启动时间较长。

但 Systemd 过于复杂，且由于各种各样的原因，现在仍有一些人反对使用 Systemd。

当前的主流发行版无论是 RHEL 或是 Debian 系，甚至 Arch 系均默认使用 Systemd。

本文将讲解如何使用 Systemd 提供的管理工具管理守护进程，设定计划任务，查看并管理日志等内容。

> 本节参考：[ArchWiki:systemd](https://wiki.archlinux.org/index.php/Systemd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) 