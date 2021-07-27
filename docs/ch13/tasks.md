# 大赛题目

由已知信息推断题目并整理，不保证与原题一致

# 登录

- Username：root
- Password：ChinaSkill20!
- Username：Chinaskills20
- Password：Chinaskills20!

除特别指定外，所有账号密码均为 `Chinaskills20!`

# 系统配置

- Region：China
- Locale：en_US.UTF-8
- KeyMap：English US

当任务为配置 SSL 或 TLS，如要求隐藏所有警告，请小心执行。

# 竞赛环境

物理机：

文件夹路径：

虚拟机:[datastore1] ( VMware ESXi )

ISO 镜像：[DATA]ISO/ ( VMware ESXi )

软件：D:\软件 ( Host )

# 项目任务描述

拓扑见样题

基本配置：

Device|Hostname|System|FQDN|IP Address|Service
-|-|-|-|-|-
Server01|Server01|预装 Linux|Server01.sdskills.com|172.16.100.201/25|DNS <br> Web <br> SSH <br> LDAP <br> DBMS 
Server02|Server02|待安装 Linux|Server02.sdskills.com|172.16.100.202/25|RAID5 <br> NFS <br> FTP <br> Mail <br> SSH <br> |
Server03|Server03|预装 Linux|Server03.skills.com|192.168.10.3/28|NTP <br> OpenVPN
Server04|Server04|待安装 Linux|Server04.skills.com|192.168.10.4/28|DNS <br> Web <br>
Rserver|Rserver|预装 Linux|Rserver.sdskills.com|172.16.100.254/25 <br> 192.168.10.2/28 <br> 10.10.10.254/24|firewall <br> DHCP <br> SSH <br> CA
Client|Client|Linux Desktop||10.10.100.x|none

网络：

Network|CIDR
-|-
office|10.10.100.0/24
service|172.16.100.128/25
internet|192.168.10.0/28


# Client Task

Client 已预装简易 Debian，要求如下：

- 要求能访问所有服务器，用于测试应用服务
- 为主机安装 GNOME 桌面环境
- 调整分辨率至 1280x768
- 测试 DHCP，IPv4 地址为自动获取
- 测试 DNS，安装 dnsutils 与 dig 命令行程序
- 测试 Web，安装 Firefox 浏览器，cUrl 命令行工具，在任何时候进行访问测试不允许弹出安全警告信息
- 测试 SSH，安装 SSH 命令行工具
- 测试 VPN 安装 VPN 客户端
- 测试 FTP，安装 FTP 客户端
- 测试文件共享，安装 Samba 命令行工具
- 测试 Mail，安装 Thunderbird，并能正常进行邮件收发
- 其他测试采用默认设定

# Rserver Task

* Network
  * 请根据基本配置信息配置服务器主机名，网卡 IP 地址配置，域名等
  * 开启路由转发功能
* iptables
  * 默认阻挡所有流量
  * 添加必要的 NAT 规则和流量放行规则，正常情况下 Internet 网络不能访问 office 网络，并使所有服务正常工作
  * 添加必要的端口转发规则
* DHCP
  * 为客户端分配 IP 范围为 10.10.100.1 - 10.10.100.50
  * 按照实际配置 DNS 与 GATEWAY
  * 配置 Client 固定获取 IP 地址为 10.10.100.51
* SSH
  * 安装 SSH 正常监听
  * 限制除 Client 以外的客户端登录，其他主机都应拒绝
  * 配置 Client 只能在 Chinaskills20 用户可以免密登录，端口为 2222，且具有 root 控制权限
* CA
  * CA 根证书路径 /CA/cacert.pem
  * 签发数字证书，签发者信息
    * 国家 = CN
    * 单位 = Inc
    * 组织机构 = www.sdskills.com
    * 公用名 = Skill Global Root CA

# Server01 Task

* Network 
  * 配置主机名 IP DNS 网关等
* DNS
  * 安装 Bind 服务
  * 建立 sdskills.com 域，为除 Internet 区域的主机建立正反解析
  * 当出现无法解析的域名时，向域 skills.com 申请更高层次的解析
* Web
  * 安装 Apache 服务
    * 由 Server01 提供 www.sdskills.com
  * 网站
    * 服务以用户 webuser 运行
    * 网站文件存放在 /data/share/webroot
    * 页面为 Shell 脚本，文件名为 index.sh
    * 网页内容为 `Current system time is:$(date "+%F %T %p")`
  * 该页面需要员工账号认证才能访问
    * 员工账号存储在 LDAP 中，账号为 lduser001
  * 网站使用 HTTPS 协议
    * SSL 使用 RServer 签发的证书，颁发给：
    * 