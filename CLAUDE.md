# CLAUDE.md

这个文件为在此代码库中工作的 Claude Code (claude.ai/code) 提供指导。

## 项目概述

MeshLab是一个基于Qt5的开源3D网格处理和编辑系统，用C++17开发。它构建在VCGLib库之上，提供了丰富的网格处理算法和可扩展的插件架构。

## 核心架构

### 代码库结构
- `src/meshlab/` - 主GUI应用程序
- `src/common/` - 共享库和核心数据结构
- `src/meshlabplugins/` - 所有插件的实现
- `src/external/` - 外部依赖管理
- `src/vcglib/` - VCG几何处理库（Git子模块）
- `scripts/` - 平台特定的构建和部署脚本

### 关键组件依赖关系

```
MeshLab应用 
    ↓
Common库（核心数据结构）
    ↓
VCGLib（几何算法）
    ↓
Qt5（GUI框架）
```

插件通过共享接口与核心系统通信，支持热加载和模块化扩展。

## 构建命令

### 基本构建
```bash
# 标准构建流程
mkdir build && cd build
cmake ..
make -j$(nproc)
```

```bash
# 使用Ninja（推荐Windows）
cmake -GNinja ..
ninja
```

### CMake配置选项
- `MESHLAB_BUILD_MINI=ON` - 仅构建核心组件
- `MESHLAB_BUILD_WITH_DOUBLE_SCALAR=ON` - 使用双精度浮点
- `MESHLAB_BUILD_STRICT=ON` - 严格符号解析（默认开启）
- `MESHLAB_ENABLE_DEBUG_LOG_FILE=ON` - 启用调试日志文件
- `MESHLAB_BUILD_ONLY_LIBRARIES=ON` - 仅构建库，不构建可执行文件

### 平台特定构建
使用`scripts/`目录中的自动化脚本：
```bash
# 设置环境依赖
bash scripts/[Linux|macOS|Windows]/0_setup_env.sh

# 构建项目
bash scripts/[Linux|macOS|Windows]/1_build.sh

# 部署应用
bash scripts/[Linux|macOS|Windows]/2_deploy.sh

# 打包分发
bash scripts/[Linux|macOS|Windows]/3_pack.sh
```

## 核心架构组件

### 数据模型层
- **MeshDocument**: 管理多个网格模型的容器
- **MeshModel**: 单个网格的包装器，包含元数据
- **CMeshO**: 核心网格数据结构（基于VCGLib）
- **FilterParameterSet**: 类型安全的参数管理系统

### 插件系统架构
支持5种插件类型：
1. **FilterPlugin**: 网格处理算法（清洗、简化、细分等）
2. **IOPlugin**: 文件格式导入/导出
3. **RenderPlugin**: 自定义渲染模式
4. **EditPlugin**: 交互式编辑工具
5. **DecoratePlugin**: 视觉装饰和信息显示

### 参数系统
- **RichParameterSet**: 强类型参数集合
- **RichFloat/RichInt/RichBool**: 基础数值参数
- **RichMesh**: 网格选择参数
- **RichDirection/RichPoint3f**: 3D向量参数

### 外部依赖管理
- **必需依赖**: Qt5.15+、VCGLib、Eigen3、OpenGL
- **可选依赖**: 根据构建配置自动下载到`src/external/downloads/`
- **插件特定依赖**: OpenMP、CGAL、PCL等

## 开发工作流程

### 添加新插件
1. 在`src/meshlabplugins/`创建新目录
2. 实现相应插件接口（FilterPlugin、IOPlugin等）
3. 创建CMakeLists.txt文件
4. 在父级CMakeLists.txt中添加子目录
5. 实现`applyFilter()`或对应的核心方法

### 修改核心功能
- 数据结构修改：主要在`src/common/ml_mesh.h`
- GUI修改：在`src/meshlab/`目录
- VCGLib集成：通过`src/vcglib/`子模块

### 添加新3D格式支持
1. 在`meshlabplugins`中创建新的IOPlugin
2. 实现`open()`和`save()`方法
3. 在`GetExportFormats()`中注册文件扩展名

## 重要文件路径

### 核心接口
- `src/common/interfaces.h` - 插件接口定义
- `src/common/ml_document.h` - 文档管理
- `src/common/ml_mesh.h` - 网格数据结构

### 主应用程序
- `src/meshlab/mainwindow.cpp` - 主窗口逻辑
- `src/meshlab/glarea.cpp` - OpenGL渲染区域

### 构建配置
- 根目录`CMakeLists.txt` - 主构建配置
- `src/CMakeLists.txt` - 源码构建配置

## 开发注意事项

### 代码规范
- C++17标准
- 使用VCGLib的命名约定（PascalCase）
- OpenGL上下文管理要谨慎（特别是多线程）

### 性能考虑
- 大网格处理：使用VCGLib的高效数据结构
- 渲染优化：考虑VBO和显示列表
- 内存管理：注意大文件加载时的内存使用

### 调试技巧
- 使用`MESHLAB_ENABLE_DEBUG_LOG_FILE=ON`启用日志
- Qt Creator调试器与CMake集成良好
- VCGLib断言在Debug模式下很有用

## 分支策略
- `main`: 稳定发布版本，仅包含错误修复
- `devel`: 新功能开发分支

## 版本信息
版本号存储在根目录的`ML_VERSION`文件中。

## 测试
虽然没有自动化测试套件，但可以使用`sample/`和`textures/`目录中的测试数据验证功能。

## 部署输出结构
构建完成后，安装目录包含：
- `meshlab`或`meshlab.exe` - 主可执行文件
- `plugins/` - 插件动态库
- `shaders/` - OpenGL着色器
- `samples/` - 示例文件