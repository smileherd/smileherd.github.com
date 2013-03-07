---
layout: post
title: ssh 配置 authorized_keys 后仍需输入密码问题
---

**{{ page.title }}**

原因在于 authorized_keys 文件写权限只能为所有者拥有。如果除所有者之外还有别的用户有写权限的话，出于安全原因 sshd 就不会使用此文件。所以仍需要用户手动输入密码。

解决方法就很简单了，只需关闭组用户和其它用户的 authorized_keys 文件写权限即可。值得注意的是**文件所在目录 .ssh 的写权限同样要对组用户和其它用户关闭。**否则也会导致文件失效。

	# 取消组用户和其它用户对 authorized_keys 文件的写权限
	chmod go-w authorized_keys
	
	# 取消组用户和其它用户对 .ssh 目录的写权限
	chmod go-w .ssh

断开链接后再 ssh 登录就不用输入密码了。

PS：所有操作环境是指目标主机…