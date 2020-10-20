---
title: koa装饰器
date: 2019-09-07 11:25:56
tags: typescript
categories: 编程
---
装饰器是一种特殊类型的声明，可以用来“装饰”三种类型的对象：类的属性/方法、访问器、类本身。装饰器使用`@expression`这种形式，expression求值后必须为一个函数，称为装饰器工厂函数，它会在运行时被调用，被装饰的声明信息做为参数传入。

注意，装饰器对类的行为的改变，是代码编译时发生的，而不是在运行时。这意味着，装饰器能在编译阶段运行代码。也就是说，装饰器本质就是编译时执行的函数
注意装饰器的调用顺序：

1. 参数装饰器，然后依次是方法装饰器，访问符装饰器，或属性装饰器应用到每个实例成员。
2. 参数装饰器，然后依次是方法装饰器，访问符装饰器，或属性装饰器应用到每个静态成员。
3. 参数装饰器应用到构造函数。
4. 类装饰器应用到类。

参数装饰器会在方法装饰器之前调用，对参数进行注解，外部装饰器可以拿到注解。

## 解析现代koa装饰器

下面来一段装饰器写法：

```ts
// CatController.ts
import { Path, Get, Post, Param, Query, Body } from 'koa-decorate';

@Path('/api/cat')
class CatController {

  @Get
  @Path('/info/:type')
  getCatInfo (
      @Param('type') type: string,
      @Query('info') info: string) {
    return { type, info }
  }

  @Post
  @Path('/info/:type')
  CreateCat (
      @Param('type') type: string,
      @Body('requestBody') requestBody: any) {
    return {
      status: 200,
      data: Object.assign(requestBody, { type }),
      message: 'Created successfully...'
    }
  }

}

export { CatController };
```

这是一段TypeScript上 koa 路由类的写法，注意到在其中，使用了@Paht @Get的写法, 并且在入参中也有`@PathParam('id') id: number`这样的写法。这就是装饰器。其中 @Path('/api')中的API是这个装饰器的入参，在这里是注解，因为这个框架通过Reflect.defineMetadata将这个入参写入到了该方法中。

>Reflect.defineMetadata是一种反射的写法，保存一个对象到缓存中，通过Reflect.getMetadata来访问缓存，无需做过多操作

思考一下该如何实现。

上面我们一共引入了六个装饰器`Path, Get, Post, Param, Query, Body`, 其中path既是类装饰器又是方法装饰器，get和post是方法装饰器，且装饰在path方法装饰器之上，Param, Query, Body 是属性装饰器。考虑到装饰器的执行顺序，先是属性装饰器，再是方法装饰器，然后是类装饰器。

### koa-decorate 参数装饰器

三个参数装饰器 Param, Query, Body 很简单，定义一个独一无二的`Symbol`类型的键,保存被修饰的参数的值和索引。

```ts
const paramSymbolKey = Symbol.for("param");
const querySymbolKey = Symbol.for("query");
const bodySymbolKey = Symbol.for("body");
// 参数装饰器，定义一个target.propertyKey[Param]对象，并设置对象的argName = argIndex
const Param = argName => {
        return (
            target: Object,
            propertyKey: string | symbol,
            argIndex: number
        ) => {
            // 获取保存在target.propertyKey[param]的值，如果不存在，就设置一个空对象
            const args =
                Reflect.getMetadata(paramSymbolKey, target, propertyKey) || {};
            // 设置对象的argName = argIndex，键为被修饰的值，值为参数的索引
            args[argName] = argIndex;
            // 设置新的target.propertyKey[param]的值
            Reflect.defineMetadata(paramSymbolKey, args, target, propertyKey);
        };
    };
```

query 和 body 参数装饰器与 param 装饰器的内容大同小异，只不过是保存的不同的键的对象。
<!-- more -->
### koa-decorate 方法装饰器

在参数装饰器保存了被修饰的参数的值和索引之后，方法装饰器就可以拿到相应的值，并处理一定的逻辑。path装饰器比较重要，这里详细说明。

```ts
const routeSymbolKey = Symbol.for("route");
const pathSymbolKey = Symbol.for("path");
/**
* 方法装饰器，传入一个字符串
*/
export const Path = (path: string): Function => {
    return (
        target: Function,
        propertyKey: string | symbol,
        decorator: PropertyDescriptor
    ) => {
        /**
         * 把原方法设置成一个函数，传递一个控制器，返回一个中间件设置ctx.param与ctx.body
        */
         // 定义routes,值为@path修饰的方法组成的数组
         const routeMethods =
             Reflect.getMetadata(routeSymbolKey, target) || [];
         routeMethods.push(propertyKey);
         Reflect.defineMetadata(routeSymbolKey, routeMethods, target);
         // 定义Path，值为@path的参数
         Reflect.defineMetadata(pathSymbolKey, path, target, propertyKey);
         // 保存原函数
         const oldMethodValue = decorator.value;
         // 这里instance是一个控制器对象，用来调用控制器中被方法修饰器中的函数
         decorator.value = (instance: Object) => async (ctx, next) => 
             const args = [];
             // 获取参数装饰器的保存的对象，为值和索引的键值对
             const param = Reflect.getMetadata(
                 paramSymbolKey,
                 target,
                 propertyKey
             );
             // 如果param存在，把ctx.params对象上的值代理到args.param对象
             param &&
                 Object.keys(param).map(
                     key => (args[param[key]] = ctx.params[key])
                 );
            /**
             * 把 ctx.query 对象上的值代理到 args.query 对象
             */
             const query = Reflect.getMetadata(
                 querySymbolKey,
                 target,
                 propertyKey
             );
             query &&
                 Object.keys(query).map(
                     key => (args[query[key]] = ctx.query)
                 );
            /**
             * 把 ctx.body 对象上的值代理到 args.body 对象
             */
             const body = Reflect.getMetadata(
                 bodySymbolKey,
                 target,
                 propertyKey
             );
             body &&
                 Object.keys(body).map(
                     key => (args[body[key]] = ctx.request.body)
                 );
            // 调用闭包中的原函数
             const result = await oldMethodValue.apply(instance, args);
             ctx.body = result;
        };
    };
};
```

get 和 post 方法装饰器

```ts
const httpMethodSymbolKey = Symbol.for("httpMethod");
const [Get, Post, Put, Delete, All] = [
    "get",
    "post",
    "put",
    "delete",
    "all"
].map(method => {
    return (target, propertyKey) => {
        /**
         * 保存被修饰函数的httpMethodSymbolKey为method(get/post)
         */
        Reflect.defineMetadata(
            httpMethodSymbolKey,
            method,
            target,
            propertyKey
        );
    };
});
```

### koa-decorate 类装饰器

path类装饰器和方法装饰器共用一个函数，这里用来定义根路径。

```ts
const rootPathSymbolKey = Symbol.for("rootPath");
export const Path = (path: string): Function => {
    return (
        target: Function,
        propertyKey: string | symbol,
        decorator: PropertyDescriptor
    ) => {
        // 类修饰器
        if (propertyKey === undefined && decorator === undefined) {
            // 保存被修饰类的rootPathSymbolKey的值为path
            Reflect.defineMetadata(rootPathSymbolKey, path, target.prototype);
        } else {
            //方法装饰器
        }
```

经过一系列装饰器的处理，我们只是保存了很多值，但是并没有使用，下面的将把这些值挂载在koa-router的Router对象上，用来实现相应的路由。

```ts
export default class Decorator {
  private router: Router;// 路由
  private controller: Object; // 控制器

  constructor (options) {
    const {router, controllers} = options;
    if (!controllers) {
      throw new Error('There is no configuration properties "controllers"');
    }
    this.router      = router || new Router();
    this.controllers = controllers;
  }

  private addRoutes (Controller) {
    // 控制器
    const instance     = new Controller();
    // 拿到rootPath
    const rootPath     = Reflect.getMetadata(rootPathSymbolKey, Controller.prototype);
    // 拿到routes,值为控制器内被@path修饰的方法所组成的数组
    const routes       = Reflect.getMetadata(routeSymbolKey, Controller.prototype);

    routes.map((routeName: string) => {
    // method为控制器内被@path修饰的方法
      const method     = instance[routeName];
      // 拿到http方法，有@get、@post等定义，与@Param相差不多，故省略
      const httpMethod = Reflect.getMetadata(httpMethodSymbolKey, Controller.prototype, routeName);
       // 拿到控制器内被@path的参数
      const path       = Reflect.getMetadata(pathSymbolKey, Controller.prototype, routeName);
      // method(instance) 返回中间件
      this.router[httpMethod](`${rootPath}${path}`, method(instance));
    });
  }

};

new Decorator(new Router(), CatController);
```

装饰器通过参数注解以`Reflect.defineMetadata`在缓存中形成一种配置，通过`Reflect.getMetadata`拿到配置，实现相应的功能。

参考文献: [koa-decorate](https://github.com/6peiweb/koa-decorate)
