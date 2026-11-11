# T1 设备专属固件加密备份

本备份来自 MacBookPro14,3 已修复后的 EFI/APPLE/EMBEDDEDOS，只能用于原电脑。固件文件包含设备专属数据，不应公开或用于其他设备。该备份的密码强度较低，请另存一份使用强密码重新加密的离线备份。

## 校验并合并

下载两个 `.part00`、`.part01` 文件和 `T1-FIRMWARE-SHA256SUMS`，在保存它们的目录执行：

```bash
sha256sum -c T1-FIRMWARE-SHA256SUMS
cat t1-device-backup.tar.gpg.part00 t1-device-backup.tar.gpg.part01 > t1-device-backup.tar.gpg
sha256sum t1-device-backup.tar.gpg
```

合并文件的 SHA-256：`c6251f691d69d2254e236a5fad13ea438c00f75ede4cceffe5a9d7447bd395ac`。

## 解密并检查

```bash
mkdir -m 700 t1-private-restore
gpg --output t1-private-restore/t1-efi.tar --decrypt t1-device-backup.tar.gpg
tar -tf t1-private-restore/t1-efi.tar
```

输入备份密码时不要将其写入命令或发布。确认 EFI 分区及现有数据后，才考虑将归档中的 `EFI/APPLE/EMBEDDEDOS` 还原到这台电脑的 EFI 分区；先备份现有目录，切勿格式化 EFI 或覆盖其他电脑的文件。恢复启动效果仍需通过重启验证。
