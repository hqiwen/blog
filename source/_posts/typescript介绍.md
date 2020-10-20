---
title: typescript介绍
date: 2020-05-08 13:40:34
tags: typescript
categories: 编程
---

TypeScript 简单来说是一门编程语言，与你熟知的 JavaScript、Java 和 C++ 之类的编程语言一样。既然有这么多的语言可供选择，为什么又要新创建一门语言？先看看官网对 TypeScript 的描述，TypeScript 是 JavaScript 类型的超集，它可以编译成纯JavaScript。那么为什么需要使用 TypeScript 而不是 JavaScript 呢？下面我将说服你为什么要使用 TypeScript。

## 历史

### javaScript是一门好语言吗

对于这个问题，不同的人有不同的想法。JavaScript 作为最多人使用的编程语言，也是在网页端取得绝对地位的语言，常常因为其动态类型、隐式转换而受到诟病。下面简要说明，一个完整的 JavaScript 包括三个不同的部分组成，核心（ESMAScript语言,简称ES），文档对象模型（DOM）和浏览器对象模型（BOM）,ESMAScript中有5种简单数据类型：undefined，null，boolean,number和string。

```js
var result1 = ("55" == 55);  //true,因为字符串“55”被转换数值55后相等
var result2 = ("55" === 55);   //false, 因为不同的类型不相等
```

这看起来令人困惑，明明差不多的操作，得到的结果却截然相反，如果由于疏忽，少写了一个等号，会得到令人检测不到的 Bug。当然，你可能精通JavaScript，不会犯这种低级错误，但是如果和你合作的某个人，你能真正地保证他不会犯这种错误吗？

### CoffeeScript的出现

为了减少犯这种错误的可能性，出现了CoffeeScript。CoffeeScript是一套javaScript的转译语言，于2009 年 12 月 24 日, 提交版本 0.0.1 ，创建者 Jeremy Ashkenas 戏称它是-JavaScript 的不那么铺张的小兄弟。因为 CoffeeScript 会将类似 Ruby 语法的代码编译成 JavaScript，而且大部分结构都相似，但不同的是 CoffeeScript 拥有更严格的语法。
在coffeeScript中，由于操作符**==**常常带来不准确的约束, 不容易达到效果, 而且跟其他语言当中意思不一致, CoffeeScript 会把 == 编译为 ===, 把 != 变异为 !==. 此外, is 编译为 ===, 而 isnt 编译为 !==，这样就避免了问题。
随着项目的日渐拓展，代码量越来越多，有了这样的代码：
<!-- more -->
```js
function Cat(name){
  this.name = name;
}
Cat.prototype.run = function() {
  console.log(this.name + "can run");
}
function Fish(name){
  this.name = name;
}
Fish.prototype.swim = function() {
  console.log(this.name + "can swim");
}
function CatHome() {
  this.home = [];
}
CatHome.prototype.add = function(cat) {
  this.home.push(cat);
}
CatHome.prototype.count = function() {
  this.home.forEach(function(cat){
    cat.run();
  })
  console.log("there are " + this.home.length + " cats")
}
---
//另外一处
var sam = new Cat("sam");
var lily = new Fish("lily");
var catHome = new CatHome();
catHome.add(sam);
catHome.add(lily);  //瞧，添加了一条鱼，却没有警告
catHome.count();  //error: cat.run is not a function，这个错误信息往往令人摸不着头脑
```

之所以报错是因为 Fish 没有实现 run 方法，只有 swim 方法。当遇到这个问题的时候，开发者需要一步一步的检查代码，去找到每一个类的实现代码。

### TypeScript的曙光

2012年十月份，微软发布了首个公开版本的TypeScript，2013年6月19日，在经历了一个预览版之后微软正式发布了正式版TypeScript 0.9，向未来的TypeScript 1.0版迈进了很大一步。与JavaScript相比，TypeScript进步的地方包括：加入注释，让编译器理解所支持的对象和函数，编译器会移除注释，不会增加开销；增加一个完整的类结构，使之更像是传统的面型对象语言。
下面我们来品味一下，看看typescript是如何解决上面这个问题的：

```ts
class Cat{
    name: string;
    constructor(name) {
        this.name = name;
    }
    run() {
        console.log(this.name + "can run");
    }
}
class Fish{
    name: string;
    constructor(name) {
        this.name = name;
    }
    swim() {
        console.log(this.name + "can swim")
    }
}
class CatHome{
    home: Cat[];
    constructor() {
        this.home = [];
    }
    add(cat:Cat) {
        this.home.push(cat);
    }
    count() {
        this.home.forEach(function (cat) {
            cat.run();
        })
    }
}
var sam = new Cat("sam");
var lily = new Fish("lily");
var catHome = new CatHome();
catHome.add(sam);
catHome.add(lily);
//Argument of type 'Fish' is not assignable to parameter of type 'Cat'.函数参数是Cat类型，传入的Fish类型不可以赋值给Cat类型
//Property 'run' is missing in type 'Fish' but required in type 'Cat'.Cat类型必要要有run方法，而Fish里面没有run方法
catHome.count();
```

可以看到编译器对catHome的内部数组进行了检测，当我们调用add方法时，会检测添加实例是不是一个Cat类。从错误信息中还能得到Cat类必须要有run方法，但是Fish类没有。

## 自定义的一个类型系统

### 语义

程序所表达的含义简称语义。语义是程序执行时所表达出来的思想，原程序通过解释器或编译器的执行，让静止不动的代码得到了含义，可以被另一种语言去执行和翻译，最终翻译成计算机可执行的指令。

程序有动态语义和静态语义之分。静态语义是指在编译阶段能够检查的语义，比如标识符未定义，类型不匹配等。动态语义是指在目标程序运行阶段能够检查的语义，比如除数为0，无效指针，数组下标越界等。

动态语义需要程序运行才能分析，但有时运行原程序不可逆或消耗时间太长，但是发现不实际执行的程序的问题很重要。分析程序的静态语义，可以让不实际程序执行计算而能发现它的近似信息。类型系统是静态分析的经典例子。

### 自定义类型

下面添加三种类型，分别是正、负和零。

```ts
class Sign{
    constructor(name) {
        this.name = name;
    }
}

Sign.POSITIVE = new Sign("positive")
Sign.NEGATIVE = new Sign("negative")
Sign.ZERO = new Sign("zero")
```

### 自定义规则

添加类型的规则和操作

```ts
class Sign{
    //...接上文的代码
    //为类型添加规则
    multiply(other_sign) {
        if (this === Sign.ZERO || other_sign === Sign.ZERO) {
            //任何一个数乘以0等于0
            return Sign.ZERO
        } else if(this === other_sign) {
            // 符号相同得正
            return Sign.POSITIVE;
        } else {
            // 符号相反得负
            return Sign.NEGATIVE;
        }
    }
}
```

### 将程序与类型相连

把源程序中的值与类型系统中的值相关联。小于0返回一个负类型，等于0返回一个0类型，大于0返回一个正类型。

```ts
class Numeric{
    constructor(value) {
        this.value = value;
    }
    // 为值添加类型
    sign() {
        if (this.value < 0) {
            return Sign.NEGATIVE;
        } else if (this.value = 0) {
            return Sign.ZERO
        } else {
            return Sign.POSITIVE;
        }
    }
}

//为了使用方便，封装一个函数
function createNumeric(value) {
    return new Numeric(value).sign();
}
```

### 解释源程序

下面调用函数解释源程序的含义。

```ts
/**
 * 不用运算结果，我们便可以知道结果是一个负值
 */
createNumeric(6).multiply(createNumeric(-9)).toString();//"negative"
```

这里我们得到了计算的结果的一点近似信息，通过类型系统的解释，结果是一个负数，显然程序结果是正还是负没有太大的意义，但是这是一个很好的起点，可以预见更加强壮的类型系统会为我们带来更多的信息，现代编译系统在构建语法树后会对程序进行语义分析，其中静态分析是很重要的一环。

编辑器插件对程序进行分析，在内部进行编译执行，让我们在程序编写时就能获得很多提示，提高开发效率和减少犯错误的几率。

## 优势

为什么要选择typescript？

### 类型检测

在Typescript里面是运行为变量指定类型的，比如当你为这个变量指定数字类型的值的时候，IDE会做出类型检查，然后告诉你这里可能会有错误，这个特性会减少你在开发阶段犯错误的几率。

### 语法提示

在IDE里面去编写TypeScript的代码时，IDE会根据你当前的上下文，把你能用的类、变量、方法和关键字都给你提示出来，你只要直接去选就可以了，这个特性会大大提升你的开发效率

### 项目重构

重构使你可以很方便的去修改你的变量或者方法的名字或者是文件的名字，当你做出这些修改的时候，IDE会帮你自动引用这个变量或者调用这个方法地方的代码自动帮你修改掉，这个特性一个是会提高你的开发效率，另一个是可以很容易的提升你的代码质量

Typescript的好处很明显，在编译时就能检查出很多语法问题而不是在运行时。不过由于是面向对象思路，如果是纯前端的人员（没有后端语言基础），那用起来应该是比较吃力的。好的框架和语言能间接帮助开发者写出规范的代码，Typescript能使团队易于协同开发，提高效率，这是typescript生态取代JavaScript生态的原因。

## 谁在使用它们

既然typeScript有那么好，这样的语言有实践先例吗？

### Game

著名游戏引擎白鹭，遵循HTML5标准的2D、3D引擎，解决了HTML5性能问题及碎片化问题，灵活地满足开发者开发2D或3D游戏的需求，并有着较强的跨平台运行能力。在 Egret 中，需使用 TypeScript 编写程序，最终编译成浏览器可读的 JavaScript。

### Heroku

Heroku是一个支持多种编程语言的云平台即服务。在2010年被Salesforce.com收购。Heroku作为最开始的云平台之一，从2007年6月起开发，当时它仅支持Ruby，但后来增加了对Java、Node.js、Scala、Clojure、Python 以及（未记录在正式文件上）PHP和Perl的支持。基础操作系统是Debian，在最新的堆栈则是基于 Debian 的 Ubuntu。

### 微信小程序

由于目前腾讯没有小程序的TypeScript版本的API，所以 OneCode team 针对目前腾讯放出的所有的小程序 JavaScript API 开发了一个 TypeScript 版本的API类型定义文件 wxAPI.d.ts，只需要在您的程序中引用该文件，如果是使用 Visual Studio 来开发的话，就能有代码提示了。

### Vue 3.0

Vue 3.0 会改进对 TypeScript 的支持，允许在编辑器中进行高级的类型检查和有用的错误和警告。

## Typescript 组成

TypeScript 为 JavaScript 添加了类型注释，通过添加 d.ts 文件，让 TypeScript 衔接 JavaScript 生态，所有版本的JavaScript将会得到 TypeScript良好的支持。在 ESMAScript 版本中，ESMAScript2015 又称 ES6 是一个比较重要的版本，TypeScript 包括了ES6的语法，从某种程度上来说，ts = js + d.ts。

## 如何初始化一个 TypeScript 项目

### 安装 TypeScript

有两种主要的方式来获取 TypeScript 工具：

* 通过npm（Node.js包管理器）
* 安装Visual Studio的TypeScript插件
Visual Studio 2017和Visual Studio 2015 Update 3默认包含了TypeScript。 如果你的Visual Studio还没有安装TypeScript，你可以下载它。

针对使用npm的用户：

> npm install -g typescript

### 编写我们的第一个 ts 文件

新建 add.ts 文件，内容如下：

```ts
//声明一个类型
type operator = string | number;

//对operator类型进行操作
function add<operator>(operator1: operator, operator2: operator) {
  //判断类型为string并相加
    if (typeof operator1 === "string" && typeof operator2 === "string") {
        return operator1 + operator2;
    }
  //判断类型为number并相加
    if (typeof operator1 === "number" && typeof operator2 === "number") {
        return operator1 + operator2;
    }
}

add(1, 1);//2

add("hello", " world");//hello world
```

### 编译和执行我们的 TypeScript 文件

在命令行上，运行 TypeScript 编译器编译文件：

> tsc add.ts

输出结果为一个add.js文件，它包含了和输入文件中相同的JavaScript代码。 一切准备就绪，我们可以运行这个使用TypeScript写的JavaScript应用了！

> node add.js

由于我们什么都没有输出，也没有阻塞进程，程序执行完就会被退出。
