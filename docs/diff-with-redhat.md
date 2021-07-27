如果你有使用过 `Red Hat` 一类的 Linux 发行版的经验话 ( 包括 `CentOS` 等 )，实际上 `Debian` 与 `Red Hat` 并无非常巨大的区别，毕竟他们都只是一种普通的 Linux 发行版而已。

多数时候，我们使用 Linux 是希望使用这个生态环境下的服务 ( 如 `apache`, `docker` 等 )，这些服务的配置无论是在 `Debian` 下抑或是 `Red Hat` 系的发行版下并无太大区别。

由于底层内核一致 ( `Debian` 多数时候使用 Linux 内核，`Red Hat` 也为 Linux 内核 )，软件包大多也一致，因此两者其实并不存在技术上的差别。

实际上，不同的发行版只是集成了不同的软件包，使用系统时我们使用的也只是这些软件包。

但，`Debian` 与 `Red Hat` 最大的区别在于他们的软件包管理方式。

| 项目 | `Debian` | `Red Hat` |
|-|-|-|
| 二进制软件包 | `deb` | `rpm` |
| 包管理程序 | `dpkg` | `rpm` |
| 软件仓库 | `apt` | `yum` ( RHEL 7 之前 ) <br> `dnf` ( RHEL 8 之后 ) |

除此以外， `Debian` 与 `Red Hat` 还存在一些细微差别

| 项目 | `Debian` | `Red Hat` |
|-|-|-|
| 维护方 | 社区 | 企业 ( 由 Red Hat 公司维护 ) |
| 查看系统版本 | `/etc/debian_version` <br> ( 但多数时候使用 `lsb_release` ) | `/etc/redhat-release` |
| SELinux 支持 | 支持，但默认不安装 | 支持，且默认安装并启用强制模式 |
| 安全性 | 默认启用 `AppArmor` | 使用 `SELinux` |
| 官方维护软件包数量 | 多 | 少 ( 不使用 `EPEL` 或 `ELRepo` 等社区维护源 ) |
| 软件包更新速度 | 快 (对于 `unstable` 与 `test` 版本 ) <br> 慢 (对于 `stable` 版本) | 非常慢 (多数时候需跟随大版本更新) |
| 技术支持 | 无 ( 社区支持 ) | 有技术支持服务 ( 由 Red Hat 公司提供技术支持 ) |
| 维护时长 | 3 年 ( 对于 `unstable` 与 `test` 版本 ) <br> 5 年 ( 对于 `stable` 版本 ) | 10 年 ( 包含 LTS 支持 ) |

对于技术支持部分，如果有这种需求也可以考虑由 `Canonical` 公司支持并维护的 [`Ubuntu`](https://ubuntu.com/) ( 一个基于 `Debian` 的 `unstable` 分支的 Linux 发行版 )，并且 `Ubuntu` 也提供长达 10 年的支持。

本书的实验环境与例子均使用 `Debian 10.6.0` ，但多数操作也应可以在其他基于 `Debian` 的发行版上完成 ( 如 [`Ubuntu`](https://ubuntu.com/), [`Linux Mint`](https://linuxmint.com/), [`Kali Linux`](https://kali.org/) 等 )

