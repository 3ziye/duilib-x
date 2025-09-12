
---

# duilib 编译指南

## 1. 环境要求

### 1.1 操作系统支持

| 操作系统    | 支持状态 | 特殊说明                                                                                                    |
| ------- | ---- | ------------------------------------------------------------------------------------------------------- |
| Windows | 完全支持 | 支持 WebView2 控件，需提前安装 [WebView2 Runtime](https://developer.microsoft.com/zh-cn/microsoft-edge/webview2/) |
| macOS   | 完全支持 | 依赖系统框架（如 AppKit），需 Xcode 开发环境                                                                           |
| Linux   | 完全支持 | 需安装 Freetype、Fontconfig 等依赖库                                                                            |
| FreeBSD | 部分支持 | 禁用 CEF 功能，需手动安装 Freetype、Fontconfig 依赖                                                                  |

### 1.2 编译工具链



| 操作系统          | 推荐工具链                  | 最低版本要求             |
| ------------- | ---------------------- | ------------------ |
| Windows       | MSVC（Visual Studio 组件） | Visual Studio 2019 |
| macOS         | Clang（Xcode 命令行工具）     | Xcode 12.0         |
| Linux/FreeBSD | GCC 或 Clang            | GCC 9.0 / Clang 10 |

### 1.3 依赖组件

duilib 编译依赖以下第三方库，需确保源码已放置在指定目录（默认路径：`duilib/third-party/`）：



*   **Skia**：2D 渲染引擎，路径需为 `third-party/skia/`

*   **SDL**：多媒体与窗口管理库，路径需为 `third-party/SDL/`

*   **CEF**：网页渲染控件，路径需为 `third-party/libcef_<平台>/`（如 Windows 对应 `libcef_win`）

*   **其他内置依赖**：apng、stb\_image、svg、zlib（minizip）、libudis86（仅 Windows），源码已包含在 `third-party/` 目录，无需额外下载

## 2. 编译前准备

### 2.1 目录结构确认

确保 duilib 源码目录结构符合以下规范（否则需修改 CMake 脚本中的路径配置）：



```
duilib/                      # 根目录（DUILIB\_SRC\_ROOT\_DIR）

├── cmake/                   # CMake 辅助脚本目录

│   └── duilib\_compiles.cmake # 编译器参数配置脚本

├── src/                     # duilib 核心源码目录（CMakeLists.txt 所在目录）

│   ├── Animation/           # 动画相关源码

│   ├── Box/                 # 布局容器源码

│   ├── CEFControl/          # CEF 控件源码

│   ├── Control/             # 基础控件源码

│   ├── Core/                # 核心逻辑（窗口、消息等）

│   ├── Image/               # 图片处理源码

│   ├── Render/              # 渲染基础源码

│   ├── RenderSkia/          # Skia 渲染实现

│   ├── Utils/               # 工具函数源码

│   └── WebView2/            # Windows WebView2 控件源码

└── third-party/             # 第三方依赖目录

&#x20;   ├── apng/                # APNG 图片支持

&#x20;   ├── libcef\_<平台>/       # CEF 库（按平台区分）

&#x20;   ├── libudis86/           # Windows 反汇编工具（仅 Windows）

&#x20;   ├── SDL/                 # SDL 库

&#x20;   ├── skia/                # Skia 渲染引擎

&#x20;   ├── stb\_image/           # 图片加载库

&#x20;   ├── svg/                 # SVG 解析库

&#x20;   ├── zlib/                # 压缩库（minizip）

&#x20;   └── xml/                 # XML 解析库
```

### 2.2 依赖安装（Linux/FreeBSD）

Linux 和 FreeBSD 需手动安装系统依赖库，通过包管理器执行以下命令：

#### Linux（Debian/Ubuntu 系列）



```
sudo apt-get update

sudo apt-get install -y build-essential cmake libfreetype6-dev libfontconfig1-dev
```

#### FreeBSD



```
pkg install -y cmake gcc freetype2 fontconfig
```

## 3. 编译参数配置

CMake 脚本支持以下自定义参数，可通过 `cmake -D<参数名>=<值>` 方式指定：



| 参数名                  | 可选值               | 默认值       | 说明                                                            |
| -------------------- | ----------------- | --------- | ------------------------------------------------------------- |
| `DUILIB_MD`          | `ON`/`OFF`        | `OFF`     | Windows 平台 MSVC 运行时库选择：`ON`=MD/MDd（动态运行时），`OFF`=MT/MTd（静态运行时） |
| `DUILIB_ENABLE_SDL`  | `ON`/`OFF`        | `OFF`     | Windows 平台是否启用 SDL 支持（启用后定义 `DUILIB_SDL=1`）                   |
| `CMAKE_BUILD_TYPE`   | `Debug`/`Release` | `Release` | 编译类型：Debug（调试版，含符号）、Release（发布版，优化编译）                         |
| `CMAKE_CXX_STANDARD` | `20`              | `20`      | C++ 标准版本（固定为 C++20，不建议修改）                                     |

## 4. 编译步骤

duilib 采用 **out-of-tree build**（离树编译）模式，禁止在源码目录内直接编译，需按照以下步骤执行：

### 4.1 创建编译目录

在 duilib 根目录外创建独立的编译目录（如 `build`），并进入该目录：



```
\# 示例（Linux/macOS/Windows PowerShell）

mkdir -p \~/duilib\_build  # 自定义编译目录路径

cd \~/duilib\_build
```

### 4.2 生成构建文件

通过 `cmake` 命令指定源码目录（`-S`）和编译目录（`-B`），并配置自定义参数（可选）：

#### 基础命令（默认配置）



```
\# 通用命令（需将 \<DUILIB\_SRC\_ROOT> 替换为 duilib 根目录绝对路径）

cmake -S \<DUILIB\_SRC\_ROOT>/src -B .
```

#### 带自定义参数的命令（示例）



```
\# Windows 平台：启用动态运行时（MD/MDd）+ SDL 支持

cmake -S D:/duilib/src -B . -DDUILIB\_MD=ON -DDUILIB\_ENABLE\_SDL=ON -DCMAKE\_BUILD\_TYPE=Debug

\# Linux 平台：指定 GCC 编译器，启用 Debug 模式

cmake -S /home/user/duilib/src -B . -DCMAKE\_CXX\_COMPILER=g++ -DCMAKE\_BUILD\_TYPE=Debug
```

### 4.3 执行编译

使用 `cmake --build` 命令执行编译，可通过 `-j` 参数指定并行编译线程数（建议设为 CPU 核心数，加快编译速度）：



```
\# 通用命令（-j8 表示 8 线程编译，根据 CPU 核心数调整）

cmake --build . -j8
```

#### 编译结果输出

编译生成的库文件（静态库 `.a`/`.lib` 或动态库 `.so`/`.dll`）将自动输出到：



```
\<DUILIB\_SRC\_ROOT>/lib/  # duilib 根目录下的 lib 文件夹
```

## 5. 常见问题解决

### 5.1 "Prevented in-tree build" 错误

**原因**：在源码目录（如 `src/`）内直接执行了 `cmake`。

**解决**：删除源码目录内的 `CMakeCache.txt` 和 `CMakeFiles/`，重新按照步骤 4.1 创建独立编译目录。

### 5.2 Windows 平台 WebView2 相关错误

**原因**：未安装 WebView2 Runtime，或 `WebView2` 源码目录缺失。

**解决**：



1.  安装 [WebView2 Runtime](https://developer.microsoft.com/zh-cn/microsoft-edge/webview2/)；

2.  确保 `src/WebView2/` 目录存在且包含源码。

### 5.3 Linux/FreeBSD "Freetype/Fontconfig not found" 错误

**原因**：未安装 Freetype 或 Fontconfig 依赖库。

**解决**：参考步骤 2.2 执行包管理器安装命令。

### 5.4 Skia 编译参数不匹配错误

**原因**：Skia 需启用 AVX/AVX2 指令集，与 duilib 编译参数不一致。

**解决**：CMake 脚本会自动检测 CPU 支持的指令集并适配，无需手动修改；若仍报错，可检查 `cmake/duilib_compiles.cmake` 中的指令集配置。

## 6. 注意事项



1.  **CEF 支持限制**：FreeBSD 平台不支持 CEF，编译时会自动禁用；Windows/macOS/Linux 需确保 `third-party/libcef_<平台>/` 目录存在且包含 CEF 头文件。

2.  **编码格式**：MSVC 编译器已配置 `/utf-8` 选项，确保源码文件为 UTF-8 编码，避免中文乱码。

3.  **多线程编译**：Windows 平台通过 `/MP<CPU_CORES>` 自动启用多线程编译，Linux/macOS 需通过 `-j` 参数手动指定线程数。

4.  **静态 / 动态运行时**：Windows 平台若需静态链接运行时（MT/MTd），需确保所有依赖库（如 Skia、SDL）也使用相同的运行时配置，避免链接错误。