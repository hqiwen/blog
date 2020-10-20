---
title: typescript变量声明
date: 2020-05-08 13:46:17
tags: typescript
categories: 编程
---

前一章我们写了一个相加程序，可以完成数字和数字的相加，也可以完成字符串和字符串的相加，根据第一个参数，提示第二个参数的类型。如果把一个数字和字符串相加，编译器就会报错，是不是很智能呢？细心的会发现声明了一个类型：`type operator = string | number;`

> type是声明一个类型的关键字，不会新建一个类型，它创建了一个新名字来引用那个类型。

这是我们完成类型检测的关键。下面我们来系统地学习typescript类型的奥秘，其中插入ES6语法的拓展。

## 声明变量let、var、const

### var导致的声明提升

在ES6之前的变量声明,var的使用,看看下面的道理：

```js
console.log(a);// undefined
var a = 1;
console.log(a);// a = 1;
console.log(b);//Uncaught ReferenceError: b is not defined,引用错误b没有被声明
```

a和b打印出来的值不一样，为何？在javascript运行的过程中先会执行一遍预解析，使得变量a会被录入，但是没有执行赋值，故第一次打印a的值为undefined,赋值后打印a的值为1，变量b无声明，得到b is not defined。

### 经典闭包问题的解决

在JavaScript的面试中经常会碰到闭包问题，你能回答出下面的结果吗？

```js
for (var i = 0; i < 5; i++) {
    setTimeout(function () {
        console.log(i);
    }, 0);
}
```

答案是5个5，思考一下。
<!-- more -->
如果还弄不清为什么，先来看看定义：
> 闭包：能够读取其他函数内部变量的函数，在JavaScript中，只有函数内部的子函数才可以读取局部变量

在JavaScript中，有全局作用域和局部作用域之分。变量没有在函数内声明或者没有带var就是全局变量，拥有全局作用域，在代码任何地方都可以访问，如window是BOM提供的全局对象，在函数内部并且以var修饰的变量就是局部变量，只能在函数体内使用，函数的参数也是局部变量。

在**全局作用域**中存在着变量i,for循环5次添加了5个函数，在函数内部引用了变量i，所有函数均指向**同一变量**i，当循环结束后，变量i的值为5，此时执行console.log函数，打印出5个5。

闭包本质上将函数内部和函数外部连接起来的桥梁，例子中外部作用域的变量i被内部函数保存了起来。闭包常用来保护函数内的变量安全和在内存中维持一个变量，例子中的变量i被引用，得不到javascript垃圾收集机制的回收，容易造成内存泄露。

### 块级作用域的引入

let的出现使问题的得到了优雅的解决。只需要如下改动：

```js
for (let j = 0; j < 5; j++) {
    setTimeout(function () {
        console.log(j);
    }, 0);
}// 0,1,2,3,4
```

没错，将var替换成let就可以解决问题。let在循环体内声明了五个**局部变量**，每次循环变量j都会被销毁，使得console.log函数读取到的值不同，打印出想要的结果。

### 常量的不可变性

有时候我们需要一个全局变量，不可以被随意改动，这样就不会被破坏。

```js
const a = 3;
a = 5;// Assignment to constant variable.对常量进行重复赋值。
const pai = 3.1415
```

如果有人尝试给const的变量赋值，就会得到编译器的警告。

## 与JavaScript相通的类型

JavaScript有string、number、boolean、null、undefined、Symbol、Array、Function和Object 9种类型。Typescript提供了相应的类型。

### string

字符串是我们常接触的一种基础的类型，是有数字、字母、下划线组成的一串字符，是编程语言中表示**文本**的数据类型，和javascript一样，可以使用双引号（"）或单引号（'）表示字符串。

```ts
let name:string = "Alice";
```

同时支持ES6的模板字符串，方便定义多行文本和内嵌表达式

```ts
let weather: string = "Sunny";
let today: string = "Friday";
let reporter = `${today} is ${weather}, have a nice day!:smile:`
```

### number

和JavaScript一样，TypeScript里的所有数字都是浮点数。 除了支持十进制和十六进制字面量，TypeScript还支持ES6中引入的二进制和八进制字面量。以下分别是10的各进制表示：

```ts
let num: number = 10;/**十进制 */
let hexNum: number = 0x000a;/**十六进制 */
let binaryNum: number = 0b1010;/**二进制 */
let octNum:number = 0o012;/**八进制 */
```

### boolean

布尔值只有两个值true和false，常用于逻辑的判断，被各种语言支持。

```ts
let isFinished: boolean = false;
if(isFinished) {
    console.log("good,free to play!!")
} else {
    console.log("you need work hard!!")
}
```

### array

数组是有序的元素序列，用于储存多个相同类型数据的集合。在typescript中有两种方式定义数组,与在JavaScript中的定义相差无几，只不过针对的是类型。

- 间接法，在元素类型后面接[]，表示该类型的集合。

```ts
let height: number[] = [155, 168, 177];
```

- 直接法，在Array中注释类型，注意使用**<>**，又称数组泛型。

```ts
let height: Array<number> = [155, 168, 177];
```

### null和undefined

了解javaScript的人，对这两个一定不陌生。没有声明的变量使用时就是undefined，声明但是没有赋值就null，本身的类型用处不大。

```ts
let u: undefined = undefined;
let n: null = null;
```

默认情况下null和undefined是所有类型的子类型,可以被其他类型所接受。指定了`--strictNullChecks`标记时，null和undefined只能赋值给void和它们各自

### function

函数是JavaScript应用程序的基础。 它帮助你实现抽象层，模拟类，信息隐藏和模块。由于函数很重要，将会拿出第四章专门介绍，这里简要提及。对函数的参数和函数返回类型进行描述。

```ts
function hello(msg: string): string {
    return "hello" + msg;
}
```

### symbol

自ES6起，symbol成为了一种新的原生类型，就像number和string一样。symbol类型的值只能通过Symbol构造函数创建的，其值是不可改变和唯一的。关于symbol的细节不多讲，可以参考阮一峰的ES6教程。

```ts
let apple = Symbol("fruit");
let orange = Symbol("fruit")
apple == orange//false
```

### object

object表示非原始类型，也就是除number，string，boolean，symbol，null或undefined之外的类型。使用object类型，就可以更好的表示像Object.create这样的API。

```ts
declare function create(o: object | null): void;

create({ prop: 0 }); // OK
create(null); // OK

create(42); // Error
create("string"); // Error
create(false); // Error
create(undefined); // Error
```

## typescript特有的类型

### any

当我们无法再编译阶段弄清楚一个变量的类型时，可以用`any`类型来描述这些变量。在对现有代码进行改写的时候，`any`类型是十分有用的，它允许你在编译时可选择地包含或移除类型检查。

```ts
let box : any =  "black";
box = 4;
box = ["red", "black"];
```

### void

`void`类型像是与`any`类型相反，它表示没有任何类型。常用来表示函数没有返回值。

```ts
function warnUser(): void {
    console.log("This is my warning message");
}
```

### tuple

元祖类型可以表示一个已知元素数量和类型的数组,是描述数组**内部变量**的一种类型。

```ts
type name = string;
type age = number;
//定义类型
let student: [name, age];
//初始化类型
student = ["tom", 14]
console.log(student[0]);//"tom"
console.log(student[1]);//14
```

### 字符串变量类型

字符串也可作为单独的一种类型,只能赋值给本身。

```ts
let zero: "0" = "0";
let one: "1" = "1";
let two: "2" = "10"//Error：Type '"10"' is not assignable to type '"2"'.“10”类型不可以赋值给“2”类型
```

因为字符串类型具有唯一性，可以当做对象的标识。

```ts
type Circle = {
    kind: "Circle";
    radius: number;
}

type Square = {
    kind: "Square";
    sideLength: number;
}
```

### unknown

typescript 3.1新添加了一种类型 unknown,任何类型都是unknown的子类型，任何值都可以赋值给unknown的变量。

```ts
let magic: unknown = null;
magic = Symbol("a");
magic = false;
magic = 101;
magic = "3394";
```

### never

never类型表示的是那些永不存在的值的类型。具体来说never类型是一个从来不会有返回值的函数或者一个总是会抛出错误的函数。never类型是所有类型的子类型。

```ts
// 总是会抛出错误的函数
function error(message: string): never {
    throw new Error(message);
}

// 从来不会有返回值的函数
function infiniteLoop(): never {
    while (true) {
    }
}
```

>never与void的差别：当一个函数没有返回值时，它返回了一个 void 类型，但是，当一个函数根本就没有返回值时（或者总是抛出错误），它返回了一个 never，void 指可以被赋值的类型（在 strictNullChecking 为 false 时），但是 never 不能赋值给其他任何类型，除了 never。

所有的类型，unknown是类型系统的上界，never是类型系统的下界。
