# OpenWrt v24+ 升级指南 (WWDK M7628)

## 升级信息

**目标版本**: OpenWrt latest (main branch)  
**设备**: WWDK M7628 (MT7628AN)  
**更新日期**: 2026-06-25  
**前版本**: v23.05.4  

---

## 📝 主要更新

### 1. **Workflow 配置变更** (.github/workflows/openwrt-ci.yml)

| 项目 | v23.05.4 | main | 说明 |
|------|----------|------|------|
| OpenWrt 版本 | v23.05.4 | main | 使用最新开发分支获得最新特性和安全补丁 |
| Build Domain | Geophyson | WWDK | 更新为设备制造商标识 |
| UCI 脚本 | 99_custom | 99_custom_wwdk | 重新命名避免冲突 |

### 2. **Patch 文件优化** (0001-WWDK7628.patch)

#### UCI 配置改进
```bash
# ✅ 改进前后对比

# 旧版本问题:
- DHCPV6 配置重复
- encryption='none' 不安全
- 硬编码 SSID 和密码
- 缺少网络分界线（LAN/WAN）

# 新版本优化:
✓ 使用 psk2（WPA2-PSK） 加密
✓ 清晰的网络配置分界
✓ 支持 IPv6（可选）
✓ 改进的评论和结构化配置
✓ 添加 DEVICE_PACKAGES（USB 支持）
```

#### 设备树文件更新
- ✅ 添加第二个 LED 指示灯（LAN）
- ✅ 改进 GPIO 组配置
- ✅ 更新固件分区大小（0xfb0000）
- ✅ 改进的代码注释

#### 图像配置改进
```makefile
# 新增 USB 支持
DEVICE_PACKAGES := uboot-envtools kmod-usb2 kmod-usb-ohci
```

### 3. **编译配置优化** (.config)

```ini
# 移除已弃用的选项:
- CONFIG_KERNEL_IPV6_MULTIPLE_TABLES (不再需要)
- CONFIG_KERNEL_IPV6_SUBTREES (不再需要)
- CONFIG_KERNEL_IPV6_MROUTE (不再需要)
- CONFIG_PACKAGE_kmod-mppe (PPP 相关)

# 添加现代化选项:
+ CONFIG_PACKAGE_luci-base (LuCI 基础)
+ CONFIG_PACKAGE_luci-ssl (HTTPS 支持)
```

---

## 🔍 兼容性检查

### 文件结构验证

```
✅ target/linux/ramips/dts/mt7628an_wwdk_m7628.dts
   - 位置: 相同 ✓
   - 格式: 保持兼容 ✓

✅ target/linux/ramips/image/mt76x8.mk
   - 位置: 相同 ✓
   - 设备定义格式: 保持兼容 ✓

✅ target/linux/ramips/mt76x8/base-files/etc/board.d/02_network
   - 位置: 相同 ✓
   - 添加设备: wwdk,m7628 ✓

✅ package/base-files/files/etc/uci-defaults/99_custom_wwdk
   - 位置: 相同 ✓
   - 脚本可执行性: ✓ (755 权限)
```

---

## 🚀 升级步骤

### 方式 1: 使用提供的 Patch（推荐）

```bash
# 1. 克隆最新 OpenWrt
git clone https://github.com/openwrt/openwrt.git
cd openwrt
git checkout main  # 或最新稳定版本

# 2. 应用 Patch
git apply /path/to/0001-WWDK7628.patch

# 3. 更新 feeds
./scripts/feeds update -a
./scripts/feeds install -a

# 4. 编译
make menuconfig  # (可选) 调整配置
make download
make -j$(nproc)
```

### 方式 2: GitHub Actions（自动构建）

Workflow 已配置为：
- 监听仓库 watch 事件自动构建
- Release 发布时自动构建
- 支持手动触发 (workflow_dispatch)

---

## 📦 编译产物

构建完成后，产物包括：

```
bin/targets/ramips/mt76x8/
├── openwrt-ramips-mt76x8-wwdk_m7628-squashfs-sysupgrade.bin  (固件)
├── openwrt-ramips-mt76x8-wwdk_m7628-initramfs-kernel.bin      (内存启动)
└── ...build info files
```

---

## ⚙️ 刷机步骤

### 从原固件升级

```bash
# 通过 Web 界面或 SSH:
sysupgrade -F -n openwrt-ramips-mt76x8-wwdk_m7628-squashfs-sysupgrade.bin

# -F: 强制升级（即使版本号相同）
# -n: 不保留配置文件（推荐第一次升级）
```

### 首次启动配置

刷机后首次启动：

1. **连接设备**
   ```
   IP: 192.168.1.1
   用户名: root
   密码: (空)
   ```

2. **配置 WAN 口**
   - 系统 → 网络 → 接口 → WAN
   - 设置为 DHCP 或固定 IP

3. **配置无线**
   - 网络 → 无线 → 修改 SSID 和密码

---

## 🐛 常见问题

### Q1: Patch 应用失败？

**原因**: 目标版本中文件结构不匹配

```bash
# 解决方案:
git apply --check 0001-WWDK7628.patch  # 检查兼容性
git apply -p1 0001-WWDK7628.patch     # 使用不同的路径前缀
```

### Q2: 编译失败？

**常见原因**:
- 缺少依赖库
- feeds 未更新
- 磁盘空间不足（需要 50GB+）

```bash
# 解决方案:
./scripts/feeds update -a
./scripts/feeds install -a
make clean
make -j1 V=s  # 详细输出以找出错误
```

### Q3: 设备启动异常？

**检查项**:
- 无线 MAC 地址是否正确读取（来自 factory 分区）
- 重置按钮是否正常工作
- LED 状态是否符合预期

```bash
# 调试命令:
logread -f  # 查看实时日志
iwconfig    # 检查无线状态
```

---

## 📋 UCI 配置脚本说明

**文件**: `99_custom_wwdk`  
**位置**: `/etc/uci-defaults/`  
**执行时机**: 首次启动或恢复出厂设置  
**权限**: 可执行 (755)

### 配置项

| 配置 | 默认值 | 说明 |
|------|--------|------|
| 系统名 | WWDK-M7628 | 显示在 Web 界面 |
| 时区 | CST-8 | 中国标准时区 |
| WLAN SSID | WWDK-M7628 | 可修改 |
| 加密 | psk2 | WPA2-PSK（推荐） |
| LAN IP | 192.168.1.1 | 管理员访问地址 |
| WAN | DHCP | 自动获取 IP |

---

## 🔄 回退到旧版本

如需回退到 v23.05.4：

```bash
cd openwrt
git checkout v23.05.4
git apply 0001-WWDK7628-v23.patch  # 使用旧版 patch
make clean && make -j$(nproc)
```

---

## ✅ 验证清单

升级前检查：

- [ ] 备份现有固件和配置
- [ ] 确保有足够的磁盘空间 (50GB+)
- [ ] 编译环境已正确配置
- [ ] Patch 文件完整性已验证

升级后检查：

- [ ] 设备正常启动
- [ ] Web 界面可访问
- [ ] 无线正常工作
- [ ] 网络连接正常
- [ ] LED 指示灯状态正确
- [ ] 日志中无错误信息

---

## 📞 技术支持

如遇问题，请检查：

1. OpenWrt 官方文档: https://openwrt.org/docs
2. 设备社区论坛
3. GitHub Issues: https://github.com/openwrt/openwrt/issues

---

**文档版本**: 2.0  
**最后更新**: 2026-06-25
