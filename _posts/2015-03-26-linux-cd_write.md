---
layout:		post
title:		Linux 光盘写入
category:	[Linux]
tags:		[Linux]
published:	true
---
# Linux 光盘写入
---

某些时刻你可能会希望将系统上最重要的数据给他备份出来，虽然目前闪盘已经有够便宜，你可以使用这玩意儿来备份。不过某些重要的、需要重复备份的数据(可能具有时间特性)，你可能会需要使用类似 DVD 之类的储存媒体来备份出来！

文字模式的烧录行为处理

* 先将所需要备份的数据建置成为一个映像档(iso)，利用 mkisofs 命令来处理；
* 将该映像档烧录至光盘或 DVD 当中，利用 cdrecord 命令来处理。

<!--break-->

## mkisofs：创建映像档

<pre>
mkisofs [-o 映像档] [-rv] [-m file] 待备份文件.. [-V vol]  -graft-point isodir=systemdir ...

选项与参数：
-o ：后面接你想要产生的那个映像档档名。
-r ：透过 Rock Ridge 产生支持 Unix/Linux 的文件数据，可记录较多的资讯；
-v ：显示建置 ISO 文件的过程
-m file ：-m 为排除文件 (exclude) 的意思，后面的文件不备份到映像档中
-V vol  ：创建 Volume，有点像 Windows 在文件总管内看到的 CD title 的东西
-graft-point：graft有转嫁或移植的意思，相关数据在底下文章内说明。
</pre>

一般默认的情况下，所有要被加到映像档中的文件都会被放置到映象档中的根目录，如此一来可能会造成烧录后的文件分类不易的情况。所以，你可以使用 -graft-point 这个选项，当你使用这个选项之后， 可以利用如下的方法来定义位於映像档中的目录，例如：

* 映像档中的目录所在=实际 Linux 文件系统的目录所在
* /movies/=/srv/movies/ (在 Linux 的 /srv/movies 内的文件，加至映像档中的 /movies/ 目录)
* /linux/etc=/etc (将 Linux 中的 /etc/ 内的所有数据备份到映像档中的 /linux/etc/ 目录中)

## cdrecord：光盘烧录工具

<pre>
cdrecord -scanbus dev=ATA                  &lt;==查询烧录机位置
cdrecord -v dev=ATA:x,y,z blank=[fast|all] &lt;==抹除重复读写片
cdrecord -v dev=ATA:x,y,z -format          &lt;==格式化DVD+RW
cdrecord -v dev=ATA:x,y,z [可用选项功能] file.iso

选项与参数：
-scanbus        ：用在扫瞄磁碟汇流排并找出可用的烧录机，后续的装置为 ATA 介面
-v              ：在 cdrecord 运行的过程中，显示过程而已。
dev=ATA:x,y,z   ：后续的 x, y, z 为你系统上烧录机所在的位置，非常重要！
blank=[fast|all]：blank 为抹除可重复写入的CD/DVD-RW，使用fast较快，all较完整
-format         ：仅针对 DVD+RW 这种格式的 DVD 而已；
[可用选项功能] 主要是写入 CD/DVD 时可使用的选项，常见的选项包括有：
   -data   ：指定后面的文件以数据格式写入，不是以 CD 音轨(-audio)方式写入！
   speed=X ：指定烧录速度，例如CD可用 speed=40 为40倍数，DVD则可用 speed=4 之类
   -eject  ：指定烧录完毕后自动退出光盘
   fs=Ym   ：指定多少缓冲内存，可用在将映像档先缓存至缓冲内存。默认为 4m，
             一般建议可添加到 8m ，不过，还是得视你的烧录机而定。
针对 DVD 的选项功能：
   driveropts=burnfree ：打开 Buffer Underrun Free 模式的写入功能
   -sao                ：支持 DVD-RW 的格式
</pre>

侦测你的烧录机所在位置：

文字模式的烧录确实是比较麻烦的，因为没有所见即所得的环境嘛！要烧录首先就得要找到烧录机才行！ 而由於早期的烧录机都是使用SCSI 介面，因此查询烧录机的方法就得要配合著 SCSI 介面的认定来处理了。 查询烧录机的方式为：

<pre>
cdrecord -scanbus dev=ATA
</pre>

进行 CD 的烧录动作：

<pre>
# 0. 先抹除光盘的原始内容：(非可重复读写则可略过此步骤)
cdrecord -v dev=ATA:1,1,0 blank=fast

# 1. 开始烧录：
cdrecord -v dev=ATA:1,1,0 fs=8m -dummy -data /tmp/system.img

# 2. 烧录完毕后，测试挂载一下，检验内容：
mount -t iso9660 /dev/cdrom /mnt
df -h /mnt

ll /mnt
umount /mnt    &lt;==不要忘了卸载
</pre>

**Tip** 事实上如果你忘记抹除可写入光盘时，其实 cdrecord 很聪明的会主动的帮你抹除啦！因此上面的资讯你只要记得烧录的功能即可。

## 进行 DVD-RW 的烧录动作：

<pre>
# 0. 同样的，先来抹除一下原本的内容：
cdrecord -v dev=ATA:1,1,0 blank=fast

# 1. 开始写入 DVD ，请注意，有些选项与 CD 并不相同了喔！
cdrecord -v dev=ATA:1,1,0 fs=8m -data -sao driveropts=burnfree /tmp/system.img

# 2. 同样的，来给他测试测试！
mount /dev/cdrom /mnt
df -h /mnt
umount /mnt
</pre>