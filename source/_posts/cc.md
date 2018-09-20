---
title: 闭包
date: 2018-09-06 17:25:54
tags:
---

## 闭包

闭包，函数内部的函数就可以叫做是一个闭包

```
function first(){
	var a='fsaf';
    function sec($a){//此处的sec就是一个闭包
      	document.write(a+$a);
    }
    return sec;  
}
//闭包之中，可以访问方法first中的局部变量
```

这种用闭包的方法比较少，因为其没什么大的作用。通常的用法是
```
function foo($index,callback){
	var idx=$index*10;
    document.write(idx+'<br>');
    callback(idx);//此处，将方法的变量注入回调函数
}

//在回调中可以忽略掉注入的变量
foo(10,function(){
	document.write('fff');
})
//100
//fff


//也可以将注入的变量显示化出来
foo(10,function($m){
	document.write($m);
})
//100
//100
```

### axios 例子
```
axios.get(PATH, {

}).then(function ($res) {
	
});
//这里的get()方法发起一个请求，返回一个response ,
//response先是进过then函数处理，再通过回调让用户自定义

```








