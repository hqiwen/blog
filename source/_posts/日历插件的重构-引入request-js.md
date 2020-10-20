---
title: 日历插件的重构--引入request.js
date: 2018-03-30 16:59:56
tags: request.js
categories: 编程
---
在项目过程和状态增加的时候，项目的复杂度增加了，如何对项目进行有效的管理？使用模块化，提炼业务代码的逻辑，有利于适应需求的快速变化。作者使用 request.js 对项目进行了简单重构，阅读源码前最好有相应的 request.js 的基础知识，这将帮助你更好的理解源码。

## request.js 介绍

对于不懂 request.js 的同学，可以参阅[阮一峰老师的教程](http://www.ruanyifeng.com/blog/2012/11/require_js.html),这里介绍了 request.js 的详细用法，本文略带提一下。

> 定义并引入模块

```js
//math.js
define(["moduleA", "moduleB"], function(moduleA, moduleB) {
    var add = function(x, y) {
        return x + y;
    };
    // moduleA some code
    // moduleB some code
    return {
        add: add
    };
});
```

> 加载模块

```js
require(["math"], function(math) {
    alert(math.add(1, 1));
});
```
<!--- more --->
## 业务的分析和分离

### data.js(数据分析)

项目是日历的编写，那么数据来源是浏览器提供的 date 对象，日历展示的是一个月的数据，通过年月构造函数 getMonthData()来获取一个月的数据。

```js
define(function() {
    function getMonthData(year, month) {
        //new Date
        //处理格式
        //返回数据
        return { year: year,
                month: month,
                days: [{
                    month: month,
                    date: date
                }]};

    return{
        getMonthData: getMonthData
    }
});
```

### render.js（展示分析）

拿到数据并将数据展示出来

```js
define(['data'], function(data) {
    //获取数据
    monthData = data.getMonthData();
    //建立DOM结构
    function buildUI(monthdata){
        //some code
        return html
    }
    //上下月判断处理
    function direction(dir){}
    //渲染DOM结构
    function render(dir){
        direction(dir)
        //some code
        $wrapper.innerHTML = html;
        document.body.appendChild($wrapper);
    }

    retrun {
        render: render
    }
});
```

### event.js（功能分析）

- 当点击输入输入框的时候可以开放或者关闭日历
- 当点击上一月的按钮时，切换到上一个月的数据
- 当点击下一月的按钮时，切换到下一个月的数据
- 当点击日历上的数据时，在输入框中显示其日期

```js
define(['render'],function(render){
    function showInput(){}

    function changeMonthData(){
        render("prev/next")
    }

    function getDate(){}

    return {
        showInput: showInput,
        changeMonthData: changeMonthData,
        getDate:getDate,
    }
})
```

### mian.js

项目的入口，实行项目的初始化，提供可供用户操作的接口。

```js
define(["render","event"],function(render,event){
    var init = function(){
        //初始渲染
        render.render();
        //事件监听与绑定
        $input.addEventListener("click", event.showInput, false);
        $wrapper.addEventListener("click", event.changeMonthData, false);
        $wrapper.addEventListener("click", event.getDate, false)
    }
    return {
        init: init,
    }
})
```

## 模块依赖

{% asset_img 模块.png 400 266 %}

## 总结

通过对项目进行重构，简化了业务的逻辑，对模块化的思想有了更加深刻的认识，现代大型项目的构建离不开模块化。比较好的模块化框架有[common.js](http://javascript.ruanyifeng.com/nodejs/module.html),[request.js](http://requirejs.org/),[sea.js](https://seajs.github.io/seajs/docs/)。学会模块化的思维，将帮助我们成长为更好的工程师。

项目地址[https://github.com/hqiwen/datepicker](https://github.com/hqiwen/datepicker)