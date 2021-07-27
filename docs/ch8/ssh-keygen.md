```sh
# 例如 访问 192.168.100.10
$ ssh-keygen
$ ssh-copy-id username@192.168.100.10
# 正常 SSH 即可，现在不需要密码
```

若要禁用密码登录，修改 `/etc/ssh/sshd_config`

查找 `PasswordAuthentication` 修改为 no 后重启 `sshd` 服务