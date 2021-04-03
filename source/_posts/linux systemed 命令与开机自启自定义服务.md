---
title: linux systemed 命令与开机自启自定义服务
date: 2019-05-01 19:22:15
tags: linux
--- 

# linux systemed 命令与开机自启自定义服务

标签（空格分隔）： linux

---

## **1. Systemd[(简体中文)][1]**

输出激活的单元：`systemctl` === `systemctl list-units`
输出执行失败的单元：`systemctl --failed`
查看全部已安装服务：`systemctl list-unit-files`
马上激活单元：`systemctl start <单元>`
马上停止单元：`systemctl stop <单元>`
重新启动单元：`systemctl restart <单元>`
检查单元是否配置为自己主动启动：`systemctl is-enabled <单元>`
取消开机自己主动激活单元：`systemctl disable <单元>`

常用：

> 查看服务当前状态 `systemctl status shadowsocks.service`   
开机自己主动激活单元：`systemctl enable shadowsocks.service`
查看所有已启动的服务: `systemctl list-units --type=service`  
杀进程: `systemctl kill httpd.service`


## **2. 开机启动**

对于那些支持 Systemd 的软件，安装的时候，会自动在 `/usr/lib/systemd/system` 目录添加一个配置文件。

    systemctl enable httpd

该相当于在 `/etc/systemd/system` 目录添加一个符号链接，指向 `/usr/lib/systemd/system` 里面的 `httpd.service` 文件。

`/etc/systemd/system` 目录里面的配置文件优先级**较高**。

### **配置文件**
```
$ systemctl cat sshd.service

[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.service
Wants=sshd-keygen.service

[Service]
EnvironmentFile=/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
Type=simple
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

然后

    $ systemctl enable sshd.service