# R7500v1 带 Quantenna 5G 固件云编译

## 目标
为 **Netgear Nighthawk X4 R7500 v1** (`NETGEAR_HW_ID := 29764841+0+128+256+3x3+4x4`)
编译一个**带 Quantenna QV840 5G 驱动**的 OpenWrt 固件。

## 原理
Lean's OpenWrt (`coolsnowwolf/lede`) 源码树自带完整 quantenna 包：

```
package/firmware/quantenna/
├── Makefile                     (PKG_VERSION 37.3.2.44)
├── src/drivers/pcie2/host/arm/  → qdpc-host.ko
├── files/lib/wifi/quantenna.sh
├── files/qtn-utils.init
└── files/qtn.wireless
```

需要的包：
- `kmod-qtn-pcie2`  内核驱动 (DEPENDS @TARGET_ipq806x)
- `qtn-utils`       管理工具 qcsapi_sockrpc
- `qtn-firmware`    u-boot.bin + topaz-linux.lzma.img
- `qtn-proto`       协议栈

## 使用方法

1. 在 GitHub 创建新仓库（或用本目录内容初始化）
2. 推送 `main` 分支
3. Actions → **Build R7500v1 5G** → Run workflow
4. 约 1~2 小时后下载 artifact `r7500-firmware`

## 刷机

产物中会有：

| 文件 | 用途 |
|---|---|
| `*sysupgrade.bin` | 从当前 OpenWrt 网页升级用（推荐） |
| `*factory.img` | 从 Netgear 原厂固件刷入 |
| `*initramfs-uImage` | TFTP 救砖 / 内存运行测试 |

推荐先用 `initramfs-uImage` 走内存测试，确认 5G 能起来再刷 sysupgrade。

## 救砖

- **nmrpflash**（Windows，插网线即可）
  https://github.com/jclehner/nmrpflash
- **TFTP**（U-Boot 开机监听，参考 R7800 wiki）
- **串口 `firmware recovery`**（需拆机）

## 重要提醒

- R7500 v1 用 `Quantenna QT3840BC + QT2518B`（QSR1000 / QV840）
- 官方 OpenWrt stock image **不支持 5G**，只有 Lean lede 这套包能驱动
- 5G 配置在 `/etc/config/wireless` 里为 `radio1`，类型 `quantenna`，设备 `host0`
- LuCI 网页对 quantenna 支持不完整，可能仍需手动编辑配置

## 回滚

刷机前请备份当前配置：
```
ssh root@192.168.5.1
cp /etc/config/* /tmp/  # 或直接用 LuCI 备份
```
