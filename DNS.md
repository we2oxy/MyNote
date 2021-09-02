打开虚拟机后挂在光盘

```
cd /media/RHEL_6.5\ i386\ Disc\ 1/Packages/
rpm -qa | grep bind
rpm -ivh bind-9.8.2-0.17.rc1.el6_4.6.i686.rpm 
rpm -ivh bind-chroot-9.8.2-0.17.rc1.el6_4.6.i686.rpm
```


关闭防火墙和selinux
serivce iptables stop
setenforce 0

编辑全局配置文件修改
vi /etc/named.conf 
listen-on port 53 { any; };
allow-query     { any; };
：wq


编辑主配置文件添加如下内容

```
vi /etc/named.rfc1912.zones

zone "hwl.com" IN {
        type master;
        file "hwl.com.zone";
        allow-update { none; };
};

zone "1.168.192.in-addr.arpa" IN {
        type master;
        file "zone.hwl.com";
        allow-update { none; };
};

```

```
:wq
```



```
cd /var/named
cp named.localhost hwl.com.zone
```



编辑区域配置文件，确保hwl.tianyi.com对应自己的LINUX服务器IP地址

```
vi hwl.com.zone
$TTL 1D
@       IN SOA  dns.hwl.com. root(
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum

hwl.com.        IN      NS      dns.hwl.com.

dns.hwl.com.    IN      A       192.168.1.1
www.hwl.com.    IN      A       192.168.1.2
ftp.hwl.com.    IN      A       192.168.1.3
mail.hwl.com.   IN      A       192.168.1.4

down            IN      CNAME   ftp
ns              IN      CNAME   dns
```

  




```
$TTL 1D
1.168.192.in-addr.arpa. IN SOA  dns.hwl.com. root(
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum

1.168.192.in-addr.arpa. IN      NS      dns.hwl.com.

1.1.168.192.in-addr.arpa.       IN      PTR     dns.hwl.com.

1       IN      PTR     dns.hwl.com.
2       IN      PTR     www.hwl.com.
3       IN      PTR     ftp.hwl.com.
4       IN      PTR     mail.hwl.com.
```


更改区域文件所属组并启动服务
ll
chgrp named hwl.com.zone
service named start

配置本机的DNS服务器为本机
vi /etc/resolv.conf
domain hwl.com
search hwl.com
nameserver 192.168.1.1

测试DNS服务

```
nslookup
> 192.168.1.1
Server:         192.168.1.1
Address:        192.168.1.1#53

1.1.168.192.in-addr.arpa        name = dns.hwl.com.
> 192.168.1.2
Server:         192.168.1.1
Address:        192.168.1.1#53

2.1.168.192.in-addr.arpa        name = www.hwl.com.
> 192.168.1.3
Server:         192.168.1.1
Address:        192.168.1.1#53

3.1.168.192.in-addr.arpa        name = ftp.hwl.com.
> 192.168.1.4
Server:         192.168.1.1
Address:        192.168.1.1#53

4.1.168.192.in-addr.arpa        name = mail.hwl.com.
> dns
Server:         192.168.1.1
Address:        192.168.1.1#53

Name:   dns.hwl.com
Address: 192.168.1.1
> www
Server:         192.168.1.1
Address:        192.168.1.1#53

Name:   www.hwl.com
Address: 192.168.1.2
> ftp
Server:         192.168.1.1
Address:        192.168.1.1#53

Name:   ftp.hwl.com
Address: 192.168.1.3
> mail
Server:         192.168.1.1
Address:        192.168.1.1#53

Name:   mail.hwl.com
Address: 192.168.1.4
> ns
Server:         192.168.1.1
Address:        192.168.1.1#53

ns.hwl.com      canonical name = dns.hwl.com.
Name:   dns.hwl.com
Address: 192.168.1.1
> down
Server:         192.168.1.1
Address:        192.168.1.1#53

down.hwl.com    canonical name = ftp.hwl.com.
Name:   ftp.hwl.com
Address: 192.168.1.3
```




