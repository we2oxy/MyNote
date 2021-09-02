# bash配置文件（管理员可修改全局配置文件）

## 登录类型：
- 交互式登录shell进程
- 非交互式登录的shell进程

## 配置文件类型
- profile类
- bashrc类

### 交互式登录shell进程：
1. 直接通过某终端输入账号密码后登录打开的shell进程；  
1. 使用su命令：su - username,或者执行su -l username执行的登录切换；


#### profile类：为交互式登录的shell提供配置。  
**功用**：
1. 用来定义环境变量；
1. 运行命令或脚本；  

##### 交互式登录的shell读取顺序：
```
/etc/profile-->/etc/profile.d/*.sh-->~/.bash_profile-->~/.bashrc-->/etc/bashrc
```

全局配置：   

```
/etc/profile  
/etc/profile.d/*.sh
```
用户个人：仅对当前用户有效；  
```
~/.bash_profile
```

### 非交互式登录shell进程：
1. su username执行的登录切换；  
1. 图形界面下打开的终端；  
1. 运行脚本的时候；  

#### bashrc类：为非交互式登录的shell提供配置
**功用：**  
1.用来定义本地变量；  
2.定义命令别名；

### 非交互式登录的shell读取顺序：
```
~/.bashrc-->/etc/bashrc-->/etc/profile.d/*
```
**全局：对所有用户都生效**
```
/etc/bashrc
```
**用户个人：仅对当前用户有效**
```
~/.bashrc
```
- 命令行中定义的特性，例如变量和别名作用于为当期shell进程的生命周期；
- 配置文件定义的特性，只对随后新启动的shell进程有效；

###### 让配置文件定义的特性立即生效：  
通过命令行重复定义一次；  
让shell进程重读配置文件；  

```
source /path/conf_file
./path/conf_file
```



