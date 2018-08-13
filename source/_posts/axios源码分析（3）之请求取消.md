---
layout: title
title: axios源码分析（3）之请求取消
date: 2018-08-11
tags:
- js
- axios
categories:
- 源码分析
---

# axios源码分析（3）之请求取消

之前分析了2篇文章：

[axios分析之请求](../axios源码分析（1）之请求)
[axios分析之拦截器](../axios源码分析（2）之拦截器)

接下来，我们今天分析axios的取消这块的实现，老规矩，先来段使用api助兴一下
```
var CancelToken = axios.CancelToken;
var source = CancelToken.source();
axios.get('/get?name=xmz', {
    cancelToken : source.token
}).then((response)=>{
    console.log('response', response)
}).catch((error)=>{
    if(axios.isCancel(error)){
        console.log('取消请求传递的消息', error.message)
    }else{
        console.log('error', error)
    }
})
// 取消请求
source.cancel('取消请求传递这条消息');
```
根据这个例子，我又找到了`axios/lib/axios.js`
```
axios.CancelToken = require('./cancel/CancelToken');
```
找到`axios/lib/cancel/CancelToken.js`
```
CancelToken.source = function source() {
  var cancel;
  var token = new CancelToken(function executor(c) {
    cancel = c;
  });
  return {
    token: token,
    cancel: cancel
  };
};
```
这里看出source执行完之后，返回一个还有token跟cancel的对象，token，也是一个对象，这个cancel到底是什么还不得而知，继续看下CancelToken这个类
```
function CancelToken(executor) {
  if (typeof executor !== 'function') {
    throw new TypeError('executor must be a function.');
  }

  var resolvePromise;
  this.promise = new Promise(function promiseExecutor(resolve) {
    resolvePromise = resolve;
  });

  var token = this;
  executor(function cancel(message) {
    if (token.reason) {
      // Cancellation has already been requested
      return;
    }

    token.reason = new Cancel(message);
    resolvePromise(token.reason);
  });
}
```
从这里我们可以看出CancelToken中含有一个promise，并且上面source中的c就是这里的cancel函数，所以我们使用的`source.cancel('取消请求传递这条消息');`就是这里的cancel函数，所以这里`source.cancel('取消请求传递这条消息');`最重要的就是执行`resolvePromise()`,这个`resolvePromise`是什么呢？其实就是this.promise中执行then中的回调函数，那这个then中的回调函数在哪里，在`axios/lib/adapters/xhr.js`
```
if(config.cancelToken){
    // 具体是如何取消的，是在这个判断内定义的；
    config.cancelToken.promise.then(function(cancel){
        request.abort();
        reject(cancel);
        request = null;
    })
}
// 发送请求
request.send(requestData);

```
看见了这段，就连接起来了，`resolvePromise()`就是要执行这里的then的回调函数，就达到了取消request的效果

好了，总结一下：执行cancel是让token的promise变成成功，在真正发送请求之前，验证token.promise的状态是否已经变了，如果变了，就取消请求，就是这样一个简单的思想来进行取消请求的