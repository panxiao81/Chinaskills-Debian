在过去 ( 甚至包括现在 ) 的 *nix 系统中存在运行级别的概念。

systemd 使用新的相似又不同的概念：目标 ( target ) 取代了过去的运行级别。

目标不再使用数字表示，且可以同时启用多个。且目标间可互相继承。

Systemd 默认提供了一些模仿旧概念的目标。

# 查看与切换

虽然旧式的 `runlevel` 命令与 `telinit` 命令仍可用，但已不再推荐使用了。

## 查看当前目标

使用：

```sh
$ systemctl list-units --type=target
UNIT                LOAD   ACTIVE SUB    DESCRIPTION
basic.target        loaded active active Basic System
cryptsetup.target   loaded active active Local Encrypted Volumes
getty.target        loaded active active Login Prompts
graphical.target    loaded active active Graphical Interface
local-fs-pre.target loaded active active Local File Systems (Pre)
local-fs.target     loaded active active Local File Systems
multi-user.target   loaded active active Multi-User System
network.target      loaded active active Network
paths.target        loaded active active Paths
remote-fs.target    loaded active active Remote File Systems
slices.target       loaded active active Slices
sockets.target      loaded active active Sockets
swap.target         loaded active active Swap
sysinit.target      loaded active active System Initialization
time-sync.target    loaded active active System Time Synchronized
timers.target       loaded active active Timers

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

16 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
```

上文已写出，由于 Systemd 目标的概念与旧的运行级别概念并不完全相同，所以存在多种已加载的目标是正常的。

这个例子充分体现了 Systemd 的目标间**继承**的关系。

## 切换

要切换当前目标，使用：

```sh
# 切换至带有图形界面的目标
$ sudo systemctl isolate graphical.target
```

Systemd 的目标有一部分与 SysV 运行级别一一对应，可参照以下表格：

| SysV 运行级别 |Systemd 目标|注释|
|-|-|-|
|0|runlevel0.target, poweroff.target|中断系统 ( `halt` ) |
|1, s, single|runlevel1.target, rescue.target|单用户模式|
|2, 4|runlevel2.target, runlevel4.target, multi-user.target|用户自定义运行级别，通常识别为级别3。|
|3|runlevel3.target, multi-user.target|多用户，无图形界面。用户可以通过终端或网络登录。|
|5|runlevel5.target, graphical.target|多用户，图形界面。继承级别3的服务，并启动图形界面服务。|
|6|runlevel6.target, reboot.target|重启|
|emergency|emergency.target|急救模式 ( `Emergency shell` )|

要更改开机启动的默认目标，使用：

```sh
# 设置开机启动至文本多用户模式
$ sudo systemctl set-default multi-user.target
# 查看当前默认启动目标
$ systemctl get-default
```

