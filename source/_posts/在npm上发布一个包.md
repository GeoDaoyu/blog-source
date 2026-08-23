---
title: 在npm上发布一个包
date: 2021-08-14 11:39:15
tags: 
- npm
- GitHub
categories:
- Git
---
在学习ArcGIS JS API的过程中，我尝试自己来实现一个esri/core/Accessor类。所以我这么做了，并且把它做成了一个基类发布到了npm上。

这里主要记录如何发布一个包到npm上。

<!-- more -->

## 创建项目

首先创建一个项目，正常的`npm init -y`初始化就好。然后传到仓库。比如我传到了GitHub。

## 准备工作

先安装你的依赖（如果你有依赖的话），和填写`README.md`、`LICENSE`等。

然后我需要配置`package.json`和`tsconfig.json`。

`package.json`：

``` json
{
  "name": "@geodaoyu/accessor", // 包名，比如要安装当前包，则 npm i @geodaoyu/accessor
  "version": "1.0.1", // 版本号，a b c三位
  "description": "Implement esri/core/Accessor by myself.", // 描述
  "main": "index.js", // 主文件地址
  "module": "dist/index.js", // 导出的模块地址
  "scripts": { // 脚本命令，我这里只写了两个，通过npm run来执行
    "build": "npx tsc", // 打包编译，因为我是ts库
    "test": "mocha" // mocha做测试
  },
  "keywords": [ // 关键字，方便别人检索
    "Accessor",
    "Esri",
    "ArcGIS"
  ],
  "author": "GeoDaoyu", // 作者
  "license": "MIT", // 许可
  "type": "module", // 类型
  "repository": { // git仓库地址
    "type": "git",
    "url": "https://github.com/GeoDaoyu/Accessor.git"
  },
  "devDependencies": { // 开发依赖
    "mocha": "^8.3.2",
    "typescript": "^4.2.4"
  }
}
```

`tsconfig.json`：

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "module": "ESNext",
    "target": "ESNext",
    "sourceMap": false,
    "rootDir": "./src", // 根地址
    "outDir": "./dist", // 输出地址
    "esModuleInterop": true,
    "declaration": true,
    "skipLibCheck": true,
    "moduleResolution": "node",
  },
  "include": [
    "src/**/*.ts",
  ],
  "exclude": []
}
```

## 编写代码

然后就是正常的编写你的代码。调试通过`node`来调试。

## 测试代码

编写`xxx.test.js`等测试代码，然后`npm run test`来测试。

## 发布

需要一个npm的账号，然后`npm publish`即可发布。