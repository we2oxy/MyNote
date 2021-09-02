# FTP:File Transfer Protocol	21/tcp

文件共享服务:应用层,
	NFS:Network File System(RPC:Remote Procedure Call 远程过程调用)
	samba:CIFS/SMB

FTP:tcp,两个链接
	命令连接,控制链接:21/tcp
	数据连接:
		主动模式:20/tcp
		被动模式:端口随机

	数据传输模式:
		ftp server --> ftp client


	vsftpd:
		/etc/vsftpd:配置文件目录
		/etc/init.d/vsftpd:服务脚本
		/usr/sbin/vsftpd:主程序
	基于PAM实现用户认证
		/etc/pam.d/*
		/lib/security/*
		/lib64/security/*
		支持虚拟用户

vsftpd:/var/ftp

上传和下载:

ftp：系统用户
	匿名用户：anonymous_enable
	系统用户：local_enable
	虚拟用户-->系统用户

	/var/ftp：ftp用户的家目录
		匿名用户访问目录

	chroot：禁锢用户与其家目录中

	系统用户：
		write_enbale=YES  上传权限

	文件服务权限：文件系统权限*文件共享权限

	守护进程:
		独立进程
		顺时守护
				有xinetd代为管理

	vsftpd：
		max_clients=#
		max_per_ip=#  每一个ip最多同时发起几个链接请求

	安全通信方式：
		ftps：ftp+ssl/tls
		sftp：OpenSSH，SubSystem，sft(ssh）
		


