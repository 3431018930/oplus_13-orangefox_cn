# 🦊 OnePlus 13 (PJZ110) OrangeFox Recovery 自动构建工作流

## 📋 工作流说明

本工作流专门为 **OnePlus 13 (PJZ110)** 设备定制，用于自动化编译 OrangeFox Recovery。所有步骤提示均为中文，方便查看和理解。

## 🚀 触发方式

### 1. 自动触发
当你向 `main` 分支推送代码时，工作流会自动运行：
```bash
git add .
git commit -m "更新设备树"
git push origin main
```

### 2. 手动触发
在 GitHub 仓库页面：
1. 点击 **Actions** 标签
2. 选择 **构建 OnePlus 13 (PJZ110) OrangeFox Recovery**
3. 点击 **Run workflow** 按钮
4. 配置以下参数（可选）：
   - **OrangeFox 分支版本**: 选择 `12.1` 或 `14.1`（默认：`14.1`）
   - **Magisk 版本**: 输入 Magisk 版本号（默认：`v28.1`）
   - **启用 CCache 加速编译**: 是否启用 CCache（默认：`true`）

## 📦 工作流步骤

| 步骤 | 说明 | 执行内容 |
|------|------|----------|
| 1️⃣ | 检出仓库代码 | 从 GitHub 克隆设备树代码 |
| 2️⃣ | 安装构建依赖 | 安装编译所需的系统软件包 |
| 3️⃣ | 配置 Git | 设置 Git 用户信息 |
| 4️⃣ | 同步 OrangeFox 源码 | 使用官方脚本同步 OrangeFox 源码树 |
| 5️⃣ | 克隆 Android 13+ 依赖 | 仅在 12.1 分支时执行，更新依赖 |
| 6️⃣ | 修补源代码 | 仅在 12.1 分支时执行，应用补丁 |
| 7️⃣ | 克隆设备树 | 将设备树复制到源码树 |
| 8️⃣ | 更新内置 Magisk | 更新 OrangeFox 内置的 Magisk 版本 |
| 9️⃣ | 更新内置 magiskboot | 更新 magiskboot 工具 |
| 🔟 | 配置 CCache | 启用编译缓存以加速编译（可选） |
| 1️⃣1️⃣ | 编译 OrangeFox Recovery | 执行完整编译流程 |
| 1️⃣2️⃣ | 显示 CCache 统计 | 显示缓存使用情况 |
| 1️⃣3️⃣ | 设置发布信息 | 准备发布相关信息 |
| 1️⃣4️⃣ | 检查编译产物 | 验证生成的文件 |
| 1️⃣5️⃣ | 上传到 Release | 创建 GitHub Release 并上传文件 |
| 1️⃣6️⃣ | 上传到 Artifacts | 备份编译产物（保留 30 天） |
| ✅ | 构建完成 | 显示构建摘要信息 |

## 🎯 编译产物

编译成功后，会生成以下文件：

- `OrangeFox-*.img` - Recovery 镜像文件
- `OrangeFox-*.zip` - 可通过 Recovery 刷入的卡刷包

这些文件会自动上传到：
1. **GitHub Release**: 永久保存，带有详细的发布说明
2. **GitHub Artifacts**: 保留 30 天，用于备份

## 📥 下载编译产物

### 从 Release 下载
1. 进入仓库的 **Releases** 页面
2. 找到最新的发布版本
3. 在 **Assets** 下载所需文件

### 从 Artifacts 下载
1. 进入 **Actions** 标签
2. 点击最近的工作流运行记录
3. 在底部的 **Artifacts** 区域下载

## 🔧 自定义配置

### 修改 Magisk 版本
编辑工作流文件中的默认值：
```yaml
MAGISK_VERSION:
  default: 'v28.1'  # 修改为你需要的版本
```

### 修改 OrangeFox 分支
编辑环境变量：
```yaml
MANIFEST_BRANCH: ${{ inputs.MANIFEST_BRANCH || '14.1' }}
```
将 `'14.1'` 改为你需要的默认分支。

### 禁用 CCache
如果不需要编译缓存，设置：
```yaml
USE_CCACHE: ${{ inputs.USE_CCACHE || false }}
```

## ⚙️ 设备特定配置

本工作流已针对 **OnePlus 13 (PJZ110)** 进行以下定制：

| 配置项 | 值 |
|--------|-----|
| 设备代号 | `PJZ110` |
| 设备路径 | `device/oplus/PJZ110` |
| 厂商 | `oplus` |
| 编译目标 | `twrp_PJZ110-eng` |
| 构建类型 | `recoveryimage` |

## 📱 设备功能支持

已验证功能：
- ✅ 显示
- ✅ 触摸（包括 FastbootD）
- ✅ 解密
- ✅ 刷机
- ✅ 备份与恢复
- ✅ MTP/OTG 存储
- ✅ ADB/FastbootD
- ✅ 恢复出厂设置
- ✅ 振动器
- ✅ 显示与振动设置

## 🛠️ 内置工具

设备树中包含以下 Root 工具（位于 `/FFiles/`）：
- **KernelSU** - KernelSU 安装包
- **KernelSU_uninstaller** - KernelSU 卸载程序
- **KernelSUN** - KernelSU 变体
- **SukiSU** - SukiSU Root 方案

## ⚠️ 注意事项

1. **编译时间**: 完整编译通常需要 2-4 小时，取决于 GitHub Actions 服务器负载
2. **存储空间**: 编译需要约 100GB 磁盘空间，GitHub Actions 免费提供
3. **并发限制**: 免费账户同时只能运行有限数量的工作流
4. **分支兼容**: 12.1 分支会自动应用额外的补丁和依赖更新
5. **网络问题**: 如果同步失败，工作流会自动重试

## 🐛 故障排除

### 工作流运行失败
1. 检查 Actions 日志，找到具体错误信息
2. 确认设备树文件完整且无语法错误
3. 查看是否是网络超时导致（重新运行即可）

### 编译产物不完整
1. 检查 `第 14 步` 的输出，确认文件是否生成
2. 查看 `第 11 步` 的编译日志，确认是否有错误

### 无法下载 Release
1. 确认工作流成功完成
2. 检查 Release 页面是否创建成功
3. 如果 Release 失败，可从 Artifacts 下载

## 📚 相关资源

- [OrangeFox 官方网站](https://orangefox.download/)
- [OrangeFox GitLab](https://gitlab.com/OrangeFox)
- [设备树仓库](${{ github.server_url }}/${{ github.repository }})

## 📄 许可证

本工作流遵循 Apache 2.0 许可证。

---

**免责声明**: 此为非官方编译版本，仅供学习交流使用。刷机有风险，操作需谨慎！

