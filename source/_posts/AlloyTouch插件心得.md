---
layout: title
title: AlloyTouch插件心得
date: 2018-08-10 14:44:04
tags:
- js
categories:
- 插件使用
---

# AlloyTouch插件心得

### Install
```
npm install alloytouch
```
### API

```
var alloyTouch = new AlloyTouch({
            touch:"#wrapper",//反馈触摸的dom
            vertical: true,//不必需，默认是true代表监听竖直方向touch
            target: target, //运动的对象
            property: "translateY",  //被运动的属性
            min: 100, //不必需,运动属性的最小值
            max: 2000, //不必需,滚动属性的最大值
            sensitivity: 1,//不必需,触摸区域的灵敏度，默认值为1，可以为负数
            factor: 1,//不必需,表示触摸位移与被运动属性映射关系，默认值是1
            step: 45,//用于校正到step的整数倍
            bindSelf: false,
            initialValue: 0,
            change:function(value){  },
            touchStart:function(evt, value){  },
            touchMove:function(evt, value){  },
            touchEnd:function(evt,value){  },
            tap:function(evt, value){  },
            pressMove:function(evt, value){  },
            animationEnd:function(value){  } //运动结束
 })
```
通过对象的实例可以自行运动DOM:
```
alloyTouch.to(value, time, ease)
```

## wiki

[wiki地址](https://github.com/AlloyTeam/AlloyTouch/wiki) 可以参照这里

## 下拉刷新心得

主要是用[transform.js](http://alloyteam.github.io/AlloyTouch/transformjs/)，设置dom元素的transform属性，这样可以使元素动画的时候性能比较好，然后将页面的content放在一个div中
```
<div class="app">
    <div class="scroll">
     // 这里是页面的content
        <div class="head"></div>
        <div class="body"></div>
    </div>
</div>
```
这样下拉的时候可以出现整个页面下拉的效果,然后一般是下拉的时候头部会有放大效果，所以我们 `cloneNode（head）` 这个元素到app的外面，下拉的时候对外面的clone的元素进行放大，对scroll里面的head进行隐藏，就会出现下拉放大的效果，大概原理即是如此