# ImmortalWrt for Gemtek XG2010G

基于 [ImmortalWrt](https://github.com/immortalwrt/immortalwrt) 上游的 Gemtek XG2010G 移植项目。

> 当前状态：移植基线阶段。设备树、分区和镜像定义已建立；首次刷写前必须使用串口确认启动参数，并完整备份原厂 NAND 分区。

## 硬件概览

| 项目 | 已确认信息 |
| --- | --- |
| SoC | Airoha/Econet AN7581 / EN7581 |
| CPU | 4x Cortex-A53，ARM64 |
| 内存 | 1 GiB DDR4-2666，起始地址 `0x80000000` |
| 闪存 | Winbond W25N04K SPI-NAND，512 MiB |
| 10G 网口 | RTL8261N，MDIO 地址 5 和 8 |
| 2.5G 网口 | Airoha EN8811H，MDIO 地址 `0xf` |
| 1G 网口 | 内置交换机 ePHY，实测仅 LAN4 出线 |
| PON | EN7572 BOSA，原厂 XGSPON |
| 无线/USB/eMMC | 当前硬件资料显示无可用模块或物理接口 |

## 端口与启动注意事项

- phy@5 对应原厂 `eth0.8`，phy@8 对应 `eth0.7`；两者的物理 LAN 编号仍建议上机复核。
- `serdes_ethernet=411` 是原厂实机参数，不要照抄旧资料中的 `421`。
- 原厂 NAND 布局与本项目 UBI 布局不同。刷写前备份 `bootloader`、`uenv`、`dsd`、`tclinux_slave` 和 `art`。
- U-Boot 可能通过 `bootflag` 在主/备系统间切换。任何升级操作都应保留可恢复的串口/TFTP/HTTP Recovery 路径。
- XG2010G 的 PON 模式、BOSA 校准、SLIC/语音和 10G PHY 链路尚未在 ImmortalWrt 上完成实机验证。

## 移植资料

- [XG2010G 逆向分析](https://github.com/naoki66/ImmortalWrt-for-Gemtek-XG2010G/blob/master/XG2010G-ANALYSIS.md)
- [DTS 逆向说明](https://github.com/naoki66/ImmortalWrt-for-Gemtek-XG2010G/blob/master/XG2010G-DTS-REVERSE.md)
- [固件重打包说明](https://github.com/naoki66/ImmortalWrt-for-Gemtek-XG2010G/blob/master/XG2010G-FIRMWARE-REPACK.md)
- 原始解包与 SDK 参考资料位于本地工作目录的 `XG2010G-fw-mod`，不直接纳入固件构建输入。

## 本地构建

请在 Debian/Ubuntu 或 WSL 的 Linux 原生文件系统中构建，不要在 `/mnt/c`、`/mnt/d` 等 Windows 挂载目录构建。

```bash
sudo apt update
sudo apt install build-essential clang gcc g++ binutils bzip2 gawk gettext git \
  libncurses-dev libssl-dev python3 python3-setuptools rsync unzip wget \
  xsltproc zlib1g-dev file which perl sed make
git clone https://github.com/naoki66/ImmortalWrt-for-Gemtek-XG2010G.git
cd ImmortalWrt-for-Gemtek-XG2010G
./scripts/feeds update -a
./scripts/feeds install -a
make menuconfig
make -j$(nproc) world
```

目标设备位于 `Airoha -> an7581 -> Gemtek XG2010G`，输出目录为 `bin/targets/airoha/an7581/`。

## GitHub Actions

- `Build Firmware`：手动触发，执行依赖安装、feeds、配置和固件构建，并上传构建产物。
- `Sync Upstream`：定时或手动将 `immortalwrt/immortalwrt` 的 `master` 合并到本仓库。

Actions 构建只验证编译产物，不代表设备已经可以安全刷写；刷机前仍需串口验证 DTS、分区和启动链。

## 许可证

本项目继承 ImmortalWrt 的 GPL-2.0-only 许可证。硬件分析资料和原厂 SDK 的版权归其原作者/版权所有者所有。
