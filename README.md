# MacBookPro14,3 Ubuntu 硬件恢复备份

适用对象：2017 年 15 英寸 MacBook Pro（MacBookPro14,3），在 Ubuntu 26.04、Linux 7.0.0-30 上使用。包含 Touch Bar 驱动源码、T1 恢复工具源码及其离线构建源码，以及 Broadcom BCM43602 Wi-Fi 板级配置模板。

## 重要：重装时保留 T1 固件

本仓库不包含设备专属的 `EFI/APPLE/EMBEDDEDOS`、FDRData 或签名票据。T1 当前已修复；重装系统时请勿格式化 EFI 分区或删除其中的 `EFI/APPLE/EMBEDDEDOS`。请在重装之前另行制作加密的离线备份；专属文件不可在不同设备之间复用。下面的驱动源码不能替代这些文件。

## 解压备份

先下载仓库中的 `macbookpro14-ubuntu-driver-backup.tar.gz`，然后运行：

```bash
tar -xzf macbookpro14-ubuntu-driver-backup.tar.gz
cd macbookpro14-driver-backup
```

## 校验文件

在仓库根目录运行 `sha256sum -c SHA256SUMS`。

## Wi-Fi：BCM43602

Ubuntu 内核自带 `brcmfmac` 驱动和固件包；`wifi/brcmfmac43602-pcie.template.txt` 是社区提供的通用板级参数，不是本机原厂校准数据，信号和兼容性可能有所差异。模板中的 MAC 地址是占位符，不能直接使用。使用本机已有的有效地址，或生成一个稳定的本地管理地址；不要使用其他设备的真实地址。

```bash
sudo apt install linux-firmware-broadcom-wireless
mac_suffix=$(od -An -N5 -tx1 /dev/urandom | tr -s ' ' ':' | sed 's/^://')
sed "s/^macaddr=.*/macaddr=02:$mac_suffix/" wifi/brcmfmac43602-pcie.template.txt > /tmp/brcmfmac43602-pcie.txt
sudo install -m 0644 /tmp/brcmfmac43602-pcie.txt /lib/firmware/brcm/brcmfmac43602-pcie.txt
printf 'options cfg80211 ieee80211_regdom=CN\n' | sudo tee /etc/modprobe.d/macbook-wifi-regdom.conf
sudo update-initramfs -u
```

上述地区设置为中国，仅在实际使用地区为中国时使用。重启后使用 `iw list` 查看是否出现 Band 2，并用 `iw dev` 确认连接频率。此次配置下 5 GHz 仅部分信道可用，路由器设为 149 后连接成功。不要在已有同名配置时盲目覆盖；先备份并检查现有设置。

配置来源：<https://gist.github.com/MikeRatcliffe/9614c16a8ea09731a9d5e91685bd8c80>。

## Touch Bar 驱动

先确认 `lsusb` 显示 `05ac:8600`（iBridge）。若显示 `05ac:1281`，T1 处于恢复模式，需要先处理设备专属固件；驱动本身不能修复固件。请勿使用其他电脑的 T1 固件数据。

```bash
mkdir -p /tmp/t1-touchbar-source
tar -xzf vendor/t1-touchbar-source.tar.gz -C /tmp/t1-touchbar-source --strip-components=1
cd /tmp/t1-touchbar-source
sudo ./install.sh --dry-run
sudo ./install.sh
```

安装脚本需要 Ubuntu 软件源、匹配当前内核的头文件、DKMS 和编译工具。驱动必须保留 `skip_acpi_power=1`，不要启用 `ASOC.SOCW(1)`，否则可能造成机器冻结。驱动作者说明通常需要重启才能完成绑定。`usbmuxd` 可能抢占 iBridge；仅在确认冲突时，为其创建本地 udev 规则覆盖，保留其他 Apple 设备匹配。

来源：<https://github.com/AJ-dev-i60/t1-touchbar>。驱动使用上游源码原有许可。

## T1 固件恢复工具

`vendor/t1-revive-source.tar.gz` 为 0.1.5 源码；`vendor/t1-revive-dependencies.tar` 为构建时使用的 8 个固定提交源码归档。该工具官方支持 Arch 系发行版，不正式支持 Ubuntu；本机曾经过人工准备依赖并成功恢复，但不同环境不可保证。固件恢复会写入设备专属数据并可能不可撤销，应先核对作者的说明、备份现有 EFI 并接通电源，不要对正常启动为 `05ac:8600` 的 T1 重复执行恢复。

来源：<https://github.com/niconistal/t1-revive>。
