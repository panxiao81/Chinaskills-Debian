# SSH 密钥对

SSH 不仅支持密码认证，同时支持使用密钥对认证，这种认证方式使用非对称加密，比起密码方式登录更加安全，且无需每次登录时输入密码。

使用 `ssh-keygen` 工具即可生成一对密钥对，默认保存在 `~/.ssh/` 目录下，可发现 `id_rsa` 文件与 `id_rsa.pub` 文件。

其中 `id_rsa` 为私钥，需妥善保存，`id_rsa.pub` 为公钥，接下来将会把公钥传送到要访问的主机上。

SSH 提供了一个工具名为 `ssh-copy-id` 可以方便的完成传输公钥到对端主机的工作。

例如 访问 192.168.100.10

```console
$ ssh-copy-id -P 22 username@192.168.100.10
$ # 正常 SSH 即可，现在不需要密码
```

这就是所谓的免密登录

若要禁用密码登录，修改 `/etc/ssh/sshd_config`

查找 `PasswordAuthentication` 修改为 no 后重启 `ssh` 服务。
