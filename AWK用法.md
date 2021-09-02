# GNU awk:
	
## 	文本处理三工具:grep,sed,awk
		grep,egrep,fgrep:文本过滤工具:pattern
		sed:行编辑器
			模式空间,保持空间
		awk:报表生成器,格式化文本输出;[Aho;Weinberger;Kernighan]
		基本用法:gawk [options] 'program' file
			program:patern[action statements]
				语句之间用分号分隔

				print,printf

			选项:
				-F:指明输入时用到的字段分隔符;
				-v var=value:自定义变量

### 		1.print

			print item1,item2

			要点:
				(1).都好分隔符;
				(2).输出的每个item可以字符串,也可以是数值;当前记录的字段,变量或awk的表达式;
				(3).如果省略item,相当与print $0;

### 		2.变量
#### 			2.1内建变量
				FS:input file seperator,默认输入分隔符为空白字符;
				OFS:output file seperator,默认输出分隔符为空白字符
				RS:input record seperator,默认输入换行符为空白字符;
				ORS:output record seperator,默认输入换行符为空白字符;

				NF:number of field,字段数量
					{print NF},{print $NF}

				NR:number of record,行数
				FNR:各文件分别计数:行数;

				filename:当前文件名;

				ARGC:命令行参数的个数;
				ARGV:数组,保存的是命令行所给定的各参数;

				[root@01dns ~]# awk 'BEGIN{print ARGC}' /etc/fstab /etc/issue
				3
				[root@01dns ~]# 
				[root@01dns ~]# awk 'BEGIN{print ARGV[0]}' /etc/fstab /etc/issue 
				awk
				[root@01dns ~]# awk 'BEGIN{print ARGV[1]}' /etc/fstab /etc/issue 
				/etc/fstab
				[root@01dns ~]# awk 'BEGIN{print ARGV[2]}' /etc/fstab /etc/issue 
				/etc/issue
				[root@01dns ~]#

#### 			2.2自定义变量:
				(1) -v var=value

					变量名区分字符大小写:

				(2)在program中直接定义
					[root@01dns ~]# awk -v test='hello gawk' 'BEGIN{print test}'
					hello gawk
					[root@01dns ~]# awk 'BEGIN{test="hello gawk";print test}'
					hello gawk
					[root@01dns ~]#

### 		3.printf命令

			格式化输出:printf FORMAT,item1,item2, ...

				(1)FORMAT必须给出
				(2)不会自动换行,需要显式给出换行符,\n
				(3)FORMAT中需要分别为后面的每个item指定一个格式化符号;

				格式符:
					%c:显示字符的ASCII码;
					%d,%i:显示十进制数;
					%e,%E:科学计数法数值显示;
					%f:显示为浮点数;
					%g,%G:以科学计数法或浮点形式显示数值;
					%s:显示字符串;
					%u:无符号整数;
					%%:显示%自身;

				修饰符:
					#[.#]:

