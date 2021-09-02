# NetWork File System

## 1.NFS安装

NFS Server:nfs-utils.x86_64 1:1.3.0-0.66.el7_8

## 2.NFS挂载

```shell
mount.nfs 172.16.0.68:/data/mysql /mysql
```

showmount -e nfs_server:查看nfs_server上导出的所有文件系统；

showmount -a ：在nfs_server上查看nfs服务所有的客户端列表；

exportfs：

1. -r：重新导出
2. -a：所有文件系统
3. -v：详细信息
4. -u：取消导出文件系统