在 Debian 这类的 Unix-like 系统里，文件以树状的文件夹阶层组织。`/` 文件夹称为 “根文件夹”；其他的文件夹都是此根文件夹的子文件夹。“挂载” 是把周边设备 (通常是磁盘) 纳入系统文件树的工作。例如以其他磁盘保存用户个人的数据，将 “挂载” 于 `/home/` 文件夹。根文件系统由系统内核永远挂载于根；其他设备可则稍后再通过启动顺序或以 `mount` 命令挂载进来。

一些可移动式设备 ( 如 U 盘，移动硬盘等 )，会在连接上时被自动挂载，尤其是当使用 `GNOME` 一类的桌面环境时，其他情况下则需要用户手动挂载。同样，既然存在挂载，也就存在卸载 ( 从文件树中移除 )，普通用户通常没有执行 `mount` 和 `umount` 命令的权限，管理员用户才可以。但是在 `/etc/fstab` 中可以通过 `user` 选项指定具体的某个挂载点具有操作权限。

`mount` 命令不指定任何参数会显示出当前的挂载点列表，如要只显示 `/etc/fstab` 文件中存在的挂载点则使用 `findmnt --fstab` 命令。对于简单的情况，要挂载一个磁盘，例如 `/dev/sdc1` 使用 `ext3` 文件系统，挂载到 `/mnt/tmp` 文件夹下，命令为:

```sh
$ sudo mount -t ext3 /dev/sdc1 /mnt/tmp
```

`/etc/fstab` 文件列出所有开机自动或手动挂载的设备，每个挂载点由一列文本表述。

例如，以下文件为安装完成后安装程序创建的 `/etc/fstab` 文件：

```sh
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda2 during installation
UUID=3b76a24a-0164-4a6e-8235-c76b7c92776b /               ext4    errors=remount-ro 0       1
# /boot/efi was on /dev/sda1 during installation
UUID=2079-B588  /boot/efi       vfat    umask=0077      0       1
# swap was on /dev/sda3 during installation
UUID=2d2332fb-19bd-4313-8329-0d17ad4e2612 none            swap    sw              0       0
/dev/sr0        /media/cdrom0   udf,iso9660 user,noauto     0       0
```

格式如下：

| 字段 | 说明 |
|-|-|
| 设备 | 这个字段指定要被挂载的设备名，他可以是一个本地设备，或者一个远程文件系统 (如 NFS ) <br> 此字段可以使用设备的 ID 取代 ( 使用 `blkid device` 命令查看 )|
| 挂载点 | 这个字段代表将设备挂载到本地文件系统的位置 |
| 类型 | 指定要挂载设备使用的文件系统 <br> 如 `ext4`、`vfat` 、`ntfs` 等 <br> `swap` 指定设备为交换分区，`auto` 代表让 `mount` 程序自行检测 |
| 选项 | 依文件系统不同也存在不同的选项，可参照 `mount` 手册。可以指定为 `default` |
| dump | 这一位绝大多数情况都设置为 0,他配合 `dump` 工具使用 |
| pass | 这一位指定是否需要开机时进行检测，通常设定为 0 表示不检测，根目录一定要设置为 1 ，其他需要检测的设置为 2 |


按照需求将设备填入 `/etc/fstab` 即可实现开机自动挂载。

还有一种开机不会直接挂载，直到用到为止才进行挂载的操作，会在后续章节再进行讨论。