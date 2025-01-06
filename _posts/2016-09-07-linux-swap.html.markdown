---
title: linux swap
date: 2016-09-07
tags: linux
---

### Ubuntu增加虚拟内存
```
Swap作为交换区，太有用了，功能类似windows下的虚拟内存，就是利用硬盘空间来扩展内存。

阿里云的系统默认都不使用swap，为了让你多买内存，而内存是最贵的一项。

如果是性能控可以掠过，毕竟阿里云的磁盘io真的不咋的。

我的ecs就挂了几个小网站，总是内存报警然后mysql挂掉，必须上swap了。

下面是步骤

首先移除关闭swap的选项，否则后面的设置会在重启后失效，在/etc/rc.local文件中移除swapoff -a 行。

然后在根目录创建swap目录，并在swap下划分一个1024MB的连续空间给swap使用，这里的1024可以根据你的内存大小来，一般两倍就够。
dd if=/dev/zero of=swapfile bs=1M count=1024
创建这个swapfile

mkswap swapfile
挂起

swapon swapfile
在free -m中查看是否已经生效

现在再用free -m命令查看一下内存和swap分区大小，就发现增加了1024M的空间了。不过当计算机重启了以后，发现swap还是原来那么大，新的swap没有自动启动，还要手动启动。那我们需要修改/etc/fstab文件，增加如下一行

/mnt/swap swap swap defaults 0 0

你就会发现你的机器自动启动以后swap空间也增大了。
除此以外还需要设置swapness，范围0-100，数值越大使用swap越积极
可以通过cat /proc/sys/vm/swappiness来查看当前数值
如需修改在文件/etc/sysctl.conf的最后加上这样一行:
vm.swappiness=10
重启后生效
手动改变swappiness的值,重启后丢失
sudo sysctl vm.swappiness=10
```
