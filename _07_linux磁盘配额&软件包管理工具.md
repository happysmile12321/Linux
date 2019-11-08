# linux磁盘配额

> 为什么需要磁盘配额？

```
linux磁盘就那么大，如果是个人使用还好，可以一个公司往往会让很多人都使用。在这种情况下，就需要对磁盘的用量进行限制。这种限制不仅仅是针对用户，还要精确到每个设备的分区。
另外，如果磁盘太满的话，把/boot分区给占用，那么会导致操作系统无法启动。
为了减少这样的安全隐患，我们需要对磁盘进行限制。
```

## 进行磁盘配额

- 创建分区&文件系统(注意不要在挂载点为/的分区进行操作，不然很容易出问题)
- 修改/etc/fstab文件&启用系统的磁盘配额功能
- 重新挂载文件系统，使配置生效
- 创建配额文件&生成磁盘用量表
- 使用edquota(ed -- 编辑，quota--配额)为用户指定磁盘配额
- 重启扫描磁盘配额

[^tips]: 磁盘配额实际运行时是对整个分区进行限制的。

### 1.挂载关键的步骤

> 挂载其他的步骤上一节有就不讲了。

```c
vim /etc/fstab
#加上usrquota,grpquota(user quota&group quota，中间用,隔开并且不要有空格)
/dev/sda5	/mnt/aa	ext3	defaults,usrquota,grpquota	0	0
```

```
#不重启重新挂载文件系统
mount  -a	
#检查挂载情况
mount -s
```

### 2.创建配额文件，生成磁盘用量表

####  quotacheck

```
#重新挂载文件系统后，系统就能够使用磁盘配额了，接着需要用quotacheck 命令检查启用了配额的文件系统，并为文件系统创建配额文件aquota.user和aquota.group，最后建立当前磁盘用量表。
quotacheck [-avug] [/mount_point]
【常用选项】
-a：扫描所有含有磁盘配额支持的文件系统。有了这个参数，就可以不指定/mount_point了。
-v：显示扫描文件与目录的使用情况。
-u：针对用户扫描文件与目录的使用情况，会建立aquota.user配额文件。
-g：针对用户组扫描文件与目录的使用情况，会建立aquota.group配额文件。
#在启用了磁盘配额功能的分区生成配额文件
quotacheck –avug
```

> 如果该命令报错，可能是因为：

```
#如果运行命令后，可能提示没有权限，这是由于selinux防火墙的问题，需要先把selinux防火墙关闭
linux中关闭防火墙
1.临时关闭
setenforce 0
2.永久关闭
vim /etc/selinux/config
将SELINUX=enforcing 改为 SELINUX=disabled ->修改完成后重启服务器才生效。
```

> 如果出现上面问题，接下来重新生成配额文件

```
quotacheck –avug
#查看配额文件是否生成
ls -a /mnt/挂载点
```

> 文件系统创建后，生成每个启用了配额的文件系统的当前磁盘用量表 

```
quotacheck –av	
```

### 3.为用户启用配额限制

```
edquota [-u username] [-g groupname]
```

#### edquota

```
edquota [-u username] [-g groupname]
edquota -t
edquota -p source_username -u target_username
【参数】
-u：后接用户名称。可以进入用户磁盘配额的编辑界面（vi）。
-g：后接用户组名称。可以进入用户组磁盘配额的编辑界面（vi）。
-t：可以修改宽限时间。
-p：复制范本。可以将一个用户的配额设置复制给另一用户。
```

> 为用户名为u1的用户启用配额文件
>
> 注意，用户名不能以数字开头，否则会以这个数字作为uid。

```
edquota -u u1
#接下来会出现一张表，看例子
```

```
/dev/sda6 0 2000 5000 0 3 5
#使用空间大小（blocks）的软限制为2M，硬限制为5M；创建文件的个数（inodes）软限制为3个，硬限制为5个
#inodes限制比较不容易把握，一般情况下是不设置的。
#给用户u1一个操作这个文件夹的权限
#把/mnt/挂载点这个目录以及这个目录下所有文件和文件夹都权限都设置为777
chmod -R 777 /mnt/挂载点
#显示这个目录的权限情况
ls -ld /mnt/aa
```

#### quotaon

```
#开启磁盘配额功能，即让aquota.user和aquota.qroup两个文件的限制生效
quotaon -av|quotaon /mnt/挂载点
```

> 至此，为用户设置磁盘配额工作已全部完成

### 4.检查磁盘配额设置情况

```
切换到挂载点，切换到那个用户，然后touch file。
```

```
#检查用户的磁盘陪额情况
quota 用户名
```

```
#如果想测试使用空间大小是否符合最大5MB的设置，可以通过以下命令创建一个指定大小的空文件：
#创建一个大小为6MB的文件
dd  if=/dev/zero  of=bigfile  bs=1MB  count=6		
```

# linux软件包管理工具

> linux共有两个软件包管理工具，
>
> 老式的叫rpm，新式的叫yum。
>
> 其中，rpm仍未过时。主要原因如下：
>
> yum会自动安装一个软件的所有依赖，如果这个软件所依赖的包恰恰是另一个已安装的软件包所依赖的，那么它会自动升级这个依赖。
>
> yum卸载也同样的，会卸载掉这个软件和这个软件所依赖的所有软件。而rpm -e 只是卸载单个软件。
>
> 因此我们常用yum安装软件，用rpm卸载软件。

## rpm

```
•	-q 在系统中查询软件或查询指定 rpm 包的内容信息 
•	-i 在系统中安装软件 
•	-U 在系统中升级软件 
•	-e 在系统中卸载软件 
•	-h 用 #(hash) 符显示 rpm 安装过程 
•	-v 详述安装过程 
•	-p 表明对 RPM 包进行查询，通常和其它参数同时使用，如： 
•	-qlp 查询某个 RPM 包中的所有文件列表 
•	-qip 查询某个 RPM 包的内容信息 
```

> 下面以finger为例用rpm安装以及卸载finger

```
#进入到都是rpm包的那个文件夹。
#即挂载点那个文件夹下的package文件夹下
#查询finger是否安装
rpm -q finger
#过滤列出包含finger的软件列表
ls | grep finger
#i--安装，v--详细过程，h-用hash符(#)显示安装进度
rpm -ivh finger-0.17-39.el6.i686.rpm
#测试是否安装
finger
#卸载finger
rpm -e finger
#查询finger是否已经卸载
rpm -q finger
```

## yum

> yum源可以配置http,ftp,file的baseurl

```
#1.指定yum源
#yum源可以编辑以.repo结尾的文件在/etc/yum.repos.d/目录下
####################mydvd.repo######################
[dvd]
#name随便定
name = install dvd
#安装镜像路径，镜像已被我们挂载到文件系统了，
#所以这一步是写挂载点的路径
#注意file://是协议名，不能省略
baseurl = file:///aa
#其他默认，不需要变
enabled = 1
#是否检查一些验证的，防止软件源上的软件被篡改
gpgcheck = 0
####################################################
用yum命令安装软件时，不能正常安装，首先检查光驱是否已经挂载到了指定目录，然后检查dvd.repo配置文件是否有错误，如果都没有问题，则需要删除/etc/yum.repos.d目录中除dvd.repo之外的其他文件。
#####################################################
#安装finger
#-y是yes,省得安装时一直按y
yum -y install finger
```

