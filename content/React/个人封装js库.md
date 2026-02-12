---
title: 打包
draft: false
tags:
---
 # npm 类库打包
tsup
安装
npm i tsup -D
pnpm add tsup -D
## 用法
tsup [...files]
## 配置
- tsup.config.ts
- tsup.config.js
- tsup.config.cjs
- tsup.config.json
## 生成 declaration 文件
```bash
tsup index.ts --dts
```
## 生成 sorcemap 文件
```bash
tsup index.ts --sourcemap
```
## Bundle formats
```bash
tsup src/index.ts --format esm,cjs,iife
```
## 编译时 环境变量
```bash
tsup src/index.ts --env.NODE_ENV production
```
Note that `--env.VAR_NAME` only recognizes `process.env.VAR_NAME` and `import.meta.env.VAR_NAME`. If you use `process.env`, it will only take effect when it is used as a built-in global variable. Therefore, do not import `process` from `node:process`.

# npm app 打包
## turbopack
## vite
tsc -b && vite build
打包后将dist上传到服务器~/dist
然后配合nginx 
sudo mv ~/dist/* /var/www/html/
或者 sudo rsync -av --delete ~/dist/ /var/www/html/



# 路由
对于单页面应用来说，路由是必不可少的，可以通过自己封装也可以使用流行的框架
- 自定义
- nextjs
## 自定义
自己实现的思路主要是通过一些变量来控制哪些组件、页面是否显示

# 状态
对于声明式组件，状态也是必修课
好的状态管理对软件的扩展性和维护效率都有很大提示
react的哲学就很不错，组件中只能写纯ui相关的，一旦牵扯到业务或者外部系统，就要使用一些脱围机制
# 数据
常见的场景
- 增加数据
- 删除数据
- 修改数据
- 查询数据

## 增加数据