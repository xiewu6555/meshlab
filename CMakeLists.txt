# Copyright 2019, 2020, Collabora, Ltd.
# Copyright 2019, 2021, Visual Computing Lab, ISTI - Italian National Research Council
# SPDX-License-Identifier: BSL-1.0

cmake_minimum_required(VERSION 3.18)

# vcpkg支持 - 必须在project()之前设置
option(MESHLAB_USE_VCPKG "Use vcpkg for dependency management" OFF)

if(MESHLAB_USE_VCPKG AND NOT DEFINED CMAKE_TOOLCHAIN_FILE)
    # 如果环境变量中有CMAKE_TOOLCHAIN_FILE，使用它
    if(DEFINED ENV{CMAKE_TOOLCHAIN_FILE})
        set(CMAKE_TOOLCHAIN_FILE "$ENV{CMAKE_TOOLCHAIN_FILE}")
    else()
        # 否则使用项目本地的vcpkg
        set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_SOURCE_DIR}/vcpkg/scripts/buildsystems/vcpkg.cmake")
    endif()

    message(STATUS "Using vcpkg toolchain: ${CMAKE_TOOLCHAIN_FILE}")
endif()

project(MeshLab)

### Build options
option(MESHLAB_BUILD_MINI "Build only common and meshlab - other plugin targets must be set manually" OFF)
option(MESHLAB_BUILD_STRICT "Strictly enforce resolution of all symbols" ON)
option(MESHLAB_BUILD_WITH_DOUBLE_SCALAR "Use double type instead of float type for scalars" OFF)
option(MESHLAB_ENABLE_DEBUG_LOG_FILE "If enabled, all the logs of MeshLab will be also saved into a log file" OFF)

option(MESHLAB_BUILD_ONLY_LIBRARIES "Build only meshlab-common and plugins, excluding executables" OFF)
option(MESHLAB_USE_DEFAULT_BUILD_AND_INSTALL_DIRS "If set to OFF, it expects that you set manually the binary and install directories" ON)

option(MESHLAB_IS_NIGHTLY_VERSION "Nightly version of meshlab will be used instead of ML_VERSION" OFF)

add_subdirectory(src)