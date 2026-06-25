# 🚀 快速开始指南

## OpenWrt v24+ 编译和刷机

### 📦 快速编译

```bash
# 1. 克隆OpenWrt最新版本
git clone https://github.com/openwrt/openwrt.git
cd openwrt

# 2. 应用WWDK M7628 patch
git apply /path/to/0001-WWDK7628.patch

# 3. 更新feeds
./scripts/feeds update -a
./scripts/feeds install -a

# 4. 配置和编译
make menuconfig          # 可选，调整编译选项
make download -j8        # 下载源码
make -j$(nproc)          # 编译（使用所有CPU核心）
```

### 💾 刷机步骤

#### 使用Web界面刷机（推荐新手）
1. 连接到设备Web界面: `http://192.168.1.1`
2. 进入 **系统** → **备份/升级** → **刷写新的固件**
3. 上传固件文件: `openwrt-ramips-mt76x8-wwdk_m7628-squashfs-sysupgrade.bin`
4. 勾选 **不保留设置** 
5. 点击 **刷写** 并等待完成

#### 使用SSH命令刷机（高级）
```bash
# SSH连接到设备
ssh root@192.168.1.1

# 刷机命令
sysupgrade -F -n openwrt-ramips-mt76x8-wwdk_m7628-squashfs-sysupgrade.bin

# 参数说明：
# -F: 强制刷新（即使版本号相同）
# -n: 不保留设置
```

#### 紧急恢复（U-Boot）
如果设备无法正常启动：
1. 按住**重置按钮**不放
2. 等待**LED**闪烁
3. 使用TFTP工具上传固件到192.168.1.1

### ⚙️ 首次配置

刷机后首次访问：

```
IP地址: 192.168.1.1
用户名: root
密码: (空，直接回车)
```

**首次登录后需配置：**

1. **设置密码**
   - 系统 → 管理员密码

2. **配置WAN口** (互联网连接)
   - 网络 → 接口 → WAN
   - 设置为DHCP或固定IP

3. **配置无线**
   - 网络 → 无线 → 修改SSID和密码
   - 建议密码长度8字符以上

### 🔧 常用命令

```bash
# 查看系统状态
logread -f                # 查看实时日志
uci show network         # 查看网络配置
iwconfig                  # 查看无线状态

# 重启网络
/etc/init.d/network restart

# 重启无线
wifi restart

# 恢复出厂设置
factory reset (通过Web) 或
mtd -r erase rootfs_data (通过SSH)
```

### 📁 编译产物位置

```
bin/targets/ramips/mt76x8/
├── openwrt-ramips-mt76x8-wwdk_m7628-squashfs-sysupgrade.bin    ✅ 刷机用
├── openwrt-ramips-mt76x8-wwdk_m7628-initramfs-kernel.bin        (内存启动)
├── openwrt-ramips-mt76x8-wwdk_m7628.buildinfo
├── openwrt-ramips-mt76x8-wwdk_m7628.manifest
└── ... (其他)
```

### ⚡ 性能优化编译

```bash
# 使用4倍任务数加速编译
make -j$(nproc) -l$(nproc)

# 或显式指定
make -j16 -l16

# 仅编译固件（跳过packages）
make -j$(nproc) IGNORE_ERRORS=1
```

### 🆘 故障排除

**Q: 编译失败怎么办？**
```bash
# 清理后重试
make clean
make distclean
make -j1 V=s  # 使用详细输出
```

**Q: 找不到编译产物？**
```bash
find bin/ -name "*wwdk_m7628*"
```

**Q: 设备无法启动？**
```bash
# 通过SSH查看日志
ssh root@192.168.1.1
dmesg | tail -50
```

### 🎯 验证编译成功

```bash
# 检查文件大小
ls -lh bin/targets/ramips/mt76x8/*.bin

# 检查文件完整性
md5sum bin/targets/ramips/mt76x8/*.bin
```

### 📖 更多文档

- 完整升级指南: [`UPGRADE_GUIDE_v24.md`](UPGRADE_GUIDE_v24.md)
- 版本更新日志: [`CHANGELOG.md`](CHANGELOG.md)
- 官方文档: https://openwrt.org/docs/guide-user/start

---

**快速参考版本**: 1.0  
**最后更新**: 2026-06-25
