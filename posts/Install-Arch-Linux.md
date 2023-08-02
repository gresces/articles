---
title: "Arch Linux双系统安装简记"
date: 2023-08-02T10:56:39+08:00
draft: false
tags: ["Linux", "计算机"]
description: 记录安装Archlinux与Windows双系统的文章
katex: false
toc: true
summary: |
    记一次Archlinux与Windows双系统的安装
---

## 说明

本文中所描述的安装步骤基本来源于两篇文章[^1][^2]。

本文只是用于下一次安装的快速复现，所以没有图片，如有必要，提供终端输出内容。

## 安装

### 进入安装环境

在Windows下使用Rufus制作启动盘。

并且使用磁盘管理进行分区操作。

**注意这里要关闭UEFI的安全模式*

安装失败可以利用UEFI中的选项进行恢复。

### 确认是否为 UEFI 模式 

```zsh
ls /sys/firmware/efi/efivars
```

输出一堆东西为正常

### 连接网络

```zsh
iwctl 
device list 
station <wlan0> scan 
station <wlan0> get-networks 
station <wlan0> connect <wifi-name>
exit
```

测试网络连通性

```zsh
ping www.bilibili.com
```

### 更新系统时钟

这里安装时先使用ntp来进行联网矫时，在安装完成后更换为rtc时间

```zsh
timedatectl set-ntp true
timedatectl status
```

### 更换国内源

```zsh
vim /etc/pacman.d/mirrorlist
```

在最前面加入

```zsh
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch 
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch 
Server = https://repo.huaweicloud.com/archlinux/$repo/os/$arch 
Server = http://mirror.lzu.edu.cn/archlinux/$repo/os/$arch
```

### 分区与格式化

#### 分区

自带的分区工具体验最好的是`cfdisk`。

|目录   | 大小  |描述 |
|:----|:----|:--|
|/    |67GB |与 /home 共同使用|
|/home|67GB | 与根目录共同使用   |
|/boot|280MB|Windows自带，仅增加内容|
| Swap|8GB  |16GB*0.5|

```zsh
cfdisk /dev/nvme0n1
```

安装完成后的磁盘结构，只有`nvme0n1p8`和`nvme0n1p9`为Linux的空间。

```zsh
❯ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme0n1     259:0    0 476.9G  0 disk
├─nvme0n1p1 259:1    0   260M  0 part /boot
├─nvme0n1p2 259:2    0    16M  0 part
├─nvme0n1p3 259:3    0   198G  0 part
├─nvme0n1p4 259:4    0     2G  0 part
├─nvme0n1p5 259:5    0 150.7G  0 part
├─nvme0n1p6 259:6    0    50G  0 part
├─nvme0n1p7 259:7    0  1000M  0 part
├─nvme0n1p8 259:8    0     8G  0 part [SWAP]
└─nvme0n1p9 259:9    0    67G  0 part /home
                                      /
```

由于本电脑使用NVME协议的安装方式，如为SATA协议可以查看[^1]

注意在这里要在`cfdisk`中将SWAP分区进行指定类型

选中操作 `[Type]` > 然后按下回车 `Enter` > 通过方向键 `↑` 和 `↓` 选中 `Linux swap` > 最后按下回车 `Enter`

#### 格式化

双系统共存在一个硬盘上，不重新格式化原有的 EFI 分区。

```zsh
mkswap /dev/nvme0n1p8
```

将整一个分区格式化为 `Btrfs` 文件系统。

```zsh
mkfs.btrfs -L myArchMain /dev/nvmexn1pn
```

-L 选项后指定该分区的 LABLE，为自定义，但不能使用特殊字符以及空格，且最好有意义

创建子卷（这一步和别的教程不大一样，但可用）

先将 `Btrfs` （将来的\和\home）挂载到 `/mnt` 下

```zsh
mount -t btrfs -o compress=zstd /dev/nvme0n1p9 /mnt

df -h # 复查挂载情况
```

#### 创建 Btrfs 子卷

```zsh
btrfs subvolume create /mnt/@ # 创建 / 目录子卷
btrfs subvolume create /mnt/@home # 创建 /home 目录子卷
```

```zsh
btrfs subvolume list -p /mnt # 复查子卷情况
```

卸载`\mnt`

```zsh
umount /mnt
```

### 挂载

```zsh
mount -t btrfs -o subvol=/@,compress=zstd /dev/nvme0n1p9 /mnt # 挂载 / 目录
mkdir /mnt/home # 创建 /home 目录
mount -t btrfs -o subvol=/@home,compress=zstd /dev/nvme0n1p9 /mnt/home # 挂载 /home 目录
mkdir -p /mnt/boot # 创建 /boot 目录
mount /dev/nvme0n1p1 /mnt/boot # 挂载 /boot 目录
swapon /dev/nvme0n1p8 # 挂载交换分区
```

复查

```zsh
df -h # 复查挂载情况
free -h # 复查 Swap 分区挂载情况
```

### 安装系统

通过如下命令使用 pacstrap 脚本安装基础包：

```bash
pacstrap /mnt base base-devel linux linux-firmware btrfs-progs
```

在这之前可以

```bash
pacman -S archlinux-keyring
```

之后继续安装其他软件

```bash
pacstrap /mnt networkmanager vim sudo zsh zsh-completions man amd-ucode ntfs-3g vi
```

### 生成 fstab 文件

```zsh
genfstab -U /mnt > /mnt/etc/fstab
```

复查

```zsh
cat /mnt/etc/fstab
```

安装完成后的`fstab`文件

```zsh
❯ cat /etc/fstab
# /dev/nvme0n1p9 LABEL=myArchMain
UUID=89bf7cd7-1305-4d05-8f08-1c3fbbe3d5f8	/         	 btrfs     	 rw,relatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvolid=256,subvol=/@	     0 0

# /dev/nvme0n1p9 LABEL=myArchMain
UUID=89bf7cd7-1305-4d05-8f08-1c3fbbe3d5f8	/home     	 btrfs     	 rw,relatime,compress=zstd:3,ssd,discard=async,space_cache=v2,subvolid=257,subvol=/@home	0 0

# /dev/nvme0n1p1 LABEL=SYSTEM_DRV
UUID=4CFD-BDD7      	/boot     	vfat      	 rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro	0 2

# /dev/nvme0n1p8
UUID=c9a68139-6142-4960-87e0-1dbaf6b2be73	none      	 swap      	 defaults  	 0 0
```

### change root

```zsh
arch-chroot /mnt
```

### 设置主机名与时区


主机名

```bash
vim /etc/hostname
```

Hosts文件

```bash
vim /etc/hosts
```

```bash
127.0.0.1   localhost
::1         localhost
127.0.1.1   <arch>.localdomain <arch> # <arch>替换为上一步的hostname
```

时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

### 硬件时间

```bash
hwclock --systohc
```

### Locale

```bash
vim /etc/locale.gen
```

去掉`en_US.UTF-8 UTF-8` 以及 `zh_CN.UTF-8 UTF-8`的注释

```bash
locale-gen
echo 'LANG=en_US.UTF-8'  > /etc/locale.conf
```

### root用户

```bash
passwd root
```

### 微码

由于之前`pacstrap`中安装过，这里安装时可能有问题，重新安装或者忽略即可

```bash
pacman -S intel-ucode # Intel
pacman -S amd-ucode # AMD
```

### 引导设置

```bash
pacman -S grub efibootmgr os-prober

grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH
```

```bash
vim /etc/default/grub
```

 - 去掉 `GRUB_CMDLINE_LINUX_DEFAULT` 一行中最后的 `quiet` 参数
 - 把 `loglevel` 的数值从 `3` 改成 `5`。这样是为了后续如果出现系统错误，方便排错
 - 加入 `nowatchdog` 参数，这可以显著提高开关机速度
 - GRUB_TIMEOUT改为10，增加启动时的等待时间

引导 win10，最后一行 `GRUB_DISABLE_OS_PROBER=false`去掉注释

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

`os-prober`可能暂时没有输出，在安装完成后再次进行`grub-mkconfig`操作即可，判断是否有输出可以单独运行`os-prober`

### 完成安装

```bash
exit # 退回安装环境
umount -R /mnt # 卸载新分区
reboot # 重启
```

再次进入系统后

可以进行之前的`grub`操作以补全boot文件

```bash
systemctl enable --now NetworkManager # 设置开机自启并立即启动 NetworkManager 服务
ping www.bilibili.com # 测试网络连接
```

连接Wifi

```bash
nmtui
```

[^1]:[archlinux 基础安装 ](https://arch.icekylin.online/guide/rookie/basic-install.html)
[^2]:[官方安装指南](https://wiki.archlinuxcn.org/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97)
