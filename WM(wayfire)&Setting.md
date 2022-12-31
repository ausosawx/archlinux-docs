# WM(wayfire)的安装配置

官方文档: [安装后的工作](https://wiki.archlinux.org/index.php/General_recommendations)


## 1.确保系统为最新

如果你在做完上一节的内容后，重启并放置过一段时间，那需要先按照上节末尾处的方式重新连接网络，然后更新系统。

```bash
pacman -Syu    # 升级系统中全部包
```

## 2.准备非 root 用户

添加用户，比如新增加的用户叫 wx

```bash
useradd -m -G wheel -s /bin/bash wx  # wheel附加组可sudo，以root用户执行命令 -m同时创建用户家目录
```

设置新用户 wx 的密码

```bash
passwd wx
```

编辑 sudoers 配置文件

```bash
EDITOR=nvim visudo  # 需要以 root 用户运行 visudo 命令
```

找到下面这样的一行，把前面的注释符号 `#` 去掉，`:wq` 保存并退出即可。

```sudoers
#%wheel ALL=(ALL:ALL) ALL
```

## 3.开启 32 位支持库

```bash
nvim /etc/pacman.conf
```

去掉[multilib]一节中两行的注释，来开启 32 位库支持。

最后:wq 保存退出，刷新 pacman 数据库

```bash
pacman -Syu
```

## 4.设置 DNS
nvim 编辑/etc/resolv.conf，删除已有条目，并将如下内容加入其中
```bash
nameserver 8.8.8.8
nameserver 2001:4860:4860::8888
nameserver 8.8.4.4
nameserver 2001:4860:4860::8844
```
如果你的路由器可以自动处理 DNS,resolvconf 会在每次网络连接时用路由器的设置覆盖本机/etc/resolv.conf 中的设置，执行如下命令加入不可变标志，使其不能覆盖如上加入的配置。
```bash
sudo chattr +i /etc/resolv.conf
```

## 5.wayfire前置

首先进行网络配置

安装networkmanager
```bash
sudo pacman -S networkmanager
```

```bash
sudo systemctl disable iwd                                                  # 确保iwd开机处于关闭状态，其无线连接会与NetworkManager冲突
sudo systemctl stop iwd                                                     # 同上，立即关闭iwd
sudo systemctl enable --now NetworkManager                                  # 确保先启动NetworkManager，并进行网络连接 若iwd已经与NetworkManager冲突 则执行完上一步重启一下电脑即可。
nmtui                                                                       # 图形化网络配置
```


添加[archlinuxcn源](https://www.archlinuxcn.org/archlinux-cn-repo-and-mirror/)

在 /etc/pacman.conf 文件末尾添加以下两行
```bash
[archlinuxcn]
Server = https://repo.archlinuxcn.org/$arch

然后安装keyring
sudo pacman -Syu
sudo pacman -S archlinuxcn-keyring
```

注意添加后安装archlinuxcn-keyring报错的话执行下列命令再安装
```bash
sudo rm -rf /etc/pacman.d/gnupg
sudo pacman-key --init
sudo pacman-key --populate
```

在做进一步的图形化之前，对系统可以做一个最精简的备份(因为后来确实可能遇到某些问题不容易解决而且即使恢复系统到前不久也没有用，因此保存一个此时的备份可以免去安装的重复性工作)
```bash
sudo pacman -S timeshift
```
注意安装完成以后需要`systemctl enable --now cronie.service`, 这在完成安装时会有提示
然后使用`timeshift`命令行进行备份，注意`timeshift`必须在`root`用户(不是`sudo`)下运行。可以命令行输入`timeshift --help`查看具体备份方法。

注意如果你使用的不是`linux`可以识别的磁盘，需要按照之前安装步骤中的方法将其格式化为相应格式，比如`ext4`。

## 8.配置文件恢复
参考我的[dotfiles](https://github.com/ausosawx/dotfiles)
