# 🗂️ 磁盘空间优化说明

## ⚠️ 问题描述

GitHub Actions 免费 runner 的磁盘空间限制约为 **14GB**，而编译 OrangeFox Recovery 需要：
- 源码同步：~50GB
- 编译产物：~30GB
- CCache 缓存：5-10GB
- 总计需求：**~100GB**

这导致编译过程中出现 `No space left on device` 错误。

## ✅ 优化措施

### 1. **磁盘清理步骤（第 2 步）**
删除 GitHub Actions runner 预装的不必要软件，释放约 **30GB** 空间：

| 清理项目 | 释放空间 |
|---------|---------|
| .NET SDK | ~10GB |
| Android SDK | ~8GB |
| Haskell (GHC) | ~5GB |
| CodeQL | ~3GB |
| Docker 镜像 | ~2GB |
| 其他工具 | ~2GB |
| **总计** | **~30GB** |

### 2. **减少 CCache 大小**
- 原配置：10GB
- 新配置：**5GB**
- 节省空间：5GB

### 3. **精简源码同步**
使用 OrangeFox 官方同步脚本的精简模式：
- 仅同步必要的代码仓库
- 不下载完整的 Git 历史
- 使用浅克隆（shallow clone）

### 4. **编译前清理（第 13 步）**
在开始编译前清理临时文件：
```bash
sudo rm -rf /tmp/*
sudo apt-get clean
```

### 5. **实时监控磁盘空间**
在关键步骤后显示磁盘使用情况：
- 初始状态（第 1 步）
- 清理后（第 2 步）
- 同步后（第 6 步）
- 编译前（第 13 步）

## 📊 优化效果对比

| 阶段 | 优化前 | 优化后 | 改善 |
|------|--------|--------|------|
| 可用空间 | 14GB | ~44GB | +30GB |
| CCache 占用 | 10GB | 5GB | -5GB |
| 清理策略 | 无 | 多阶段 | 更安全 |

## 🔧 进一步优化建议

如果仍然遇到空间不足，可以尝试：

### 选项 1: 使用自托管 Runner
```yaml
runs-on: self-hosted  # 使用自己的服务器
```

### 选项 2: 分离编译步骤
将源码同步和编译分为两个独立的作业：
```yaml
jobs:
  sync:
    # 同步源码并缓存
  build:
    needs: sync
    # 使用缓存的源码编译
```

### 选项 3: 使用 Ubuntu 24.04 Runner
GitHub 新的 runner 可能有更多空间：
```yaml
runs-on: ubuntu-24.04
```

### 选项 4: 禁用 CCache
如果不需要加速编译：
```yaml
USE_CCACHE: false  # 节省 5GB 空间
```

### 选项 5: 只编译 Recovery 镜像
不编译不必要的目标：
```bash
# 只编译 recoveryimage，不编译 adbd
mka recoveryimage
```

## 📋 监控命令

在工作流中随时检查磁盘空间：

```bash
# 显示磁盘使用情况
df -h

# 查看最大的目录
du -h --max-depth=1 / | sort -hr | head -20

# 查看 /tmp 目录大小
du -sh /tmp/*
```

## 🚨 常见错误处理

### 错误 1: "No space left on device"
**解决方案**：
1. 检查是否执行了第 2 步（磁盘清理）
2. 减少 CCache 大小到 3GB
3. 清理 `.repo` 目录中的临时文件

### 错误 2: "Disk quota exceeded"
**解决方案**：
1. 使用 `df -h` 检查哪个分区满了
2. 清理 `/tmp` 和 `~/.cache` 目录
3. 删除不必要的中间产物

### 错误 3: 编译中途空间不足
**解决方案**：
```bash
# 在编译过程中清理
cd out
find . -name "*.o" -delete  # 删除对象文件
find . -name "*.a" -delete  # 删除静态库
```

## 💡 最佳实践

1. **始终监控磁盘空间**：在每个关键步骤后运行 `df -h`
2. **及时清理临时文件**：不要等到空间不足
3. **使用精简同步**：只同步必要的仓库
4. **合理设置 CCache**：5GB 通常足够
5. **定期更新清理脚本**：GitHub Actions 环境可能变化

## 📚 参考资源

- [GitHub Actions Runner Images](https://github.com/actions/runner-images)
- [OrangeFox 官方文档](https://wiki.orangefox.tech/)
- [AOSP 编译优化](https://source.android.com/docs/setup/build/building)

---

**最后更新**: 2025-11-07
**适用版本**: OrangeFox 15.0+ / Android 15+

