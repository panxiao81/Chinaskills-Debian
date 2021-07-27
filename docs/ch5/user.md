Debian 在用户配置上与其他发行版无任何区别。

但因 Linux 使用时最好遵守的最小权限法则，通常要求维护与守护进程均不要使用 `root` 账户运行。

因此我们需要创建多个账户，并在临时需要管理员权限时使用 `sudo` 命令进行临时提权。

`sudo` 会在后续章节讲述，而当前章节则会重复唠叨一些 Linux 用户都知道的知识。

用户的清单均存储在 `/etc/passwd` 文件中，`/etc/shadow` 存储了用户的密码，群组的数据则存储在 `/etc/group` 中

尽管完全可以通过直接编辑文件来达到添加与删除的目的，但更推荐的方式是使用命令行工具。例如：

```shell
# 创建一个名为 debian 的用户
useradd debian
# 创建一个禁止登录的名为 apache 的用户
useradd -s /sbin/nologin apache
# 创建一个 admin 用户，并指定其附加组具有 sudo 组
useradd -G sudo admin
# 完全删除用户 ( 包括用户的家目录与邮箱目录 )
userdel -r debian
```

之后对用户设置密码即可让用户正常登录

```shell
# 交互式设定
passwd debian
# 或者一条命令
echo passwordhere | passwd debian --stdin
```
