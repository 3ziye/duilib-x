# duilib-x
duilib-x 是一套功能全面、适配性强的桌面应用 UI 开发框架，核心优势体现在 “灵活定制”“功能完备”“高效适配” 三大维度。

duilib-x 是一个跨平台的 UI 库，支持 Windows、macOS、Linux 等操作系统，集成了 Skia 渲染引擎、SDL 多媒体库、CEF（Chromium Embedded Framework）网页控件及 WebView2（Windows 平台）等核心组件，提供高效、灵活的 UI 开发能力。本指南将详细介绍 duilib 的编译环境要求、编译参数配置及具体编译步骤。

# 前置条件
## 下载submodule方法
### 1. Git Clone时下载
``` bash
git clone --recursive https://github.com/3ziye/duilib-x.git
```

### 2. Git Clone后再下载
``` bash
git submodule update --init --recursive
```

# 快速开始和编译
# 使用cmake编译
```
mkdir build
cd build 
cmake ..
```
