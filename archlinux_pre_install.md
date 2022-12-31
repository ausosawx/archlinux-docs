# 安装前的准备

由于当前 UEFI 已普及十余年，安装将全部以 UEFI+GPT 的形式进行，传统 BIOS 方式不再赘述。

## 1.确保网络环境

如果你可以使用路由器分接出来的网线，以 dhcp 的方式直接上网，那么不用准备什么。如果你的环境只能使用无线网络安装，需要事先把自己所用的 wifi 名称改成自己能记住的英文名称。因为**安装时无法显示和输入中文名的 wifi**，你会看到一堆不知道是什么的方块，并且在安装过程中你将没有办法输入中文的无线名称进行连接。虽然通过一些繁琐的步骤可以解决终端中文的问题，但是显然这么做在安装 Arch Linux 时毫无必要。

其次，有些笔记本电脑上存在无线网卡的硬件开关或者键盘控制，开机后安装前需要**确保你的无线网卡硬件开关处于打开状态**。

## 2.刻录启动优盘

准备一个 2G 以上的优盘，刻录一个安装启动盘。安装镜像 iso 在[下载页面](https://archlinux.org/download/)下载，你需要选择通过磁力链接或 torrent 下载，下载完成后，还需要在 archlinux 下载页面下载`PGP signature`签名文件(不要从镜像源下载签名文件)，将签名文件和 iso 镜像置于同一文件夹，随后进行对镜像的签名校验，以保证下载的镜像是完整，无错误的，未被篡改的。若你使用 Linux,执行以下命令，确保输出完好的签名。具体镜像名根据名字自行修改。如果你使用其他系统，请自行搜索验证签名的方式。

```bash
gpg --keyserver-options auto-key-retrieve --verify archlinux-202x.0x.01-x86_64.iso.sig
```

注意，这里的签名校验**非常重要**，这可以保证你的安装镜像是未被篡改的，同时可以保证你在使用安装盘安装系统时，用正确的公钥校验安装包。

---

Windows 下推荐使用[ventoy](https://www.ventoy.net/cn/doc_start.html)或者[Rufus](https://rufus.ie/)或者[etcher](https://github.com/balena-io/etcher)进行优盘刻录。三者皆为自由软件。具体操作请自行查阅，都非常简单。

linux下可以采用多种方式进行刻录，可以参考[archwiki](https://wiki.archlinux.org/title/USB_flash_installation_medium)

这里列出比较方便的一种
```bash
cat path/to/archlinux-version-x86_64.iso > /dev/sdx
```

## 3.进入主板 BIOS 进行设置

插入优盘并开机。在开机的时候，按下 F2/F8/F10/DEL 等(取决与你的主板型号，具体请查阅你主板的相关信息)按键，进入主板的 BIOS 设置界面。

## 4.关闭主板设置中的 Secure Boot

在类似名为 `security` 的选项卡中，找到一项名为 Secure Boot(名称可能略有差异)的选项，选择 Disable 将其禁用。

## 5.调整启动方式为 UEFI

在某些旧的主板里，需要调整启动模式为 UEFI,而非传统的 BIOS/CSM。在类似名为 `boot` 的选项卡中，找到类似名为 Boot Mode 的选项，确保将其调整为 UEFI only，而非 legacy/CSM。

## 6.调整硬盘启动顺序

在类似名为 `boot` 的选项卡中，找到类似名为 Boot Options(名称可能略有差异)的设置选项，将 USB 优盘的启动顺序调至首位。

## 7.准备安装

最后保存 BIOS 设置并退出，一般的按键是 F2,F10等。此时系统重启，不出意外你应该已经进入 archlinux 的安装界面。

请参考[基础安装](https://github.com/ausosawx/archlinux-docs/blob/master/basic_install.md)部分继续安装。
