---
layout: title
title: 浅谈js对象的属性设置
date: 2018-08-10 14:44:04
tags:
- js
categories:
- 随谈
---

# 浅谈js对象的属性设置

### 何为面向对象

面向对象的定义是比较好比喻的,比如一个人,有什么特征,这是特征就是对象属性,而不同的人又相同的地方和自身亮点; 
这样就是从单一对象过渡到继承,拓展;[注:ES6之前没有类的说法,原生的继承也是不存在的;模拟居多]

ECMA的定义就是: “无序属性的几何,其属性可以包含基本值,对象或者函数.”

在JS中,声明对象的方法有这么两种:声明和字面量;
```js
var x = {}; //字面量
vay y = new Object(); //声明式

x.name = "crper"  //设置属性  Object {name: "crper"}
```

### 属性类型

数据属性有四个行为特性: 

1. [[Configurable]] : 若是设置为true,可以通过delete删除属性;若是false,则不可!!! 默认为true; 
2. [[Enumerable]] : 是否允许for-in这类循环遍历属性值,默认为true; 
3. [[Writable]] : 是否允许修改对象的属性值,默认为true; 
4. [[Value]] : 对象的属性值,默认为undefined;

若是要修改其对象的行为特性,需要用Object.defineProperty来实现，格式:Object.defineProperty(obj,attribute,{行为特性或带值}) 代码如下;
```js
    var x = {};
    x.name = "crper";

    Object.defineProperty(x,"age",{
        writable:false,
        configurable:false,
        value:23
    }) 
    /*非严格模式下,操作age这个值会忽略;严格模式会直接抛出异常;
   因为writable 阻止了;

    而当你设置了configurable这个值为false的时候,非严格模式下删除对象属性值,会忽略;
   严格模式也会直接抛出异常;

   若是要修正这些问题,只能再次调用Object.defineProperty来修改特性,允许多次调用;

   默认情况下,你调用了Object.defineProperty,但不设置特性的情况下,
   会把configurable[操作],enumerable[枚举属性],writable[读写]都设置为false;
  */
```
访问器属性有四个特性: 
1. [[Configurable]] : 与数据属性一致; 
2. [[Enumberable]] : 与数据属性一直; 
3. [[Get]] : 在读取属性的时候调用函数,默认值为undefined; 
4. [[Set]] : 在写入属性值的时候调用函数,默认值为undefined;

```js
/*getter 和setter  就我个人所知的其中一个作用,就是隐藏设置值;
      ES5严格模式中,不可以单独设置getter,会报错!!!
      直接搬<<JS高级编程第三版的例子>> 
  */
   var book = { 
        _year:2004,
        edition:1
    };

   Object.defineProperty(book,"year",{
      get:function(){
                return this._year;
     },
     set:function(newValue){
         if(newValue > 2004){
            this._year = newValue;
            this.edition += newValue - 2004;
         }
      }
  });

    book.year = 2005;
    console.log(book.edition);    // 2

   /*这个例子中,getter不做任何设置,直接返回值; setter里面进行了一个判断,
     若是年份大于当前的设置的年份,年份为设置的值且版本加上设置年份减去已有年份的值*/
```

定义多个属性

```
/*
     用法相当简单,基本和defineProperty一致,唯一的区别就是支持多个属性的同时设置 
   */
   var my = {};
   Object.defineProperties(my,{
       name:{
            value:"crper"
       },
       age:{
          value:23
      },
      read:{
           get:function(){
                 return this.age;
           },
           set:function(value){
                if(value > 24){
                      return this.age  = (value - 24);
              }
          }
      }
   });
```

读取属性特性

读取属性特性可以使用Object,getOwnPropertyDescriptor(obj,attribute)方法; 
返回值是数据属性和访问器属性 
接着上面一个例子:

```
Object.getOwnPropertyDescriptor(my,"age") ;   //Object {value: 23, writable: false, enumerable: false, configurable: false}

Object.getOwnPropertyDescriptor(my,"name");  //Object {value: "crper", writable: false, enumerable: false, configurable: false}

Object.getOwnPropertyDescriptor(my,"read"); //Object {enumerable: false, configurable: false}
```