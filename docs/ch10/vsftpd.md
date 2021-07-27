`apt install vsftpd`

虚拟用户安装 `libpam-pwdfile`

阻止文件使用　`deny_file={*.doc,*.docx,*.xlsx}`

可以设置 local 用户登录，直接创建账户，将家目录指定为 ftp 要访问的目录即可

将 `chroot_local_user` 行取消注释，即可让用户被限制在家目录