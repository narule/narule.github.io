---
title: java面向对象设计原则
author: Narule
date: 2018-10-11 20:10:00 +0800
categories: [Blogging, Technology^技术, Java]
tags: [writing, Java]
---



## 总原则 开闭原则（OCP：Open Closed Principle） 

　　对扩展开放，对修改关闭。设计功能模块的时候，应当使这个模块在不被修改的前提下可以被扩展（功能）

### 一 里氏替换原则 （LSP：Liskov Substitution Principle）

　　对于父类出现的地方，都可以用子类代替（多态，继承）

### 二 单一职责原则（SRP：Single responsibility principle）

　　一个类或模块应该只做一件事(一个类或者模块对应一个功能类),高内聚，低耦合，专注于单一功能（高内聚）

### 三 接口隔离原则（ISP：Interface Segregation Principle）

　　一个接口最好只有一个方法（功能），让实现一个接口的类重写一种方法（功能）。

　　针对不同功能应该有不同接口，使接口的功能有选择性，不强迫必须实现不需要的功能。

### 四 依赖倒置原则（DIP：Dependence Inversion Principle）

　　依赖抽象不依赖具体，高层模块不应该依赖底层模块，两者应该依赖抽象，抽象不应该依赖细节（具体实现），细节依赖抽象（依赖接口）

　　提高可维护性

### 五 迪米特原则 (知道最少)（LOD：Law of Demeter）

　　对象之间联系越少越好，对于对象的使用，方法调用，具体内部细节知道的越少越好（高内聚，低耦合） 可维护性强

### 六 组合/聚合原则（CRP：Composite Reuse Principle）

　　尽量使用对象组合，而不是继承对象达到功能复用的目的，一个新对象A能使用已有对象B达到功能复用(B对象的功能)，就不要通过继承(B)对象来达到功能复用