---
title: typescript接口
date: 2019-09-06 16:24:03
tags: typescript 接口
categories: 编程
---
在计算机中，接口是计算机系统中两个独立的部件进行信息交换的共享边界。这种交换可以发生在计算机软、硬件，外部设备或进行操作的人之间，也可以是它们的结合。
Typescript通过类型检查的方式减少了程序犯错误的机率，但是不能约束代码的产出方式，模块暴露出的API变更之后会产生一系列的麻烦。在TypeScript里，接口的作用就是规范代码，为需要约束的类型命名和为你的代码或第三方代码定义产出方式。因为typescript认为约束条件一样时是属于同一类，所以它有时被称做“鸭式辨型法”或“结构性子类型化”。

## 具名接口

通过`interface`关键词为**约束条件**添加名字，主要为对象和函数添加约束，利用typescript中提到的类型：

1. string
2. number
3. boolean
4. symbol
5. array
6. function
7. object
8. null
9. undefined
10. any
11. void
12. tuple
13. unknown
14. never
15. 字面量类型

### 对象的接口

```ts
interface User{
    name: string;
    id: number;
}

interface Car{
    name: string;
    id: number;
}

let user: User = {
    name: "lily",
    id: 3
}
let car: Car = {
    name: "tick",
    id: 1
}

function getUserName(user: User) {
    return user.name;
}

getUserName(car);
```

函数getUserName限制传入参数为User类，可是我们传入了一个Car类，并且没有报错，这是为什么呢？User类的约束条件为`function test(obj) { return obj === {} && obj.name !== undefined && obj.id !== undefined;}`,同时Car类的约束条件也是一样的，所以函数可以传入Car类，所以typescript有时被称做“鸭式辨型法”或“结构性子类型化”。

### 函数的接口

为函数添加约束条件，约束函数的传入参数和返回值，与第四章添加函数的类型检查时一样。

```ts
interface isArray{
    (array: []): boolean;
}

const isArray : isArray = function(array: []) {
    return Object.prototype.toString.call(array) === '[object Array]';
}
```

## 属性的约束

检查对象的属性分为必须、可选、只读，函数的参数有必须、可选之分

### 可选属性

有时候不确定某个属性是否存在时，在声明的属性后面加入一个`?`，就可以使属性变得可选。

```ts
interface User{
    name: string;
    id?: number;
}

let user: User = {
    name: "boy",
}

let user1:User = {
    name: "jack",
    id: 4
}

function createUser(name: string, id?: number): User {
    return {
        name: name,
    }
}
```

检查条件为`function test(obj) { return (obj === {} && obj.name !== undefined && obj.id !== undefined;) || obj === {} && obj.id !== undefined;}`
<!-- more -->
```ts
interface User{
    name: string;
    id?: number;
}
let app;
let user1: User = (app = {
    name: "ccc",
    card: "d",
    number: 1
});

//Error: card属性不存在User类型里面
console.log(user1.card);
```

```js
var app;
var user1 = (app = {
    name: "ccc",
    card: "d",
    number: 1
});
console.log(user1.card);// "d"
```

虽然声明user1对象时，没有报错，但是使用除了name和id之外的属性时，会报错，这里看到app对象**绕过**了User的类型检查，但是强制翻译ts文件（不推荐这么做），可以得到`user1.card`的值为d,虽然typescript在类型检查阶段报错了，但是并不程序影响正确执行

### 只读属性

当不希望属性的值被更改时，可以设置属性可读。

```ts
interface User{
    name: string;
    readonly id: number;
}

let user1:User = {
    name: "jack",
    id: 4
}

user1.id = 1;//错误：无法对id赋值因为它是一个只读属性。
```

检查条件为

```js
function test(obj) {
    Object.defineProperty(obj, "id", {
        writable: false,
    })
    return obj === {} && obj.name !== undefined && obj.id !== undefined;
}
```

### 额外的属性检查(对象字面量和数组)

由于JavaScript的动态性，可以先声明对象，在添加属性，typescript为了实现这一特性，添加了额外的属性检查，这允许动态添加属性。对象字面量会被特殊对待而且会经过 额外属性检查，当将它们赋值给变量或作为参数传递的时候。

```ts
interface Card{
    readonly id: number;
    name: string;
    [key: string]: string | number;
}

let card:Card = {
    name: "jack",
    id: 4
}

card.size = 20;
card.price = `$1`;

interface StringArray {
    [index: number]: string;
}

let myArray: StringArray = ["Bob", "Fred"];
myArray.push("Alice");

let myStr: string = myArray[0];
```

需要注意的地方是对象所有属性的`key`为string类型，`value`为string或者number类型。

## 实现接口

### 类实现接口

面向对象编程中最重要的实现是类，如何用接口去约束类，是typescript检查的重点。

```ts
interface ClockInterface {
    currentTime: Date;
    getTime() : Date;
}

class Clock implements ClockInterface {
    currentTime: Date;
    constructor(h: number, m: number) { }
    getTime() {
        return this.currentTime;
    }
}
```

这里简单的声明了一个接口，实现接口的类必须要有一个属性currentTime为Date类型,一个方法叫做`getTime`并返回一个Date类型的值。

当你操作类和接口的时候，你要知道类（类是由函数模拟而来）是具有两个类型的：静态部分的类型（存在于函数的属性上）和实例的类型（存在于函数的原型链上）。因为当一个类实现了一个接口时，只对其实例部分进行类型检查。 constructor存在于类的静态部分，所以不在检查的范围内。

```ts
//不能检查构造函数
interface ClockConstructor {
    new (hour: number, minute: number);
}

class Clock implements ClockConstructor {
    currentTime: Date;
    constructor(h: number, m: number) { }
}

// 构造函数通过函数来检查
function createClock(ctor: ClockConstructor, h: number, m: number): ClockConstructor {
    return new ctor(h: number, m: number);
}
```

### 接口继承接口

```ts
interface Point{
    x: number;
    y: number;
}
interface ThreePoint extends Point{
    z: number;
}

let point: ThreePoint = {
    x: 0,
    y: 0,
    z: 0
}
```

前面说到typescript为结构性子类型化，可以看到接口threePoint的检查类型比接口Point的更严格，在检查x,y的属性上额外添加了对z的属性检查。

### 接口继承类

```ts
class Box {
    private state: boolean;
    pick() { };
}

interface selectBox extends Box {
    select(): void;
}

//Error: Property 'state' is private in type 'selectBox' but not in type 'textForm'.
class textForm implements selectBox {
    state: boolean;
    select() {}
    pick() {}
}

class Button extends Box implements selectBox {
    select() {}
}
```

可以看到Box类私有属性state被selectBox接口继承了,则会生成下面这个接口：

```ts
interface selectBox {
    private state: boolean;
    pick() { };
    select(): void;
}
```

textForm类必须实现state，pick()方法,select()方法，由于state为私有属性，所以会报错。因为state是私有成员，所以只能Box的子类们才能实现selectBox接口，只有Box的子类才能够拥有一个声明于Box的私有成员state。

> 无论父类中的成员变量是私有的、共有的、还是其它类型的，子类都会拥有父类中的这些成员变量。但是父类中的私有成员变量，无法在子类中直接访问，必须通过从父类中继承得到的protected、public方法（如getter、setter方法）来访问。这是因为子类对父类进行了**屏蔽**。

## 类与接口的关系

类型和接口之间有一对多和多对一的关系，下面将列举出这些常见的概念，以方便读者理解接口与类型在复杂环境下的实现关系。

### 一个类实现多个接口

一个类型可以同时实现多个接口，而接口间彼此独立，不知道对方的实现。

网络上的两个程序通过一个双向的通信连接实现数据的交换，连接的一端称为一个 Socket。Socket 能够同时读取和写入数据，这个特性与文件类似。因此，开发中把文件和 Socket 都具备的读写特性抽象为独立的读写器概念。Socket 和文件一样，在使用完毕后，也需要对资源进行释放。把 Socket 能够写入数据和需要关闭的特性使用接口来描述，请参考下面的代码：

```ts
// 可被写入的抽象
interface Writer {
    write(p: Buffer): void;
}
// 可被关闭的抽象
interface Closer {
    close(error?: Error) : void;
}
// 实现两个接口
class Socket implements Writer, Closer {
    writer: NodeJS.WriteStream;
    write(data: Buffer): void {
        this.writer.write(data);
    }

    close(error?: Error): void {
        this.writer.destroy(error);
    }
}

// 使用Writer的代码, 并不知道Socket和iCloser的存在
function usingWriter(writer: Writer) {
    writer.write(new Buffer("Hello"));
}
// 使用iCloser, 并不知道Socket和Writer的存在
function usingCloser(closer: Closer) {
    closer.close();
}

function main() {
    let s = new Socket();
    usingWriter(s);
    usingCloser(s);
}
```

{% asset_img 图片1.png 多个类实现一个接口 %}

socket 类实现了 writer 接口和 closer 接口，函数 usingWriter 不需要知道内部细节，只需要知道拥有write方法，函数在使用时传入一个实现了writer 接口的类型，无需关心其他，实现了调用与实现无关。

### 多个类实现一个接口

一个接口的方法，不一定需要由一个类型完全实现，接口的方法可以通过在类型中嵌入其他类型或者结构体来实现。也就是说，使用者并不关心某个接口的方法是通过一个类型完全实现的，还是通过多个结构嵌入到一个结构体中拼凑起来共同实现的。

Service 接口定义了两个方法：一个是开启服务的方法（Start()），一个是输出日志的方法（Log()）。使用 GameService 类来实现 Service，GameService 自己的结构只能实现 Start() 方法，而 Service 接口中的 Log() 方法已经被一个能输出日志的日志器（Logger）实现了，无须再进行 GameService 封装，或者重新实现一遍。所以，选择将 Logger 嵌入到 GameService 能最大程度地避免代码冗余，简化代码结构。详细实现过程如下：

```ts
// 一个服务需要满足能够开启和写日志的功能
interface Service{
    Start()  // 开启服务
    Log(string)  // 日志输出
}
// 日志器
class Logger{
    // 实现Service的Log()方法
    Log(l: string) {

    }
}

// 游戏服务
class GameService extends Logger {
    // 实现Service的Start()方法
    Start() {

    }
}

var s: Service = new GameService();
s.Start()
s.Log("Hello")
```

{% asset_img 图片2.png 多个类实现一个接口 %}

通过接口的方式，分离了接口内和接口外的环境，使得系统设计可以更加容易的分解。

### 用接口实现一个排序系统

下面我们来用接口实现一个排序系统，简单的来说排序就是要遍历集合中的所有元素，比较元素之间的大小或某种次序，移动元素的相应的位置，使整个集合有序的操作。先定义出可排序的基本条件，可被遍历，大小比较，移动元素。声明一个排序接口如下：

```ts
interface Sort {
    // 获取元素数量
    Len(): number;
    // 小于比较
    Less(i: number, j: number): boolean;
    // 交换元素
    Swap(i: number, j: number);
}
```

声明完接口之后，我们就可以对接口提供的逻辑进行使用了。

```ts
// 这里是一个简单的冒泡排序
function sort(data: Sort) {
    for (let i = 0; i < data.Len(); i++) {
        for (let j = 0; j < data.Len(); j++) {
            if (data.Less(i, j)) {
                data.Swap(i, j);
            }
        }
    }
}
```

下一步就是实现接口，这是内部逻辑的填充，接口只是一个声明，无法执行相应的逻辑。在编译成 JavaScript 后，相应的接口也会被檫除。

```ts
// 第一个实现的是对字符串的排序
class MyStringList implements Sort {
    // 声明字符串数组
    List: string[];
    constructor(List: string[]) {
        this.List = List;
    }
    // 获取字符串数组的长度
    Len() {
        return this.List.length;
    }
    // 字符串大小比较的规则
    Less(i: number, j: number) {
        return this.List[i] < this.List[j];
    }
    // 交换
    Swap(i: number, j: number) {
        [this.List[i], this.List[j]] = [this.List[j], this.List[i]];
    }
}
```

所有的工作都已经做完了，下面我们来测试一下实际效果。

```ts
const names = new MyStringList([
    "3. three sheep",
    "5. five sheep",
    "2. two sheep",
    "4. four sheep",
    "1. one sheep"
]);

sort(names);

console.log(names);
```

很好地完成了排序工作，现在这里有一个数组，同样也需要排序。现在不比费心去重新去写一遍，在原来排序接口的接口上添加数组的排序逻辑即可。

```ts
// 数组实现排序的接口
class myArray implements Sort {
    List: number[];
    constructor(List: number[]) {
        this.List = List;
    }
    Len() {
        return this.List.length;
    }
    // 注意这里虽然逻辑相同，但是数字大小的比较，而上面是字符串的比较
    Less(i: number, j: number) {
        return this.List[i] < this.List[j];
    }
    Swap(i: number, j: number) {
        [this.List[i], this.List[j]] = [this.List[j], this.List[i]];
    }
}
```

同样地检测一下排序效果，给定一个数组。

```ts
const list = new myArray([3, 4, 1, 2, 5]);
sort(list);

console.log(list);
```

现在我们把排序的概念延伸一下，上面的排序系统不仅能排序数组和字符串数组这种简单的数据结构，还能排序其他的数据结构，比如常见的对象。

```ts
// 声明军队的分类
enum ArmyKind{
    None = 1,
    Air,
    Sea,
    land
}
// 定义一个军队
class Army {
    name: string;
    kind: ArmyKind;
    constructor(name : string, kind : number) {
        this.name = name;
        this.kind = kind;
    }
}
// 实现排序逻辑
class Armies implements Sort {
    armyList: Army[];
    constructor(armyList: Army[]) {
        this.armyList = armyList;
    }
    Len() {
        return this.armyList.length;
    }
    Less(i: number, j: number) {
        let s = this.armyList;
        // 如果军队的分类不一致时, 优先对分类进行排序
        if (s[i].kind != s[j].kind) {
            return s[i].kind < s[j].kind;
        }
        // 默认按军队名字字符升序排列
        return s[i].name < s[j].name;
    }
    Swap(i: number, j: number) {
        let s = this.armyList;
        [s[i], s[j]] = [s[j], s[i]];
    }
}
```

下面看看效果会怎么样，给出一个`Armies`类的实例。

```ts
let armies = new Armies([
    new Army("A3", ArmyKind.Air),
    new Army("A8", ArmyKind.Air),
    new Army("S1", ArmyKind.Sea),
    new Army("S4", ArmyKind.Sea),
    new Army("L2", ArmyKind.land)
]);
// 使用sort包进行排序
sort(armies);

console.log(armies);
```

MyStringList、MyArray和Armies类都实现了sort接口，可见他们有一种共性。这种共性抽象出了一个Sort概念（可被排序的基本要求），接下来用MyStringList、MyArray和Armies类去实现sort接口的要求，使得提供len、less和swap方法的类就可以被排序。

在排序算法中使用Sort接口即可完成排序，无需关心其实现，直接与MyStringList、MyArray和Armies类提供的上层抽象sort相关联。调用时，只要传入MyStringList、MyArray和Armies类的实例即可。同样抽象类也可以，MyStringList、MyArray和Armies类都继承同一个抽象类，排序算法算法直接与抽象类打交道。当然还有更高效的排序算法，这里就不一一列举了。

这样抽象就建立起来了，通过抽象类和接口去描述抽象，使得系统设计可以更加容易的分解和解耦，为大型系统的设计铺路。

## 接口与抽象类的区别

对于面向对象编程来说，**抽象**是它的一大特征之一。在面对对象的语言Java中，可以通过两种形式来体现OOP的抽象：接口和抽象类。这两者有太多相似的地方，又有太多不同的地方。typescript同样也实现了接口和抽象类，两种语言的概念差别不大。

抽象类是什么：
>抽象类是从多个具体类中抽象出来的父类，它具有更高层次的抽象。从多个具有相同特征的类中抽象出一个抽象类，以这个抽象类作为其子类的模板，从而避免了子类的随意性。抽象类不能创建实例，它只能作为父类被继承。

```ts
abstract class Book {
    pages: Page[];
    currentPage = 0;
    open(pageNumber: number) {
        return this.pages.filter(value => value.page = pageNumber);
    }

    close() {
        this.currentPage = 0;
        return this.pages[this.currentPage];
    }
}

class Page{
    page = 0;
    content = "";
    constructor(page: number, content: string) {
        this.page = page;
        this.content = content;
    }
}

class mathBook extends Book {
    name: string;
    constructor(name: string, pages: Page[]) {
        super();
        this.name = name;
        this.pages = pages;
    }
}

let math = new mathBook("math", [
    new Page(1, "3 + 5"),
    new Page(2, " 3 - 1"),
    new Page(3, " 3 * 2"),
    new Page(4, " 9 / 3"),
]);
math.open(3);
math.close();

class englishBook extends Book {
    name: string;
    constructor(name: string, pages: Page[]) {
        super();
        this.name = name;
        this.pages = pages;
    }
}

let english = new englishBook("english", [
    new Page(1, "hello"),
    new Page(2, "china"),
    new Page(3, "food"),
    new Page(4, "weather"),
])

english.open(2);
english.close();
```

抽象类`Book`封装了书的概念，书可以被打开和关闭，数学书和英语书继承了书的概念，符合我们现实生活中的观点。

接口是什么：
>接口本身是一种对行为的一种抽象，实现接口的类将按照接口的规范约束行为。我们开发中经常说给别人提供接口，而不是说给别人提供实现类。我们将属性私有，通过接口中的行为来操作。这样封装了内部的实现。当我们对外提供接口，而不是直接暴露实现类，这样调用的类就实现了与提供类之间的解耦。

```ts
interface Open{
    open(): void;
}

class Door implements Open{
   private closed = true;
    open() {
        this.closed = false;
    }
    close() {
        this.closed = true;
    }
}

class Computer implements Open{
    open() {
        this.setup();
    }
    private setup() {
        console.log("hello world");
    }
}

let door: Open = new Door();
door.open();

let mac: Open = new Computer();
mac.open();
```

上面的两个类`Door`和`Computer`很难有共同特点，但是有一个`open`方法，使用抽象类固然可以，但是一个功能一个继承，多重继承会带来很多麻烦，使用接口能更加细粒度地描述抽象，即接口比抽象类更加抽象和灵活，细数两者之间的区别：

1. 抽象类可以提供成员方法的实现细节，而接口中只能存在public abstract方法；
2. 抽象类中的成员变量可以是各种类型的，而接口中的成员变量只能是public static final类型的；
3. 接口中不能含有静态代码块以及静态方法，而抽象类可以有静态代码块和静态方法；
4. 一个类只能继承一个抽象类，而一个类却可以实现多个接口。

## 参考链接

[Go语言类型与接口的关系](http://c.biancheng.net/view/79.html)

[Go语言排序](http://c.biancheng.net/view/81.html)

[深入理解Java的接口和抽象类](https://blog.csdn.net/universe_ant/article/details/59491738)
