---
title: 百花齐放下的前端MVC、MVP、MVVM设计模式框架
date: 2016-10-29 12:34:56
tags: 
- 设计模式
- MVVM
- MVC
- MVP
---
目前前端的工程化、结构化越来越普及，各种MV*系列的框架应运而生，比较多的有三种设计模式：MVC、MVP、MVVM，本文阐述一下自己对这3种设计模式的看法。

## MVC模式
MVC即Model-VIew-Controller。MVC模式致力于关注点的切分，这意味着model和controller的逻辑是不与用户界面（View）挂钩的。因此，维护和测试程序变得更加简单容易。
MVC设计模式将应用程序分离为3个主要的方面：Model，View和Controller

![MVC设计模式图](/images/mvc.png "MVC Pattern")

<!--more-->

### Model
> Model代表了描述业务路逻辑，业务模型、数据操作、数据模型的一系列类的集合。这层也定义了数据修改和操作的业务规则。

### View
> View代表了UI组件，像CSS，jQuery，HTML等。他只负责展示从controller接收到的数据。也就是把model转化成UI。

### Controller
> Controll负责处理流入的请求。它通过View来接受用户的输入，之后利用Model来处理用户的数据，最后把结果返回给View。Controll就是View和Model之间的一个协调者。

### JS代表框架
Backbone.js、Angular.js

## MVP模式
这个模式把Presenter换成Controller就非常和MVC相像了。这个设计模式把应用程序分成了3个主要方面：Model、View和Presenter。

![MVP设计模式图](/images/mvp.png "MVP Pattern")

### Model
> Model层代表了描述业务逻辑和数据的一系列类的集合。它也定义了数据修改和操作的业务规则。

### View
> View代表了UI组件，像CSS，JQuery，html等。他只负责展示从Presenter接收到的数据。也就是把模型（非Modle层模型）转化成UI。

### Presenter
> Presenter负责处理View背后所有的UI事件。它通过View接收用户输入，之后利用Model来处理用户的数据，最后把结果返回给View。与View和Controller不同，View和Presenter之间是完全解耦的，他们通过接口来交互。另外，presenter不像controller处理进入的请求。

这个模式被普遍的引用于ASP.NET Web Forms 应用程序。并且也应用于windows form。

### MVP模式关键点
1. 用户和View交互。
2. View和Presenter是一对一关系。意味着一个Presenter只映射一个View。
3. View持有Presenter的引用（通过接口交互，并不直接引用Presenter），但是View不持有Model的引用。
4. 在View和Presenter之间可以双向交互。

### JS代表框架
Riot.js

## MVVM模式
MVVM即Model-View-View Model。这个模式提供对View和View Model的双向数据绑定。这使得View Model的状态改变可以自动传递给View。典型的情况是，View Model通过使用obsever模式（观察者模式）来将View Model的变化通知给model。

![MVVM设计模式图](/images/mvvm.png "MVVM Pattern")

### Model
> Model层代表了描述业务逻辑和数据的一系列类的集合。它也定义了数据修改和操作的业务规则。

### View
> View代表了UI组件，像CSS，JQuery，html等。他只负责展示从Presenter接收到的数据。也就是把模型转化成UI。

### View Model
> View Model负责暴漏方法，命令，其他属性来操作VIew的状态，组装model作为View动作的结果，并且触发view自己的事件。

这个模式被广泛应用于WPF，Silverlight，Caliburn，nRoute 等。

### MVVM模式关键点
1. 用户和View交互。
2. View和ViewModel是多对一关系。意味着一个ViewModel只映射多个View。
3. View持有ViewModel的引用，但是ViewModel没有任何View的信息。
4. View 和ViewModel之间有**双向数据绑定关系**。

### JS代表框架
Vue.js、Knockout.js、Angular.js