# Arch Linux 基础安装

让我们从安装最基础的无图形化 ArchLinux 系统开始。[官方安装指南](https://wiki.archlinux.org/index.php/Installation_guide)

## 0.禁用 reflector

reflector 会为你选择速度合适的镜像源，但其结果并不准确，同时会清空配置文件中的内容，我们首先对其进行禁用。

如果安装中出现错误要重启，记得重启之后确保禁用reflector

```bash
systemctl stop reflector.service
```

## 1.再次确保是否为 UEFI 模式

在一系列的信息刷屏后，可以看到已经以 root 登陆安装系统了，此时可以执行的命令：

```bash
ls /sys/firmware/efi/efivars
```

若输出了一堆东西，即 efi 变量，则说明已在 UEFI 模式。否则请确认你的启动方式是否为 UEFI。

## 2.连接网络

一般来说，你连接的网络几乎均可以通过 DHCP 的方式来进行 IP 地址和 DNS 的相关设置，你无需进行额外操作。在没有合适网络的情况下，使用手机的移动热点也是很方便的选择，**特别是针对校园网需要特别认证的请情况**。如果你的网络环境需要配置静态 IP 和 DNS,请自行参考 Arch Wiki。

**一般对于非校园网的情况**，有线连接时，直接插入网线即可。

对于无线连接，则需进行如下操作进行网络连接。

**对于校园网而言**，可以用手机连接wifi再热点共享。

无线连接使用 iwctl 命令进行，按照如下步骤进行网络连接：

```bash
iwctl                                           # 执行iwctl命令，进入交互式命令行
device list                                     # 列出设备名，比如无线网卡看到叫 wlan0
station wlan0 scan                              # 扫描网络
station wlan0 get-networks                      # 列出网络 比如想连接YOUR-WIRELESS-NAME这个无线
station wlan0 connect YOUR-WIRELESS-NAME        # 进行连接 输入密码即可
exit                                            # 成功后exit退出
```

可以等待几秒等网络建立链接后再进行下面测试网络的操作。

```bash
ping www.gnu.org
```

---

**如果**你不能正常连接网络，首先确认系统已经启用网络接口[[1]](https://wiki.archlinux.org/index.php/Network_configuration/Wireless#Check_the_driver_status)。

```bash
ip link                 # 列出网络接口信息，如不能联网的设备叫wlan0
ip link set wlan0 up    # 比如无线网卡看到叫 wlan0
```

**如果**随后看到类似`Operation not possible due to RF-kill`的报错，继续尝试`rfkill`命令来解锁无线网卡。

```bash
rfkill unblock wifi
```

**如果**还是不能，则检查是否分配ip地址，如果没有，则使用dhcpcd
```bash
dhcpcd
```

## 3.更新系统时钟

```bash
timedatectl set-ntp true    # 将系统时间与网络时间进行同步
timedatectl status          # 检查服务状态
```

## 4.分区

这是一个在我个人电脑上实行的方案。此步骤会清除磁盘中全部内容，请事先确认。

- EFI 分区[[2]](https://wiki.archlinux.org/title/EFI_system_partition#Mount_the_partition)： `/efi` 800M
- 根目录： `/` 200G
- 用户主目录： `/home` 剩余全部

首先将磁盘转换为 gpt 类型，这里假设比如你想安装的磁盘名称为 sdx。如果你使用 NVME 的固态硬盘，你看到的磁盘名称可能为 nvme0n1。

```bash
lsblk                       # 显示分区情况 找到你想安装的磁盘名称
parted /dev/sdx             # 执行parted，进入交互式命令行，进行磁盘类型变更
(parted)mktable             # 输入mktable
New disk label type? gpt    # 输入gpt 将磁盘类型转换为gpt 如磁盘有数据会警告，输入yes即可
quit                        # 最后quit退出parted命令行交互

```

接下来使用 cfdisk 命令对磁盘分区。进入 cfdisk 后的操作很直观，用键盘的方向键、Tab 键、回车键配合即可操作分配各个分区的大小与格式。一般建议将 EFI 分区设置为磁盘的第一个分区，据说有些主板如果不将 EFI 设置为第一个分区，可能有不兼容的问题。其中 EFI 分区选择`EFI System`类型，其余两个分区选择`Linux filesystem`类型。

**记得分区之后选择`write`写入磁盘再退出。**

```bash
cfdisk /dev/sdx   # 来执行分区操作,分配各个分区大小，类型
fdisk -l          # 分区结束后， 复查磁盘情况
```

## 5.格式化

建立好分区后，需要对分区用合适的文件系统进行格式化。这里用`mkfs.ext4`命令格式化根分区与 home 分区，用`mkfs.fat`命令格式化 EFI 分区。如下命令中的 sdax 中，x 代表分区的序号。格式化命令要与上一步分区中生成的分区名字对应才可以。

磁盘若事先有数据，会提示你: 'proceed any way?' 按 y 回车继续即可。

```bash
mkfs.ext4  /dev/sdax            # 格式化根目录和home目录的两个分区
mkfs.fat -F 32 /dev/sdax        # 格式化efi分区
```

## 6.挂载

在挂载时，挂载是有顺序的，先挂载根分区，再挂载 EFI 分区。
这里的 sdax 只是例子，具体根据你自身的实际分区情况来。

```bash
mount /dev/sdax /mnt
mkdir /mnt/efi                   # 创建efi目录
mount /dev/sdax /mnt/efi
mkdir /mnt/home                 # 创建home目录
mount /dev/sdax /mnt/home
```

## 7.镜像源的选择

使用如下命令编辑镜像列表：

```bash
vim /etc/pacman.d/mirrorlist
```

此时你可以选择使用[Mirrorlist Generator](https://archlinux.org/mirrorlist/)去生成最新的镜像源，挑选几个非威权国家的镜像源加入到`mirrorlist`中，你可以在[Mirror Status](https://archlinux.org/mirrors/status/)上查看镜像源的状态。

例子如下:
```bash
Server = https://mirror.cov.ukservers.com/archlinux/$repo/os/$arch      # UK
Server = https://mirroir.wptheme.fr/archlinux/$repo/os/$arch            # France
Server = https://arch.yourlabs.org/$repo/os/$arch                       # France
```

## 8.安装系统

必须的基础包

```bash
pacstrap /mnt base base-devel linux linux-headers linux-firmware     # base-devel对于AUR包的安装是必须的
```

必须的功能性软件

```bash
pacstrap /mnt dhcpcd iwd neovim # 一个有线所需(iwd也需要dhcpcd) 一个无线所需 一个编辑器
```

## 9.生成 fstab 文件

fstab 用来定义磁盘分区

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

复查一下 /mnt/etc/fstab 确保没有错误

```bash
cat /mnt/etc/fstab
```

## 10.change root

把环境切换到新系统的/mnt 下

```bash
arch-chroot /mnt
```

为了防止安装时出现蜂鸣声，简单粗暴的卸载pcspkr模块，加入黑名单
```
rmmod pcspkr
echo “blacklist pcspkr” > /etc/modprobe.d/nobeep.conf
```

## 11.时区设置

设置时区，在/etc/localtime 下用/usr 中合适的时区创建符号连接。如下设置上海时区。

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

随后进行硬件时间设置，将当前的正确 UTC 时间写入硬件时间。

```bash
hwclock --systohc
```

## 12.设置 Locale 进行本地化

Locale 决定了地域、货币、时区日期的格式、字符排列方式和其他本地化标准。

首先使用 neovim 编辑 /etc/locale.gen，去掉 en_US.UTF-8 所在行以及 zh_CN.UTF-8 所在行的注释符号（#）。

```bash
nvim /etc/locale.gen
```

然后使用如下命令生成 locale。

```bash
locale-gen
```

最后向 /etc/locale.conf 导入内容

```bash
echo 'LANG=en_US.UTF-8'  > /etc/locale.conf
```

## 13.设置主机名

首先在`/etc/hostname`设置主机名

```bash
nvim /etc/hostname
```

加入你想为主机取的主机名，这里比如叫 `arch`。

接下来在`/etc/hosts`设置与其匹配的条目。

```
nvim /etc/hosts
```

加入如下内容

```bash
127.0.0.1   localhost
::1         localhost
127.0.1.1   arch.localdomain    arch      # 注意这里与你的主机名对应
```

## 14.为 root 用户设置密码

```bash
passwd root
```

## 15.安装微码

```bash
pacman -S intel-ucode   # Intel
pacman -S amd-ucode     # AMD
```

## 16.安装引导程序

```bash
pacman -S grub efibootmgr            #grub是启动引导器，efibootmgr被 grub 脚本用来将启动项写入 NVRAM
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
```

**optional**  接下来编辑/etc/default/grub 文件，去掉`GRUB_CMDLINE_LINUX_DEFAULT`一行中最后的 quiet 参数，同时把 log level 的数值从 3 改成 5。这样是为了后续如果出现系统错误，方便排错。

**recommend** 同时在同一行加入 nowatchdog 参数，这可以显著提高开关机速度。

```bash
nvim /etc/default/grub
```

最后生成 GRUB 所需的配置文件

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

## 17.完成安装

```bash
exit                # 退回安装环境#
umount -R  /mnt     # 卸载新分区
reboot              # 重启
```

注意，重启前要先拔掉U盘，否则你重启后还是进安装程序而不是安装好的系统。

## 18.进入系统

连接网络
```bash
systemctl start dhcpcd  # 立即启动dhcp
ping www.gnu.org        # 测试网络连接
```

若为无线连接，则还需要启动 iwd 才可以使用 iwctl 连接网络

```bash
systemctl start iwd     # 立即启动iwd
iwctl                   # 和之前的方式一样，连接无线网络
```

如果你是`archlinux`单系统，那么到此为止，一个基础的，无 UI 界面的 Arch Linux 已经安装完成了。

如果你是`arch`和`windows`双系统，那么还需要做一些工作来使得`grub`可以探测到`windows`磁盘
```bash
sudo pacman -S os-prober ntfs-3g  # os-prober用来探测，ntfs-3g识别windows的磁盘
```

启用os-prober:默认禁用掉了，将GRUB_DISABLE_OS_PROBER=false前的注释符#去掉。
```bash
sudo nvim /etc/default/grub
```

别忘了生成 GRUB 所需的配置文件

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```
重新启动即可进入到接下来的[图形界面](https://github.com/ausosawx/archlinux-docs/blob/master/WM(wayfire)%26Setting.md)安装配置。
