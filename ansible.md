# ansible

环境准备：三台centos7

10.10.20.105 node1（安装ansible）
10.10.20.104 node2
10.10.20.103 node3

## ansible安装

```shell
yum install epel-release -y
yum install ansible -y
```



## ansible配置

/etc/ansible/hosts文件增加如下主机清单

```shell
cat >> /etc/ansible/hosts << EOF
[test01]
10.10.20.103
10.10.20.104
EOF
```



## ansible实验

```shell
ansible test -m command -a "free -h"
-m:指定模块
-a:指定参数
test 对应 /etc/ansible/hosts中的[test]
all 对应 /etc/ansible/hosts所有ip
注意：默认模块不支持管道通信
颜色说明
绿：正常
红：异常
黄：有变动
```



### ping模块

```shell
[root@node1 ~]# ansible all -m ping --list-hosts  #查看主机清单
  hosts (2):
    10.10.20.103
    10.10.20.104
[root@node1 ~]# ansible all -m ping -C  #-C 测试并不执行
10.10.20.104 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
10.10.20.103 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
[root@node1 ~]#
```



### group模块

```shell
[root@node1 ~]# ansible-doc -l #列出可用模块

#创建组名为mygrp，gid=3000
[root@node1 ~]# ansible all -m group -a "gid=3000 name=mygrp state=present system=no" #present参数为创建
10.10.20.103 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "gid": 3000, 
    "name": "mygrp", 
    "state": "present", 
    "system": false
}
10.10.20.104 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "gid": 3000, 
    "name": "mygrp", 
    "state": "present", 
    "system": false
}
[root@node1 ~]#
#节点2主机
[root@node2 ~]# tail -1 /etc/group
mygrp:x:3000:
[root@node2 ~]#
#节点3主机
[root@node3 ~]# tail -1 /etc/group
mygrp:x:3000:
[root@node3 ~]# 
 #absent参数为删除
[root@node1 ~]# ansible all -m group -a "gid=3000 name=mygrp state=absent system=no" 
10.10.20.104 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "name": "mygrp", 
    "state": "absent"
}
10.10.20.103 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "name": "mygrp", 
    "state": "absent"
}
[root@node1 ~]# 
```



### user模块

```shell
#创建用户
[root@node1 ~]# ansible all -m user -a "uid=5000 name=testuser state=present groups=mygrp shell=/bin/tcsh"
10.10.20.103 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "comment": "", 
    "create_home": true, 
    "group": 5000, 
    "groups": "mygrp", 
    "home": "/home/testuser", 
    "name": "testuser", 
    "shell": "/bin/tcsh", 
    "state": "present", 
    "system": false, 
    "uid": 5000
}
10.10.20.104 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "comment": "", 
    "create_home": true, 
    "group": 5000, 
    "groups": "mygrp", 
    "home": "/home/testuser", 
    "name": "testuser", 
    "shell": "/bin/tcsh", 
    "state": "present", 
    "system": false, 
    "uid": 5000
}
[root@node1 ~]#
#节点2主机
[root@node2 ~]# id testuser
uid=5000(testuser) gid=5000(testuser) groups=5000(testuser),3000(mygrp)
[root@node2 ~]#  tail -1 /etc/passwd
testuser:x:5000:5000::/home/testuser:/bin/tcsh
[root@node2 ~]# 
#节点3主机
[root@node3 ~]# id testuser
uid=5000(testuser) gid=5000(testuser) groups=5000(testuser),3000(mygrp)
[root@node3 ~]# tail -1 /etc/passwd
testuser:x:5000:5000::/home/testuser:/bin/tcsh
[root@node3 ~]# 
```



### copy模块

```shell
[root@node1 ~]# ansible all -m copy -a "src=/etc/fstab dest=/root/fstab.ansible mode=600 owner=testuser group=mygrp"
10.10.20.103 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "checksum": "aeb2bcdf54eefdb31de0967d9d18bdd9991ce339", 
    "dest": "/root/fstab.ansible", 
    "gid": 3000, 
    "group": "mygrp", 
    "md5sum": "d81248928b7989f025d91a7fe57fb196", 
    "mode": "0600", 
    "owner": "testuser", 
    "size": 848, 
    "src": "/root/.ansible/tmp/ansible-tmp-1598478283.01-10368-31775414173528/source", 
    "state": "file", 
    "uid": 5000
}
10.10.20.104 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "checksum": "aeb2bcdf54eefdb31de0967d9d18bdd9991ce339", 
    "dest": "/root/fstab.ansible", 
    "gid": 3000, 
    "group": "mygrp", 
    "md5sum": "d81248928b7989f025d91a7fe57fb196", 
    "mode": "0600", 
    "owner": "testuser", 
    "size": 848, 
    "src": "/root/.ansible/tmp/ansible-tmp-1598478283.03-10370-243822379572701/source", 
    "state": "file", 
    "uid": 5000
}
[root@node1 ~]#
[root@node2 ~]# ls -l /root
total 12
-rw-------. 1 root     root  4036 Aug 19 03:19 anaconda-ks.cfg
-rw-------  1 testuser mygrp  848 Aug 27 05:44 fstab.ansible
-rw-------. 1 root     root  3966 Aug 19 03:19 original-ks.cfg
[root@node2 ~]# 
[root@node3 ~]# ls -l /root
total 12
-rw-------. 1 root     root  4036 Aug 19 03:19 anaconda-ks.cfg
-rw-------  1 testuser mygrp  848 Aug 27 05:44 fstab.ansible
-rw-------. 1 root     root  3966 Aug 19 03:19 original-ks.cfg
[root@node3 ~]# 
#直接根据content内容创建文件
[root@node1 ~]# ansible all -m copy -a "content='hi ansible test' dest=/root/hi.ansible mode=770 owner=testuser group=mygrp"
10.10.20.103 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "checksum": "ddad8b148f0bc800a9a71441476cd3b118e16015", 
    "dest": "/root/hi.ansible", 
    "gid": 3000, 
    "group": "mygrp", 
    "md5sum": "16c88f69e76d484f997a158a93e8fba0", 
    "mode": "0770", 
    "owner": "testuser", 
    "size": 15, 
    "src": "/root/.ansible/tmp/ansible-tmp-1598478655.06-10480-45583591293126/source", 
    "state": "file", 
    "uid": 5000
}
10.10.20.104 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "checksum": "ddad8b148f0bc800a9a71441476cd3b118e16015", 
    "dest": "/root/hi.ansible", 
    "gid": 3000, 
    "group": "mygrp", 
    "md5sum": "16c88f69e76d484f997a158a93e8fba0", 
    "mode": "0770", 
    "owner": "testuser", 
    "size": 15, 
    "src": "/root/.ansible/tmp/ansible-tmp-1598478655.09-10482-273153972832645/source", 
    "state": "file", 
    "uid": 5000
}
[root@node1 ~]# 
[root@node2 ~]# ls -lh  /root
total 16K
-rw-------. 1 root     root  4.0K Aug 19 03:19 anaconda-ks.cfg
-rw-------  1 testuser mygrp  848 Aug 27 05:44 fstab.ansible
-rwxrwx---  1 testuser mygrp   15 Aug 27 05:50 hi.ansible
-rw-------. 1 root     root  3.9K Aug 19 03:19 original-ks.cfg
[root@node2 ~]# 
[root@node3 ~]# ls -lh  /root
total 16K
-rw-------. 1 root     root  4.0K Aug 19 03:19 anaconda-ks.cfg
-rw-------  1 testuser mygrp  848 Aug 27 05:44 fstab.ansible
-rwxrwx---  1 testuser mygrp   15 Aug 27 05:50 hi.ansible
-rw-------. 1 root     root  3.9K Aug 19 03:19 original-ks.cfg
[root@node3 ~]# 
```



### command模块

```shell
[root@node1 ~]# ansible all -m command -a "pwd"
10.10.20.104 | CHANGED | rc=0 >>
/root
10.10.20.103 | CHANGED | rc=0 >>
/root
[root@node1 ~]# 
```



### shell模块

```shell
[root@node1 ~]# ansible all -m shell -a " echo 111111 | passwd --stdin testuser"10.10.20.104 | CHANGED | rc=0 >>
Changing password for user testuser.
passwd: all authentication tokens updated successfully.
10.10.20.103 | CHANGED | rc=0 >>
Changing password for user testuser.
passwd: all authentication tokens updated successfully.
[root@node1 ~]# 
```



### file模块

```shell
[root@node1 ~]# ansible all -m file -a "path=/var/tmp/hi.dir state=directory"
10.10.20.104 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "gid": 0, 
    "group": "root", 
    "mode": "0755", 
    "owner": "root", 
    "path": "/var/tmp/hi.dir", 
    "size": 6, 
    "state": "directory", 
    "uid": 0
}
10.10.20.103 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "gid": 0, 
    "group": "root", 
    "mode": "0755", 
    "owner": "root", 
    "path": "/var/tmp/hi.dir", 
    "size": 6, 
    "state": "directory", 
    "uid": 0
}
[root@node1 ~]# 
```

### fetch模块

```shell
[root@node1 ansible_platbooks]# ansible 10.10.20.104 -m fetch -a "src=/etc/redis.conf dest=./ "
10.10.20.104 | CHANGED => {
    "changed": true, 
    "checksum": "07eedef3014b6ed6d95b95b38577dff5ac3ecf12", 
    "dest": "/ansible_platbooks/10.10.20.104/etc/redis.conf", 
    "md5sum": "d98629fded012cd2a25b9db0599a9251", 
    "remote_checksum": "07eedef3014b6ed6d95b95b38577dff5ac3ecf12", 
    "remote_md5sum": null
}
[root@node1 ansible_platbooks]#
[root@node1 ansible_platbooks]# ls
10.10.20.104  test.yaml
[root@node1 ansible_platbooks]# cd 10.10.20.104/
[root@node1 10.10.20.104]# ls
etc
[root@node1 10.10.20.104]# cd etc/
[root@node1 etc]# ls
redis.conf
[root@node1 etc]# pwd
/ansible_platbooks/10.10.20.104/etc
[root@node1 etc]# 
```



### cron模块

```shell
[root@node1 ~]# ansible test01 -m cron -a "name='echo test' minute=*/1 job='echo ansible cron test' "
10.10.20.103 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "envs": [], 
    "jobs": [
        "echo test"
    ]
}
10.10.20.104 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "envs": [], 
    "jobs": [
        "echo test"
    ]
}
[root@node1 ~]#
[root@node2 ~]# crontab -l
#Ansible: echo test
*/1 * * * * echo ansible cron test
[root@node2 ~]# 
[root@node3 ~]# crontab -l
#Ansible: echo test
*/1 * * * * echo ansible cron test
[root@node3 ~]# 
```

### yum模块

```shell
[root@node1 ~]# ansible test01 -m yum -a "name=nginx state=installed"
[root@node1 ~]# 
[root@node2 ~]# rpm -q nginx
nginx-1.16.1-1.el7.x86_64
[root@node2 ~]# 
[root@node3 ~]# rpm -q nginx
nginx-1.16.1-1.el7.x86_64
[root@node3 ~]# 
```



### service模块

```shell
[root@node1 ~]# ansible all -m service -a "name=nginx state=started"
[root@node1 ~]# 
[root@node2 ~]# ss -tnlp
State      Recv-Q Send-Q   Local Address:Port                  Peer Address:Port              
LISTEN     0      128                  *:80                               *:*                   users:(("nginx",pid=11241,fd=6),("nginx",pid=11240,fd=6))
LISTEN     0      128                  *:22                               *:*                   users:(("sshd",pid=1062,fd=3))
LISTEN     0      128               [::]:80                            [::]:*                   users:(("nginx",pid=11241,fd=7),("nginx",pid=11240,fd=7))
LISTEN     0      128               [::]:22                            [::]:*                   users:(("sshd",pid=1062,fd=4))
[root@node2 ~]# 
[root@node3 ~]# ss -tnlp
State      Recv-Q Send-Q   Local Address:Port                  Peer Address:Port              
LISTEN     0      128                  *:80                               *:*                   users:(("nginx",pid=11267,fd=6),("nginx",pid=11266,fd=6))
LISTEN     0      128                  *:22                               *:*                   users:(("sshd",pid=1062,fd=3))
LISTEN     0      128               [::]:80                            [::]:*                   users:(("nginx",pid=11267,fd=7),("nginx",pid=11266,fd=7))
LISTEN     0      128               [::]:22                            [::]:*                   users:(("sshd",pid=1062,fd=4))
[root@node3 ~]# 
[root@node1 ~]# ansible all -m service -a "name=nginx state=stopped"
```



### script模块

```shell
[root@node1 ~]# cat /root/test.sh
#!/bin/bash
echo "test" > /tmp/test.out
[root@node1 ~]# 
[root@node1 ~]# ansible all -m script -a "/root/test.sh"
10.10.20.103 | CHANGED => {
    "changed": true, 
    "rc": 0, 
    "stderr": "Shared connection to 10.10.20.103 closed.\r\n", 
    "stderr_lines": [
        "Shared connection to 10.10.20.103 closed."
    ], 
    "stdout": "", 
    "stdout_lines": []
}
10.10.20.104 | CHANGED => {
    "changed": true, 
    "rc": 0, 
    "stderr": "Shared connection to 10.10.20.104 closed.\r\n", 
    "stderr_lines": [
        "Shared connection to 10.10.20.104 closed."
    ], 
    "stdout": "", 
    "stdout_lines": []
}
[root@node2 ~]# cat /tmp/test.out 
test
[root@node2 ~]# 
[root@node3 ~]# cat /tmp/test.out 
test
[root@node3 ~]# 
```

## Playbook文件配置

Playbook的核心元素：

1. Hosts：主机
2. Tasks：任务列表
3. Variables：变量
4. Templates：包含模板语法的文件
5. Handlers：特定条件触发的任务



task：
	模块，模块参数
	格式：
			（1）action：module arguments
			（2）module：arguments

### Tasks配置

```shell
[root@node1 ansible_platbooks]# pwd
/ansible_platbooks
[root@node1 ansible_platbooks]# cat test.yaml 
- hosts: all
  remote_user: root
  tasks: 
  - name: install redis
    yum: name=redis state=installed
  - name: start redis
    service: name=redis state=started enabled=yes

[root@node1 ansible_platbooks]# 
[root@node1 ansible_platbooks]# ansible-playbook --syntax-check test.yaml 

playbook: test.yaml
[root@node1 ansible_platbooks]# 
[root@node1 ansible_platbooks]# ansible-playbook --list-hosts  --list-tasks test.yaml 

playbook: test.yaml

  play #1 (all): all    TAGS: []
    pattern: [u'all']
    hosts (2):
      10.10.20.103
      10.10.20.104
    tasks:
      install redis     TAGS: []
      start redis       TAGS: []
[root@node1 ansible_platbooks]#
[root@node1 ansible_platbooks]# ansible-playbook -C  test.yaml 

PLAY [all] *************************************************************************************

TASK [Gathering Facts] *************************************************************************
ok: [10.10.20.104]
ok: [10.10.20.103]

TASK [install redis] ***************************************************************************
changed: [10.10.20.104]
changed: [10.10.20.103]

TASK [start redis] *****************************************************************************
changed: [10.10.20.103]
changed: [10.10.20.104]

PLAY RECAP *************************************************************************************
10.10.20.103               : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.10.20.104               : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[root@node1 ansible_platbooks]#
```

### Handlers配置

特定条件触发任务

notify:handlers_name

```shell

[root@node1 playbook]# cat file.yaml
- hosts: all
  remote_user: root
  tasks: 
  - name: install redis
    yum: name=redis state=installed
  - name: copy config file
    copy: src=/playbook/redis.conf dest=/etc/redis.conf owner=redis mode=640
    notify: restart redis #调用handlers任务“restart redis”
  - name: start redis
    service: name=redis state=started enabled=yes
  handlers: 
  - name: restart redis
    service: name=redis state=restarted
[root@node1 playbook]# ansible-playbook -C file.yaml

PLAY [all] *******************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************
ok: [10.10.20.104]
ok: [10.10.20.103]

TASK [install redis] *********************************************************************************************************************************************************************************************
ok: [10.10.20.104]
ok: [10.10.20.103]

TASK [copy config file] ******************************************************************************************************************************************************************************************
changed: [10.10.20.104]
changed: [10.10.20.103]

TASK [start redis] ***********************************************************************************************************************************************************************************************
ok: [10.10.20.104]
ok: [10.10.20.103]

RUNNING HANDLER [restart redis] **********************************************************************************************************************************************************************************
changed: [10.10.20.104]
changed: [10.10.20.103]

PLAY RECAP *******************************************************************************************************************************************************************************************************
10.10.20.103               : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.10.20.104               : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

### tags配置

指定条件触发任务

```shell
[root@node1 playbook]# cat file.yaml
- hosts: all
  remote_user: root
  tasks: 
  - name: install redis
    yum: name=redis state=installed
  - name: copy config file
    copy: src=/playbook/redis.conf dest=/etc/redis.conf owner=redis mode=640
    notify: restart redis
    tags: config and restart #定义tags
  - name: start redis
    service: name=redis state=started enabled=yes
  handlers: 
  - name: restart redis
    service: name=redis state=restarted

[root@node1 playbook]#
[root@node1 playbook]#  ansible-playbook -t "config and restart" -C file.yaml #指定执行tags为"config and restart"的任务

PLAY [all] *******************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************
ok: [10.10.20.104]
ok: [10.10.20.103]

TASK [copy config file] ******************************************************************************************************************************************************************************************
changed: [10.10.20.103]
changed: [10.10.20.104]

RUNNING HANDLER [restart redis] **********************************************************************************************************************************************************************************
changed: [10.10.20.104]
changed: [10.10.20.103]

PLAY RECAP *******************************************************************************************************************************************************************************************************
10.10.20.103               : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.10.20.104               : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[root@node1 playbook]#  ansible-playbook -t "config and restart" file.yaml

PLAY [all] *******************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************
ok: [10.10.20.103]
ok: [10.10.20.104]

TASK [copy config file] ******************************************************************************************************************************************************************************************
changed: [10.10.20.103]
changed: [10.10.20.104]

RUNNING HANDLER [restart redis] **********************************************************************************************************************************************************************************
changed: [10.10.20.103]
changed: [10.10.20.104]

PLAY RECAP *******************************************************************************************************************************************************************************************************
10.10.20.103               : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.10.20.104               : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

### Variables配置

 (1) facts：可直接调用；
 注意：可使用setup模块直接获取目标主机的facters；
 (2) 用户自定义变量：

 (a) ansible-playbook命令的命令行中的
 -e VARS, --extra-vars=VARS

 (b) 在playbook中定义变量的方法：
 vars:
 - var1: value1
 - var2: value2

 变量引用：{{ variable }}

 (3) 通过roles传递变量；
 (4) Host Inventory
 (a) 用户自定义变量
 (i) 向不同的主机传递不同的变量；
 IP/HOSTNAME varaiable=value var2=value2
 (ii) 向组中的主机传递相同的变量；
 [groupname:vars]
 variable=value

 (b) invertory参数
 用于定义ansible远程连接目标主机时使用的参数，而非传递给playbook的变量；
 ansible_ssh_host
 ansible_ssh_port
 ansible_ssh_user
 ansible_ssh_pass
 ansbile_sudo_pass
 ...

 补充模块：
 setup模块：

### Templates配置





