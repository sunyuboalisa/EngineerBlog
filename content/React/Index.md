---
longform:
  format: scenes
  title: React
  workflow: Default Workflow
  sceneFolder: /
  scenes: []
  ignoredFiles: []
---

# 前端环境
在windows下如果提示powershell执行不了的时候可以执行下列命令
```
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned -Force
```
## Git
https://git-scm.com/downloads
```
git config --global user.name "alisa"   
git config --global user.email sunyubo.alisa@outlook.com
```
### 简单教程
创建仓库
```
cd 项目目录
git init
```
将文件添加到版本控制中
```
git add 文件
# 查看仓库中文件状态
git status
# 对比差异
git diff
```
将文件删除
```
rm 文件
git rm 文件
```
撤销修改
```
git checkout --<file>
```
提交到本地仓库
```
git commit -m "提交日志"
```

远程仓库
```
# 添加远程仓库
git remote add pb https://github.com/paulboone/ticgit
# 查看远程仓库
git remote -v
# 从远程仓库中拉取代码
git fetch <remote>
# 从远程仓库中拉取代码并自动尝试合并
git pull <remote>
# 推送到远程仓库
git push remote branch
# 查看远程仓库信息
git show remote
```
标签
```
# 查看标签
git tag
# 打标签
git tag v1
# 切换到某个标签
git checkout -b branch tagname
```
分支
```
# 添加分支
git branch branchname
# 切换到分支
git checkout branchname
# 根据分支为基新建一个新的分支
git checkout branchname -b newbranchname
# 合并分支
git merge otherbranchname
```

配置ssh
windows中
```
ssh-keygen -t rsa -b 4096 -C "sunyubo.alisa@outlook.com"
# 将生成的公钥内容复制到github中

# 编辑用户目录下.ssh/config 文件（如果没有就创建一个）
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa

# 也可以选择ssh agent的方式
Get-Service -Name ssh-agent | Set-Service -StartupType Manual
Start-Service ssh-agent
ssh-add ~/.ssh/id_rsa
```
## NVM
### Linux 环境
https://github.com/nvm-sh/nvm

### Windows环境
https://code.visualstudio.com/download
从GitHub下载安装包后，安装完验证下是否成功，默认接下来终端都是powershell
```
nvm version
```
国内的用户可以配置镜像
```
nvm node_mirror https://npmmirror.com/mirrors/node/
nvm npm_mirror https://npmmirror.com/mirrors/npm/
npm config set registry https://registry.npmmirror.com/
npm config get registry

```
安装最新的nodejs lts
```
nvm install lts
```
使用已安装的nodejs
```
nvm ls
# 假设输出22.17.0
nvm use 22.17.0
node -v 或者 nvm current
# 应显示v22.17.0
```
要注意全局安装的包，在每次切换使用的nodejs版本后都要重新安装
```
nvm use 14.0.0
npm install -g yarn
nvm use 12.0.1
npm install -g yarn
```
## VS Code
### Windows 环境
https://code.visualstudio.com/docs/setup/windows
#### 常用插件
Prettier - Code formatter
#### 支持 TypeScript
```
npm install -g typescript
# 验证是否安装成功
tsc --version
```
#### React
- next.js
```
npx create-next-app@latest
```
- vite
```
npm create vite@latest my-app -- --template react
```
- react-router
```
npx create-react-router@latest
```


# Electron

# TypeScript
安装 react 库
```
npm install @types/react @types/react-dom
```

# VS Code
安装 prettier

## 保存并自动格式化

理想情况下，你应该在每次保存时自动格式化代码。VS Code 就包含该配置!

1. 在 VS Code, 按快捷键 `Ctrl/Cmd + Shift + P`.
2. 输入 “settings”
3. 按回车键
4. 在搜索栏, 输入 “format on save”
5. 确保勾选 “format on save” 选项！

禁用 eslint 格式检查
安装 eslint-config-prettier
```
npm i -D eslint-config-prettier   
```
然后在`.eslintrc.*` file 或者 `eslint.config.json`中添加
```
# .eslintrc.*
{
  "extends": [
    "some-other-config-you-use",
    "prettier"
  ]
}

# eslint.config.js
import someConfig from "some-other-config-you-use";
// Note the `/flat` suffix here, the difference from default entry is that
// `/flat` added `name` property to the exported object to improve
// [config-inspector](https://eslint.org/blog/2024/04/eslint-config-inspector/) experience.
import eslintConfigPrettier from "eslint-config-prettier/flat";

export default [
  someConfig,
  eslintConfigPrettier,
];
```
可以用 `npx @eslint/config-inspector` 检查是否成功应用