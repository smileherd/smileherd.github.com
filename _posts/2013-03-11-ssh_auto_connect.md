---
layout: default
title: Mac 下 ssh 自动连接
---

Mac 下借助 autossh 工具可以实现 ssh 断线自动连接功能。（在此之前，首先需要配置 [ssh 免输入密码登录][1]）

autossh 通过监控 ssh 进程运行和 ssh 通道通信情况来判断 ssh 是否处于正常通信状态，一旦通信断开 autossh 将自动重启 ssh 连接。

如果机器上已安装包管理器 Homebrew 或 MacPorts，那么安装 autossh 是很简单的：

	# Homebrew 安装 autossh
	brew install autossh
	
	# MacPorts 安装 autossh
	port install autossh

当然，也可以自己下载 autossh 编译安装。

安装完 autossh 之后就可以通过这条命令来启动 ssh 自动连接：

	autossh -M 0 -D 1984 username@sshserver.com
	
其中 `-D 1984 username@sshserver.com` 的含义和 ssh 命令完全一致，实际上autossh 会将其原封不动的传递给 ssh 命令。`-M` 选项用来指定 autossh 用来发送探测包以监控通信状态的端口。指定为 0 将关闭此功能，autossh 会只在 ssh 退出（exit）时重启。Mac 下的 OpenSSH 通过设置 ServerAliveInterval 和 ServerAliveCountMax 选项可以让 SSH 客户端在失去服务器连接时自动退出。所以我们在这里并不需要监控端口。

ok，将这两个选项的配置附加到 /etc/ssh/ssh_config 文件里：

	# 每隔 60 秒发送一次请求给服务器以保持连接
	ServerAliveInterval 60
	
	# 发送请求后服务器无响应达到 3 次的话自动断开连接
	ServerAliveCountMax 3

这样当 ssh 发现无法连接服务器时会 exit，autossh 监听到 exit 信号将重启 ssh 通信连接。

设置完自动连接，还可以通过 launchd 来设置开机自动启动 ssh。在 /Library/LaunchDaemons/ 或 ~/Library/LaunchAgents/ 下创建一个 plist 文件，假如名为 com.yourdomain.sshd.plist，文件输入以下内容：

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
	<plist version="1.0">
    	<dict>
        	<key>Label</key>
        	<string>com.yourdomain.sshd</string>
        	<key>ProgramArguments</key>
        	<array>
            	<string>/usr/local/bin/autossh</string>
            	<string>-M</string>
            	<string>0</string>
            	<string>-D</string>
            	<string>1984</string>
            	<string>username@sshserver.com</string>
        	</array>
        	<key>KeepAlive</key>
        	<true/>
        	<key>RunAtLoad</key>
        	<true/>
    	</dict>
	</plist>

然后将以上配置好的服务加入开机启动服务列表：

	launchctl load ~/Library/LaunchAgents/com.yourdomain.sshd.plist

重启计算机，all done~

**参考：**

http://blog.hebine.com/archives/1621.html
http://www.2cto.com/os/201301/182222.html

[1]:http://smileherd.github.com/2013/03/07/authorized_keys_config