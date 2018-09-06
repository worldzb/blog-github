---
title: es6 变量作用域
date: 2018-09-06 16:33:40
tags:
---

### es6 变量作用域

　　ES6的模块化的基本规则或特点， 欢迎补充：

1：每一个模块只加载一次， 每一个JS只执行一次， 如果下次再去加载同目录下同文件，直接从内存中读取。 一个模块就是一个单例，或者说就是一个对象；

2：**每一个模块内声明的变量都是局部变量， 不会污染全局作用域**；

3：模块内部的变量或者函数可以通过export导出；

4：一个模块可以导入别的模块


### 作用域解析

在main.js 当中 

```
//1

import Vue from 'vue';//只是将Vue引入当前文件，在其它文件使用 Vue 时，会显示Vue 为 undifinded

window.Vue=Vue;//将Vue 注册为全局对象，因为windo是全局对象，将Vue注册到window 下，则可通过全局调用Vue

```

```
//2

import config from './config/app-config.js';//引入配置文件
Vue.prototype.config=config;//将配置添加到Vue类当中,再实例化则将config注册到 vue实例当中

//因为Vue是全局对象，而注册到Vue下则也可通过全局来调用。

```
```
//3

//a.js
export a=10;

//b.js
import {a} from './a.js';
```
> 每个文件都有单独的作用域，A文件引入B文件，则只能使用从B文件当中导出的几个变量，或者是函数。其余B当中的定义的变量，如果没有导出是不能在A当中不能使用的。当然，B文件更是没法使用A当中的变量。

### Vue 和 它的组件

Vue是一个构造函数
```
console.log(Vue);

//结构如下
ƒ Vue (options) {
  if ("development" !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword');
  }
  this._init(options);
}

console.log(Vue.prototype);
//结构如下
{_init: ƒ, $set: ƒ, $delete: ƒ, $watch: ƒ, $on: ƒ, …}
```

而Vue组件，默认是一个 VueComponent 对象.而它的原型是一个Vue对象，即VueComponent是在Vue对象的基础上的扩展，而这个Vue对象是main.js实例化对象。实际上 就是说，Vue是VueComponent的父对象。

