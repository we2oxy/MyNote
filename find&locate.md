## locate

依赖于事先构建好的索引库；

系统自动实现（周期性任务）；

手动更新数据库（updatedb）；

    
工作特性： 
        查找速度快；
        模糊查找；
        非实时查找；
        
        locate [option] ... pattern...
        -b:只匹配路径中的基名;
        -c:统计出共有多少个符合条件的文件;
        -r:BRE
注意：索引构建过程层需要遍历符合整个根文件系统，极度消耗资源;
## find


测试:结果通常为布尔型("true","false")

根据文件名查找:
	-name "pattern"
	-iname "pattern"
		支持glob风格的通配符;
		*,?,[],[^]  
		
批量清空文件
```
find ./ -type f -size +51M | xargs -i sh -c ' > {}'
find ./ -type f -size +30M -exec cp /dev/null {} \;
```

根据文件从属关系查找:

```
    find . -user root
	find . -group root
	find . -uid 500
	find . -gid 500
	find . -nouser
	find . -nogroup
```


根据文件的类型查找:

```
-type TYPE:
		f:普通文件
		d:目录
		l:符号链接文件
		b:块设备文件
		c:字符设备文件
		p:管道文件
		s:套接字文件
```




```
[root@huangwL ~]# find /dev -type b -ls
```



组合测试:
	与:-a,默认组合逻辑;
	或:-o,
 	非:-not,!

- 1.找出/tmp目录下属主为非root的所有文件;

```
- [root@huangwL ~]# find /tmp -not -user root -type f
```

- 2.找出/tmp目录下文件名中不包含fstab字符串的文件;

```
- [root@huangwL ~]# find /tmp/ ! -iname "*fstab*"
```

- 3.找出/tmp目录下属主为非root,而且文件名中不包含fstab字符串的文件;

```
- [root@huangwL ~]# find /tmp -type f ! -iname "*fstab*" -a ! -user root
- [root@huangwL ~]# find /tmp -type f ! \( -user root -o -iname "*fstab*" \) -ls
```



!A -a !B = !(A -o B)
!A -o !B = !(A -a B)

根据文件的大小查找:
	-size[+|-]#UNIT
		常用单位:k,M,G

		#UNIT:(#-1,#]
		-#UNIT:[0,#-1]
		+#UNIT:(#,00)


```
[root@huangwL tmp]# ls -l
total 576
-rw-r--r-- 1 root root 291492 Jul 15 14:19 passwd
-rw-r--r-- 1 root root 291202 Jul 15 14:20 passwd.2
[root@huangwL tmp]# ls -lh     
total 576K
-rw-r--r-- 1 root root 285K Jul 15 14:19 passwd
-rw-r--r-- 1 root root 285K Jul 15 14:20 passwd.2
[root@huangwL tmp]# find /tmp/ -size 285k
/tmp/passwd
/tmp/passwd.2
[root@huangwL tmp]# find /tmp/ -size 285k
/tmp/passwd
[root@huangwL tmp]# find /tmp/ -size 284k
/tmp/passwd.2
[root@huangwL tmp]# find /tmp/ -size -285k
/tmp/
/tmp/.ICE-unix
/tmp/passwd.2
[root@huangwL tmp]# find /tmp/ -size -286k
/tmp/
/tmp/.ICE-unix
/tmp/passwd
/tmp/passwd.2
[root@huangwL tmp]# find /tmp/ -size +284k
/tmp/passwd
```



根据时间戳查找:
	以"天"为单位:
		-atime[访问]:[+|-]#
			#:[#,#-1)
			-#:(#,0]
			+#:(00,#-1]
		-mtime[修改]:
			#:[#,#-1)
			-#:(#,0]
			+#:(00,#-1]
		-ctime[改变]:
			#:[#,#-1)
			-#:(#,0]
			+#:(00,#-1]

	以"分钟"为单位:
		-amin[访问]:
			#:[#,#-1)
			-#:(#,0]
			+#:(00,#-1]
		-mmin[修改]:
			#:[#,#-1)
			-#:(#,0]
			+#:(00,#-1]
		-cmin[改变]:
			#:[#,#-1)
			-#:(#,0]
			+#:(00,#-1]


```
根据"权限"查找:
	-perm[/|-]mode
		mode:精确权限匹配
		/mode:任何一类用户(u.g.o)的权限中的任何一位(r.w.x)符合条件即满足;
		-mode:每一类用户的权限中的每一位都符合条件既满足;
```



```
处理动作:
	-print:输出之标准输出;默认动作;
	-ls:类似与对查找到的文件执行ls -l命令,输出文件的详细信息;
	-delete:删除查找到的文件;
	-fls path/to/somefile:把查找到的所有文件的长格式信息保存至指定的文件中;
	-ok command {} \; :对查找到的每个文件执行command命令;每次操作都要用户确认;
	-exec command {} \; :对查找到的每个文件执行command命令;
```



```
注意:find传递查找到的文件路径至后面的命令时,会先查找出所有哦符合条件的文件路径,然后一次性传递给后面的命令;
	但是有些命令不能接受过长的参数,此时命令执行会失败;另一种方式可规避此问题;
	find | xargs command
```



练习:
	1.查找/var目录下属主为root,且属组为mail的所有文件和目录;

```
[root@huangwL ~]# find /var -user root -a -group mail -ls
```
2.查找/usr目录下不属于root,bin或hadoop的所有文件或目录;使用两种方法查询

```
[root@huangwL ~]# find /usr ! -user root -a -user bin -a -user hadoop -ls
[root@huangwL ~]# find /usr ! \( -user root -o -user bin -o -user hadoop \) -ls
```

3.查找/etc目录下最近一周内七内容修改过,且属主不是root用户也不是hadoop用户的文件或目录.

```
[root@huangwL ~]# find /etc/ -mtime -7 -a ! -user root -a ! -user hadoop -ls
[root@huangwL ~]# find /etc/ -mtime -7 -a ! \( -user root -o -user hadoop \) -ls
```
4.查找当前系统上没有属主或属组,且最近一周内被访问过的文件或目录.

```
[root@huangwL ~]# find / -atime -7 \( -nouser -o -nogroup \) -ls
```

5.查找/etc目录下大于1M且类型为普通文件的所有文件;

```
[root@huangwL ~]# find /etc/ -size +1M -type f -exec ls -lh {} \;
```

6.查找/etc目录下所有用户都没有写权限的文件;

```
[root@huangwL ~]# find /etc/ ! -perm /222 -type f -exec ls -lh {} \;
```

7.查找/etc目录只有有一类用户没有执行权限的文件;

```
[root@huangwL ~]# find /etc/ ! -perm -111 -type f -ls
```

8.查找/etc/init.d/目录下,所有用户都有执行权限,且其他用户有写权限的所有文件;

```
[root@huangwL ~]# find /etc/init.d/ -perm -113 -type f -ls
```










