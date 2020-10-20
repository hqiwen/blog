---
title: 从数据看vue组件
date: 2018-06-09 08:34:39
tags: vue
categories: 编程
---

三大前端框架 Vue 、react 和 angular，都提到了一个概念——组件，那么什么是组件——组件（Component）是对数据和方法的简单封装。考虑一个页面，有 UI 的渲染，有状态的改变，有事件的监听。组件正是由这三个方面组成的。现在来看一个简单的组件：

```html
    //绑定class，进行UI渲染
    <div id="app" v-bind:class="container">
        //传递message值
        <p>{{ message }}<p>
        //监听click事件
        <button v-on:click="reverseMessage">逆转消息</button>
    </div>
```

```css
    .container{
        margin:0;
        padding:0;
    }
```
<!--more-->
```js
    var app = new Vue({
        el: '#app',
        //页面状态的初始化
        data: {
            message: 'Hello Vue!'
        }
        //事件的处理
        methods: {
            reverseMessage: function () {
                this.message = this.message.split('').reverse().join('')
            }
        }
    })
```

组件关心着三个方面，就是 **UI**，**state**，**event**。vue 用 js 方式注入了样式，状态和事件，当我们编写vue组件时，要考虑这三个方面。样式由css定义，事件是浏览器标准事件，我们主要说说状态。

## 组件的状态

每个组件都可以有自己的状态，从根组件到叶子组件，形成了一个组件树。父组件通过 Prop 向子组件传递数据，同时子组件本身还维持自身的状态。

### 受控组件与非受控组件

如果一个组件的状态完全由父组件prop传入，那么这个组件就叫做受控组件，又称纯组件。我们把上面的例子改成受控组件，省略css。

```html
<div id="app" v-bind:class="container">
    <child :message = "message" @reverse_message="reverse_message">
</div>
```

```js
var app = new Vue({
    el: '#app',
    data: {
        message: 'Hello Vue!'
    }
    components: {
        'child': child,
    }
    methods: {
        reverse_message: function () {
            this.message = this.message.split('').reverse().join('')
        }
    }
})

Vue.component('child', {
    props: ['message'],
    template: `
            <div>
                <p>{{ message }}<p>
                <button @click="$emit('reverse_message')">逆转消息</button>
            </div>
            `,
})
```

在例子中child是受控组件，父组件不是，可以发现 message 状态的维护就交给了父组件，这样称作 **状态提升**，一直把状态提升到首页，就是全局状态处理的问题，这是 Vuex 引入的原因。受控组件没有副作用，不会改变应用的状态，便于理解和维护，这也是受欢迎的原因。

### 组件的组合与slot

slot 是 父组件提供内容，子组件接受内容。下面给出一个例子：

```html
<navigation-link url="/profile">
  Your Profile
</navigation-link>

<!--<navigation-link>-->
<a
    v-bind:href="url"
    class="nav-link"
>
<slot></slot>
</a>
```

## 结束语

认清页面中数据的流动方向，从更高的抽象上理解组件的状态，知道状态提升，理解全局状态——实现状态共享和同步。