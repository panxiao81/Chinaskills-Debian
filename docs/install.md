早期的 Debian 安装会比较麻烦，而当前时代的 Debian 引入了 GUI 安装向导，使得安装变得非常容易且轻松。

多数情况下我们需要在没有操作系统的主机上安装 Debian，因此本文也仅演示这种情况下的操作。

安装 Debian 主要分几个步骤：

- 准备安装介质
- 确认引导方式
- 引导并进行安装
- 安装后配置

# 准备安装介质

安装前，我们需要准备好 Debian 的安装 ISO，Debian 的安装镜像可从 [Debian.org](https://www.debian.org/CD/http-ftp/) 下载。

官方下载因为国内网络原因比较慢，可以使用国内镜像下载。

源内有多个镜像，我们只需下载 DVD1 即可。

Debian 支持多种 CPU 平台，我们常用的 `x86_64` 架构的电脑需要下载 `amd64` 镜像。 

因此，这一次，我们下载后准备好的镜像文件名为 `debian-10.6.0-amd64-DVD-1.iso`。

对于虚拟机，以及服务器设备支持远程管理的情况下，允许我们直接使用 ISO 文件安装，而如果需要在物理机上安装，则需要将镜像写进 U 盘。

在 Linux 环境中可以直接用 `dd` 命令写 U 盘。

而在 Windows 中，可以使用 `refus` 来写 U 盘。

`refus` 的下载地址在[这里](http://rufus.ie/)

下载镜像后直接写入即可。

# 确认引导方式

安装前，我们需要确认计算机的引导方式。

引导方式分两种，分别为：

- 传统 BIOS 方式
- UEFI 方式

两种不同的引导方式需要了解不同的分区方式。

# 引导并进行安装

当安装介质准备完毕后，就可以正式开始安装了。

本例中将在 VMware 虚拟化环境下使用 UEFI 引导模式进行安装演示，其他环境均大同小异。

VMware 虚拟化环境可以看作一台完整的电脑，因此安装系统与真机并无太大差异。

当在虚拟机中挂载 ISO 进入虚拟光驱，并设定从光驱启动后，会进入 Debian 安装程序的引导菜单。

![图 1](images/install/d32cabe8cbbfaed0ce3928cabcd4dca44fff436cffc9fbb56a930fabdf45df92.png)  

这些选项分别代表：

- 图形化安装
- 安装 ( 指使用文本界面安装 )
- 高级选项
- 轻松访问选项菜单 ( 为视障人士准备 )
- 带有朗读合成器的安装 ( 为视障人士准备 )

在菜单中，按 `Enter` 键选择确认。

为方便使用，本例使用图形化方式安装，文本安装与图形化安装大同小异，但更节省系统资源。

选择第一项 `Graphical Install` 并按 `Enter` 键，在一系列系统引导过程后，会进入 Debian 安装器应用。

![图 2](images/install/ea40b08840c1b9e82c6dd7828d02b65360cca5493be02f9b10144d9e3f16aa46.png)  

第一步需要选择语言，此处的选择也会影响安装系统后的默认语言，笔者比较推荐使用英语，这样不会导致平时使用时可能会出现的终端乱码问题，当然如果英语真的比较差，也可以选择中文。

本例中将选择英语进行安装。

![图 3](images/install/fd8921e503128fc49e61755f154107710f5d0a77f2f870f7731f58fc76156c3c.png)  

这一步选择区域，我们选择中国，但这个列表中不会直接列出 `China`,需要寻找 `Other - Asia - China`

![图 4](images/install/f165fd1ac67183374999a268a50338989644ee819eb543a1baa47cf3c6c6e4dd.png)  

由于中文区并没有英语，所以会有一个手动指定编码的选项，保持默认的 `en-US.UTF-8` 即可，

![图 5](images/install/7e35d29a9665f767a826153ccf42067e7c3f2fcc2d1dd6e280ecf301dec20e03.png)  

选择键盘布局，这里保持默认的美式英语布局。

接下来安装向导会对光盘进行挂载并扫描光盘内的内容，扫描完成之后继续让我们输入配置。

![图 6](images/install/5a224791faa125256fda08425020d34d5d78183a6dd623eaadb04cee4d08ffd9.png)  

配置主机名，主机名可在系统安装完成后再修改，这里可以保持默认。

![图 7](images/install/465628f2d8994f4ea1207add805e245115301161b8dcaa07bf5d6eafb661a7f4.png)  

配置域名，这一步也可在安装完之后修改，保持默认为空。

![图 8](images/install/01b0d4c5959690a9d125b9485841bc22f99ae70824b8b9695233320b0309d95a.png)  

接下来配置 `root` 用户密码，如果此处选择留空，则 `root` 账户将默认被禁用。

即便 `root` 账户被禁用的情况下，普通用户也可使用 `sudo` 命令进行临时提权，这是 Debian 的推荐使用方式。

输入完成并输入确认密码后点击下一步。

![图 9](images/install/eda15230813f9cb4bc82bbd84355f3003ffc646efc5c407f5a1e9c509758c66f.png)  

接下来的步骤将创建一个普通用户。

这几步分别对应输入用户全名，输入用户名，以及设置密码。

接下来安装向导将寻找 NTP 服务器并自动配置时间。

![图 10](images/install/aa2a846ad450e0ab9875eedfac87ac8230539e2b0ffadf6d391f87f0f1b007ea.png)  

这一步将对硬盘进行分区。

这几个选项分别对应 ：

- 向导 - 使用整块硬盘
- 向导 - 使用整块硬盘并设置 LVM
- 向导 - 使用整块硬盘并设置加密的 LVM
- 手动配置

如无特殊需求，此处选择第一项即可。

![图 11](images/install/2c82596e2da4892702017e28707d4f549d823b2a32b870150ceb2aa66945cb64.png)  

选择安装硬盘，这台虚拟机只有一块硬盘，直接点击 `Continue`

![图 12](images/install/cf2fc06da95cd8eb63cb2ed45e788cdd960173d1f3b9b397b7a8f78b414b45fb.png)  

此处安装向导给出了三种分区方式，分别为：

- 所有文件存储在单个分区中 ( 推荐新用户使用 )
- 单独分出 `/home` 分区
- 单独分出 `/home`, `/var` 和 `/tmp` 分区

无特殊需求的话，可以直接选择第一项。

磁盘管理会在后续有专门一章节进行讲解。

![图 13](images/install/e3e4008e5efdc65c04c52035da6d41fd701707301d0549d3d40c2686d25c08e8.png)  

此处安装向导让我们确认磁盘分区情况，并默认选择 *完成分区并写入磁盘* 。

确认无误之后即可点击 `Continue`

![图 14](images/install/6ac304bb6e4074660d7e4d0ce4b70b2aba9e0c25b5d73864c1e4d6614c096d23.png)  

此处会再次确认，点击 `Continue`

![图 15](images/install/7c9c6f503a7ed5cab320a687eb9c0119baa39110e0792694bcd203b86a4f253c.png)  

安装向导开始安装系统基础组件。

![图 16](images/install/ada6a0b390a915a7f87fff0e14265a4725a6ef4e1b09c9c456e70c5d6c80da8e.png)  

这一步安装向导提示我们是否要扫描其他 CD/DVD 作为本地源使用，我们只准备了一张光盘，并且多数软件包会在安装完成后通过网络安装，因此此处选择 `No` 并继续。

![图 17](images/install/657a57531e02595ded6612431eb6b0f72359a00c32ced4315ccb939ab7bd2054.png)  

这一步会提示我们配置源镜像，由于国内访问官方源速度较慢，虽然可以在系统安装完毕后修改镜像源，但也可以在安装时直接修改，此处选择 `Yes`

选择中国后，会跳出源镜像选择列表

![图 18](images/install/6a60f4411e44e83f5e8a63bb1c09d1bf5020757ebdc6e2a2b3535c50d6a4e212.png)  

此处可以选择清华源或 `163` 源，华为云源镜像等，此处选择配置 `163` 源。

![图 19](images/install/f657166566da10ca2d257ed6c445e9df951da33e980b758a2f21d9c5224f2805.png)  

此处配置 HTTP 代理，如需要使用代理连接外部网络，则在此处配置，否则留空并下一步。

接下来安装向导会配置源与包管理器。

![图 21](images/install/5b6361fda704339b07628caf052a742cbb75aa5630c204126cf2b20065ea36cf.png)  

提示是否同意发送数据给 Debian 社区来改进系统，这里可以根据想法选择。

![图 22](images/install/8f2902a27394a82cc3b13381f57abc535a7269cb453b01c5df0b3f1ec3955a6f.png)  

此步骤选择安装组件，为接下来的章节方便，将不安装除了系统工具与 SSH 服务器以外的任何包，因此将取消选择 `Debian Desktop environment` 与 `print server`

点击 `Continue` ，安装向导将继续安装选择的选项。

![图 23](images/install/c912566658f3357730a214a82d56c09d60fdb4ebabafc641b666c5ce2c4e99cf.png)  

安装完成，提示移除安装媒体并重新启动。

到这里 Debian 的安装就完成了。

> [官方的 Debian 安装手册](https://www.debian.org/releases/stable/amd64/index.zh-cn.html)

# 安装后配置

在安装向导没有进行配置的项目，均可在安装完成后再进行修改。

这些都将在以后的章节中讲到。

![图 24](images/install/b5a14f45105d44f60d23be0b067d2004cb5e42cc55adc5dba0d48c439447d134.png)  

安装后，如果像我一样没有安装 GUI 的话，将进入 TTY 模式。

可以使用安装时设定的 `root` 账户或创建的普通用户登录。

安装完成后，通常会做几件事情。

- 配置网络
- 配置远程访问
- 如需要，修改主机名
- 确认时间与时区
- 确认源并安装常用软件包

这些将会在下一章节详细解说。

但就目前而言，为了方便后续远程连接，可以先配置允许 `root` 用户进行远程访问。

方法是修改 `/etc/ssh/sshd_config`, 在 32 行的

```bash
#PermitRootLogin prohibit-password
```

修改为：

```bash
PermitRootLogin yes
```

之后可以通过 SSH 远程登陆，方便后续实验。

> 注意，实际应用中应尽量避免使用 `root` 用户