#### 光盘安装DHCP服务端软件：
```
cd /media/RHEL_6.5\ i386\ Disc\ 1/Packages/
rpm -qa | grep dhcp
rpm -ivh dhcp-4.1.1-38.P1.el6.i686.rpm 
```
#### 参考帮助文档
```
more /usr/share/doc/dhcp-4.1.1/dhcpd.conf.sample /etc/dhcp/dhcpd.conf
```
```
cd /etc/dhcp/
vi dhcpd.conf
option domain-name "example.org";    //dns的域名
option domain-name-servers ns1.example.org, 
ns2.example.org;   //指定dns服务器  最多三个
default-lease-time 600;   //默认租约时间,单位秒
max-lease-time 7200;   //最大租约时间,单位秒
#ddns-update-style none;   //不接受更新
log-facility local7;   //定义日志服务,默认路径/var/log/bootlog,在/var/log/meddages 里也记录dhcp的日志.
  
 
subnet 10.5.5.0 netmask 255.255.255.224 {       //分配的网段和子网掩码
  range 10.5.5.26 10.5.5.30;    //IP地址池
  option domain-name-servers ns1.internal.example.org;   //dns服务器
  option domain-name "internal.example.org";   //dns域名
  option routers 10.5.5.1;    //指定路由器网关
  option broadcast-address 10.5.5.31;   //指定广播地址
  default-lease-time 600;   //默认租约时间,单位秒
  max-lease-time 7200;   //最大租约时间,单位秒
}

host fantasia {    //绑定MAC和IP,host后面名字随意
  hardware ethernet 08:00:07:26:c0:a5;  //mac地址
  fixed-address fantasia.fugue.com;   //ip地址
}

class "foo" {   //定义多个子网,class后填组名
  match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
}

shared-network 224-29 {   //定义多个子网,从大往小写
  subnet 10.17.224.0 netmask 255.255.255.0 {
    option routers rtr-224.example.org;
  }
  subnet 10.0.29.0 netmask 255.255.255.0 {
    option routers rtr-29.example.org;
  }
  pool {
    allow members of "foo";
    range 10.17.224.10 10.17.224.250;
  }
  pool {
    deny members of "foo";
    range 10.0.29.10 10.0.29.230;
  }
}

service dhcpd restart 
cat /var/lib/dhcpd/dhcpd_leases   //记录已分配的IP地址信息
```