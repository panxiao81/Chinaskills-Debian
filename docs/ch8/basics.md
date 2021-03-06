# SSH

SSH 可以远程连接进行开 Shell，传输文件等操作，其中远程连接主机使用 `ssh`，而文件传输使用 `scp`。

## SSH 远程连接

使用以下方式连接远程计算机：

```sh
$ ssh 用户名@计算机名或 IP 地址 -p 端口号 

# 其中若使用默认的 22 号端口则可省略端口号
# 如：
# 使用用户名 debian 连接 192.168.50.153，端口号为 22
$ ssh debian@192.168.50.153

# 或，使用 root 连接 us.ddupan.top,端口号为 1234
$ ssh root@us.ddupan.top -p 1234
```

除去 `-p` 参数外，还有一些其他常用参数，如打开 X11 转发，建立 SSH 隧道转发等：

| 参数 | 说明                                      |
| ---- | ----------------------------------------- |
| `-X` | 打开 X11 转发                             |
| `-L` | 打开一个 TCP 本地转发                     |
| `-R` | 打开一个 TCP 远程转发                     |
| `-J` | 按照先后顺序进行连接 ( 使用跳板机的场景 ) |
| `-T` | 不分配伪终端 (通常与 -N 参数配合使用)     |
| `-N` | 不采用交互提示 ( 通常与 -T 参数配合使用 ) |

## TCP 转发

SSH 可以转发任意 TCP 流量，有两种转发方式，本地转发与远程转发

- 本地转发：在本机开一个端口，将连接到本机的此端口的所有 TCP 流量转发至远程目标端口
- 远程转发：将连接到远程主机指定端口的 TCP 流量通过本机转发至目标

远程转发可用于将 NAT 后的主机流量转发至一个中继。

参数的格式为：`<本地端口>:<远程目标地址>:<远程端口>`

例如：

```sh
$ ssh -TNL 8080:us.ddupan.top:8080 root@us.ddupan.top -p 1234
# 将连接到本机的 8080 端口的流量转发至 us.ddupan.top 的 8080 端口，不分配伪终端，不采用交互模式
```

当需要开机启动，或者常驻后台时，可以使用 nohup 或作为 service 单元写入 systemd.

