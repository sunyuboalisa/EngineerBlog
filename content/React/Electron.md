---
title: Electron
draft: false
tags:
---
# 安装
国内需要提前配置镜像，涉及两部分
- npm 镜像
- electron 镜像
直接修改用户目录下或项目目录下.npmrc文件添加以下内容，没有的话新建一个
```
registry=https://registry.npmmirror.com
ELECTRON_MIRROR=https://npmmirror.com/mirrors/electron/
```
## 创建项目
```sh
mkdir my-app
npx create-electron-app@latest my-app
```