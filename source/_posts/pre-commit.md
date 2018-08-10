---
layout: title
title: pre-commit使用
date: 2018-08-10 14:44:04
tags:
- pre-commit
categories:
- 插件使用
---

# pre-commit

### 简介
一个结合git对项目的代码提交进行拦截的工具，可以配合eslint对代码风格的统一规范

### 安装
```
npm install --save-dev pre-commit
npm install --save-dev eslint-friendly-formatter
npm install --save-dev eslint
```

### 使用

```
//在srcipt 中加入如下命令
"lint": "eslint --format node_modules/eslint-friendly-formatter **/*.js",   //需要安装 eslint-friendly-formatter
"lint:fix": "npm run lint -- --fix",

//在package.json文件中 加入如下属性
"pre-commit": [
    "lint:fix"
  ]
```

