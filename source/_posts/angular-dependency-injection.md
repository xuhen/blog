---
title: angular 依赖注入
date: 2020-10-04 11:05:44
tags:
  - angular
---

## 定义

> Dependency Injection (DI) is a technique in which we provide an instance of an object to another object, which depends on it. This is technique is also known as “Inversion of Control” (IoC)

## Angular 依赖注入框架

Angular Dependency Injection framework implements the Dependency injection Pattern in Angular. It creates & maintains the Dependencies and injects them into the Components or Services which requests for it.

### 依赖注入的组成部分

1. Consumer（消费者）： The Component that needs the Dependency
2. Dependency（依赖）：The Service that is being injected
3. DI Token ：The DI Token uniquely identifies a Dependency. We use DI Token when we register dependency
4. Provider（提供者）：The Providers Maintains the list of Dependencies along with their Tokens. The DI Token is used to identify the Dependency.
5. Injector（注入者）：Injector holds the Providers and is responsible for resolving the dependencies and injecting the instance of the Dependency to the Consumer

### @Injectable() 装饰器

`@Injectable()` decorator is not needed, if the class already has other Angular decorators like `@Component`, `@pipe` or `@directive` etc. Because all these are a subtype of Injectible.

> @Injectible is also not needed if the class does not have any dependencies to be injected. However it is best practice is to decorate every service class with @Injectable(), even those that don’t have dependencies.

### 注意点

Angular does not have any options add providers in the Service Class. The Providers must be added to the Component/Directive/Pipe or to the Module.

## services 寻找规则

1. 全局服务：
   The services injected at the module level are app-scoped, which means that they can be accessed from every component/service within the app. Any service provided in the Child Module is available in the entire application.
2. 局部模块服务：
   The services is provided in a lazy module are module scoped and available only to the lazy loaded module.
3. 组件级别服务：
   The services provided in the Component level are available only to the Component & and to the child components.
