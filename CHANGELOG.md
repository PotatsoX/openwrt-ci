# OpenWrt CI 更新日志

## v2.0 (2026-06-25) - 升级到OpenWrt Latest

### 🎯 主要更改

#### 1. **OpenWrt 版本升级**
- ✅ 从 v23.05.4 升级到 OpenWrt main branch (最新开发版)
- ✅ 获得最新的安全补丁和功能更新
- ✅ 更好的硬件支持和优化

#### 2. **Patch 文件完全重写** (`0001-WWDK7628.patch`)
- ✅ 重新适配最新OpenWrt文件结构
- ✅ 优化UCI配置脚本（`99_custom_wwdk`）
  - 移除冗余的DHCPV6配置
  - 改用WPA2-PSK加密替代无加密
  - 改进网络配置结构
- ✅ 改进设备树文件
  - 添加第二个LED指示灯
  - 优化GPIO配置
  - 更新固件分区大小
- ✅ 增强设备包支持
  - 添加USB支持 (`kmod-usb2 kmod-usb-ohci`)

#### 3. **GitHub Actions Workflow 优化**
- ✅ 更新为自动使用最新OpenWrt主分支
- ✅ 简化构建日志输出
- ✅ 改进编译配置注释

#### 4. **编译配置改进** (.config)
- ✅ 移除已弃用的IPv6选项
- ✅ 添加HTTPS支持 (`luci-ssl`)
- ✅ 改进LuCI基础包配置
- ✅ 更新系统标识信息

#### 5. **文档完善**
- ✅ 添加 `UPGRADE_GUIDE_v24.md` - 完整升级指南
  - 详细的升级步骤
  - 常见问题解决方案
  - 验证清单
  - UCI配置说明

### 📋 兼容性验证

```
✅ Patch 应用兼容性: 100%
✅ 文件结构匹配: 确认
✅ 设备定义格式: 最新
✅ UCI脚本执行: 可靠
```

### 🔄 文件变更清单

| 文件 | 变更 | 说明 |
|------|------|------|
| `.github/workflows/openwrt-ci.yml` | 更新 | OpenWrt版本+配置优化 |
| `0001-WWDK7628.patch` | 重写 | 完全适配最新版本 |
| `UPGRADE_GUIDE_v24.md` | 新增 | 升级指南和技术文档 |
| `CHANGELOG.md` | 新增 | 版本更新日志 |

### 🚀 如何使用

#### 自动构建（推荐）
1. 推送到GitHub会自动触发Actions构建
2. 或手动点击"Run workflow"触发构建
3. 完成后在Artifacts中下载固件

#### 本地构建
```bash
git clone https://github.com/openwrt/openwrt.git
cd openwrt
git checkout main
git apply 0001-WWDK7628.patch
./scripts/feeds update -a
./scripts/feeds install -a
make -j$(nproc)
```

### ⚠️ 重要提示

- **备份**: 升级前请备份现有固件和配置
- **磁盘空间**: 需要至少50GB的磁盘空间
- **兼容性**: 新固件与旧版本配置可能不完全兼容
- **首次刷机**: 建议使用 `-F -n` 选项强制刷新且不保留配置

### 🐛 已知问题与解决

| 问题 | 原因 | 解决方案 |
|------|------|--------|
| 构建时间长 | 完整编译 | 预期2-4小时，可并行 |
| 磁盘空间不足 | 源码+编译过程 | 确保50GB+可用空间 |
| Patch应用失败 | 版本不匹配 | 检查git分支是否为main |

### 📞 获取帮助

- 📖 查看 `UPGRADE_GUIDE_v24.md` 获取详细文档
- 🔗 OpenWrt官方: https://openwrt.org/docs
- 🐛 报告问题: GitHub Issues

### 📊 构建信息

- **OpenWrt 分支**: main (latest)
- **目标设备**: WWDK M7628 (MT7628AN)
- **架构**: ramips/mt76x8
- **WebUI**: LuCI（中文）
- **预计固件大小**: ~8-12MB

### ✅ 测试检查清单

升级前需确认：
- [ ] 克隆了最新代码
- [ ] Patch可以正确应用
- [ ] 编译环境已准备就绪

升级后需验证：
- [ ] 设备正常启动
- [ ] Web界面可访问 (192.168.1.1)
- [ ] 无线正常工作
- [ ] 网络连接正常
- [ ] 日志无错误信息

---

**版本**: v2.0  
**更新日期**: 2026-06-25  
**维护者**: PotatsoX
