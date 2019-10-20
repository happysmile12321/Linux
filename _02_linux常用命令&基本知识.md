## tips

> Ctrl+shift+“+”可以增大终端的字体 ， ctrl+“-”减小字体。

## 图形界面

X window

- X Server
- X Client
- X protocol
- Desktop Environment,window manage

---

## Gnome

- panel
- Desktop
- menu system
- file manager

---

## 字符界面和图形界面切换

字符->图形：Alt	+	F1~6	/	Ctrl+Alt	+	F1~6

图形->字符：Ctrl+Alt	F1~6

---

## 运行级别

> 排名根据常用度

```
0-关机
3-对外提供网络服务的纯终端模式
5-带GUI的模式
1-root用户无密码登陆，用以执行一些紧急操作
-------------------------
2-不对外提供网络服务的纯终端模式
6-重启
4-保留，用于以后扩展
```

### 修改默认的运行级别

#### 立刻生效，但不写入硬盘

> 切换到root

```
su
```

```cmd
init 0
init 1
init 2
init 3
init 4
init 5
init 6
```

#### 滞后生效，但永久成立(谨慎使用)

> 切换到root

```
su
```

> 修改默认启动级别

```
vim /etc/inittab
```

> 如果采用这种方式，设置成0，就会开机然后又立刻关机；设置成6，就会开机然后又重启。

### 相关命令

> 查看当前运行级别，N代表上次运行级别，数字代表当前运行级别

```cmd
runlevel
```



> 关机，底层是给init发送运行级别

```
shutdown 【选项】【时间】【警告信息】
①-r：reboot
②-h：halt after shutdown
④-P：power off after shutdown
★halt：挂起→同步数据→关闭主机
poweroff：关机
⑥-k：不关机，只发出警告信息
⑦-c：取消关机
```

```cmd
	①	shutdown –k 45
	提示当前时间，并指出45分钟后关机（-k参数不能由-c参数撤销，因为此参数并只是发出警告信息，并不是真正关机）
	②	shutdown –h 45
		shutdown –h now
	③	shutdown –c
	取消关机，此命令无法在当前终端中发出，应再打开一个终端，重新登录root，发出此命令
	④	shutdown –r now “警告信息”
	发出警告信息，关机并重启
```



> 列出当前路径 pwd  (print working directory)

```cmd
pwd
```



> 改变路径 cd (change directory)

```cmd
①cd /etc	#切换到“/etc”目录
②cd ..	#更改至当前目录的父目录（上一级）
cd .		#当前目录
③cd ~		#更改至当前登录用户的工作目录
④cd ~rjxy	#更改至用户rjxy的宿主目录（宿主目录，即用户的个人目录）
```



> 列出当前目录信息 ls (list)

```cmd
ls 【选项】 【目录或文件】
①ls /home			#查看/home目录下的文件（不包括隐藏文件）
②ls –a /home		#显示/root目录下所有文件（包括隐藏文件，隐藏文件前面带“.”）
③ls –l /etc		#长格式显示所有内容（相当于ll命令）
```



> 文件定位 locate

```cmd
文件定位命令：locate（搜索文件速度最快，并输出文件完整的路径）
locate inittab
★可能提示“locate: can not stat () `/var/lib/mlocate/mlocate.db': No such file or directory”
原因：没有找到指定的数据库
解决方法：使用updatedb命令升级数据库（注意root用户才有权限）
```



> 查看历史命令 history

```cmd
①history
②!4						#执行history结果中显示的第4条命令
```



> 在线帮助文档 man (manaul)

```cmd
【例】man ls，man halt
组成内容：
	①NAME：该内容的简单说明
	②SYNOPSIS：大致说明，对于命令来说是命令的语法，对于函数来说是函数的定义
	③DESCRIPTION：该内容的简明介绍
	④OPTIONS：命令参数的详细解释
	⑤SEE ALSO：给用户一些提示，介绍一些参考内容
	⑥BUGS：该命令或函数存的的bug
数字表示手册页的不同类型
	【1】一般使用者类型，如ls，init
	【2】系统调用命令
	…………
	【8】有关系统维护的命令，如rpm，grub
	★退出man：按“q”
```



> 离线帮助手册  -- help (linux软件编写习惯，选项后为单字母用一个-，多字母用两个-)

```cmd
在命令后输入“--help”，即可显示该shell命令的用法
★man和help区别：
	man是装系统时安装的文档，help是软件编写人员在编写时提供的内置查询参数，查询参数是在程序或者命令内部，而man的查询结果在程序或命令之外，即如果系统的中缺少某条命令的文档，则man命令无法无法返回结果。
```

