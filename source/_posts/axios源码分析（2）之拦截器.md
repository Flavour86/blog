---
layout: title
title: Axios 源码分析（2）之拦截器
date: 2018-08-11
tags:
- js
- axios
categories:
- 源码分析
---

# Axios 源码分析（2）之拦截器

上一篇我们主要分析了请求这块的流程，想了解的[点击此处](../axios源码分析（1）之请求)了解

这一篇讲主要来分析下axios是如何实现拦截器的，这里只分析请求拦截器，
老规矩先来看下api如何使用的
```
axios.interceptors.request.use(function(config){
    // ...
    return config;
}, function(err){
    // ...
    return Promise.reject(err)
})
```
根据第一篇我们分析的，
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
`axios.interceptors`应该是在`Axios.prototype`或者`context`中
所以在`axios/lib/core/Axios.js`中，我们看见
```
function Axios(instanceConfig) {
  this.defaults = instanceConfig;
  this.interceptors = {
    request: new InterceptorManager(),
    response: new InterceptorManager()
  };
}
```
可以看出request跟response都是new了一个InterceptorManager，都是InterceptorManager的实例，接着往下，我们看看`axios/lib/core/InterceptorManager.js`
```
function InterceptorManager() {
  this.handlers = [];
}

InterceptorManager.prototype.use = function use(fulfilled, rejected) {
  this.handlers.push({
    fulfilled: fulfilled,
    rejected: rejected
  });
  return this.handlers.length - 1;
};
InterceptorManager.prototype.eject = function eject(id) {
  if (this.handlers[id]) {
    this.handlers[id] = null;
  }
};
InterceptorManager.prototype.forEach = function forEach(fn) {
  utils.forEach(this.handlers, function forEachHandler(h) {
    if (h !== null) {
      fn(h);
    }
  });
};
```
相当简短的代码，逻辑很清楚了，use，只不过将`fulfilled, rejected`2个方法保存在this.handlers中，便于后面调用，那什么时候调用呢？
答案肯定是请求前后啦，因为拦截器request跟response本来就是为了拦截请求之前，跟请求之后,所以我们看看第一篇中，请求的时候干了什么

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
  // 拦截器的逻辑这里开始
  this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
    chain.unshift(interceptor.fulfilled, interceptor.rejected);
  });

  this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
    chain.push(interceptor.fulfilled, interceptor.rejected);
  });

  while (chain.length) {
    promise = promise.then(chain.shift(), chain.shift());
  }

  return promise;
}
```
我们仔细看看这段拦截器，这里的`forEach`在`InterceptorManager`重新被定义了，通过这里的2个forEach，我们最后拿到的chain是
```
[request.fulfilled, request.rejected,
dispatchRequest, undefined,
response.fulfilled, response.rejected]
```
这样的一个形式，然后循环赋给`promise.then()`，所以最后拿到的promise就是这样
```
Promise.resolve(config)
.then(request.fulfilled, request.rejected)
.then(dispatchRequest, undefined)
.then(response.fulfilled, response.rejected)
```
这样就很清晰了，再发送请求`dispatchRequest`之前，先执行了request.fulfilled或者 request.rejected，请求之后立刻又执行了response.fulfilled, response.rejected

好了，这就是拦截器的分析，
大概总结一下，就是通过InterceptorManager这个类的use方法去搜集拦截的回调，在request的时候去执行他们