---
layout: title
title: Axios 源码分析（1）之请求
date: 2018-08-11
tags:
- js
- axios
categories:
- 源码分析
---


# Axios 源码分析（1）之请求

首先在开始之前，我当做你对Promise已经很熟悉啦，如果不熟悉的，请自行再去了解一下Promise

首先，axios是什么东西呢？
这里简单的介绍一下，Axios 是一个基于 promise 的 HTTP 库，可以运行在浏览器和 node.js 中

我们先来看看axios的使用api吧
```
import axios from 'axios'

axios.get('/user?ID=12345')
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```
这是axios的最基础的使用方法，这里直接用axios进行一个get请求，然后返回一个Promise

于是乎，我们打开源码找到```axios/lib/axios.js```这个文件，找到
```
module.exports.default = axios
```
接着看见
```
var axios = createInstance(defaults);
```
```
function createInstance(defaultConfig) {
  var context = new Axios(defaultConfig);
  var instance = bind(Axios.prototype.request, context);

  // Copy axios.prototype to instance
  utils.extend(instance, Axios.prototype, context);

  // Copy context to instance
  utils.extend(instance, context);

  return instance;
}
```
从这里我们看见，context是一个对象，是Axios这类的实例，而instance是Axios的原型上的那个request的方法，只不过，通过bind，将它的this指向了context，
虽然大概猜到bind的用途，但是我们还是得看看他的具体实现,
```
function bind(fn, thisArg){
  return function wrap(){
    var args = new Array(arguments.length);
      for(var i = 0; i < args.length; i++){
          args[i] = arguments[i]
      }
      return fn.apply(thisArg, args)
    }
}

```
所以axios其实就是instance,是一个方法，但是axios为什么还有```.get()```方法呢？
我们接着往下看
```
// Copy axios.prototype to instance
utils.extend(instance, Axios.prototype, context);
// Copy context to instance
utils.extend(instance, context);
```
这里写的很清楚，将```axios.prototype, context```这2个对象copy给instance，我们要看看extend的具体实现
```
function extend(a, b, thisArg) {
  forEach(b, function assignValue(val, key) {
    if (thisArg && typeof val === 'function') {
      a[key] = bind(val, thisArg);
    } else {
      a[key] = val;
    }
  });
  return a;
}
```
这里对b对象进行遍历，然后如果属性值是```function```的话，就将这个```function```的this对象指向到```thisArg```,然后赋给a，如果不是```function```直接赋给a，最后返回a，所以`axios.get`就是`Axios.prototype`中继承过来的，让我们来看看`Axios.prototype`,于是乎，我们找到
`axios/lib/core/Axios.js`，打开它,我本来是想找`Axios.prototype.get`,搜了下，没搜到，尴尬了，然后看见下面这段
```
utils.forEach(['delete', 'get', 'head', 'option'], function(method){
    Axios.proptotype[method] = function(url, config){
        // 先不去看util.merge了，我猜也就是把config 和 后面的对象进行下合并
        return this.request(util.merge(config||{}, {
            method: method,
            url: url
        }))
    }
})

```
这里看出`Axios.prototype.get`其实就是去执行了`Axios.prototype.request`,找之
```
Axios.prototype.request = function request(config) {
  // Allow for axios('example/url'[, config]) a la fetch API
  if (typeof config === 'string') {
    config = arguments[1] || {};
    config.url = arguments[0];
  } else {
    config = config || {};
  }

  config = mergeConfig(this.defaults, config);
  config.method = config.method ? config.method.toLowerCase() : 'get';
  // 前面就是对config于默认的config进行合并，并且对method进行toLowerCase处理


  // Hook up interceptors middleware
  var chain = [dispatchRequest, undefined];
  var promise = Promise.resolve(config);
  this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
    chain.unshift(interceptor.fulfilled, interceptor.rejected);
  });

  this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
    chain.push(interceptor.fulfilled, interceptor.rejected);
  });

  while (chain.length) {
    promise = promise.then(chain.shift(), chain.shift());
  }
  // 这一段是拦截器的一些逻辑，我们暂时先忽略

  return promise;
}
```
我们可以看见，`request`函数返回的是promise，所以到这里就和`axios.get().then().catch()`对上了，但是这里如何发请求出去呢？这里混合一些拦截器的逻辑，我们暂先跳过，明眼的一看就知道是这个`dispatchRequest`,我们它的具体逻辑,找到`axios/lib/core/dispatchRequest.js`
```
function dispatchRequest(config){
    // ...
    // 开始就对config的 baseUrl, data, headers进行处理
    var adapter = config.adapter || defaults.adapter;
    // 这个adapter是什么？一路以来并没有关注啊？
    // 执行adapter是一个Promise对象，resolve的函数的参数还是response
    // adpater肯定是去发送请求了啊
    return adapter(config).then(function(response){
        // ...
        return response
    }, function(reason){
        // ...
        return Promise.reject(reason)
    })
}
```
`dispatchRequest`这个代码有点长，我挑重点的贴出来，这里就是去执行`adapter`这个方法，然后成功之后返回response, 失败的话返回`Promise.reject(reason)`, 我再看一下adapter
```
var defaults.adapter = getDefaultAdapter();
function getDefaultAdapter(){
    var adapter;
    if(typeof XMLHttpRequest !== 'undefined'){
        // 浏览器环境
        adapter = require('./adapter/xhr');
    }else if(typeof process !== 'undefined'){
        // node环境
        adapter = require('./adapter/http');
    }
    return adapter;
}
```
可以看出adapter就是根据浏览器环境跟node环境进行区分，浏览器环境就require浏览器的xhr对象，node环境就是使用node的http或者https的模块
```
// xhrAdapter
 var request = new XMLHttpRequest();


// httpAdapter
var http = require('http');
var https = require('https');
transport = isHttpsProxy ? https : http;
```
好了在一堆的忽略下，把请求的这块流程分析清晰，我们请求这部分的代码分析到这里，哈哈！



