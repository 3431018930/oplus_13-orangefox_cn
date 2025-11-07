# 🐧 Ubuntu 本地构建 OrangeFox Recovery 完整教程

## 📋 前置要求

- **操作系统**: Ubuntu 20.04/22.04/24.04 LTS（推荐 22.04）
- **硬盘空间**: 至少 **150GB** 可用空间
- **内存**: 至少 **16GB RAM**（推荐 32GB）
- **处理器**: 64 位多核处理器
- **网络**: 稳定的互联网连接

---

## 🚀 第一部分：系统准备和依赖安装

### 步骤 1: 更新系统

打开终端（Ctrl+Alt+T），执行以下命令：

```bash
# 更新软件包列表
sudo apt update && sudo apt upgrade -y

# 安装基础工具
sudo apt install -y software-properties-common
```

### 步骤 2: 安装编译所需的依赖包

```bash
# 安装所有编译依赖（一次性完成）
sudo apt install -y \
    bc bison build-essential ccache curl flex \
    g++-multilib gcc-multilib git git-lfs gnupg gperf \
    imagemagick lib32readline-dev lib32z1-dev \
    libelf-dev liblz4-tool libsdl1.2-dev libssl-dev \
    libxml2 libxml2-utils lzop pngcrush rsync \
    schedtool squashfs-tools xsltproc zip zlib1g-dev \
    openjdk-11-jdk python3 python3-pip python-is-python3 \
    aria2 wget unzip nano vim
```

### 步骤 3: 配置 Git

```bash
# 配置 Git 用户信息（必须）
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global color.ui auto

# 验证配置
git config --list
```

### 步骤 4: 安装 Repo 工具

```bash
# 创建 bin 目录
mkdir -p ~/bin
export PATH=~/bin:$PATH

# 下载 repo 工具
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo

# 将 bin 目录永久添加到 PATH
echo 'export PATH=~/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# 验证安装
repo --version
```

---

## 📥 第二部分：同步 OrangeFox 源码

### 步骤 5: 创建工作目录

```bash
# 创建 OrangeFox 工作目录
mkdir -p ~/orangefox
cd ~/orangefox
```

### 步骤 6: 克隆同步脚本

```bash
# 克隆 OrangeFox 同步脚本
git clone https://gitlab.com/OrangeFox/sync.git
cd sync
```

### 步骤 7: 同步 OrangeFox 源码

⚠️ **注意：此步骤会下载约 40-60GB 数据，需要 2-6 小时**

根据 README.md 中的说明，使用 14.1 分支：

```bash
# 同步 OrangeFox 14.1 源码
./orangefox_sync.sh --branch 14.1 --path ~/fox_14.1
```

**如果网络不稳定，可以使用国内镜像（可选）：**

```bash
# 设置清华大学镜像源（国内用户推荐）
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'

# 如果同步失败，可以重新运行同步命令
cd ~/fox_14.1
repo sync -c --force-sync --no-tags --no-clone-bundle -j4
```

### 步骤 8: 验证源码同步

```bash
# 检查源码目录
ls -lh ~/fox_14.1/

# 应该看到这些目录：
# bootable/ build/ device/ external/ frameworks/ vendor/ 等等
```

---

## 🌳 第三部分：克隆设备树

### 步骤 9: 克隆 OnePlus 13 (PJZ110) 设备树

```bash
# 进入源码目录
cd ~/fox_14.1

# 创建设备厂商目录
mkdir -p device/oplus
cd device/oplus

# 克隆设备树（使用你的仓库）
git clone https://github.com/YOUR_USERNAME/oplus_13-orangefox_cn.git PJZ110

# 验证设备树
ls -lh PJZ110/
```

**注意**: 请将上面命令中的 `YOUR_USERNAME` 替换为实际的 GitHub 用户名。

---

## 🔧 第四部分：配置编译环境

### 步骤 10: 配置环境变量

```bash
# 进入源码根目录
cd ~/fox_14.1

# 设置必要的环境变量
export ALLOW_MISSING_DEPENDENCIES=true
export FOX_BUILD_DEVICE=PJZ110

# 启用 CCache 加速编译（可选但推荐）
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
ccache -M 20G  # 设置缓存大小为 20GB
```

### 步骤 11: 初始化编译环境

```bash
# 加载编译环境
source build/envsetup.sh

# 选择编译目标（根据 README.md）
lunch twrp_PJZ110-ap2a-eng
```

你会看到类似输出：

```
============================================
PLATFORM_VERSION_CODENAME=REL
PLATFORM_VERSION=99.87.36
TARGET_PRODUCT=twrp_PJZ110
TARGET_BUILD_VARIANT=eng
TARGET_ARCH=arm64
...
============================================
```

---

## 🔨 第五部分：开始编译

### 步骤 12: 清理旧编译产物（首次编译可跳过）

```bash
# 如果是重新编译，清理旧文件
make clean

# 或者只清理当前设备
make installclean
```

### 步骤 13: 开始编译 🚀

⚠️ **编译时间：2-4 小时，取决于你的 CPU 性能**

根据 README.md 中的说明：

```bash
# 编译 adbd 和 recoveryimage
mka adbd recoveryimage
```

**或者使用多线程加速编译：**

```bash
# 使用所有 CPU 核心
mka adbd recoveryimage -j$(nproc)

# 或者指定线程数（例如 8 个线程）
mka adbd recoveryimage -j8
```

编译过程中会看到大量输出，这是正常的。

### 步骤 14: 监控编译进度（可选）

在另一个终端窗口中：

```bash
# 打开新终端，监控 CPU 和内存使用
top

# 或者使用 htop（更友好的界面）
sudo apt install htop
htop
```

---

## 🎉 第六部分：获取编译产物

### 步骤 15: 检查编译结果

编译完成后（看到 **"#### build completed successfully"** 字样）：

```bash
# 进入产物目录
cd ~/fox_14.1/out/target/product/PJZ110

# 查看生成的文件
ls -lh OrangeFox*.img
ls -lh OrangeFox*.zip
ls -lh recovery.img

# 应该看到类似：
# OrangeFox-PJZ110-Unofficial-20241107.img
# OrangeFox-PJZ110-Unofficial-20241107.zip
# recovery.img
```

### 步骤 16: 复制到便于访问的位置

```bash
# 创建输出目录
mkdir -p ~/orangefox_builds

# 复制编译产物
cp OrangeFox*.img ~/orangefox_builds/ 2>/dev/null || true
cp OrangeFox*.zip ~/orangefox_builds/ 2>/dev/null || true
cp recovery.img ~/orangefox_builds/ 2>/dev/null || true

# 查看复制的文件
ls -lh ~/orangefox_builds/

echo "✅ 编译产物已复制到: ~/orangefox_builds/"
```

---

## 📊 完整命令流程总结（快速复制版）

一旦环境配置好，后续编译只需要这些命令：

```bash
# 1. 进入源码目录
cd ~/fox_14.1

# 2. 更新设备树（可选）
cd device/oplus/PJZ110
git pull
cd ~/fox_14.1

# 3. 初始化环境
source build/envsetup.sh
lunch twrp_PJZ110-ap2a-eng

# 4. 设置环境变量
export ALLOW_MISSING_DEPENDENCIES=true
export FOX_BUILD_DEVICE=PJZ110

# 5. 开始编译
mka adbd recoveryimage -j$(nproc)

# 6. 复制产物
cd out/target/product/PJZ110
cp OrangeFox*.* ~/orangefox_builds/ 2>/dev/null || true
cp recovery.img ~/orangefox_builds/ 2>/dev/null || true
```

---

## 🛠️ 常见问题排查

### 问题 1: 磁盘空间不足

**查看空间：**
```bash
df -h
```

**解决方案：**
```bash
# 清理 CCache
ccache -C

# 清理旧编译产物
cd ~/fox_14.1
rm -rf out/

# 清理 repo 缓存
rm -rf .repo/repo
```

### 问题 2: 编译错误 "out of memory"

**解决方案：**
```bash
# 减少并发线程数
mka adbd recoveryimage -j4  # 使用 4 个线程

# 或者增加 swap 分区
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# 永久生效
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### 问题 3: repo sync 失败

**解决方案：**
```bash
# 使用镜像源（国内用户）
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'

# 重新同步
cd ~/fox_14.1
repo sync -c --force-sync --no-tags --no-clone-bundle -j4

# 如果某个仓库失败，单独同步
repo sync -c PROJECT_NAME
```

### 问题 4: 缺少依赖包

**检查并重新安装：**
```bash
# 更新包列表
sudo apt update

# 重新安装依赖
sudo apt install -y --reinstall build-essential

# 检查 Java 版本
java -version
# 应该显示 openjdk 11
```

### 问题 5: 找不到编译产物

**检查编译是否真的成功：**
```bash
# 查看编译日志
tail -100 ~/fox_14.1/out/build*.log

# 或者重新运行编译，查看错误信息
cd ~/fox_14.1
source build/envsetup.sh
lunch twrp_PJZ110-ap2a-eng
mka adbd recoveryimage 2>&1 | tee build.log
```

### 问题 6: Python 相关错误

**解决方案：**
```bash
# 确保 python3 是默认的 python
sudo apt install python-is-python3

# 验证
python --version
# 应该显示 Python 3.x.x
```

---

## ⚡ 性能优化建议

### 1. 启用 CCache（大幅加速重新编译）

```bash
# 设置 CCache 大小
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
ccache -M 50G  # 如果有足够空间，可以设置更大

# 查看 CCache 统计
ccache -s

# 将配置添加到 .bashrc
echo 'export USE_CCACHE=1' >> ~/.bashrc
echo 'export CCACHE_EXEC=/usr/bin/ccache' >> ~/.bashrc
```

### 2. 使用 RAM Disk（需要大内存）

```bash
# 创建 30GB RAM Disk（需要至少 48GB 内存）
sudo mkdir -p /mnt/ramdisk
sudo mount -t tmpfs -o size=30G tmpfs /mnt/ramdisk

# 将 out 目录链接到 RAM Disk
cd ~/fox_14.1
mv out /mnt/ramdisk/
ln -s /mnt/ramdisk/out out
```

### 3. 优化编译器设置

```bash
# 使用 ninja 并行编译
export USE_NINJA=true

# 设置合适的线程数（CPU 核心数）
export MAKE_JOBS=$(nproc)
```

### 4. 使用 SSD

如果可能，将整个构建目录放在 SSD 上，而不是机械硬盘。

---

## 🎯 快速启动脚本

创建一个自动化脚本方便使用：

```bash
# 创建编译脚本
nano ~/build_orangefox.sh
```

粘贴以下内容：

```bash
#!/bin/bash
set -e

echo "🦊 开始编译 OrangeFox Recovery for OnePlus 13 (PJZ110)"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

# 进入源码目录
cd ~/fox_14.1

echo "📦 初始化编译环境..."
source build/envsetup.sh
lunch twrp_PJZ110-ap2a-eng

# 设置环境变量
export ALLOW_MISSING_DEPENDENCIES=true
export FOX_BUILD_DEVICE=PJZ110
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache

echo "🔨 开始编译..."
echo "使用 $(nproc) 个线程"
mka adbd recoveryimage -j$(nproc)

echo "📤 复制产物..."
mkdir -p ~/orangefox_builds
cp out/target/product/PJZ110/OrangeFox*.img ~/orangefox_builds/ 2>/dev/null || true
cp out/target/product/PJZ110/OrangeFox*.zip ~/orangefox_builds/ 2>/dev/null || true
cp out/target/product/PJZ110/recovery.img ~/orangefox_builds/ 2>/dev/null || true

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "✅ 编译完成！"
echo "📂 文件位置: ~/orangefox_builds/"
ls -lh ~/orangefox_builds/
```

保存后赋予执行权限：

```bash
chmod +x ~/build_orangefox.sh
```

以后只需要运行：

```bash
~/build_orangefox.sh
```

---

## 🧹 清理脚本

创建清理脚本：

```bash
nano ~/clean_build.sh
```

粘贴内容：

```bash
#!/bin/bash
cd ~/fox_14.1
echo "🧹 清理编译产物..."
make clean
ccache -C
echo "✅ 清理完成！"
df -h
```

赋予执行权限：

```bash
chmod +x ~/clean_build.sh
```

---

## 📖 相关文档链接

- [OrangeFox 官方文档](https://wiki.orangefox.tech/)
- [Android 编译指南](https://source.android.com/docs/setup/build/building)
- [设备树仓库](https://github.com/koaaN/android_device_oplus_13-orangefox.git)
- [Repo 工具文档](https://source.android.com/docs/setup/download)

---

## ✅ 检查清单

完整流程检查：

- [ ] Ubuntu 系统已更新
- [ ] 所有依赖包已安装
- [ ] Git 已配置（用户名和邮箱）
- [ ] Repo 工具已安装
- [ ] OrangeFox 源码已同步（~50GB）
- [ ] 设备树已克隆到正确位置
- [ ] 编译环境已初始化
- [ ] 编译成功完成
- [ ] 产物已复制到 ~/orangefox_builds/

---

## 📱 刷入设备

编译完成后，刷入 OnePlus 13：

```bash
# 重启到 Fastboot 模式
adb reboot bootloader

# 刷入 recovery.img
fastboot flash recovery ~/orangefox_builds/recovery.img

# 或者使用 OrangeFox.img（如果生成了）
fastboot flash recovery ~/orangefox_builds/OrangeFox-*.img

# 重启到 Recovery
fastboot reboot recovery
```

---

## 🔄 后续编译流程

第二次及以后的编译，只需要：

```bash
# 快速编译命令
cd ~/fox_14.1 && \
source build/envsetup.sh && \
lunch twrp_PJZ110-ap2a-eng && \
export ALLOW_MISSING_DEPENDENCIES=true && \
mka adbd recoveryimage -j$(nproc)
```

或者直接运行：

```bash
~/build_orangefox.sh
```

---

## 💡 提示和技巧

1. **保持源码更新**
   ```bash
   cd ~/fox_14.1
   repo sync -c -j$(nproc)
   ```

2. **查看编译时间**
   ```bash
   time ~/build_orangefox.sh
   ```

3. **并行下载源码**
   ```bash
   repo sync -c -j16 --force-sync
   ```

4. **编译前检查磁盘空间**
   ```bash
   df -h ~
   # 确保至少有 50GB 可用空间
   ```

5. **使用 tmux 或 screen 防止意外断开**
   ```bash
   sudo apt install tmux
   tmux new -s build
   # 在 tmux 中运行编译命令
   # 按 Ctrl+B 然后 D 可以分离会话
   # tmux attach -t build 重新连接
   ```

---

**祝你编译顺利！🎉**

如有问题，请检查：
- 编译日志：`~/fox_14.1/out/error.log`
- 系统日志：`dmesg | tail -100`
- 磁盘空间：`df -h`
- 内存使用：`free -h`

如遇到其他问题，可以查看 OrangeFox 官方文档或社区支持。

