---
title: Tauri
draft: false
tags:
---
Tauri is a framework for building tiny, fast binaries for all major desktop and mobile platform
# 环境
- 需要提前安装 Rust 相关开发环境
- WebView2 （一般windows10以上都自带）
- Node.js
- VS Code
# 基本用法
创建项目
```shell
bun create tauri-app
```

生成icon
```shell
#注意要提前在根目录放app-icon.png 且是正方形的
bun tauri icon
```

运行
```shell
bun tauri dev
```

打包
```shell
bun tauri build
```