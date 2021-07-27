本实验使用 2020 年全国职业院校技能大赛：网络系统管理项目-模块 A 样题一，且在原始题目上有所修改，使其贴近原题。

可在如下位置下载得到：

[下载地址](http://www.chinaskills-jsw.org/_img/2020/09/29/20200929122117510007.zip)

实验过程中可能还需要 [WordPress](https://www.wordpress.org/) 和 [phpMyAdmin](https://www.phpmyadmin.net/) ，可从各自的官网下载最新版本。

我根据实验环境需求制作了一个 VMware 镜像，可直接导入 VMware Workstation 中，获得类似的实验环境。

由于实验环境需求内存量非常大，建议运行此实验环境的主机最好具有 16G 以上内存。

实验需要的其他未提供的软件可自行去其官网下载

# 实验环境使用方法

将下载的虚拟机解压后，将虚拟机导入 VMware Workstation，根据实际配置增大 CPU 核心数 ( 合理范围内越大越好 )

根据实验环境修改网卡为桥接模式或仅主机模式。

打开虚拟机后，待 ESXi 启动完成，之后在宿主机用浏览器打开显示的 IP 地址，或使用宿主机的 VMware Workstation 连接此 IP 地址即可。

- 账号：root
- 密码：Chinaskills20!

每次实验结束需要恢复初始状态时只需恢复快照即可