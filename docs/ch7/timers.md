# Timers

systemd 自带定时器功能，可用来取代 cron 来执行计划任务。

比起 cron，systemd 的 timers 存在一些优势如下：

- 简化调试，`systemctl` 可以直接列出退出代码，日志，以及计划任务执行的历史
- 可以配置为运行在特定环境中
- 可以依赖其他 systemd 单元
- 可以对计划任务的 CPU 与内存使用量进行限制

对于熟悉 cron 的人来说，可以安装 `systemd-cron` 软件包，这样使用 `crontab` 设定计划任务会自动转换成 `.timer` 并用 systemd 管理。

安装此软件包之后，启动 `cron.target`，并正常使用 `crontab` 即可。此脚本将自动将 `cron` 配置转换成 systemd 脚本。

```sh
$ sudo apt install systemd-cron
$ sudo systemctl enable cron.target
$ sudo systemctl start cron.target
```

当然也可以手写 `.timer` 文件，注意，文件名应和对应的 `.service` 文件相同。

本文不打算详解 `.timer` 文件的写法，可查看参考文献：

> systemd.timer(5) [man page](https://manpages.debian.org/buster/systemd/systemd.timer.5.en.html)
> 
> systemd.time(7) [man page](https://manpages.debian.org/buster/systemd/systemd.time.7.en.html)
> 
> [Using systemd as a better cron](https://medium.com/horrible-hacks/using-systemd-as-a-better-cron-a4023eea996d)
