---
layout:		post
title:		Linux 完整备份
category:	[Linux]
tags:		[Linux]
published:	true
---
# Linux 完整备份
---

某些时刻你想要针对文件系统进行备份或者是储存的功能时，不能不谈到这个 dump 命令！

## dump

其实 dump 的功能颇强，他除了可以备份整个文件系统之外，还可以制定等级喔！什么意思啊！ 假设你的 /home 是独立的一个文件系统，那你第一次进行过 dump 后，再进行第二次 dump 时， 你可以指定不同的备份等级，假如指定等级为 1 时，此时新备份的数据只会记录与第一次备份所有差异的文件而已。

![dump]({{ BASE_PATH }}/img/post/linux-full-backups/dump-1.gif)

如上图所示，上方的『即时文件系统』是一直随著时间而变化的数据，例如在 /home 里面的文件数据会一直变化一样。 而底下的方块则是 dump 备份起来的数据，第一次备份时使用的是 level 0 ，这个等级也是完整的备份啦！ 等到第二次备份时，即时文件系统内的数据已经与 level 0 不一样了，而 level 1 仅只是比较目前的文件系统与 level 0 之间的差异后，备份有变化过的文件而已。

<!--break-->

虽然 dump 支持整个文件系统或者是单一各别目录，但是对於目录的支持是比较不足的，这也是 dump 的限制所在。 简单的说，如果想要备份的数据如下时，则有不同的限制情况：

* 当待备份的数据为单一文件系统：

	如果是单一文件系统 (filesystem) ，那么该文件系统可以使用完整的 dump 功能，包括利用 0~9 的数个 level 来备份， 同时，备份时可以使用挂载点或者是装置档名 (例如 /dev/sda5 之类的装置档名) 来进行备份！

* 待备份的数据只是目录，并非单一文件系统：

	例如你仅想要备份 /home/someone/ ，但是该目录并非独立的文件系统时。此时备份就有限制啦！包括：

	* 所有的备份数据都必须要在该目录 (本例为：/home/someone/) 底下；
	* 且仅能使用 level 0 ，亦即仅支持完整备份而已；
	* 不支持 -u 选项，亦即无法创建 /etc/dumpdates 这个各别 level 备份的时间记录档；

<pre>
dump [-Suvj] [-level] [-f 备份档] 待备份数据
dump -W

选项与参数：
-S    ：仅列出后面的待备份数据需要多少磁碟空间才能够备份完毕；
-u    ：将这次 dump 的时间记录到 /etc/dumpdates 文件中；
-v    ：将 dump 的文件过程显示出来；
-j    ：加入 bzip2 的支持！将数据进行压缩，默认 bzip2 压缩等级为 2
-level：就是我们谈到的等级，从 -0 ~ -9 共十个等级；
-f    ：有点类似 tar 啦！后面接产生的文件，亦可接例如 /dev/st0 装置档名等
-W    ：列出在 /etc/fstab 里面的具有 dump 配置的 partition 是否有备份过？
</pre>

## 用 dump 备份完整的文件系统

假如你要将系统的最小的文件系统捉出来进行备份，那该如何进行呢？

<pre>
# 1. 先找出系统中最小的那个文件系统，如下所示：
df -h

# 2. 先测试一下，如果要备份此文件系统，需多少容量？
dump -S /dev/hdc1

# 3. 将完整备份的档名记录成为 /root/boot.dump ，同时升级记录档：
dump -0u -f /root/boot.dump /boot

# 在命令的下达方面，dump 后面接 /boot 或 /dev/hdc1 都可以的！
# 而运行 dump 的过程中会出现如上的一些信息，您可以自行仔细的观察！

# 4. 观察一下系统主动创建的记录档：
cat /etc/dumpdates
</pre>

现在让我们来进行一个测试，检查看看能否真的创建 level 1 的备份

<pre>
# 0. 看一下有没有任何文件系统被 dump 过的数据？
dump -W

# 如上列的结果，该结果会捉出 /etc/fstab 里面第五栏位配置有需要 dump 的 
# partition，然后与 /etc/dumpdates 进行比对，可以得到上面的结果啦！
# 尤其是第三行，可以显示我们曾经对 /dev/hdc1 进行过 dump 的备份动作喔！

# 1. 先恶搞一下，创建一个大约 10 MB 的文件在 /boot 内：
dump -1u -f /root/boot.dump.1 /boot

# 2. 开始创建差异备份档，此时我们使用 level 1 吧：
dump -1u -f /root/boot.dump.1 /boot

# 看看文件大小，岂不是就是刚刚我们所创建的那个大文件的容量吗？ ^_^
ll /root/boot*

# 3. 最后再看一下是否有记录 level 1 备份的时间点呢？
dump -W
</pre>

用 dump 备份非文件系统，亦即单一目录的方法

<pre>
# 让我们将 /etc 整个目录透过 dump 进行备份，且含压缩功能
dump -0j -f /root/etc.dump.bz2 /etc
</pre>

一般来说 dump 不会使用包含压缩的功能，不过如果你想要将备份的空间降低的话，那个 -j 的选项是可以使用的。

## restore

备份档就是在急用时可以回复系统的重要数据，所以有备份当然就得要学学如何复原了！ dump 的复原使用的是 restore 这个命令！

<pre>
restore -t [-f dumpfile] [-h]        &lt;==用来察看 dump 档
restore -C [-f dumpfile] [-D 挂载点] &lt;==比较dump与实际文件
restore -i [-f dumpfile]             &lt;==进入互动模式
restore -r [-f dumpfile]             &lt;==还原整个文件系统

选项与参数：
相关的各种模式，各种模式无法混用喔！例如不可以写 -tC 啦！
-t  ：此模式用在察看 dump 起来的备份档中含有什么重要数据！类似 tar -t 功能；
-C  ：此模式可以将 dump 内的数据拿出来跟实际的文件系统做比较，
      最终会列出『在 dump 文件内有记录的，且目前文件系统不一样』的文件；
-i  ：进入互动模式，可以仅还原部分文件，用在 dump 目录时的还原！
-r  ：将整个 filesystem 还原的一种模式，用在还原针对文件系统的 dump 备份；
其他较常用到的选项功能：
-h  ：察看完整备份数据中的 inode 与文件系统 label 等资讯
-f  ：后面就接你要处理的那个 dump 文件罗！
-D  ：与 -C 进行搭配，可以查出后面接的挂载点与 dump 内有不同的文件！
</pre>

用 restore 观察 dump 后的备份数据内容

<pre>
# 要找出 dump 的内容就使用 restore -t 来查阅即可！例如我们将 boot.dump 的文件内容捉出来看看！

restore -t -f /root/boot.dump 

restore -t -f /root/etc.dump
</pre>

比较差异并且还原整个文件系统

<pre>
# 0. 先尝试变更文件系统的内容：
cd /boot
mv config-2.6.18-128.el5 config-2.6.18-128.el5-back

# 1. 看使进行文件系统与备份文件之间的差异！
restore -C -f /root/boot.dump

# 2. 将文件系统改回来啊！
mv config-2.6.18-128.el5-back config-2.6.18-128.el5
cd /root
</pre>

如同上面的动作，透过曾经备份过的资讯，也可以找到与目前实际文件系统中有差异的数据呢！ 如果你不想要进行累积备份，但也能透过这个动作找出最近这一阵子有变动过的文件说！了解乎？ 那如何还原呢？由於 dump 是记录整个文件系统的，因此还原时你也应该要给予一个全新的文件系统才行。 

<pre>
# 1. 先创建一个新的 partition 来使用，假设我们需要的是 150M 的容量
fdisk /dev/hdc
partprobe   &lt;==很重要的动作！别忘记！

# 这样就能够创建一个 /dev/hdc8 的 partition ，然后继续格式化吧！
mkfs -t ext3 /dev/hdc8
mount /dev/hdc8 /mnt

# 2. 开始进行还原的动作！请您务必到新文件系统的挂载点底下去！
cd /mnt
restore -r -f /root/boot.dump
</pre>

仅还原部分文件的 restore 互动模式
某些时候你只是要将备份档的某个内容捉出来而已，并不想要全部解开，那该如何是好？此时你可以进入 restore 的互动模式 (interactive mode)。

<pre>
cd /mnt
restore -i -f /root/etc.dump
</pre>