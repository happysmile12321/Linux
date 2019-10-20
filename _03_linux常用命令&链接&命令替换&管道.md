## 常用命令

> 创建文件 touch

```cmd
touch 【-c】【-d<日期时间>】【-r<参考文件或目录>】【-t<日期时间>】【文件】
-c：假设目的文件不存在，不创建新文件
-r：使用参考档的时间
-t：设定（修改）文件的修改时间，日期格式为“MMDDHHmm”
```

```cmd
①touch file file1 file2		#创建文件“file”、“file1”和“file2”
②ls –l file				#查看file的修改时间
③touch –t 10010900 file	#把file文件的时间改为10月1日9点
touch –t 10012600 file	#错误！时间格式不对，没有“26”时
④ls –l file				#再查看file的修改时间
⑤touch –c –t 10100100 file	#把ff文件的修改时间修改为10月10日1点，但若file文件不存在，则不创建此文件
⑥touch –r file1 file		#把file的时间改为file1的时间（以file1为参考）
ls –l file file1
```

```cmd
★在linux里，文件的后缀名从技术角度来说没有任何的特殊意义，仅仅是文件名的一个简单的组成部分。
```



> 创建目录 mkdir(make directory)

```cmd
mkdir 【选项】 【目录名】
-p：后跟路径名，若路径中某些目录不存在，则系统将自动创建这些目录，即一次可以创建多个目录
-m：对新建目录设置存取权限，不加此选项时，默认权限为755
①mkdir a
②mkdir a/b
③mkdir a/b/c/d		#错误！
mkdir –p a/b/c/d
④ls –dl a			#-d选项可以查看目录本身的信息
mkdir –m 777 b		#创建目录b，权限为777
ls –dl a 
【注意】绝对路径和相对路径
★绝对路径：从根目录出发，以根目录为参考，如/home，表示根目录下的home子目录；
★相对路径：从当前目录出发，以当前目录为参考，如home，表示当前目录下的home子目录。
```



> 删除 rm

```
rm -rf 文件/文件夹
删除一切
	参数解释:
		-r 级联地，-f 强制
rmdir -p 目录
递归地删除目录
```



> 复制 cp(copy)

```cmd
如果同时指定两个以上的文件或目录，且最后的目的地是一个已存在的目录，则此命令会把前面指定的所有文件或目录复制到该目录中。
-l：不做拷贝，只是链接文件（硬链接）
★ 若用cp –l a/aaa b，则此时b中的aaa只是a/aaa的一个链接文件。若此时运行“cp a/aaa b”则会提示“a/aaa和b/aaa是同一个文件”
-r：若源文件是目录，则递归复制该目录下所有子目录和文件
①	mkdir a b
touch a/aaa
cp –l a/aaa b		#复制链接
cp  a/aaa b			#复制文件时，提示两者是同一个文件
②	cp a/aaa b/ddd		#把a/aaa复制到b目录下，并改名为ddd
③	mkdir –p a/b/c/d/e
mkdir w
cp a b				#略过目录
cp –r a b			#复制a到b
cp –r a/b w			#把a下的b目录复制到w目录
④	cp /etc/*.conf a		#把/etc目录下所有后缀为conf的文件复制到a目录
```



> 移动|改名|复制 mv(move)

```cmd
-f：非交互式，强制覆盖
①mv  a/asound.conf b			#把asound.conf移至b中，a中不再有此文件
②mv  b/asound.conf a/a.c		#把asound.conf移至a中并改名为a.c
③mv  a/b  b					#把目录a/b移至b中
④mv  a/b  b/x					#把目录a/b移至b中并改名为x
```



> 显示更多分页 more(常用在纯终端环境)

```cmd
-num：一次显示的行数
-s：多个空白行替换为一个空白行
+num：从第num行开始显示
①more -s file2
②more -20 +10 file2			#从第10行开始，每次显示20行
```



> 回卷显示文本文件 less

```cmd
▲作用和more类似，但允许往回卷动，按“q”退出
①less file2					#滚动到底后可回滚
```



> 显示文本内容 cat

```cmd
-n：由1开始对所有行编号
-b：空白行不编号
-s：两个以上的空白行替换为1行显示
	①cat –n file1
	②cat –b file1
	③cat –s file1
	④cat -ns file1
	 cat -bs file1
	⑤cat –n file1 > file2
	把file1的内容加上行号输入文件file2中
	 cat  –n file file2
	先显示file，再显示file2，一起编号
	⑥cat –b file >>file2
	file文件加行号（空白行不加）后，附加到file2的末尾
★注意“>”和“>>”的区别
```



> 显示文件开头 head

```cmd
-n：显示的行数，默认是10行
-v：显示文件名
①head -8 file2				#显示文件file2的前8行
②head -v -10 file2
head -10 -v file2			#对比两个命令，一个对一个错
```



> 显示文件结尾 tail

```cmd
-n：从距文件尾n行处开始，默认为10行
①tail -6 file1
②tail和head并用
head -10 file1 | tail -6			#“head -10 file1”表示file1文件的前10行，“tail -6”表示在上一步的基础上读最后6行，最后结果是读取源文件的5~10行
---
tail -f 文件名 动态地显示文件结尾10行
```



## 链接

> 理解inode是理解链接的前提

### inode

```
linux的诞生是很早的，那时候硬盘大行其道，是最主要的存储介质。硬盘有扇区，磁道，柱面，盘面等一些物理特性。
本文用到一些术语，汉译多有歧义，建议用英文理解。
	- sector 扇区
	- block 块
sector是硬盘一次读取的最小单位。可以想象sector又是Byte的基本单位。
但，这对于硬盘来说，还是太小了，于是人为地让机器多读几个sector，就成了block。我们现在所经常提到的硬盘一次读写的512Byte，就是1 block。

linux怎么搞呢？
	这里必须要说清楚一点儿事儿。linux一切皆文件。人家只是拿文件表示一切，又没真的说外设它就是文件。有的文件还不能用vim打开呢。这个文件我的理解就是为了方便描述而抽象的一种概念。就像c语言对int的抽象，使得申请一个变量的内存空间不必像汇编那样繁琐。
	前面关于sector和block的辩证已经知晓，那么，8 block = 4 KB 。这个大小在linux中是一个很神奇的单位，就如同65535对于计算机网络来说是一个经常用到的量一样。

必备知识来了~

linux文件系统基本结构
	inode + 文件本身(也就是文件的数据)。
	linux在装系统时，忽略硬盘的物理细节，抽象为inode + 文件本身。
	inode = 0.5 kB(512 Bytes) 
	文件的一片儿 = 4 KB
    ---
    文件大于4KB？
    文件 = 8KB。 inode + 文件片儿 + inode + 文件片儿
    ---
    文件不够或刚好超过8KB？
    	文件不够8KB
        inode + 文件片儿 + inode + 文件片儿
		文件超过8KB
		inode + 文件片儿 + inode + 文件片儿 + inode + 文件片儿
	---
	总结：没用到就浪费。反正4KB也不大
	---

重点来了~

inode号是linux内部找寻文件的唯一凭证。文件名的话容易有特殊字符，文件的信息(属性)都存在inode那0.5KB之中，我们在shell中输入文件名，linux默认转化为inode号进行处理。一个文件可以有多个别名。

linux系统链接
	硬链接
	软链接
	
硬链接 
	文件inode区域，起一个alias(别名)。
	文件inode号和source file相同。
软链接
	文件data区，存放另一个文件地址。
	即链接到的那个文件的路径是这个软链接文件的内容，点击这个文件就会执行，按照它里面的内容找到另一个文件并打开。
	文件inode号和source file不同。
```

### ln

> 创建硬链接

```cmd
ln source-file linkFile
```

> 创建软链接

```cmd
ln –s source-file linkFile
```

## 命令替换

```cmd
Command1 `Command2`
or
Command1 $(Command2)
```

> Command2的输出作为Command1的参数。

```cmd
①方法一：gedit $（locate inittab）
②方法二：gedit `locate inittab`
★“`”为Tab键上方的键，和“~”同一个键
```

## 管道

> 管道不管在linux运维还是在嵌入式，都是一个十分重要的概念，学好它非常重要。

```cmd
Linux理念：汇集许多小程序，每个程序都有特殊的专长，复杂的任务不是由大型软件完成，而是组合许多小程序共同完成
Command1|Command2
管道：将Command1命令的输出作为Command2命令的输入
①ls /etc|grep ab					#grep表示筛选，筛选出符合后面条件的内容
②ls /etc和ls /etc|more的输出做对比
```











