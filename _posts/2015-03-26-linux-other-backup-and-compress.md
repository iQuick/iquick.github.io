---
layout:		post
title:		Linux 其他常见的压缩与备份工具
category:	[Linux]
tags:		[Linux]
published:	true
---
# Linux 其他常见的压缩与备份工具
---

还有一些很好用的工具得要跟大家介绍介绍，尤其是 dd 这个玩意儿呢！

<!--break-->

## dd

dd 可以读取磁碟装置的内容(几乎是直接读取磁区"sector")，然后将整个装置备份成一个文件

```
dd if="input_file" of="output_file" bs="block_size" count="number"

选项与参数：
if   ：就是 input file 罗～也可以是装置喔！
of   ：就是 output file 喔～也可以是装置；
bs   ：规划的一个 block 的大小，若未指定则默认是 512 bytes(一个 sector 的大小)
count：多少个 bs 的意思。
```


* 范例一：将 /etc/passwd 备份到 /tmp/passwd.back 当中

<pre>
dd if=/etc/passwd of=/tmp/passwd.back

ll /etc/passwd /tmp/passwd.back
# 仔细的看一下，我的 /etc/passwd 文件大小为 1945 bytes，因为我没有配置 bs ，
# 所以默认是 512 bytes 为一个单位，因此，上面那个 3+1 表示有 3 个完整的 
# 512 bytes，以及未满 512 bytes 的另一个 block 的意思啦！
# 事实上，感觉好像是 cp 这个命令啦～
</pre>

* 范例二：将自己的磁碟之第一个磁区备份下来

<pre>
[root@www ~]# dd if=/dev/hdc of=/tmp/mbr.back bs=512 count=1
1+0 records in
1+0 records out
512 bytes (512 B) copied, 0.0104586 seconds, 49.0 kB/s
# 第一个磁区内含有 MBR 与 partition table ，透过这个动作，
# 我们可以一口气将这个磁碟的 MBR 与 partition table 进行备份哩！
</pre>

* 范例三：找出你系统最小的那个分割槽，并且将他备份下来：

<pre>
df -h

dd if=/dev/hdc1 of=/tmp/boot.whole.disk

ll -h /tmp/boot.whole.disk

# 等於是将整个 /dev/hdc1 通通捉下来的意思～如果要还原呢？就反向回去！
# dd if=/tmp/boot.whole.disk of=/dev/hdc1 即可！非常简单吧！
# 简单的说，如果想要整个硬盘备份的话，就类似 Norton 的 ghost 软件一般，
# 由 disk 到 disk ，嘿嘿～利用 dd 就可以啦～厉害厉害！
</pre>

## cpio

cpio 可以备份任何东西，包括装置设备文件。不过 cpio 有个大问题， 那就是 cpio 不会主动的去找文件来备份。一般来说， cpio 得要配合类似 find 等可以找到档名的命令来告知 cpio 该被备份的数据在哪里。

<pre>
cpio -ovcB  > [file|device] 	&lt;==备份
cpio -ivcdu &lt; [file|device] 	&lt;==还原
cpio -ivct  &lt; [file|device] 	&lt;==察看


备份会使用到的选项与参数：
  -o ：将数据 copy 输出到文件或装置上 
  -B ：让默认的 Blocks 可以添加至 5120 bytes ，默认是 512 bytes ！ 
　  　 这样的好处是可以让大文件的储存速度加快(请参考 i-nodes 的观念)还原会使用到的选项与参数：
  -i ：将数据自文件或装置 copy 出来系统当中 
  -d ：自动创建目录！使用 cpio 所备份的数据内容不见得会在同一层目录中，因此我们必须要让 cpio 在还原时可以创建新目录，此时就得要 -d 选项的帮助！
  -u ：自动的将较新的文件覆盖较旧的文件！
  -t ：需配合 -i 选项，可用在"察看"以 cpio 创建的文件或装置的内容一些可共享的选项与参数：
  -v ：让储存的过程中文件名称可以在萤幕上显示 
  -c ：一种较新的 portable format 方式储存 
</pre>

范例：找出 /boot 底下的所有文件，然后将他备份到 /tmp/boot.cpio 去！

<pre>
find /boot -print

find /boot | cpio -ocvB > /tmp/boot.cpio
ll -h /tmp/boot.cpio
</pre>

cpio 可以将系统的数据完整的备份到磁带机上头去喔
<pre>
备份：find / | cpio -ocvB > /dev/st0
还原：cpio -idvc < /dev/st0
</pre>