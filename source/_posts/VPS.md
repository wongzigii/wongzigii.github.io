---
title: VPS
date: 2017-03-14 22:21:30
tags:
---

搬瓦工搞了个 19刀 一年的良心活动，而且竟然支持支付宝付款。

1. Vim 中文乱码
2. SSH Key
3. Shadowsocks
4. Supervisor
5. MySQL 远程连接问题
7. 自建 Git 仓库 及 git-hook 自动部署
8. Nginx

## Vim 中文乱码
在 `/etc/vim/vimrc` 这个配置文件中添加文件编码：

````bash
set fileencodings=utf-8,gb2312,gbk,gb18030
set termencoding=utf-8
set encoding=prc
````

## SSH Key
首先，先本地生成一个新的 SSH key 用来连接到远程服务器，然后加到服务器上 `.ssh/authorized_keys` 文件中。

````bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
cat ~/.ssh/id_rsa.pub | ssh <user>@<hostname> 'umask 0077; mkdir -p .ssh; cat >> .ssh/authorized_keys && echo "Key copied"'
````

参数：
- -t: 加密算法：RSA 、RSA1、DSA、ECDSA、ED25519
- -b: 多少位：4096
- -C: 注释

更安全的做法：
- 定期更换 SSH 端口
- 禁用密码登录，本地配置好 ssh_config 文件
- 特定 IP 加白名单
- 用 [fail2ban](https://github.com/fail2ban/fail2ban) 来 ban 掉一些尝试登录的爬虫

## Shadowsocks
自建一个 bare git 仓库用来管理项目，并用 git hook 来自动部署。

## Supervisor
自然少不了用 Supervisor 这个守护进程来管理和监控进程，有几个命令很容易搞混乱：

````bash
// 启动一个进程
supervisorctl start <name>
````
````bash
// 重启 supervisor 但不会改变配置，它会停止并重新启动所有进程
service supervisor restart 
````
````bash
// 重启 <name> 进程，但不会改变配置
supervisorctl restart <name>
````

注意：如果你改了 supervisor 的配置，那么上面的命令都不会对最新的配置生效
````bash
// 将最新的配置应用到所有（通过重启）进程
service supervisor stop
service superviosr start
````
````bash
// 或者，你可以用 reread 来更新配置，注意：仅仅是更新，并没有重启相关进程
supervisorctl reread

// reread 通常需要跟 update 一起使用，将所有更新了配置的进程重启
supervisorctl update
````

## [Nginx](http://markjberger.com/flask-with-virtualenv-uwsgi-nginx/)
