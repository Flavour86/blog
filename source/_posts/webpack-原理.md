---
layout: title
title: webpack 原理
date: 2018-07-02 14:44:04
tags:
- webpack
- Commonjs
categories:
- 随谈
---

# webpack原理
1. 从入口文件，利用类似于babylon这种js模块解析器，对所有的js依赖进行递归解析，将它们解析成AST（抽象语法树）
2. 得到语法树之后用webpack配置的loader对所有的资源模块进行解析，转化，比如es6用babel转化成es5，
3. 这个过程中其实已经对所有的js模块进行模块的id的分配，以及所有依赖关系的确定，就是每次js模块所依赖的模块都已经分配好了，并且已经将所有的需要到的js文件包依赖，统一处理成模块函数，并且形成chunk，
4. 根据webpack的配置，把所有公共提取的模块，异步的模块进行打包，打到一个文件上，
5. 然后生成manifest的配置，manifest就是对所有模块函数根据模块id进行管理，引用，将每个模块函数变成我们需要的模块对象

### loader与plugin的区别
loader：用于转换webpack模块的各种资源，因为webpack只能解析基于commonjs规范的js，对于一些图片，.jsx, .vue, css等等都不支持，所以用这些loader转化这些资源，使的webpack可以解析这些资源

plugin：是webpack的一个扩展器，它可以拓展webpack的很多功能，比如公共模块提起，比如样式提取等等，shide