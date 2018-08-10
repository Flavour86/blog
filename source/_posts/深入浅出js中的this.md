---
layout: title
title: 深入浅出js中的this
date: 2018-08-10 14:44:04
tags:
- js
- this
categories:
- 随谈
---

# 深入浅出js中的this

### this应当如何确定！

说到这个，就不得不说JS中this的绑定额；JS中有五种绑定模式及一种特殊绑定模式【ES6的箭头函数】

#### 1.默认绑定 

默认绑定的this值取决于严格模式和非严格模式；
```
var a = 10086;

//在非严格模式中，单纯的函数调用的上下文年默认是指向window;所以this之下window
function test1(){
  console.log(this.a);
}
test1(); //结果集:10086


//而在严格模式中，this不指向window，只会绑定到undefined,
function test2(){
   "use strict"
    console.log(this.a); 
}
test2();//抛出异常,值为undefined 
```

#### 2.new绑定

```
//这个this绑定比较好理解，new 谁对象this指向new的对象
    function test3(a){
       this.a = a;
       console.log(this.a); //10086
       console.log(a);//10086
    }
    var testv3 = new test3(10086); //test3 {a: 10086}

    /*
    * new一个对象的内部过程一般有三步
    * 1、创建一个空对象，并且 this 变量引用该对象，继承了该函数的原型prototype。
    * 2、属性和方法被加入到 this 引用的对象中。
    * 3、新创建的对象由 this 所引用，并且最后隐式的返回 this 。
    */
```

#### 3.显式绑定(硬绑定) – call ， apply ， bind 
 
call和apply的区域在于传参的方式不同，但是作用基本一致
bind是es5的绑定方法，绑定后this无法改变，上面亦是如此

```
//因为call传入是obj，所以上下文this绑定于obj
    var obj = {a:2}
    function test4(){
        console.log(this.a)
    }
    var testv4 = test4.call(obj); //2


    /*bind的使用方法和call很类似，都是将obj作为上下文的this
    * 但是bind还是和call有区别之处
    * bind是返回改变上下文this后的函数(新函数)
    * 使用call是 改变上下文this并执行函数
    */
    function test5(){
     console.log(this.a)
    }
    var obj2 = {a:10086}
    var testv7 = test7.bind(obj2)
    testv7() //10086
```

#### 4.隐式绑定 

隐式绑定是要看是否有上下文对象，调用的时候是否给某个对象拥有或者包含；则该this绑定到该对象中。

```

    /*
    * 注意点：
    * 这是对象的调用，所以this指向该对象
    * 若是var testv6 = obj3.test8 ,就是函数别名赋值，则默认是使用默认绑定而非指向那个对象；
    */
    function test6(){
       console.log(this.a)
    }

    var obj3 = {
      a:10086,
      test6:test6
    }

    obj3.test6() //打印值是10086


    //对于多个链式调用的，this的指向取决最后一个对象。
    var obj1 = {a:789 , obj2:obj2}
     var obj2 = { a:456, test8:test8}
     obj1.obj2.test6(); //456
```

#### 5.软绑定

（函数模拟的）–软绑定 是相对原生 bind 而言一个更灵活的绑定 this 的功能。bind 有一个弊端：被绑定后的新函数无法再更改 this

#### 6.箭头函数

这里说下this和箭头函数的一些要点：
1. 箭头函数的this绑定是根据外层作用域来决定
2. 箭头函数绑定this无法修改,call,apply,bind等都无法修改他
3. 特别适合用于回调函数等这些，避免了var self = this这种写法

## 总结

若是把undefined,null作为this绑定对象参数传入call,apply,bind，会采用默认绑定规则，而参数依旧传入；比如你有一个数组想要依次分割传入一个函数中，apply就可以做到；在未来ES7 中，有一个关于 bind 语法 的提议，提议将:: 作为一个新的绑定操作符，该操作符会将左值和右值(一个函数)进行绑定；若是能开展，this在一些情况下都可以不用写了