今天在写一个老项目时，发现安装的 node 版本太高了，很多依赖下载不下来，询问GPT发现是 node 版本太高了，旧依赖不兼容，这篇博客就来讲下`Windows`下如何进行 node 的版本切换；
## 介绍 NVM（Node Version Manager）
### 什么是NVM
NVM（Node Version Manager）是一个用于管理和切换 Node.js 版本的工具。它允许开发者在同一系统上安装和使用多个版本的 Node.js，方便在不同项目间切换，满足不同版本的需求。
### 主要功能

1. **安装 Node.js 版本**：NVM 可以轻松安装指定的 Node.js 版本，包括最新版本和历史版本。
2. **切换 Node.js 版本**：可以在不同版本的 Node.js 之间快速切换，而不需要重新安装。
3. **管理全局 npm 包**：每个 Node.js 版本都有独立的全局 npm 包，这样可以避免版本冲突。
4. **设置默认版本**：可以设置系统默认使用的 Node.js 版本。
### 如何安装 NVM

1. **下载 NVM for Windows**：
   - 访问[ nvm-windows releases](https://github.com/coreybutler/nvm-windows/releases) 页面，下载最新的 **nvm-setup.exe**。
2. **安装 NVM**：
   - 双击下载的安装文件，按照提示完成安装。
   - 安装就是傻瓜式安装，这里有个注意点
   - 如果你以前安装了 node ，在第二步的时候要选择你的安装目录，否则这个node不会加载进去
   - ![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1715781792607-299c64d3-8a60-49dd-a89f-eef818e823f1.png#averageHue=%23f0efef&clientId=uda8f813a-10f1-4&from=paste&height=386&id=kDFEM&originHeight=482&originWidth=582&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=27017&status=done&style=none&taskId=u70d9ef6b-7232-4fa0-b586-cb77b390329&title=&width=465.6)
3. **校验安装**
   - 安装完成之后，在命令行输入 nvm，出现以下界面表示安装成功
   - ![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1715782063820-64dba6a5-4500-4bca-80ea-af07fc2a580d.png#averageHue=%23181818&clientId=uda8f813a-10f1-4&from=paste&height=373&id=u2389d073&originHeight=760&originWidth=1481&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=100555&status=done&style=none&taskId=uf5d06b3a-0135-4821-abf3-78644ad2888&title=&width=726)

## 使用 NVM 管理 Node.js 版本
### 设置镜像源
在下载 Node.js 之前，可以先设置一下镜像源，提高下载速度（强烈推荐，我刚开始在这里卡住）
```shell
nvm node_mirror https://npmmirror.com/mirrors/node/
nvm npm_mirror https://npmmirror.com/mirrors/npm/
```
### 安装特定版本的 Node.js

1. **查看可安装版本（选其1即可）**
- `nvm list available`
- [https://registry.npmmirror.com/binary.html?path=node](https://registry.npmmirror.com/binary.html?path=node)
2. **安装特定版本**
- ![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1715782574324-b30fbc07-d1ca-44ee-8a6f-864336e10a74.png#averageHue=%230e0e0e&clientId=uda8f813a-10f1-4&from=paste&height=386&id=ufd470159&originHeight=760&originWidth=1292&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=20611&status=done&style=none&taskId=u3e3fa92d-228f-4b97-a363-fefb4120881&title=&width=656)
3. **安装成功之后，nvm 安装目录会出现对应的版本号**
- ![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1715782639436-e2496a7b-f0f1-4e40-8179-93d8343e2458.png#averageHue=%23fcfbfa&clientId=uda8f813a-10f1-4&from=paste&height=421&id=u76f9a590&originHeight=526&originWidth=826&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=45695&status=done&style=none&taskId=u127e3912-d017-4958-9564-b15c54f933f&title=&width=660.8)
4. **切换 node 版本**
- ![image.png](https://cdn.nlark.com/yuque/0/2024/png/22796888/1715782751796-3cdf16ff-82f8-4997-8bb7-aecb98c55924.png#averageHue=%230f0f0f&clientId=uda8f813a-10f1-4&from=paste&height=449&id=u3ead0f0f&originHeight=760&originWidth=1292&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=32017&status=done&style=none&taskId=ueebe4b2f-bea7-4b9f-9e64-35c9796523e&title=&width=764)
## NVM 使用技巧
### 使用技巧

1. **设置默认 Node.js 版本**：你可以设置一个默认版本，这样每次打开新终端时都会自动使用该版本。
```shell
nvm alias default <version>
```

2. **列出所有已安装的 Node.js 版本**：可以查看系统上已经安装的所有 Node.js 版本。
```shell
nvm list
```
或
```shell
nvm ls
```

3. **卸载不需要的 Node.js 版本**：清理不再需要的版本，节省磁盘空间。
```shell
nvm uninstall <version>
```

4. **全局安装 npm 包到特定版本**：切换到特定版本的 Node.js，然后安装全局 npm 包，确保不同版本之间的包隔离。
```shell
nvm use <version>
npm install -g <package>
```

5. **升级 nvm**：保持 **nvm** 本身是最新版本，以获得新功能和修复。
```shell
nvm install-latest-npm
```
### 常用命令
#### 安装和卸载 Node.js 版本

- **安装指定版本的 Node.js**：
```shell
nvm install <version>
```

- **卸载指定版本的 Node.js**：
```shell
nvm uninstall <version>
```

- **使用指定版本的 Node.js**：
```shell
nvm use <version>
```
#### 查看版本信息

- **列出所有可用的 Node.js 版本**：
```shell
nvm list available
```

- **查看当前使用的 Node.js 版本**：
```shell
nvm current
```
