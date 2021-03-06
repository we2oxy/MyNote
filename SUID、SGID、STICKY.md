## 安全上下文：  
1.进程以用户的身份运行；进程是发起次进程用的的代理，因此以此用户的身份和权限完成所有操作；  
2.权限匹配模型：  
(1).判断进程的属主，是否属于被访问的文件属组；如果是，则应用属组的权限；否则进入第二步；  
(2).判断进程的属主，是否属于被访问的文件属组；如果是，则应用属组的权限；否则进入第三部；

## SUID：  
默认情况下：用户发起的进程，进程的属主是其发起者；因此，其以发起者的身份在运行；  
SUID的功用：用户在运行某程序时，如果此程序拥有SUID权限，那么程序运行为进程时，进程的属主不是发起者，而是程序文件自己的属主；

## 管理文件的SUID权限：  
    
```
chmod u+|-s file
```  
展示位置：属主的执行权限位  
    如果属主原本有执行权限，显示为小写s；
    否则显示为大写S；
    
## SGID
功用：当前目录属组有写权限，且有SGID权限时，那么所有属于此目录的属组，且以属组身份在此目录中新建文件或目录时，新文件的属组不是用户的基本组，而是此目录的属组；  
管理文件的SGID权限：  
```
chmod g+|-s file
```  
展示位置：属组的执行权限位  
如果属组原本有执行权限，显示为小写s；  
否则，显示为大写S；  

STICKY：  
功用：对于属组或全局科协的目录，组内的所有用户或系统上的所有用户对在此目录中都能创建新文件或删除所有的已有文件；如果为此类目录设置STICKY权限，则每个用户能创建新文件，且只能删除自己的文件；

管理文件的Sticky权限：  
```
chmod o+ | -t file
```

展示位置：其他用户的执行权限位  如果属组原本有执行权限，显示为小写t；  
否则，显示为大写T；  
系统上的/tmp和/var/tmp目录均默认有Sticky权限  
基于八进制方式赋权时，可于默认的三位八进制数字左侧再加一位八进制数字

```
chmod 1777 filename
```  
## facl：file access control lists  
文件的额外赋权机制：  
在原有的u,g,o之外，另一层让普通用户能控制赋权给另外的用户或组的赋权机制；

#### getfacl命令：
```
    getfacl filename
    user：username：MODE
    group：groupname:MODE
```
#### serfacl命令：
赋权给用户：
```
setfacl -m u:username:mode file..
```
赋权给组：
```
setfacl -m g:groupname:mode file..
```

取消赋权：  
```
setfacl -x u:username:mode file..
setfacl -x g:groupname:mode file..
```