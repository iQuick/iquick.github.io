---
layout:		post
title:		Linux 文件与文件系统的压缩与打包
category:	[Linux]
tags:		[Linux]
published:	true
---
# Linux 文件与文件系统的压缩与打包
---

目前我们使用的计算机系统中都是使用所谓的 bytes 单位来计量的！不过，事实上，计算机最小的计量单位应该是 bits 才对啊，此外，我们也知道 1 byte = 8 bits 。但是如果今天我们只是记忆一个数字，亦即是 1 这个数字呢？他会如何记录？假设一个 byte 可以看成底下的模样：

□□□□□□□□

由於我们记录数字是 1 ，考虑计算机所谓的二进位喔，如此一来， 1 会在最右边占据 1 个 bit ，而其他的 7 个 bits 将会自动的被填上 0 罗！你看看，其实在这样的例子中，那 7 个 bits 应该是『空的』才对！不过，为了要满足目前我们的操作系统数据的存取，所以就会将该数据转为 byte 的型态来记录了！而一些聪明的计算机工程师就利用一些复杂的计算方式， 将这些没有使用到的空间『丢』出来，以让文件占用的空间变小！这就是压缩的技术啦！

另外一种压缩技术也很有趣，他是将重复的数据进行统计记录的。举例来说，如果你的数据为『111....』共有100个1时， 那么压缩技术会记录为『100个1』而不是真的有100个1的位存在！这样也能够精简文件记录的容量呢！ 非常有趣吧！

<!--break-->

## 系统常见的压缩命令

<pre>
选项与参数：
*.Z         compress 程序压缩的文件；
*.gz        gzip 程序压缩的文件；
*.bz2       bzip2 程序压缩的文件；
*.tar       tar 程序打包的数据，并没有压缩过；
*.tar.gz    tar 程序打包的文件，其中并且经过 gzip 的压缩
*.tar.bz2   tar 程序打包的文件，其中并且经过 bzip2 的压缩
</pre>

## compress

compress这个压缩命令是非常老旧的一款，大概只有在非常旧的 Unix 机器上面还会找到这个软件。
不过，由於 gzip 已经可以解开使用 compress 压缩的文件，因此， compress 可以不用学习啦！ 但是，如果你所在的环境还是有老旧的系统，那么还是得要学一学就是了。好了， 如果你有网络的话，那么安装其实很简单喔！

<pre>
yum install ncompress			&lt;== 这里是安装

compress [-rcv] 文件或目录  	&lt;== 这里是压缩
uncompress 文件.Z           	&lt;== 这里是解压缩
</pre>

## gzip, zcat
gzip 可以说是应用度最广的压缩命令了！目前 gzip 可以解开 compress, zip 与 gzip 等软件所压缩的文件。

<pre>
选项与参数：
-c  ：将压缩的数据输出到萤幕上，可透过数据流重导向来处理；
-d  ：解压缩的参数；
-t  ：可以用来检验一个压缩档的一致性～看看文件有无错误；
-v  ：可以显示出原文件/压缩文件的压缩比等资讯；
-#  ：压缩等级，-1 最快，但是压缩比最差、-9 最慢，但是压缩比最好！默认是 -6

gzip -v 档名 		&lt;== 压缩
gzip -d 档名.gz   	&lt;== 解压
zcat 档名.gz   		&lt;== 查看
</pre>

gzip 的压缩已经最佳化过了，所以虽然 gzip 提供 1~9 的压缩等级，不过使用默认的 6 就非常好用了！

<pre>
gzip -9 -c man.config > man.config.gz  &lt;== 用最佳的压缩比压缩，并保留原本的文件
</pre>

## bzip2, bzcat 

若说 gzip 是为了取代 compress 并提供更好的压缩比而成立的，那么 bzip2 则是为了取代 gzip 并提供更佳的压缩比而来的。

<pre>
选项与参数：
-c  ：将压缩的过程产生的数据输出到萤幕上！
-d  ：解压缩的参数
-k  ：保留原始文件，而不会删除原始的文件喔！
-z  ：压缩的参数
-v  ：可以显示出原文件/压缩文件的压缩比等资讯；
-#  ：与 gzip 同样的，都是在计算压缩比的参数， -9 最佳， -1 最快！

bzip2 [-cdkzv#] 档名  		&lt;== 压缩
bzcat 档名.bz2  			&lt;== 查看
bzip2 -d 档名.bz2 			&lt;== 解压
bzip2 -9 -c 档名 > 档名.bz2 	&lt;== 用最佳的压缩比压缩，并保留原本的文件
</pre>

## 打包命令： tar
虽然 gzip 与 bzip2 也能够针对目录来进行压缩， 不过，这两个命令对目录的压缩指的是『将目录内的所有文件 "分别" 进行压缩』的动作！ 而不像在 Windows 的系统，可以使用类似 WinRAR 这一类的压缩软件来将好多数据『包成一个文件』的样式。

<pre>
选项与参数：
-c  ：创建打包文件，可搭配 -v 来察看过程中被打包的档名(filename)
-t  ：察看打包文件的内容含有哪些档名，重点在察看『档名』就是了；
-x  ：解打包或解压缩的功能，可以搭配 -C (大写) 在特定目录解开
      特别留意的是， -c, -t, -x 不可同时出现在一串命令列中。
-j  ：透过 bzip2 的支持进行压缩/解压缩：此时档名最好为 *.tar.bz2
-z  ：透过 gzip  的支持进行压缩/解压缩：此时档名最好为 *.tar.gz
-v  ：在压缩/解压缩的过程中，将正在处理的档名显示出来！
-f filename：-f 后面要立刻接要被处理的档名！建议 -f 单独写一个选项罗！
-C 目录    ：这个选项用在解压缩，若要在特定目录解压缩，可以使用这个选项。

其他后续练习会使用到的选项介绍：
-p  ：保留备份数据的原本权限与属性，常用於备份(-c)重要的配置档
-P  ：保留绝对路径，亦即允许备份数据中含有根目录存在之意；
--exclude=FILE：在压缩的过程中，不要将 FILE 打包！ 

tar [-j|-z] [cv] [-f 创建的档名] filename... &lt;==打包与压缩
tar [-j|-z] [tv] [-f 创建的档名]             &lt;==察看档名
tar [-j|-z] [xv] [-f 创建的档名] [-C 目录]   &lt;==解压缩
</pre>

使用 tar 加入 -j 或 -z 的参数备份 /etc/ 目录

<pre>
tar -zpcv -f /root/etc.tar.gz /etc

# 由於加上 -v 这个选项，因此正在作用中的档名就会显示在萤幕上。
# 如果你可以翻到第一页，会发现出现上面的错误信息！底下会讲解。
# 至於 -p 的选项，重点在於『保留原本文件的权限与属性』之意。

tar -jpcv -f /root/etc.tar.bz2 /etc

# 显示的信息会跟上面一模一样罗！
</pre>

查阅 tar 文件的数据内容(可察看档名)，与备份档名有否根目录的意义

<pre>
tar -jtv -f /root/etc.tar.bz2

# 如果加上 -v 这个选项时，详细的文件权限/属性都会被列出来！如果只是想要知道档名而已， 那么就将 -v 拿掉即可。
</pre>

备份的数据解压缩，并考虑特定目录的解压缩动作 (-C 选项的应用)

<pre>
tar -jxv -f /root/etc.tar.bz2

# 在 tmp 目录下解压
tar -jxv -f /root/etc.tar.bz2 -C /tmp
</pre>

仅解开单一文件的方法

<pre>
# 1. 先找到我们要的档名，假设解开 shadow 文件好了：
tar -jtv -f /root/etc.tar.bz2 | grep 'shadow'

# 2. 将该文件解开！语法与实际作法如下：
tar -jxv -f 打包档.tar.bz2 待解开档名
tar -jxv -f /root/etc.tar.bz2 etc/shadow
</pre>

打包某目录，但不含该目录下的某些文件之作法

<pre>
ar -jcv  -f /root/system.tar.bz2 --exclude=/root/etc* --exclude=/root/system.tar.bz2  /etc /root
</pre>

仅备份比某个时刻还要新的文件

<pre>
# 1. 先由 find 找出比 /etc/passwd 还要新的文件
find /etc -newer /etc/passwd

# 2. 好了，那么使用 tar 来进行打包吧！日期为上面看到的 2008/09/29
tar -jcv -f /root/etc.newer.then.passwd.tar.bz2 --newer-mtime="2008/09/29" /etc/*

# 3. 显示出文件即可
tar -jtv -f /root/etc.newer.then.passwd.tar.bz2 | grep -v '/$'
</pre>

另外值得一提的是，tar 打包出来的文件有没有进行压缩所得到文件称呼不同喔！ 如果仅是打包而已，就是『 tar -cv -f file.tar 』而已，这个文件我们称呼为 tarfile 。 如果还有进行压缩的支持，例如『 tar -jcv -f file.tar.bz2 』时，我们就称呼为 tarball (tar 球？)！ 这只是一个基本的称谓而已，不过很多书籍与网络都会使用到这个 tarball 的名称！

特殊应用：利用管线命令与数据流

<pre>
# 1. 将 /etc 整个目录一边打包一边在 /tmp 解开
cd /tmp
tar -cvf - /etc | tar -xvf -

# 这个动作有点像是 cp -r /etc /tmp 啦～依旧是有其有用途的！
# 要注意的地方在於输出档变成 - 而输入档也变成 - ，又有一个 | 存在～
# 这分别代表 standard output, standard input 与管线命令啦！
# 简单的想法中，你可以将 - 想成是在内存中的一个装置(缓冲区)。
# 更详细的数据流与管线命令，请翻到 bash 章节罗！
</pre>

## 其它
* [完整备份工具：dump](http://www.imli.me/2015/03/26/linux-full-backups/)
* [光盘写入工具](http://www.imli.me/2015/03/26/linux-cd_write/)
* [其他常见的压缩与备份工具](http://www.imli.me/2015/03/26/linux-other-backup-and-compress/)