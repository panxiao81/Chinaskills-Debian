APT 可能是 Linux 中最早的包管理器了。

虽然在当下与 `dnf` ( RHEL 8 开始使用的包管理器 ) 和 `pacman` ( Arch 系发行版使用的包管理器 ) 相比，并没有显示出明显的优势，但软件源的数量却可以认为非常多。

APT 软件源将写进 `/etc/apt/sources.list` 文件中，也可以在 `/etc/apt/sources.list.d/` 中创建结尾为 `.list` 的文件并写入。

之后即可使用一系列命令安装与管理软件包。

# 写入软件源

`/etc/apt/sources.list` 的语法格式如下：

```sh
deb url distribution component1 component2 component3 [..] componentX
deb-src url distribution component1 component2 component3 [..] componentX
```

---

- deb
  - 表示此软件源存放二进制软件包
- deb-src
  - 表示此软件源存放源码包

实际使用时，可以不写 `deb-src` 可加快软件源更新速度。

---

`url` 填写软件源目标地址

软件源支持许多网络协议：

- http ( s ) ://
- file://
- ftp ( s ) ://
- cdrom:

---

`distribution` 指当前版本内部代码，可用 `lsb_release -sc` 查看：

```sh
$ lsb_release -sc
buster
```

---

由于各种各样的原因，Debian 官方软件源将软件包分割成了多个分支，如需要完整的所有软件，则需要提供多个分支。

Debian 官方软件源的分支有：

- main
- contrib
- non-free

其他第三方软件源也可能存在多个分支。

---

如上，一个安装完成后，提供在线软件源的 `/etc/apt/sources.list` 应如下：

```sh
# Security updates
deb http://security.debian.org/ buster/updates main contrib non-free
deb-src http://security.debian.org/ buster/updates main contrib non-free

## Debian mirror

# Base repository
deb https://deb.debian.org/debian buster main contrib non-free
deb-src https://deb.debian.org/debian buster main contrib non-free

# Stable updates
deb https://deb.debian.org/debian buster-updates main contrib non-free
deb-src https://deb.debian.org/debian buster-updates main contrib non-free

# Stable backports
deb https://deb.debian.org/debian buster-backports main contrib non-free
deb-src https://deb.debian.org/debian buster-backports main contrib non-free
```

如果官方地址从国内访问速度太慢，可以搜索并添加国内镜像源，替换以上的 `URL` 即可。

如要使用光盘作为软件源，通过一定方式挂载光盘到 `/mnt` 后，使用 `sudo apt-cdrom add` 命令，将自动添加光盘为软件源。

在添加软件源或一段时间后，每次更新系统或安装软件包之前，须更新 APT 缓存：

```sh
$ sudo apt update
```

在添加第三方软件源时，可能还需要添加第三方源的密钥，可使用 `apt-key` 添加。

例如，如下命令将下载 `Docker` 官方软件源的密钥，并通过管道符传给 `apt-key` 命令添加：

```sh
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add - 
```

# 管理软件包

APT 可轻松管理软件包的安装，卸载，升级，以及对整个发行版的版本号进行升级。

虽然多数时候并不推荐直接升级发行版版本，尤其是跨大版本升级。

APT 的常用子命令有：

- update 更新缓存
- upgrade 升级现有软件包
- install 安装软件包
- remove 卸载软件包 ( 保留配置 )
- purge 完全卸载软件包 ( 清除配置 )
- autoremove 自动卸载未使用的软件包 ( 当系统中存在未使用 APT 安装并管理的软件包时慎用！ )
- search ( 来自 `apt-cache` ) 搜索软件包

下面的例子将更新 APT 软件源缓存并为我们的实验环境虚拟机安装 `open-vm-tools` 工具：

```sh
$ sudo apt update && sudo apt install open-vm-tools
```