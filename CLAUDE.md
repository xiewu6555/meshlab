# MeshLab 项目指南 v2025.07.1

本文档为 MeshLab 项目开发者和 Claude Code (claude.ai/code) 提供全面的技术指导。

## 目录

1. [项目概述](#项目概述)
2. [环境配置指南](#环境配置指南)
3. [快速开始](#快速开始)
4. [核心架构](#核心架构)
5. [vcpkg依赖管理](#vcpkg依赖管理)
6. [构建系统](#构建系统)
7. [开发工作流程](#开发工作流程)
8. [故障排除](#故障排除)
9. [版本历史](#版本历史)

## 项目概述

MeshLab是一个基于Qt5的开源3D网格处理和编辑系统，使用C++17标准开发。它构建在VCGLib库之上，提供了丰富的网格处理算法和可扩展的插件架构。

### 主要特性
- 支持50+种3D文件格式的导入/导出
- 提供200+种网格处理算法
- 模块化插件架构，支持动态扩展
- 跨平台支持（Windows、Linux、macOS）
- OpenGL 4.6渲染支持
- 完整的vcpkg依赖管理

### 技术栈
- **编程语言**: C++17
- **GUI框架**: Qt 5.15+
- **几何库**: VCGLib
- **构建系统**: CMake 3.18+
- **依赖管理**: vcpkg
- **编译器**: Visual Studio 2022 (Windows) / GCC 9+ (Linux) / Clang 12+ (macOS)

## 环境配置指南

### Windows环境配置（推荐配置）

#### 系统要求
- Windows 10/11 64位
- Visual Studio 2022（含C++开发工具）
- Git for Windows
- CMake 3.18或更高版本
- 至少8GB RAM，推荐16GB
- 10GB可用磁盘空间

#### 环境准备步骤

##### 1. 安装Visual Studio 2022
```powershell
# 确保安装以下组件：
# - MSVC v143 - VS 2022 C++ x64/x86 生成工具
# - Windows 10/11 SDK
# - CMake工具
# - C++ ATL
```

##### 2. 安装CMake
```powershell
# 方式1：使用winget（推荐）
winget install Kitware.CMake

# 方式2：从官网下载
# https://cmake.org/download/
# 确保添加到系统PATH
```

##### 3. 验证环境
```powershell
# 检查编译器
cl.exe
# 应显示: Microsoft (R) C/C++ Optimizing Compiler Version 19.xx

# 检查CMake
cmake --version
# 应显示: cmake version 3.18或更高

# 检查Git
git --version
# 应显示: git version 2.x.x
```

### Linux环境配置

#### Ubuntu/Debian系统
```bash
# 更新包管理器
sudo apt update

# 安装基础开发工具
sudo apt install -y \
    build-essential \
    cmake \
    git \
    ninja-build \
    pkg-config

# 安装Qt5开发包
sudo apt install -y \
    qt5-default \
    libqt5opengl5-dev \
    libqt5xmlpatterns5-dev

# 安装OpenGL相关
sudo apt install -y \
    libgl1-mesa-dev \
    libglu1-mesa-dev \
    libglew-dev

# 安装其他依赖
sudo apt install -y \
    libeigen3-dev \
    libgmp-dev \
    libmpfr-dev \
    libboost-all-dev
```

#### Fedora/RHEL系统
```bash
# 安装开发工具组
sudo dnf groupinstall -y "Development Tools"
sudo dnf install -y cmake ninja-build

# 安装Qt5和OpenGL
sudo dnf install -y \
    qt5-qtbase-devel \
    qt5-qtxmlpatterns-devel \
    mesa-libGL-devel \
    mesa-libGLU-devel \
    glew-devel

# 安装其他依赖
sudo dnf install -y \
    eigen3-devel \
    gmp-devel \
    mpfr-devel \
    boost-devel
```

### macOS环境配置

```bash
# 安装Xcode命令行工具
xcode-select --install

# 使用Homebrew安装依赖
brew install cmake ninja qt@5 eigen glew boost

# 设置Qt路径
export Qt5_DIR=/usr/local/opt/qt@5
export PATH="/usr/local/opt/qt@5/bin:$PATH"
```

## 快速开始

### 方式一：使用vcpkg构建（推荐）

这是最简单且最可靠的构建方式，所有依赖会自动管理。

#### Windows完整构建流程

```powershell
# 1. 克隆仓库（包含子模块）
git clone --recursive https://github.com/cnr-isti-vclab/meshlab.git
cd meshlab

# 2. 初始化vcpkg（首次构建）
.\setup_vcpkg.bat
# 或者手动初始化
git clone https://github.com/Microsoft/vcpkg.git
.\vcpkg\bootstrap-vcpkg.bat

# 3. 使用CMake预设配置（推荐）
cmake --preset vcpkg-default

# 4. 构建项目
cmake --build build\vcpkg --config Release

# 5. 运行MeshLab
.\build\vcpkg\src\meshlab\Release\meshlab.exe
```

#### Linux/macOS完整构建流程

```bash
# 1. 克隆仓库
git clone --recursive https://github.com/cnr-isti-vclab/meshlab.git
cd meshlab

# 2. 设置vcpkg环境
bash setup_vcpkg.sh
source vcpkg_env.sh

# 3. 配置和构建
cmake --preset vcpkg-default
cmake --build build/vcpkg

# 4. 运行MeshLab
./build/vcpkg/src/meshlab/meshlab
```

### 方式二：传统构建方式

适用于已有系统依赖或特殊环境的情况。

```bash
# 使用平台脚本
bash scripts/Windows/0_setup_env.sh
bash scripts/Windows/1_build.sh

# 或手动构建
mkdir build && cd build
cmake -GNinja -DMESHLAB_USE_VCPKG=OFF ..
ninja
```

### 常用开发任务

- **增量构建**: `cmake --build build/vcpkg`
- **清理构建**: `rm -rf build/` 或 `rmdir /s build\`
- **调试模式**: `cmake --preset vcpkg-debug`
- **查看日志**: 添加 `-DMESHLAB_ENABLE_DEBUG_LOG_FILE=ON`
- **并行构建**: `cmake --build build/vcpkg -j8`

## 核心架构

### 项目结构详解

```
meshlab/
├── src/                          # 源代码目录
│   ├── meshlab/                  # 主应用程序
│   │   ├── main.cpp             # 程序入口
│   │   ├── mainwindow.cpp       # 主窗口实现
│   │   └── glarea.cpp           # OpenGL渲染区域
│   ├── common/                   # 共享库
│   │   ├── ml_mesh.h            # 网格数据结构
│   │   ├── ml_document.h        # 文档管理
│   │   └── interfaces/          # 插件接口定义
│   ├── meshlabplugins/          # 插件目录（50+插件）
│   │   ├── filter_*/            # 滤镜插件
│   │   ├── io_*/                # 导入导出插件
│   │   ├── edit_*/              # 编辑工具插件
│   │   └── render_*/            # 渲染插件
│   ├── external/                 # 外部依赖
│   └── vcglib/                  # VCG库（子模块）
├── vcpkg.json                   # vcpkg包清单
├── vcpkg-configuration.json     # vcpkg配置
├── CMakePresets.json            # CMake预设
├── CMakeLists.txt               # 主构建文件
└── ML_VERSION                   # 版本信息
```

### 关键组件依赖关系

```
┌─────────────────────────────────────┐
│         MeshLab GUI应用              │
├─────────────────────────────────────┤
│         插件系统 (50+插件)            │
├─────────────────────────────────────┤
│      Common库 (核心数据结构)          │
├─────────────────────────────────────┤
│       VCGLib (几何算法库)            │
├─────────────────────────────────────┤
│   Qt5 │ OpenGL │ GLEW │ Eigen3     │
└─────────────────────────────────────┘
```

### 插件系统架构

MeshLab采用模块化插件架构，支持5种插件类型：

1. **FilterPlugin** - 网格处理算法
   - 清理和修复
   - 简化和细分
   - 平滑和变形
   - 纹理处理

2. **IOPlugin** - 文件格式支持
   - 3D模型格式（OBJ、PLY、STL、FBX等）
   - 点云格式（PCD、LAS、E57等）
   - CAD格式（STEP、IGES等）

3. **EditPlugin** - 交互式编辑工具
   - 选择工具
   - 绘画工具
   - 测量工具

4. **RenderPlugin** - 渲染模式
   - 着色模式
   - 光照效果
   - 材质渲染

5. **DecoratePlugin** - 装饰和信息显示
   - 坐标轴
   - 背景设置
   - 统计信息

## vcpkg依赖管理

### vcpkg配置文件说明

#### vcpkg.json - 包清单文件
定义项目所需的所有依赖包和版本要求。

```json
{
  "name": "meshlab",
  "version": "2025.7.0",
  "dependencies": [
    "qt5-base",
    "opengl",
    "glew",
    "eigen3"
  ],
  "features": {
    "meshlab-plugins": {
      "description": "插件依赖",
      "dependencies": [
        "boost-algorithm",
        "cgal",
        "embree4",
        "qhull"
      ]
    }
  }
}
```

#### vcpkg-configuration.json - 注册表配置
指定包源和基线版本，确保构建的可重现性。

```json
{
  "default-registry": {
    "kind": "git",
    "repository": "https://github.com/Microsoft/vcpkg",
    "baseline": "最新稳定版本哈希"
  }
}
```

#### CMakePresets.json - CMake预设配置
提供标准化的构建配置选项。

### vcpkg功能特性详解

#### 核心依赖（core）
始终需要的基础依赖：
- **Qt5**: GUI框架（5.15或更高）
- **OpenGL**: 3D渲染
- **GLEW**: OpenGL扩展管理
- **Eigen3**: 线性代数库

#### 插件依赖（meshlab-plugins）
构建完整插件集所需：
- **Boost**: 算法和文件系统支持
- **CGAL**: 计算几何算法库
- **Embree**: Intel光线追踪库
- **QHull**: 凸包计算
- **MuParser**: 数学表达式解析
- **TBB**: Intel线程构建块
- **Xerces-C**: XML解析

#### 开发工具（meshlab-tools）
- **Ninja**: 快速构建系统
- **ccache**: 编译缓存（可选）

### vcpkg使用技巧

#### 1. 查看已安装的包
```bash
# Windows
.\vcpkg\vcpkg.exe list

# Linux/macOS
./vcpkg/vcpkg list
```

#### 2. 二进制缓存配置
加速重复构建：
```bash
# 设置本地缓存
export VCPKG_BINARY_SOURCES="clear;files,./vcpkg-cache,readwrite"

# 使用NuGet缓存（企业环境）
export VCPKG_BINARY_SOURCES="clear;nuget,https://your-server/v3/index.json,readwrite"
```

#### 3. 指定特定版本
在vcpkg.json中使用版本约束：
```json
{
  "dependencies": [
    {
      "name": "qt5-base",
      "version>=": "5.15.0"
    }
  ]
}
```

#### 4. 自定义triplet
针对特定平台优化：
```bash
# 创建自定义triplet
cp vcpkg/triplets/x64-windows.cmake vcpkg/triplets/community/x64-windows-meshlab.cmake
# 编辑设置，如启用静态链接
set(VCPKG_LIBRARY_LINKAGE static)
```

## 构建系统

### CMake配置选项详解

#### 基础构建选项
```cmake
# vcpkg集成
-DMESHLAB_USE_VCPKG=ON              # 启用vcpkg（推荐）

# 构建类型
-DCMAKE_BUILD_TYPE=Release          # Release/Debug/RelWithDebInfo/MinSizeRel

# 构建范围
-DMESHLAB_BUILD_MINI=OFF            # 仅构建核心组件
-DMESHLAB_BUILD_ONLY_LIBRARIES=OFF  # 仅构建库

# 精度设置
-DMESHLAB_BUILD_WITH_DOUBLE_SCALAR=OFF  # 使用双精度浮点

# 调试选项
-DMESHLAB_ENABLE_DEBUG_LOG_FILE=OFF # 启用调试日志
-DMESHLAB_BUILD_STRICT=ON           # 严格符号解析
```

#### vcpkg特定选项
```cmake
# vcpkg工具链
-DCMAKE_TOOLCHAIN_FILE=path/to/vcpkg.cmake

# 目标架构
-DVCPKG_TARGET_TRIPLET=x64-windows  # x64-windows/x64-linux/x64-osx

# 功能选择
-DVCPKG_MANIFEST_FEATURES="meshlab-plugins;meshlab-tools"

# 主机架构（交叉编译）
-DVCPKG_HOST_TRIPLET=x64-windows
```

#### 高级选项
```cmake
# 并行构建
-DCMAKE_BUILD_PARALLEL_LEVEL=8

# 安装路径
-DCMAKE_INSTALL_PREFIX=/usr/local

# 编译器选项
-DCMAKE_CXX_FLAGS="-O3 -march=native"

# 链接器选项
-DCMAKE_EXE_LINKER_FLAGS="-static-libgcc -static-libstdc++"
```

### CMake预设使用指南

#### 可用预设

1. **vcpkg-default** - 标准Release构建
   ```bash
   cmake --preset vcpkg-default
   cmake --build build/vcpkg --config Release
   ```

2. **vcpkg-debug** - 调试构建
   ```bash
   cmake --preset vcpkg-debug
   cmake --build build/vcpkg-debug --config Debug
   ```

3. **vcpkg-with-plugins** - 完整构建（含所有插件）
   ```bash
   cmake --preset vcpkg-with-plugins
   cmake --build build/vcpkg-plugins --config Release
   ```

#### 创建自定义预设

编辑CMakePresets.json添加：
```json
{
  "name": "my-custom-preset",
  "inherits": "vcpkg-default",
  "cacheVariables": {
    "MESHLAB_BUILD_WITH_DOUBLE_SCALAR": "ON",
    "CMAKE_CXX_FLAGS": "-O3 -march=native"
  }
}
```

### 平台特定构建脚本

#### Windows构建脚本
```batch
:: build_debug_simple.bat - 简单调试构建
@echo off
set BUILD_DIR=build_debug
cmake -S . -B %BUILD_DIR% -G "Visual Studio 17 2022" -A x64 ^
      -DMESHLAB_USE_VCPKG=ON ^
      -DCMAKE_BUILD_TYPE=Debug
cmake --build %BUILD_DIR% --config Debug
```

#### Linux/macOS构建脚本
```bash
#!/bin/bash
# build_with_vcpkg.sh - vcpkg完整构建

# 设置颜色输出
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m'

echo -e "${GREEN}初始化vcpkg...${NC}"
bash setup_vcpkg.sh
source vcpkg_env.sh

echo -e "${GREEN}配置CMake...${NC}"
cmake --preset vcpkg-with-plugins

echo -e "${GREEN}构建项目...${NC}"
cmake --build build/vcpkg-plugins -j$(nproc)

echo -e "${GREEN}构建完成！${NC}"
```

## 开发工作流程

### 添加新插件

#### 1. 创建插件目录结构
```bash
src/meshlabplugins/filter_mynewfilter/
├── filter_mynewfilter.h      # 插件头文件
├── filter_mynewfilter.cpp    # 插件实现
├── CMakeLists.txt            # 构建配置
└── README.md                 # 插件文档
```

#### 2. 实现插件接口
```cpp
// filter_mynewfilter.h
class FilterMyNewFilter : public QObject, public FilterPlugin
{
    Q_OBJECT
    Q_INTERFACES(FilterPlugin)
    Q_PLUGIN_METADATA(IID "vcg.meshlab.FilterPlugin/1.0")

public:
    enum { FP_MY_FILTER };

    FilterMyNewFilter();
    QString filterName(ActionIDType filter) const override;
    QString filterInfo(ActionIDType filter) const override;
    FilterClass getClass(const QAction*) const override;
    RichParameterList initParameterList(const QAction*, const MeshDocument&) override;
    bool applyFilter(const QAction*, MeshDocument&,
                    std::map<std::string, QVariant>&,
                    unsigned int&) override;
};
```

#### 3. 配置CMakeLists.txt
```cmake
set(SOURCES filter_mynewfilter.cpp)
set(HEADERS filter_mynewfilter.h)

add_library(filter_mynewfilter MODULE ${SOURCES} ${HEADERS})

target_link_libraries(filter_mynewfilter
    PUBLIC
        meshlab-common
        VCG::header_only)

set_property(TARGET filter_mynewfilter
    PROPERTY FOLDER Plugins/Filter)

install(TARGETS filter_mynewfilter
    DESTINATION ${MESHLAB_PLUGIN_INSTALL_DIR})
```

#### 4. 注册插件
在`src/meshlabplugins/CMakeLists.txt`中添加：
```cmake
add_subdirectory(filter_mynewfilter)
```

### 添加新的3D格式支持

#### 1. 创建IO插件
```cpp
class IOMyFormat : public IOPlugin
{
public:
    QList<Format> importFormats() const override
    {
        QList<Format> formatList;
        formatList << Format("My Format", tr("MYF"));
        return formatList;
    }

    bool open(const QString& format, const QString& fileName,
              MeshModel& mesh, int& mask,
              const RichParameterList& params,
              CallBackPos* cb) override
    {
        // 实现文件读取逻辑
        return true;
    }
};
```

#### 2. 实现格式解析
```cpp
bool parseMyFormat(const QString& filename, MeshModel& mesh)
{
    QFile file(filename);
    if (!file.open(QIODevice::ReadOnly))
        return false;

    QTextStream stream(&file);
    // 解析文件内容
    // 填充mesh.cm的顶点、面等数据

    return true;
}
```

### 修改核心功能

#### 数据结构修改
主要文件位置：
- `src/common/ml_mesh.h` - 网格数据结构
- `src/common/ml_document.h` - 文档管理
- `src/common/ml_mesh_type.h` - 类型定义

#### GUI修改
- `src/meshlab/mainwindow.cpp` - 主窗口
- `src/meshlab/glarea.cpp` - OpenGL渲染
- `src/meshlab/dialogs/` - 对话框

#### 添加新的参数类型
```cpp
// 在 src/common/rich_parameter.h 中
class RichMyType : public RichParameter
{
public:
    RichMyType(const QString& name,
               const MyType& defaultValue,
               const QString& description);
    // 实现必要的虚函数
};
```

## 故障排除

### 常见构建问题

#### 1. vcpkg初始化失败

**问题**: `setup_vcpkg.sh`执行失败
```
错误: 无法下载vcpkg
```

**解决方案**:
```bash
# 手动克隆vcpkg
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg
./bootstrap-vcpkg.sh  # Linux/macOS
# 或
.\bootstrap-vcpkg.bat  # Windows

# 设置环境变量
export CMAKE_TOOLCHAIN_FILE=$(pwd)/scripts/buildsystems/vcpkg.cmake
```

#### 2. Qt5未找到

**问题**: CMake配置时报错
```
CMake Error: Could not find Qt5
```

**解决方案**:

Windows:
```powershell
# 方式1：使用vcpkg（推荐）
.\vcpkg\vcpkg.exe install qt5-base:x64-windows

# 方式2：手动指定Qt路径
cmake -DQt5_DIR="C:\Qt\5.15.2\msvc2019_64\lib\cmake\Qt5" ..
```

Linux:
```bash
# 安装Qt5开发包
sudo apt install qt5-default libqt5opengl5-dev

# 或指定路径
export Qt5_DIR=/usr/lib/x86_64-linux-gnu/cmake/Qt5
```

#### 3. OpenGL相关错误

**问题**: 链接错误或运行时错误
```
undefined reference to `glCreateShader'
```

**解决方案**:
```bash
# Linux
sudo apt install libgl1-mesa-dev libglu1-mesa-dev libglew-dev

# Windows - 确保安装了正确的SDK
# Visual Studio Installer中添加Windows SDK
```

#### 4. 插件加载失败

**问题**: 运行时插件不显示
```
Plugin xxx could not be loaded
```

**解决方案**:
1. 检查插件是否正确编译
2. 确认插件在正确路径：`build/src/meshlabplugins/`
3. 检查依赖库是否完整
4. 使用调试模式查看详细错误：
   ```bash
   cmake -DMESHLAB_ENABLE_DEBUG_LOG_FILE=ON ..
   ```

#### 5. 内存不足

**问题**: 构建时内存耗尽
```
c++: fatal error: Killed signal terminated program cc1plus
```

**解决方案**:
```bash
# 减少并行任务数
cmake --build . -j2

# 或增加交换空间
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### 运行时问题

#### 1. OpenGL版本不支持

**问题**: 启动时报告OpenGL版本过低
```
OpenGL 4.6 is required
```

**解决方案**:
- 更新显卡驱动
- Windows: 使用Mesa3D软件渲染作为备选
- Linux: `export MESA_GL_VERSION_OVERRIDE=4.6`

#### 2. 字体渲染问题

**问题**: UI文字显示异常

**解决方案**:
```bash
# Linux
export QT_AUTO_SCREEN_SCALE_FACTOR=1
export QT_SCALE_FACTOR=1.0

# Windows
# 右键属性 -> 兼容性 -> 更改高DPI设置
```

#### 3. 插件崩溃

**问题**: 使用特定插件时崩溃

**调试步骤**:
1. 启用日志：`-DMESHLAB_ENABLE_DEBUG_LOG_FILE=ON`
2. 使用调试器运行：
   ```bash
   gdb ./meshlab
   run
   # 崩溃后
   backtrace
   ```
3. 检查日志文件：`~/.meshlab/meshlab_debug.log`

### 性能优化

#### 1. 编译优化
```cmake
# Release模式优化
cmake -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_CXX_FLAGS="-O3 -march=native" ..

# Link Time Optimization
cmake -DCMAKE_INTERPROCEDURAL_OPTIMIZATION=ON ..
```

#### 2. 并行构建
```bash
# 使用所有CPU核心
cmake --build . -j$(nproc)

# Windows
cmake --build . -j%NUMBER_OF_PROCESSORS%
```

#### 3. ccache加速
```bash
# 安装ccache
sudo apt install ccache  # Linux
brew install ccache      # macOS

# 配置CMake使用ccache
cmake -DCMAKE_CXX_COMPILER_LAUNCHER=ccache ..
```

## 版本历史

### v2025.07.1 (2025-01-13)
**文档更新版本**
- 完善vcpkg环境配置文档
- 添加详细的故障排除指南
- 更新CMake预设配置说明
- 补充Windows/Linux/macOS平台配置
- 添加性能优化建议

### v2025.07.0 (2025-01)
**主要版本更新**
- ✨ 新增独立vcpkg依赖管理系统
- ✨ 添加CMake预设配置支持
- ✨ 改进跨平台构建一致性
- 🔧 支持Visual Studio 2022
- 🔧 升级到Qt 5.15+
- 🔧 支持OpenGL 4.6
- 📦 生成50+插件模块

### 之前版本
- v2024.12 - 基础架构改进
- v2024.06 - 插件系统重构
- v2023.12 - Qt5迁移完成

## 测试指南

### 功能测试
虽然没有自动化测试套件，但可以使用以下方式验证：

1. **基础功能测试**
   ```bash
   # 加载示例文件
   ./meshlab samples/bunny.ply

   # 测试基本操作
   # - 旋转、缩放、平移
   # - 网格信息显示
   # - 渲染模式切换
   ```

2. **插件测试**
   - 使用`samples/`目录中的测试数据
   - 验证各滤镜功能
   - 测试导入导出格式

3. **性能测试**
   - 加载大型模型（>100万面）
   - 测试简化算法
   - 验证内存使用

### 调试技巧

1. **启用详细日志**
   ```cmake
   cmake -DMESHLAB_ENABLE_DEBUG_LOG_FILE=ON ..
   ```

2. **使用调试器**
   ```bash
   # Linux/macOS
   lldb ./meshlab
   gdb ./meshlab

   # Windows (Visual Studio)
   # 在VS中打开build/MeshLab.sln
   # F5启动调试
   ```

3. **内存检查**
   ```bash
   # Linux
   valgrind --leak-check=full ./meshlab

   # Windows
   # 使用Visual Studio诊断工具
   ```

## 部署指南

### 构建产物结构
成功构建后的目录结构：
```
build/vcpkg/
├── src/
│   ├── meshlab/
│   │   └── Release/
│   │       └── meshlab.exe      # 主程序
│   └── meshlabplugins/          # 插件目录
│       ├── filter_*/Release/    # 滤镜插件
│       ├── io_*/Release/        # IO插件
│       └── ...                  # 其他插件
├── shaders/                     # 着色器文件
└── samples/                     # 示例文件
```

### Windows部署
```powershell
# 使用部署脚本
.\scripts\Windows\2_deploy.bat

# 或手动部署
# 1. 收集所有DLL依赖
windeployqt.exe meshlab.exe

# 2. 复制插件
xcopy /E build\vcpkg\src\meshlabplugins\*\Release\*.dll deploy\plugins\

# 3. 复制资源
xcopy /E shaders deploy\shaders\
xcopy /E samples deploy\samples\
```

### Linux部署
```bash
# 创建AppImage
bash scripts/Linux/2_deploy.sh

# 或创建deb包
cpack -G DEB
```

### macOS部署
```bash
# 创建DMG
bash scripts/macOS/3_dmg.sh

# 或使用macdeployqt
macdeployqt meshlab.app -dmg
```

## 贡献指南

### 代码规范
- C++17标准
- 使用现代CMake（3.18+）
- VCGLib命名约定（PascalCase）
- 4空格缩进
- UTF-8编码

### 提交规范
```bash
# 功能添加
feat: 添加新的网格简化算法

# 错误修复
fix: 修复PLY文件导入崩溃问题

# 文档更新
docs: 更新vcpkg配置文档

# 性能优化
perf: 优化大型网格渲染性能
```

### 分支策略
- `main` - 稳定发布版本
- `devel` - 开发分支
- `feature/*` - 功能分支
- `hotfix/*` - 紧急修复

## 参考资源

### 官方资源
- [MeshLab官网](https://www.meshlab.net/)
- [GitHub仓库](https://github.com/cnr-isti-vclab/meshlab)
- [VCGLib文档](http://vcg.isti.cnr.it/vcglib/)

### 技术文档
- [CMake文档](https://cmake.org/documentation/)
- [vcpkg文档](https://vcpkg.io/)
- [Qt5文档](https://doc.qt.io/qt-5/)

### 社区支持
- [MeshLab论坛](https://www.meshlab.net/community/)
- [Stack Overflow标签](https://stackoverflow.com/questions/tagged/meshlab)
- [问题追踪](https://github.com/cnr-isti-vclab/meshlab/issues)

---

*本文档版本: v2025.07.1*
*最后更新: 2025-01-13*
*维护者: MeshLab开发团队*