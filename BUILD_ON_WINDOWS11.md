# 🪟 Windows 11 手动构建 OrangeFox Recovery 完整教程

## 📋 前置要求

- **操作系统**: Windows 11 (已激活)
- **硬盘空间**: D 盘至少 **150GB** 可用空间
- **内存**: 至少 **16GB RAM**（推荐 32GB）
- **处理器**: 支持虚拟化的 64 位处理器
- **网络**: 稳定的互联网连接

---

## 🚀 第一部分：安装 WSL2 和 Ubuntu

### 步骤 1: 启用 WSL2

打开 **PowerShell（管理员）**，执行以下命令：

```powershell
# 启用 WSL 功能
wsl --install

# 如果已经安装过 WSL，更新到 WSL2
wsl --set-default-version 2

# 安装 Ubuntu 22.04
wsl --install -d Ubuntu-22.04
```

**重启电脑** 后继续。

### 步骤 2: 配置 Ubuntu

重启后，Ubuntu 会自动打开，设置用户名和密码：

```bash
# 输入新的用户名（例如：builder）
Enter new UNIX username: builder

# 输入密码（输入时不显示，正常）
New password: ********
Retype new password: ********
```

### 步骤 3: 将 WSL 数据移动到 D 盘

默认 WSL 安装在 C 盘，我们需要移动到 D 盘以节省 C 盘空间。

在 **PowerShell（管理员）** 中执行：

```powershell
# 创建 D 盘目录
New-Item -ItemType Directory -Force -Path "D:\WSL"
New-Item -ItemType Directory -Force -Path "D:\OrangeFox_Build"

# 关闭 WSL
wsl --shutdown

# 导出 Ubuntu
wsl --export Ubuntu-22.04 "D:\WSL\ubuntu-22.04.tar"

# 注销原 Ubuntu
wsl --unregister Ubuntu-22.04

# 导入到 D 盘
wsl --import Ubuntu-22.04 "D:\WSL\Ubuntu-22.04" "D:\WSL\ubuntu-22.04.tar"

# 设置默认用户（替换 builder 为你的用户名）
ubuntu2204.exe config --default-user builder

# 删除临时文件
Remove-Item "D:\WSL\ubuntu-22.04.tar"

# 启动 WSL
wsl -d Ubuntu-22.04
```

---

## 🔧 第二部分：配置 Ubuntu 编译环境

### 步骤 4: 更新系统并安装依赖

在 WSL Ubuntu 中执行：

```bash
# 更新软件包列表
sudo apt update && sudo apt upgrade -y

# 安装编译所需的依赖（一次性安装）
sudo apt install -y \
    bc bison build-essential ccache curl flex \
    g++-multilib gcc-multilib git gnupg gperf \
    imagemagick lib32readline-dev lib32z1-dev \
    libelf-dev liblz4-tool libsdl1.2-dev libssl-dev \
    libxml2 libxml2-utils lzop pngcrush rsync \
    schedtool squashfs-tools xsltproc zip zlib1g-dev \
    openjdk-11-jdk python3 python-is-python3 \
    git-lfs aria2 wget unzip nano vim

# 配置 Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global color.ui auto
```

### 步骤 5: 安装 Repo 工具

```bash
# 创建 bin 目录
mkdir -p ~/bin
export PATH=~/bin:$PATH

# 下载 repo 工具
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo

# 将 bin 目录添加到 PATH（永久生效）
echo 'export PATH=~/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# 验证安装
repo --version
```

---

## 📥 第三部分：同步 OrangeFox 源码

### 步骤 6: 在 D 盘创建构建目录

```bash
# 在 Windows D 盘创建目录（WSL 中访问 Windows 磁盘）
cd /mnt/d/OrangeFox_Build

# 或者在 WSL 自己的空间创建（更快，但占用 WSL 空间）
# mkdir -p ~/OrangeFox_Build && cd ~/OrangeFox_Build
```

**推荐方式**：使用 WSL 自己的空间（速度更快）

```bash
# 创建工作目录
mkdir -p ~/orangefox
cd ~/orangefox
```

### 步骤 7: 同步 OrangeFox 源码

⚠️ **这一步会下载约 50GB 数据，需要 2-6 小时**

```bash
# 克隆同步脚本
git clone https://gitlab.com/OrangeFox/sync.git
cd sync

# 同步 OrangeFox 15.0 源码（Android 15）
./orangefox_sync.sh --branch 15.0 --path ~/orangefox/fox_15.0
```

同步过程中可能看到很多输出，这是正常的。耐心等待完成。

### 步骤 8: 验证源码同步

```bash
# 检查源码目录
ls -lh ~/orangefox/fox_15.0/

# 应该看到这些目录：
# bootable/ build/ device/ external/ frameworks/ 等等
```

---

## 🌳 第四部分：克隆设备树

### 步骤 9: 克隆 OnePlus 13 设备树

```bash
# 进入源码目录
cd ~/orangefox/fox_15.0

# 创建设备目录
mkdir -p device/oplus

# 克隆设备树
cd device/oplus
git clone https://github.com/3431018930/oplus_13-orangefox_cn.git PJZ110

# 验证设备树
ls -lh PJZ110/
```

---

## 🔮 第五部分：配置 Magisk 和 Root 工具

### 步骤 10: 下载 Magisk Alpha

```bash
# 创建 Magisk 目录
mkdir -p ~/magisk_alpha
cd ~/magisk_alpha

# 下载最新 Magisk Alpha（Canary 版本）
wget -O Magisk-alpha.apk https://github.com/topjohnwu/Magisk/releases/download/canary-latest/app-release.apk

# 或者下载特定版本（例如 v29.0）
# wget -O Magisk-v29.0.apk https://github.com/topjohnwu/Magisk/releases/download/v29.0/Magisk-v29.0.apk

# 验证下载
ls -lh Magisk-*.apk
```

### 步骤 11: 配置设备树使用 Magisk Alpha

```bash
# 进入设备树目录
cd ~/orangefox/fox_15.0/device/oplus/PJZ110

# 编辑 vendorsetup.sh
nano vendorsetup.sh

# 在文件末尾添加（按 Ctrl+O 保存，Ctrl+X 退出）：
# export FOX_USE_SPECIFIC_MAGISK_ZIP=~/magisk_alpha/Magisk-alpha.apk
```

或者使用命令直接添加：

```bash
echo 'export FOX_USE_SPECIFIC_MAGISK_ZIP=~/magisk_alpha/Magisk-alpha.apk' >> vendorsetup.sh
```

### 步骤 12: 更新 magiskboot（可选）

```bash
# 下载最新 magiskboot
cd ~
git clone https://github.com/magojohnji/magiskboot-linux.git

# 复制到 OrangeFox
cp -f ~/magiskboot-linux/arm64-v8a/magiskboot ~/orangefox/fox_15.0/vendor/recovery/prebuilt/arm64/magiskboot_updated
cp -f ~/magiskboot-linux/x86_64/magiskboot ~/orangefox/fox_15.0/vendor/recovery/tools/magiskboot

echo "✅ magiskboot 更新完成"
```

---

## 🔨 第六部分：开始编译

### 步骤 13: 配置编译环境

```bash
# 进入源码根目录
cd ~/orangefox/fox_15.0

# 设置环境变量
export ALLOW_MISSING_DEPENDENCIES=true
export FOX_BUILD_DEVICE=PJZ110

# 可选：启用 CCache 加速编译
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
ccache -M 10G
```

### 步骤 14: 初始化编译环境

```bash
# 加载环境
source build/envsetup.sh

# 选择编译目标
lunch twrp_PJZ110-eng
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

### 步骤 15: 清理旧编译产物（首次编译可跳过）

```bash
# 完全清理
make clean

# 或者只清理当前设备
make installclean
```

### 步骤 16: 开始编译 🚀

⚠️ **编译时间：2-4 小时，取决于你的 CPU**

```bash
# 编译 Recovery（使用所有 CPU 核心）
mka recoveryimage -j$(nproc)

# 或者指定线程数（例如 8 个线程）
# mka recoveryimage -j8
```

编译过程中会看到大量输出，这是正常的。

### 步骤 17: 监控编译进度

在另一个 WSL 窗口中：

```bash
# 打开新的 WSL 终端
wsl -d Ubuntu-22.04

# 监控 CPU 和内存使用
top

# 或者查看编译日志
tail -f ~/orangefox/fox_15.0/out/verbose.log.gz
```

---

## 🎉 第七部分：获取编译产物

### 步骤 18: 检查编译结果

编译完成后（看到 "#### build completed successfully" 字样）：

```bash
# 进入产物目录
cd ~/orangefox/fox_15.0/out/target/product/PJZ110

# 查看生成的文件
ls -lh OrangeFox*.img
ls -lh OrangeFox*.zip

# 应该看到类似：
# OrangeFox-PJZ110-Unofficial-20251107.img
# OrangeFox-PJZ110-Unofficial-20251107.zip
```

### 步骤 19: 复制到 Windows D 盘

```bash
# 创建 Windows 输出目录
mkdir -p /mnt/d/OrangeFox_Build/output

# 复制编译产物
cp OrangeFox*.img /mnt/d/OrangeFox_Build/output/
cp OrangeFox*.zip /mnt/d/OrangeFox_Build/output/

echo "✅ 文件已复制到 D:\OrangeFox_Build\output\"
```

在 Windows 文件资源管理器中打开 `D:\OrangeFox_Build\output\` 即可看到编译好的文件！

---

## 📊 完整命令流程总结（复制粘贴版）

一旦环境配置好，后续编译只需要这些命令：

```bash
# 1. 进入源码目录
cd ~/orangefox/fox_15.0

# 2. 同步最新设备树（可选）
cd device/oplus/PJZ110
git pull
cd ~/orangefox/fox_15.0

# 3. 初始化环境
source build/envsetup.sh
lunch twrp_PJZ110-eng

# 4. 清理（可选）
make clean

# 5. 开始编译
export ALLOW_MISSING_DEPENDENCIES=true
export FOX_BUILD_DEVICE=PJZ110
mka recoveryimage -j$(nproc)

# 6. 复制产物到 Windows
cd out/target/product/PJZ110
cp OrangeFox*.{img,zip} /mnt/d/OrangeFox_Build/output/
```

---

## 🛠️ 常见问题排查

### 问题 1: WSL 磁盘空间不足

**查看空间：**
```bash
df -h
```

**解决方案：**
```bash
# 清理 CCache
ccache -C

# 清理旧编译产物
cd ~/orangefox/fox_15.0
rm -rf out/

# 清理 repo 缓存
rm -rf .repo/repo
```

### 问题 2: 编译错误 "out of memory"

**解决方案：**
```bash
# 减少并发线程数
mka recoveryimage -j4  # 使用 4 个线程而不是全部
```

**在 Windows PowerShell 中增加 WSL 内存限制：**
```powershell
# 在 C:\Users\你的用户名\ 创建 .wslconfig 文件
notepad $env:USERPROFILE\.wslconfig
```

添加内容：
```ini
[wsl2]
memory=16GB
processors=8
swap=8GB
```

保存后重启 WSL：
```powershell
wsl --shutdown
wsl -d Ubuntu-22.04
```

### 问题 3: repo sync 失败

**解决方案：**
```bash
# 使用镜像源（国内用户）
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'

# 重新同步
cd ~/orangefox/fox_15.0
repo sync -c --force-sync --no-tags --no-clone-bundle -j4
```

### 问题 4: 找不到编译产物

**检查编译是否真的成功：**
```bash
# 查看编译日志最后几行
tail -100 ~/orangefox/fox_15.0/out/build*.log
```

成功的话应该看到：
```
#### build completed successfully (XX:XX (mm:ss)) ####
```

### 问题 5: Windows 和 WSL 文件互访

**从 WSL 访问 Windows D 盘：**
```bash
cd /mnt/d/
```

**从 Windows 访问 WSL 文件：**
在文件资源管理器地址栏输入：
```
\\wsl$\Ubuntu-22.04\home\builder\orangefox\
```

---

## ⚡ 性能优化建议

### 1. 启用 CCache（加速重新编译）

```bash
# 设置 CCache 大小为 50GB
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
ccache -M 50G

# 查看 CCache 统计
ccache -s
```

### 2. 使用 RAM Disk（需要大内存）

```bash
# 创建 20GB RAM Disk（需要至少 32GB 内存）
sudo mkdir -p /mnt/ramdisk
sudo mount -t tmpfs -o size=20G tmpfs /mnt/ramdisk

# 将 out 目录链接到 RAM Disk
cd ~/orangefox/fox_15.0
rm -rf out
ln -s /mnt/ramdisk/out out
```

### 3. WSL2 性能调优

在 `.wslconfig` 中添加：
```ini
[wsl2]
memory=20GB
processors=12
swap=16GB
localhostForwarding=true
```

---

## 📚 后续编译流程

第二次编译只需要：

```bash
# 1. 启动 WSL
wsl -d Ubuntu-22.04

# 2. 更新设备树（如果有更新）
cd ~/orangefox/fox_15.0/device/oplus/PJZ110
git pull

# 3. 编译
cd ~/orangefox/fox_15.0
source build/envsetup.sh
lunch twrp_PJZ110-eng
mka recoveryimage -j$(nproc)

# 4. 复制产物
cd out/target/product/PJZ110
cp OrangeFox*.* /mnt/d/OrangeFox_Build/output/
```

---

## 🎯 快速启动脚本

创建一个自动化脚本方便使用：

```bash
# 创建编译脚本
nano ~/build_orangefox.sh
```

粘贴内容：
```bash
#!/bin/bash
set -e

echo "🦊 开始编译 OrangeFox Recovery for OnePlus 13"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

cd ~/orangefox/fox_15.0

echo "📦 初始化编译环境..."
source build/envsetup.sh
lunch twrp_PJZ110-eng

export ALLOW_MISSING_DEPENDENCIES=true
export FOX_BUILD_DEVICE=PJZ110

echo "🔨 开始编译..."
mka recoveryimage -j$(nproc)

echo "📤 复制产物到 Windows..."
mkdir -p /mnt/d/OrangeFox_Build/output
cp out/target/product/PJZ110/OrangeFox*.{img,zip} /mnt/d/OrangeFox_Build/output/ 2>/dev/null || true

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "✅ 编译完成！"
echo "📂 文件位置: D:\OrangeFox_Build\output\"
ls -lh /mnt/d/OrangeFox_Build/output/OrangeFox*
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

## 🔧 额外工具和技巧

### 清理脚本

```bash
# 创建清理脚本
nano ~/clean_build.sh
```

```bash
#!/bin/bash
cd ~/orangefox/fox_15.0
echo "🧹 清理编译产物..."
make clean
ccache -C
echo "✅ 清理完成！"
```

### 查看编译时间

```bash
# 编译时记录时间
time mka recoveryimage -j$(nproc)
```

### 只编译 Recovery 不编译 vendor_boot

```bash
# 只编译 recoveryimage
mka recoveryimage

# 如果需要 vendor_boot
mka vendorbootimage
```

---

## 📖 相关文档链接

- [OrangeFox 官方文档](https://wiki.orangefox.tech/)
- [WSL2 官方文档](https://docs.microsoft.com/zh-cn/windows/wsl/)
- [Android 编译指南](https://source.android.com/docs/setup/build/building)
- [设备树仓库](https://github.com/3431018930/oplus_13-orangefox_cn)

---

## ✅ 检查清单

完整流程检查：

- [ ] WSL2 已安装并移动到 D 盘
- [ ] Ubuntu 22.04 已配置
- [ ] 所有依赖已安装
- [ ] Repo 工具已安装
- [ ] OrangeFox 源码已同步（~50GB）
- [ ] 设备树已克隆
- [ ] Magisk Alpha 已下载并配置
- [ ] 编译环境已初始化
- [ ] 编译成功完成
- [ ] 产物已复制到 D:\OrangeFox_Build\output\

---

**祝你编译顺利！🎉**

如有问题，请检查日志文件：`~/orangefox/fox_15.0/out/error.log`

