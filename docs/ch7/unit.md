# Unit

Unit ( 单元 ) 是 Systemd 管理系统的基本单位，分为：

- 系统服务 ( .service )
- 挂载点 ( .mount )
- Sockets 套接字 ( .sockets )
- 系统设备 ( .device )
- 交换分区 ( .swap )
- 文件路径 ( .path )
- 启动目标 ( .target )
- 计时器 ( .timer )

最常用的为系统服务单元，作为守护进程使用。

通常在控制单元时使用文件全名，但在指系统服务单元时可省略  `.service` 扩展名。

## systemctl

控制 systemd 的主要命令为 `systemctl`，此命令可用于查看系统状态以及管理系统与服务。

常用的子命令有：

- start 激活单元
- stop 停止单元
- restart 重启单元
- reload 在不重启服务的情况下重新加载配置文件 
- status 输出单元运行状态
- enable 设置开机自启
- disable 取消开机自启
- is-enabled 查看是否设为开机自启
- cat 查看单元配置文件内容
- daemon-reload 重新载入 systemd 系统配置，但不会重新加载变更的单元文件

一些示例：

```sh
# 启动 SSH 服务
$ sudo systemctl start ssh.service
# 如运行正常不会有额外提示

# 查看 ntpd 运行状态
$ systemctl status ntp.service
● ntp.service - Network Time Service
   Loaded: loaded (/lib/systemd/system/ntp.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2021-01-12 22:25:41 CST; 2 weeks 5 days ago
     Docs: man:ntpd(8)
 Main PID: 3783 (ntpd)
    Tasks: 2 (limit: 2258)
   Memory: 2.1M
   CGroup: /system.slice/ntp.service
           └─3783 /usr/sbin/ntpd -p /var/run/ntpd.pid -g -u 107:112

# 设置 open-vm-tools 开机自启
$ sudo systemctl enable open-vm-tools.service

# 重载配置文件，当添加新单元文件时须执行
$ sudo systemctl daemon-reload
```

## 创建或修改单元文件

### 创建单元

systemd 的系统级单元放在以下两个目录中：

- `/usr/lib/systemd/system/`  软件包安装的单元
- `/etc/systemd/system/`  自行安装的单元

systemd 可以使用用户模式运行，存放单元的目录也有所不同。

书写单元按照其他单元的格式仿写即可，例如，一个最简单的单元内容如下：

```ini
[Unit]
# 指定名称
Description=Foo

[Service]
# 设置启动单元时所要启动的项目，可为二进制程序或 Shell 脚本。
# 必须指定绝对路径，例如需要使用 Python 脚本需要写成
# /usr/bin/python3 /root/scripts.py
ExecStart=/usr/sbin/foo-daemon

[Install]
# 指定此单元在哪种 target 启动
WantedBy=multi-user.target
```

### 修改

如要修改软件包安装后附带的单元，最好不要在原始文件进行修改，而是在 `/etc/systemd/system/<单元名>.d/` 目录中新建 `*.conf` 文件，在文件内部写入要新增或修改的参数。

> 注：可能被多次赋值的变量，如 `Execstart` 等，在修改前需要先用一行将变量制空

如要修改自行建立的单元，使用 `systemctl edit 单元名` 即可打开编辑器，修改完成后会自动重新加载。

