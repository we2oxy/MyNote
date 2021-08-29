# Nginx(web server, web reverse proxy):

http协议：80/tcp.

URL:scheme://server[:port]/path/to/source

hhtp事务：request <---> response

    request：
        <method> <URL> <version>
        <HEADERS>
        
        <body>
    
    response
        <version> <status> <reason phrase>
        <HEADERS>
        
        <body>

协议格式：文本、二进制

请求的方式：

    method：
        GET、HEAD、POST、PUT、DELETE、TRACE、OPTINS
    status：
        1xx：信息类
        2xx：成功类，200
        3xx：重定向类，301、302、304
        4xx：客户端错误类，403、401、404
        5xx：服务器端错误类，502、504
        
    HEADER：
        通用首部
        请求首部：
            If-Modified-Since、If-None-Match
        响应首部
        实体首部
        扩展首部
    
    认证：
        基于IP认证：
        基于用户认证：
            basic
            digest
    资源映射：
        Alias
        DocumentRoot
        
    httpd：MPM
        prefork，worker，event
            prefork：主进程，生成多个子进程，每个子进程出来一个请求；
            worker：主进程，生成多个子进程，每个子进程生成多个线程，每个线程响应一个请求；
            event：主进程，生成多个子进程，每个子进程响应多个请求；
## I/O类型：  

同步和异步：synchronous，asynchronous
关注的是消息通知机制
    
同步：调用发出以后不会立即返回，但一旦返回，则返回的是最终结果；
异步：调用发出之后，被调用方立即返回消息、但返回的并非最终结果；被调用者通过状态、通知机制等来通知调用者，或通过回调函数来处理结果；
    
阻塞和非阻塞：block，nonblock
关注的是调用者等待被调用者返回调用结果时的状态
阻塞：调用结果返回之前，调用者会被挂起；调用者只有在得到返回结果之后才能继续；
非阻塞：调用者在结果返回之前，不会被挂起，调用不会阻塞调用者；


## I/O模型：  
- blocking IO：阻塞IO  
- nonblocking IO：非阻塞IO  
- IO multiplexing：复用IO  
- signal driven IO：事件驱动IO  
- asynchronous IO：异步IO  

## 编译安装   
```
yum -y groupinstall "Development Tools" "Server Platform Development"
yum -y install openssl-devel zlib-devel pcre-devel
mkdir -pv /nginx
tar xf nginx-1.8.1.tar.gz 
groupadd -r -g 300 nginx
useradd -r -g 300 -u 300 nginx
./configure \
 --prefix=/nginx \
 --sbin-path=/nginx/sbin/nginx \
 --conf-path=/etc/nginx/nginx.conf \
 --user=nginx \
 --group=nginx \
 --error-log-path=/nginx/log/error.log \
 --http-log-path=/nginx/log/access.log \
 --pid-path=/nginx/pid/nginx.pid \
 --lock-path=/nginx/lock/nginx.lock \
 --with-http_ssl_module \
 --with-http_stub_status_module \
 --with-http_gzip_static_module \
 --with-debug 
make -j 4 && make install


echo "export NGINX_HOME=/nginx" >> /etc/profile.d/nginx.sh
echo "export PATH=$PATH:$NGINX_HOME/sbin" >> /etc/profile.d/nginx.sh
source /etc/profile.d/nginx.sh

[root@CentOS7 nginx-1.8.1]# /nginx/sbin/nginx -t 
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@CentOS7 nginx-1.8.1]# /nginx/sbin/nginx 
[root@CentOS7 nginx-1.8.1]# 
[root@CentOS7 nginx-1.8.1]# ss -tnl
State       Recv-Q Send-Q                        Local Address:Port                                       Peer Address:Port              
LISTEN      0      128                                       *:80                                                    *:*                  
LISTEN      0      128                                       *:22                                                    *:*                  
LISTEN      0      100                               127.0.0.1:25                                                    *:*                  
LISTEN      0      128                                      :::22                                                   :::*                  
LISTEN      0      100                                     ::1:25                                                   :::*                  
[root@CentOS7 nginx-1.8.1]# ss -tnlp
State       Recv-Q Send-Q                        Local Address:Port                                       Peer Address:Port              
LISTEN      0      128                                       *:80                                                    *:*                   users:(("nginx",pid=21960,fd=6),("nginx",pid=21959,fd=6))
LISTEN      0      128                                       *:22                                                    *:*                   users:(("sshd",pid=7050,fd=3))
LISTEN      0      100                               127.0.0.1:25                                                    *:*                   users:(("master",pid=7202,fd=13))
LISTEN      0      128                                      :::22                                                   :::*                   users:(("sshd",pid=7050,fd=4))
LISTEN      0      100                                     ::1:25                                                   :::*                   users:(("master",pid=7202,fd=14))
[root@CentOS7 nginx]# vim nginx.conf 
[root@CentOS7 nginx]# /nginx/sbin/nginx -s stop
[root@CentOS7 nginx]# /nginx/sbin/nginx 


```



## CentOS6服务启动脚本

```
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/etc/nginx/nginx.conf"

[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx

lockfile=/var/lock/subsys/nginx
:<<!
make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:.*--user=" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -n "$user" ]; then
      if [ -z "`grep $user /etc/passwd`" ]; then
         useradd -M -s /bin/nologin $user
      fi
      options=`$nginx -V 2>&1 | grep 'configure arguments:'`
      for opt in $options; do
          if [ `echo $opt | grep '.*-temp-path'` ]; then
              value=`echo $opt | cut -d "=" -f 2`
              if [ ! -d "$value" ]; then
                  # echo "creating" $value
                  mkdir -p $value && chown -R $user $value
              fi
          fi
       done
    fi
}
！
start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```

## CentOS 7服务启动脚本
```
vim /lib/systemd/system/nginx.service
[Unit]
Description=nginx 
After=network.target 
   
[Service] 
Type=forking 
ExecStart=/nginx/sbin/nginx
ExecReload=/nginx/sbin/nginx reload
ExecStop=/nginx/sbin/nginx quit
PrivateTmp=true 
   
[Install] 
WantedBy=multi-user.target

systemctl enable nginx.service
systemctl disable nginx.service
systemctl start nginx.service
systemctl stop nginx.service
systemctl restart nginx.service
systemctl status nginx.service
```

## 配置https+301跳转  
```
    server {
        listen       443;
        server_name www.c.com;
        ssl on;
        ssl_certificate /etc/nginx/cert/gamutsoft.com.crt;  #证书文件
        ssl_certificate_key /etc/nginx/cert/gamutsoft.com.key;  #秘钥文件
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers AESGCM:ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
        ssl_prefer_server_ciphers on;
        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        location /ding {
            alias /data/c.com/www/;
            index index.html index.htm;
        }


        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


```
## 配置http跳转https

```
server {
        listen 80;
        server_name c.com www.c.com;
        rewrite ^(.*) https://www.c.com$1 permanent;  #301永久跳转
        }
```

## 配置反向代理  
代理主机：192.168.52.131 www.hwl.com   

```
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    include /etc/nginx/conf.d/*.conf;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    upstream backend {
        ip_hash;
        server 192.168.52.128 weight=1;
        server 192.168.52.129 weight=2;
        server 192.168.52.130 weight=3;
        #server 192.168.52.130 weight=3 backup;
        
        }
    server {
        listen       80;
        server_name  www.hwl.com;
        location / {
            root   html;
            index  index.html index.htm;
            proxy_pass http://backend;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

RS主机1：192.168.52.128  

```
    server {
        listen       80;
        server_name  192.168.52.128;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```

RS主机2：192.168.52.129  

```
server {
        listen       80;
        server_name  192.168.52.129;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}
```

RS主机3：192.168.52.130

```
    server {
        listen       80;
        server_name  192.168.52.130;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```
### 测试脚本

```
for i in `seq 10`;do curl www.hwl.com ; done
```


## 配置文件的组成部分  
#### 主配置文件：

```
nginx.conf  
include conf.d/*.conf  
include hosts/*.conf  
/etc/nginx/conf.d/*.conf
```


1. mime.typef:支持的mime类型列表  
1. fastcgi_params:fastcgi相关的配置  
1. proxy.conf:代理相关的配置  
1. site.conf:虚拟主机的配置  


#### 配置指令(分号结尾）：  
- directive value1 [value2] ；  
- 支持使用变量   
- 内置变量：由模块引入  
- 自定义变量：  
- set varlable value;  
- 引用变量$varlable  

#### 配置文件组织结构：  
- main block  
- event block  
- http block  

##### main配置段  
###### 正常运行必备的配置：  
- 	user username [groupname]; 指定用户运行worker进程的用户和组  
- 	user nginx nginx   
- 	pid /path/to/pid_file;指定nginx进程的pid文件路径；  
- 	pid /var/run/nginx/nginx.pid  
- 	worker_rlimit_nofile ; 指定一个worker进程能够打开最大文件描述符数量；  
- 	worker_rlimit_sigpending ；指定每个用户发给worker进程信号的数量；  

###### 性能优化相关的配置：  
- worker_processes；worker进程的个数；通常为物理CPU核心数量减1；  
- worker_cpu_affinity CPUMASK;CPUMASK:0001 0010 0100 1000;CPU绑定，并不能隔离；  
- worker_priority nice;指定worker进程优先级  


###### 调试、定位问题的配置：  
- daemon [off|on]；是否以守护进程方式启动   
- master_process on|off ；是否以master/worker模型启动nginx；  
- error_log /path/to/error_log level;                        错误日志文件及起级别；处于调试的需要，可以设定为debug；debug需编译时开启；  

##### event配置段
- worker_connections #;  每个worker进程能够相应的最大并发请求数；   worker_processes * worker_connections  
- use [epoll|rgsig|select|poll];定义使用的事件模型；建议配置nginx自动选择；  
- accept_mutex [on|off]；各worker接受用户的请求的负载均衡锁；启用时，表示用于多个worker轮流、序列化地相应新请求；
- lock_file /path/to/lock_file；  

##### http配置段   
**套接字或主机相关的指令:**
###### 1.server{}；定义一个虚拟主机；  

```
    server {
        listen       80;
        server_name  node1.test.com;
        root /data/www/vhost01/;
        }
    }
```
- 基于port；listen指令监听不同的端口；  
- 基于hostname；server_name指向不同的主机名；

###### 2.listen

```
    listen address[:port] [default_server] [ssl] [http2 | spdy]   
 	listen port [default_server] [ssl] [http2 | spdy]
```


1.  default_server：设置默认虚拟主机；用于基于IP地址，或者使用了不能对应任何一个server配置的server_name时返回站点；  
1.  ssl：用于限制只能通过ssl连接提供服务；  
1.  spdy：SPDY protocol；在编译时编译了spdy模块时用于SPDY协议；  
1.  http2：http version 2；  
###### 3.server_name NAME； 
可跟一个或多个主机名；名称还可以使用通配符和正则表达式（~）；

1.  首先做精确匹配；例如：www.test.com
1.  左侧通配符；例如：*.test.com
1.  右侧通配符；例如：www.test.*
1.  正则表达式，例如：~^\.test.com$  
1.  default_server

###### 4.tcp_nodelay on|off;  
对keepalive模式下的连接是否使用tcp_nodelay选项；

###### 5.tcp_nopush on|off;  
是否启用tcp_nopush（freebse）或tcp_cork选项；仅在sendfile为on时生效；

###### 6.sendfile on|off; 
是否启用sendfile功能；

**路径相关的指令:**

###### 7.root
    设置web资源的路径映射；用于指明请求的URL对应的文档的目录路径； 

```
    server {
        listen       80;
        server_name  www.test.com;
		root /data/www/vhost01/;
    }
#http://www.test.com/imgs/logo.jpg-->/data/www/vhost01/imgs/logo.jpg

    server {
        listen       80;
        server_name  www.test1.com;
        location /images/ {
           root   /data/www/imgs/;
           index  index.html index.htm;
        }
    }
#http://www.test1.com/images/logo.jpg-->/data/www/imgs/images/logo.jpg
```

###### 8.location
```
location [ = | ~ | ~* | ^~ ] uri { ... }   
location @name { ... }
```
**功能：根据用户请求的URI来匹配定义的各location，匹配到时，此请求将被相应的location块中的配置所处理**；

```
    server {
        listen       80;
        server_name  www.test.com;
		root /data/www/vhost01/;
        location /admin/ {
           root   /data/www/admin/;
           index  index.html index.htm;
        }
    }
```
1. =：URI的精确匹配；
1. ~：做正则表达式匹配，区分字符串大小写；
1. ~*：做正则表达式匹配，不区分字符大小写；
1. *~：URI的左半部分匹配，不区分分字符大小写；
2. 匹配优先级：精确匹配=、^~、~或~*、不带符号的URI；

###### 9.alias
**只能用于location配置段中，定义路径别名；**

```
location /images/{
    root /data/imgs/;
}
location /images/{
    alias /data/imgs/;
}
```
注意：  
    root指令：路径为对应的location的“/”；  
        /images/test.jpg-->/data/imgs/images/test.jpg  
    alias指令：路径为对应的location的“/URI/”；  
        /images/test.jpg-->/data/imgs/test.jpg  
        
###### 验证

```
    server {
        listen       80;
        server_name  www.a.com;
		root /data/www/vhost01/;
    }
#http://www.a.com/imgs/1.jpg-->/data/www/vhost01/imgs/1.jpg

    server {
        listen       80;
        server_name  www.b.com;
        location /images/ {
           root   /data/www/imgs/;
           index  index.html index.htm;
        }
    }
#http://www.b.com/images/2.jpg-->/data/www/imgs/images/2.jpg

    server {
        listen       80;
        server_name  www.c.com;
		location /images/{
			root /data/imgs/;
		}
    }

#http://www.c.com/images/3.jpg-->/data/imgs/images/3.jpg

    server {
        listen       80;
        server_name  www.d.com;
		location /images/{
			alias /data/imgs/;
}
    }
#http://www.d.com/images/4.jpg-->/data/imgs/4.jpg
```

###### 10.index

```
Syntax:	index file ...;
Default:	
index index.html;
Context:	http, server, location

location / {
    index index.$geo.html index.html;
}
```
###### 11.error_page
```
Syntax:	error_page code ... [=[response]] uri;  //根据http状态码重定向错误页面；
Default:	—
Context:	http, server, location, if in location
error_page 404             /404.html;
error_page 500 502 503 504 /50x.html;

//重定向至200，返回empty.gif
error_page 404 =200 /empty.gif;
```

