# MeshLab vcpkg 环境配置完整指南

## 概述

本文档提供 MeshLab 项目使用 vcpkg 依赖管理的详细配置指南。vcpkg 是微软开发的跨平台 C++ 包管理器，能够简化第三方库的安装和管理。

**文档版本**: v1.0.0
**更新日期**: 2025-01-13
**适用版本**: MeshLab 2025.07.0+

## 目录

1. [快速开始](#快速开始)
2. [详细配置步骤](#详细配置步骤)
3. [配置文件说明](#配置文件说明)
4. [构建选项](#构建选项)
5. [常见问题解决](#常见问题解决)
6. [高级配置](#高级配置)

## 快速开始

### Windows 快速构建（推荐）

```powershell
# 1. 克隆项目
git clone --recursive https://github.com/cnr-isti-vclab/meshlab.git
cd meshlab

# 2. 初始化 vcpkg
git clone https://github.com/Microsoft/vcpkg.git
.\vcpkg\bootstrap-vcpkg.bat

# 3. 配置和构建
cmake --preset vcpkg-default
cmake --build build\vcpkg --config Release

# 4. 运行
.\build\vcpkg\src\meshlab\Release\meshlab.exe
```

### Linux/macOS 快速构建

```bash
# 1. 克隆项目
git clone --recursive https://github.com/cnr-isti-vclab/meshlab.git
cd meshlab

# 2. 设置 vcpkg
bash setup_vcpkg.sh
source vcpkg_env.sh

# 3. 配置和构建
cmake --preset vcpkg-default
cmake --build build/vcpkg

# 4. 运行
./build/vcpkg/src/meshlab/meshlab
```

## 详细配置步骤

### 步骤 1: 环境准备

#### Windows 环境

1. **安装 Visual Studio 2022**
   - 下载 [Visual Studio 2022 Community](https://visualstudio.microsoft.com/downloads/)
   - 安装时选择以下工作负载：
     - 使用 C++ 的桌面开发
     - 包含 Windows 10/11 SDK
     - MSVC v143 编译器

2. **安装 CMake**
   ```powershell
   # 使用 winget
   winget install Kitware.CMake

   # 或从官网下载
   # https://cmake.org/download/
   ```

3. **安装 Git**
   ```powershell
   winget install Git.Git
   ```

#### Linux 环境

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y build-essential cmake git curl zip unzip tar pkg-config

# Fedora/RHEL
sudo dnf groupinstall "Development Tools"
sudo dnf install -y cmake git curl zip unzip tar pkg-config

# Arch Linux
sudo pacman -S base-devel cmake git
```

#### macOS 环境

```bash
# 安装 Xcode 命令行工具
xcode-select --install

# 安装 Homebrew（如果没有）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 安装必要工具
brew install cmake git
```

### 步骤 2: 设置 vcpkg

#### 方式 1: 使用自动脚本（推荐）

**Windows:**
```powershell
# 运行设置脚本
.\setup_vcpkg.bat

# 或创建脚本 setup_vcpkg.bat
@echo off
if not exist vcpkg (
    echo 正在克隆 vcpkg...
    git clone https://github.com/Microsoft/vcpkg.git
)
cd vcpkg
echo 正在构建 vcpkg...
call bootstrap-vcpkg.bat
cd ..
echo vcpkg 设置完成！
```

**Linux/macOS:**
```bash
#!/bin/bash
# setup_vcpkg.sh

if [ ! -d "vcpkg" ]; then
    echo "正在克隆 vcpkg..."
    git clone https://github.com/Microsoft/vcpkg.git
fi

cd vcpkg
echo "正在构建 vcpkg..."
./bootstrap-vcpkg.sh
cd ..

# 创建环境变量脚本
cat > vcpkg_env.sh << 'EOF'
export VCPKG_ROOT="$(pwd)/vcpkg"
export CMAKE_TOOLCHAIN_FILE="${VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake"
export PATH="${VCPKG_ROOT}:${PATH}"
echo "vcpkg 环境已设置"
echo "VCPKG_ROOT: ${VCPKG_ROOT}"
EOF

echo "vcpkg 设置完成！"
echo "请运行: source vcpkg_env.sh"
```

#### 方式 2: 手动设置

```bash
# 1. 克隆 vcpkg
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg

# 2. 构建 vcpkg
# Windows
.\bootstrap-vcpkg.bat
# Linux/macOS
./bootstrap-vcpkg.sh

# 3. 设置环境变量
# Windows (PowerShell)
$env:VCPKG_ROOT = "$(Get-Location)"
$env:CMAKE_TOOLCHAIN_FILE = "$env:VCPKG_ROOT\scripts\buildsystems\vcpkg.cmake"

# Linux/macOS
export VCPKG_ROOT=$(pwd)
export CMAKE_TOOLCHAIN_FILE=$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake
```

### 步骤 3: 配置项目

#### 使用 CMake 预设（推荐）

```bash
# 基础构建（仅核心组件）
cmake --preset vcpkg-default

# 调试构建
cmake --preset vcpkg-debug

# 完整构建（包含所有插件）
cmake --preset vcpkg-with-plugins
```

#### 手动配置

```bash
# 创建构建目录
mkdir build && cd build

# 配置（基础）
cmake -DMESHLAB_USE_VCPKG=ON \
      -DCMAKE_TOOLCHAIN_FILE="../vcpkg/scripts/buildsystems/vcpkg.cmake" \
      ..

# 配置（包含插件）
cmake -DMESHLAB_USE_VCPKG=ON \
      -DCMAKE_TOOLCHAIN_FILE="../vcpkg/scripts/buildsystems/vcpkg.cmake" \
      -DVCPKG_MANIFEST_FEATURES="meshlab-plugins;meshlab-tools" \
      ..
```

### 步骤 4: 构建项目

```bash
# 使用 CMake 构建
cmake --build build/vcpkg --config Release

# 或使用原生构建工具
# Visual Studio (Windows)
msbuild build\vcpkg\MeshLab.sln /p:Configuration=Release

# Make (Linux/macOS)
cd build/vcpkg && make -j$(nproc)

# Ninja
cd build/vcpkg && ninja
```

## 配置文件说明

### vcpkg.json - 包清单文件

```json
{
  "name": "meshlab",
  "version": "2025.7.0",
  "description": "MeshLab - 3D mesh processing and editing system",
  "homepage": "https://www.meshlab.net/",
  "license": "GPL-3.0-or-later",
  "dependencies": [
    {
      "name": "vcpkg-cmake",
      "host": true
    },
    {
      "name": "vcpkg-cmake-config",
      "host": true
    },
    "qt5-base",
    "opengl",
    "glew",
    "eigen3"
  ],
  "features": {
    "meshlab-plugins": {
      "description": "Additional dependencies for MeshLab plugins",
      "dependencies": [
        "boost-algorithm",
        "boost-filesystem",
        "boost-thread",
        "boost-system",
        "cgal",
        "embree4",
        "qhull",
        "muparser",
        "tbb",
        "xerces-c"
      ]
    },
    "meshlab-tools": {
      "description": "Development tools",
      "dependencies": [
        {
          "name": "ninja",
          "host": true
        }
      ]
    }
  }
}
```

**关键字段说明**：
- `name`: 项目名称
- `version`: 项目版本
- `dependencies`: 基础依赖列表
- `features`: 可选功能组，通过 `VCPKG_MANIFEST_FEATURES` 启用
- `host`: 标记为主机工具（构建时使用）

### vcpkg-configuration.json - 注册表配置

```json
{
  "default-registry": {
    "kind": "git",
    "repository": "https://github.com/Microsoft/vcpkg",
    "baseline": "c9f906558f9bb12ee9811d6edc98ec9255c6cda5"
  },
  "registries": []
}
```

**关键字段说明**：
- `baseline`: 锁定 vcpkg 仓库版本，确保构建可重现
- `registries`: 可添加私有或第三方注册表

### CMakePresets.json - CMake 预设配置

```json
{
  "version": 3,
  "configurePresets": [
    {
      "name": "vcpkg-default",
      "displayName": "vcpkg Default Configuration",
      "description": "Default build using vcpkg with core dependencies",
      "generator": "Visual Studio 17 2022",
      "architecture": "x64",
      "binaryDir": "${sourceDir}/build/vcpkg",
      "toolchainFile": "${sourceDir}/vcpkg/scripts/buildsystems/vcpkg.cmake",
      "cacheVariables": {
        "MESHLAB_USE_VCPKG": "ON",
        "CMAKE_BUILD_TYPE": "Release"
      },
      "environment": {
        "VCPKG_ROOT": "${sourceDir}/vcpkg",
        "VCPKG_DEFAULT_TRIPLET": "x64-windows"
      }
    },
    {
      "name": "vcpkg-debug",
      "displayName": "vcpkg Debug Configuration",
      "inherits": "vcpkg-default",
      "binaryDir": "${sourceDir}/build/vcpkg-debug",
      "cacheVariables": {
        "CMAKE_BUILD_TYPE": "Debug",
        "MESHLAB_ENABLE_DEBUG_LOG_FILE": "ON"
      }
    },
    {
      "name": "vcpkg-with-plugins",
      "displayName": "vcpkg with All Plugins",
      "inherits": "vcpkg-default",
      "binaryDir": "${sourceDir}/build/vcpkg-plugins",
      "cacheVariables": {
        "VCPKG_MANIFEST_FEATURES": "meshlab-plugins;meshlab-tools"
      }
    }
  ]
}
```

## 构建选项

### CMake 选项

| 选项 | 默认值 | 说明 |
|------|--------|------|
| `MESHLAB_USE_VCPKG` | OFF | 启用 vcpkg 依赖管理 |
| `CMAKE_BUILD_TYPE` | Release | 构建类型 (Release/Debug/RelWithDebInfo) |
| `VCPKG_MANIFEST_FEATURES` | "" | 启用的功能特性，用分号分隔 |
| `VCPKG_TARGET_TRIPLET` | 自动检测 | 目标平台 (x64-windows/x64-linux/x64-osx) |
| `MESHLAB_BUILD_MINI` | OFF | 仅构建核心组件 |
| `MESHLAB_BUILD_WITH_DOUBLE_SCALAR` | OFF | 使用双精度浮点 |
| `MESHLAB_ENABLE_DEBUG_LOG_FILE` | OFF | 启用调试日志 |

### 功能特性

| 特性 | 包含的依赖 | 用途 |
|------|------------|------|
| `meshlab-plugins` | Boost, CGAL, Embree, QHull 等 | 完整插件支持 |
| `meshlab-tools` | Ninja | 开发工具 |

### 示例构建命令

```bash
# 1. 最小构建（快速）
cmake -B build-mini \
      -DMESHLAB_USE_VCPKG=ON \
      -DMESHLAB_BUILD_MINI=ON \
      -DCMAKE_TOOLCHAIN_FILE=vcpkg/scripts/buildsystems/vcpkg.cmake

# 2. 标准构建
cmake --preset vcpkg-default

# 3. 完整构建（所有插件）
cmake --preset vcpkg-with-plugins

# 4. 调试构建
cmake --preset vcpkg-debug

# 5. 自定义构建
cmake -B build-custom \
      -DMESHLAB_USE_VCPKG=ON \
      -DCMAKE_TOOLCHAIN_FILE=vcpkg/scripts/buildsystems/vcpkg.cmake \
      -DVCPKG_MANIFEST_FEATURES="meshlab-plugins" \
      -DCMAKE_BUILD_TYPE=RelWithDebInfo \
      -DMESHLAB_BUILD_WITH_DOUBLE_SCALAR=ON
```

## 常见问题解决

### 问题 1: vcpkg 下载失败

**错误信息**：
```
Failed to download vcpkg
```

**解决方案**：

1. **使用代理**：
   ```bash
   # Windows
   set HTTPS_PROXY=http://proxy:port
   set HTTP_PROXY=http://proxy:port

   # Linux/macOS
   export HTTPS_PROXY=http://proxy:port
   export HTTP_PROXY=http://proxy:port
   ```

2. **使用镜像**：
   ```bash
   # 使用国内镜像
   git clone https://gitee.com/mirrors/vcpkg.git
   ```

3. **手动下载**：
   - 从 GitHub Releases 下载 vcpkg
   - 解压到项目目录

### 问题 2: 依赖包安装失败

**错误信息**：
```
Error: Building package xxx failed
```

**解决方案**：

1. **清理缓存**：
   ```bash
   # Windows
   .\vcpkg\vcpkg.exe remove --outdated

   # Linux/macOS
   ./vcpkg/vcpkg remove --outdated
   ```

2. **查看详细日志**：
   ```bash
   # 查看构建日志
   cat vcpkg/buildtrees/xxx/install-x64-windows-dbg-out.log
   ```

3. **单独安装失败的包**：
   ```bash
   ./vcpkg/vcpkg install package-name --debug
   ```

### 问题 3: CMake 找不到包

**错误信息**：
```
CMake Error: Could not find package xxx
```

**解决方案**：

1. **确认工具链文件**：
   ```cmake
   -DCMAKE_TOOLCHAIN_FILE=完整路径/vcpkg/scripts/buildsystems/vcpkg.cmake
   ```

2. **检查 triplet**：
   ```bash
   # 查看可用 triplet
   ./vcpkg/vcpkg help triplet

   # 指定 triplet
   -DVCPKG_TARGET_TRIPLET=x64-windows
   ```

3. **重新配置**：
   ```bash
   rm -rf build/CMakeCache.txt
   cmake --preset vcpkg-default
   ```

### 问题 4: Qt5 相关错误

**错误信息**：
```
Qt5 not found
```

**解决方案**：

1. **Windows**：
   ```powershell
   # 确保安装 Qt5
   .\vcpkg\vcpkg.exe install qt5-base:x64-windows

   # 或手动指定 Qt 路径
   cmake -DQt5_DIR="C:\Qt\5.15.2\msvc2019_64\lib\cmake\Qt5" ..
   ```

2. **Linux**：
   ```bash
   # 安装系统 Qt5
   sudo apt install qt5-default libqt5opengl5-dev

   # 或使用 vcpkg
   ./vcpkg/vcpkg install qt5-base
   ```

### 问题 5: 内存不足

**错误信息**：
```
Out of memory
```

**解决方案**：

1. **减少并行任务**：
   ```bash
   cmake --build . -j2
   ```

2. **使用二进制缓存**：
   ```bash
   export VCPKG_BINARY_SOURCES="clear;files,./vcpkg-cache,readwrite"
   ```

3. **增加交换空间**（Linux）：
   ```bash
   sudo fallocate -l 8G /swapfile
   sudo chmod 600 /swapfile
   sudo mkswap /swapfile
   sudo swapon /swapfile
   ```

## 高级配置

### 使用二进制缓存

二进制缓存可以显著加速重复构建：

```bash
# 1. 本地文件缓存
export VCPKG_BINARY_SOURCES="clear;files,C:/vcpkg-cache,readwrite"

# 2. NuGet 缓存（企业环境）
export VCPKG_BINARY_SOURCES="clear;nuget,https://your-server/v3/index.json,readwrite"

# 3. 多级缓存
export VCPKG_BINARY_SOURCES="clear;files,./cache,readwrite;nuget,https://server/v3/index.json,read"
```

### 自定义 Triplet

创建自定义 triplet 以优化构建：

```cmake
# vcpkg/triplets/community/x64-windows-meshlab.cmake
set(VCPKG_TARGET_ARCHITECTURE x64)
set(VCPKG_CRT_LINKAGE dynamic)
set(VCPKG_LIBRARY_LINKAGE static)  # 静态链接库
set(VCPKG_CMAKE_SYSTEM_NAME Windows)

# 启用优化
set(VCPKG_C_FLAGS_RELEASE "/O2 /GL")
set(VCPKG_CXX_FLAGS_RELEASE "/O2 /GL")
```

使用自定义 triplet：
```bash
cmake -DVCPKG_TARGET_TRIPLET=x64-windows-meshlab ..
```

### 依赖版本控制

在 vcpkg.json 中指定版本：

```json
{
  "dependencies": [
    {
      "name": "qt5-base",
      "version>=": "5.15.0"
    },
    {
      "name": "boost",
      "version>=": "1.75.0",
      "version<": "1.80.0"
    }
  ],
  "overrides": [
    {
      "name": "eigen3",
      "version": "3.4.0"
    }
  ]
}
```

### 集成 CI/CD

#### GitHub Actions

```yaml
# .github/workflows/build.yml
name: Build with vcpkg

on: [push, pull_request]

jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]

    steps:
    - uses: actions/checkout@v3
      with:
        submodules: recursive

    - name: Setup vcpkg
      run: |
        git clone https://github.com/Microsoft/vcpkg.git
        ./vcpkg/bootstrap-vcpkg.sh

    - name: Configure
      run: cmake --preset vcpkg-default

    - name: Build
      run: cmake --build build/vcpkg --config Release
```

#### Docker 构建

```dockerfile
# Dockerfile
FROM ubuntu:22.04

# 安装依赖
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    git \
    curl \
    zip \
    unzip

# 设置 vcpkg
RUN git clone https://github.com/Microsoft/vcpkg.git /vcpkg
RUN /vcpkg/bootstrap-vcpkg.sh

# 设置环境变量
ENV VCPKG_ROOT=/vcpkg
ENV CMAKE_TOOLCHAIN_FILE=/vcpkg/scripts/buildsystems/vcpkg.cmake

# 复制项目
COPY . /meshlab
WORKDIR /meshlab

# 构建
RUN cmake --preset vcpkg-default
RUN cmake --build build/vcpkg
```

### 性能优化

#### 1. 并行构建
```bash
# 使用所有 CPU 核心
cmake --build . -j$(nproc)

# Windows
cmake --build . -j%NUMBER_OF_PROCESSORS%
```

#### 2. 预编译头
在 CMakeLists.txt 中启用：
```cmake
target_precompile_headers(meshlab-common
  PRIVATE
    <vector>
    <string>
    <memory>
)
```

#### 3. Link Time Optimization
```cmake
cmake -DCMAKE_INTERPROCEDURAL_OPTIMIZATION=ON ..
```

#### 4. ccache 集成
```bash
# 安装 ccache
sudo apt install ccache  # Linux
brew install ccache      # macOS

# 配置 CMake
cmake -DCMAKE_CXX_COMPILER_LAUNCHER=ccache ..
```

## 调试技巧

### 启用详细输出

```bash
# vcpkg 详细输出
export VCPKG_KEEP_ENV_VARS=VCPKG_ROOT
./vcpkg/vcpkg install --debug

# CMake 详细输出
cmake --preset vcpkg-default --debug-output

# 构建详细输出
cmake --build build/vcpkg --verbose
```

### 查看依赖关系

```bash
# 查看已安装的包
./vcpkg/vcpkg list

# 查看包信息
./vcpkg/vcpkg search qt5

# 查看依赖树
./vcpkg/vcpkg depend-info qt5-base
```

### 清理和重置

```bash
# 清理构建目录
rm -rf build/

# 清理 vcpkg 缓存
./vcpkg/vcpkg remove --outdated
rm -rf vcpkg/buildtrees
rm -rf vcpkg/packages

# 完全重置 vcpkg
rm -rf vcpkg/
git clone https://github.com/Microsoft/vcpkg.git
./vcpkg/bootstrap-vcpkg.sh
```

## 最佳实践

1. **版本控制**：
   - 将 vcpkg.json 和 vcpkg-configuration.json 加入版本控制
   - 使用 baseline 锁定 vcpkg 版本

2. **缓存策略**：
   - 使用二进制缓存加速 CI/CD
   - 定期清理过期缓存

3. **依赖管理**：
   - 最小化依赖，使用 features 管理可选依赖
   - 定期更新依赖版本

4. **构建优化**：
   - 使用 CMake 预设简化配置
   - 启用并行构建和编译缓存

5. **故障恢复**：
   - 保留构建日志
   - 使用调试模式排查问题

## 总结

vcpkg 为 MeshLab 项目提供了现代化的依赖管理方案，具有以下优势：

- ✅ **跨平台一致性**：Windows、Linux、macOS 使用相同配置
- ✅ **版本控制**：精确控制依赖版本，确保构建可重现
- ✅ **自动化管理**：自动下载、编译和链接依赖
- ✅ **二进制缓存**：加速重复构建
- ✅ **模块化设计**：通过 features 管理可选依赖

通过本指南，您应该能够成功配置和使用 vcpkg 构建 MeshLab 项目。如遇到问题，请参考故障排除部分或提交 Issue。

---

**相关链接**：
- [vcpkg 官方文档](https://vcpkg.io/)
- [MeshLab GitHub](https://github.com/cnr-isti-vclab/meshlab)
- [CMake 文档](https://cmake.org/documentation/)

**版本历史**：
- v1.0.0 (2025-01-13): 初始版本，基于 MeshLab 2025.07.0