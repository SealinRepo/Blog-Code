---
title: GOF设计模式及类型简介
tags:
  - 设计模式
originContent: >-
  # 设计模式简介

  设计模式（Design
  pattern）代表了最佳的实践，通常被有经验的面向对象的软件开发人员所采用。设计模式是软件开发人员在软件开发过程中面临的一般问题的解决方案。这些解决方案是众多软件开发人员经过相当长的一段时间的试验和错误总结出来的。


  设计模式是一套被反复使用的、多数人知晓的、经过分类编目的、代码设计经验的总结。使用设计模式是为了重用代码、让代码更容易被他人理解、保证代码可靠性。 


  # 设计模式总览

  <div class="doc">


  |序号| 类型 & 描述| 包括的模式|

  |:---------:|:--------------:|:----------:|

  |1|创建型模式||

  ||这些设计模式提供了一种在创建对象的同时<br>隐藏创建逻辑的方式，而不是使用
  new<br>运算符直接实例化对象。这使得程序在<br>判断针对某个给定实例需要创建哪些对<br>象时更加灵活。|工厂模式（Factory
  Pattern）<hr />抽象工厂模式（Abstract Factory Pattern）<hr />单例模式（Singleton Pattern）<hr
  />建造者模式（Builder Pattern）<hr />原型模式（Prototype Pattern）|

  |2|结构型模式||

  ||这些设计模式关注类和对象的组合。继承<br>的概念被用来组合接口和定义组合对象<br>获得新功能的方式。|<hr />适配器模式（Adapter
  Pattern）<hr />桥接模式（Bridge Pattern）<hr />过滤器模式（Filter、Criteria Pattern）<hr
  />组合模式（Composite Pattern）<hr />装饰器模式（Decorator Pattern）<hr />外观模式（Facade
  Pattern）<hr />享元模式（Flyweight Pattern）<hr />代理模式（Proxy Pattern）|

  |3|行为型模式||

  ||这些设计模式特别关注对象之间的通信。|责任链模式（Chain of Responsibility Pattern）<hr />命令模式（Command
  Pattern）<hr />解释器模式（Interpreter Pattern）<hr />迭代器模式（Iterator Pattern）<hr
  />中介者模式（Mediator Pattern）<hr />备忘录模式（Memento Pattern）<hr />观察者模式（Observer
  Pattern）<hr />状态模式（State Pattern）<hr />空对象模式（Null Object Pattern）<hr
  />策略模式（Strategy Pattern）<hr />模板模式（Template Pattern）<hr />访问者模式（Visitor
  Pattern）|


  </div>


  # 总结

  设计模式总共可分为三大类: **创建型** / **结构型** / **行为型**
categories:
  - 技术
  - 经验
  - 设计模式
toc: true
date: 2019-07-30 10:02:44
---

# 设计模式简介
设计模式（Design pattern）代表了最佳的实践，通常被有经验的面向对象的软件开发人员所采用。设计模式是软件开发人员在软件开发过程中面临的一般问题的解决方案。这些解决方案是众多软件开发人员经过相当长的一段时间的试验和错误总结出来的。

设计模式是一套被反复使用的、多数人知晓的、经过分类编目的、代码设计经验的总结。使用设计模式是为了重用代码、让代码更容易被他人理解、保证代码可靠性。 

# 设计模式总览
<div class="doc">

|序号| 类型 & 描述| 包括的模式|
|:---------:|:--------------:|:----------:|
|1|创建型模式||
||这些设计模式提供了一种在创建对象的同时<br>隐藏创建逻辑的方式，而不是使用 new<br>运算符直接实例化对象。这使得程序在<br>判断针对某个给定实例需要创建哪些对<br>象时更加灵活。|工厂模式（Factory Pattern）<hr />抽象工厂模式（Abstract Factory Pattern）<hr />单例模式（Singleton Pattern）<hr />建造者模式（Builder Pattern）<hr />原型模式（Prototype Pattern）|
|2|结构型模式||
||这些设计模式关注类和对象的组合。继承<br>的概念被用来组合接口和定义组合对象<br>获得新功能的方式。|<hr />适配器模式（Adapter Pattern）<hr />桥接模式（Bridge Pattern）<hr />过滤器模式（Filter、Criteria Pattern）<hr />组合模式（Composite Pattern）<hr />装饰器模式（Decorator Pattern）<hr />外观模式（Facade Pattern）<hr />享元模式（Flyweight Pattern）<hr />代理模式（Proxy Pattern）|
|3|行为型模式||
||这些设计模式特别关注对象之间的通信。|责任链模式（Chain of Responsibility Pattern）<hr />命令模式（Command Pattern）<hr />解释器模式（Interpreter Pattern）<hr />迭代器模式（Iterator Pattern）<hr />中介者模式（Mediator Pattern）<hr />备忘录模式（Memento Pattern）<hr />观察者模式（Observer Pattern）<hr />状态模式（State Pattern）<hr />空对象模式（Null Object Pattern）<hr />策略模式（Strategy Pattern）<hr />模板模式（Template Pattern）<hr />访问者模式（Visitor Pattern）|

</div>

# 总结
设计模式总共可分为三大类: **创建型** / **结构型** / **行为型**