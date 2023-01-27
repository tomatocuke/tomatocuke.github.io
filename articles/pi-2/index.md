---
title: "树莓派(二): 扩展SD卡剩余空间"
date: "2021-02-07"
categories:
  - Linux
tags:
  - 树莓派

toc: true
---

<!--more-->

我也不是很懂磁盘挂载相关知识，待之后补充， 这里只贴出自己东拼西凑操作成功的记录。
我是16G的内存卡，但是查看磁盘，只有差不多一半可用。

### 操作前磁盘占用情况
```shell
local ) df -h
文件系统        容量  已用  可用 已用% 挂载点
/dev/root       7.0G  6.7G     0  100% /
devtmpfs        1.8G     0  1.8G    0% /dev
tmpfs           1.9G     0  1.9G    0% /dev/shm
tmpfs           1.9G   17M  1.9G    1% /run
tmpfs           5.0M  4.0K  5.0M    1% /run/lock
tmpfs           1.9G     0  1.9G    0% /sys/fs/cgroup
/dev/mmcblk0p1  253M   52M  201M   21% /boot
tmpfs           376M     0  376M    0% /run/user/1000
```
### 操作
1. ``cat /sys/block/mmcblk0/mmcblk0p2/start`` 记录起始位置 (532480)
2. ``fdisk /dev/mmcblk0``
	```shell
	Command (m for help): d
	Partition number (1,2, default 2): 2

	Partition 2 has been deleted.

	Command (m for help): n
	Partition type
	   p   primary (1 primary, 0 extended, 3 free)
	   e   extended (container for logical partitions)
	Select (default p): p
	Partition number (2-4, default 2): 2
	First sector (2048-31116287, default 2048): 532480 (上边记录的起始位置)
	Last sector, +/-sectors or +/-size{K,M,G,T,P} (532480-31116287, default 31116287): (这里回车默认即可)

	Created a new partition 2 of type 'Linux' and of size 14.6 GiB.
	Partition #2 contains a ext4 signature.

	Do you want to remove the signature? [Y]es/[N]o: n

	Command (m for help): w

	The partition table has been altered.
	Syncing disks.
	```
3. ``reboot``
4. 硬盘大小更新 ``resize2fs /dev/mmcblk0p2``

### 操作后磁盘占用情况
```shell
root ) df -h
文件系统        容量  已用  可用 已用% 挂载点
/dev/root        15G  6.7G  7.1G   49% /
devtmpfs        1.8G     0  1.8G    0% /dev
tmpfs           1.9G     0  1.9G    0% /dev/shm
tmpfs           1.9G  8.5M  1.9G    1% /run
tmpfs           5.0M  4.0K  5.0M    1% /run/lock
tmpfs           1.9G     0  1.9G    0% /sys/fs/cgroup
/dev/mmcblk0p1  253M   52M  201M   21% /boot
tmpfs           376M     0  376M    0% /run/user/1000
```
