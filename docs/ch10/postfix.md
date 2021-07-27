`postfix` 是一个配置简单的 SMTP 邮件服务器。

Debian 还提供了另一个强大的邮件服务器： `Exim4` 并且默认安装且推荐使用。

> [[教學] Mail Server 架設與設定指南-Postfix](https://xenby.com/b/260-%E6%95%99%E5%AD%B8-mail-server-%E6%9E%B6%E8%A8%AD%E8%88%87%E8%A8%AD%E5%AE%9A%E6%8C%87%E5%8D%97-postfix)

如果有接入数据库或 LDAP 的需求，则需要安装其他模块，例如 `postfix-ldap` 等。

默认只开放 25 端口，但实际上是通过 STARTLS 加密传输的，需要额外指定开放 465 端口

> [[Postfix] SMTP Mail Server 架設教學指南 – Postfix with Ubuntu](https://code.yidas.com/mail-server-postfix-with-ubuntu/)

安装 `postfix` 包即可

安装时会弹出配置向导，配置 Internet Site 即可

第一步填入域名，一般的邮箱为 `user@example.com` 的格式，则这里应填写 `example.com`

首先配置 DNS。

填入如下解析，本例为 `sdskills.cn`

```sh
@       IN  MX      10 server01.sdskills.cn.
smtp    IN  CNAME   server01.sdskills.cn.
imap    IN  CNAME   server01.sdskills.cn.
```

并给 `smtp.sdskills.cn` 与 `imap.sdskills.cn.` 签发两张证书，稍后会使用

配置 postfix 配置文件

有两个配置文件要进行配置，分别为 `main.cf` 与 `master.cf`

首先配置 `main.cf`

`mynetworks` 填入局域网 IP 地址段，修改 SSL 证书路径

配置 `master.cf`,取消注释 smtps 的行，并取消注释下面的两行

重新加载 `postfix` 配置

```sh
$ systemctl reload postfix
```

对于 dovecot

修改 `10-auth.conf` 在 `auth_mechanisms` 中添加 `login`

在 `10-master.conf` 中，找到 `inet_listener imaps`,将端口与 `ssl` 一行取消注释

在 `postfix` 的 `master.conf` 中注释 `smtp` 行即可停止监听 25 端口，dovecot 则要将 `imap` 监听端口从 `143` 改为 `0`