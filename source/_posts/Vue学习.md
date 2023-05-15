---
title: Vue学习
date: 2021-10-17 15:31:01
tags: vue
categories: 学习
keywords: vue
description: Vue的学习笔记
cover: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
top_img: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
---

# Vue学习

# 1.Vue是什么？

一套用于构建用户界面的渐进式JavaScript框架，vue可以自底向上逐层应用。

简单应用：只需一个轻量小巧的核心库

复杂应用：可以引用各式各样的Vue插件

# 2.Vue特点

1.  采用组件化模式，提高代码复用率、且让代码更好维护。

2.  声明式编码，让编码人员无需直接操作DOM，提高开发效率。&nbsp;

3.  使用虚拟DOM+优秀的Diff算法，尽量服用Dom节点。

# 3.学习Vue

## 3.1初识Vue

1.  想让Vue工作，就必须创建一个vue实例，且要传入一个配置对象。

2.  root容器里的代码依然符合html规范，只不过混入了一些特殊的Vue语法&nbsp;

3.  root容器里的代码被称为【Vue模板】

4.  Vue实例和容器是一一对应的&nbsp;

5.  真实开发中只有一个Vue实例，并且会配合着组件一起使用

6.  {{xxx}}中的xxx要写进js表达式，且xxx可以自动读取到data中的所有属性&nbsp;

7.  一旦data中的数据发生改变，那么页面中用到该数据的地方也会自动更新

8.  注意区分：1.js表达式 和 js代码（语句）

    1.  表达式：一个表达式会产生一个值，可以放在任何一个需要值的地方

        1.  a

        2.  a+b&nbsp;

        3.  demo(1)

        4.  x === y? 'a' : 'b'&nbsp;

    2.  js代码（语句）

        1.  if(){}

        2.  for(){}

## 3.2模板语法

### 3.2.1插值语法

功能：用于解析标签体内容

写法：{{xxx}}，xxx是js表达式，且可以直接读取到data中所有属性。

### 3.2.1指令语法

功能：用于解析标签（包括：标签属性、标签体内容、绑定时间。。。。）

举例：v-bind:href="xxx" 或 简写为:href="xxx"，xxx同样要写js表达式。且可以直接读取到data中的所有属性。

备注：Vue中有很多的指令，且形式都是：v-???，此处我们只是拿v-bind举个例子。

## 3.3数据绑定

1.  单项绑定（v-bind）：数据只能从data流向页面

2.  双向绑定（v-model）：数据不仅能从data流向页面，还可以从页面流向data。&nbsp;

3.  备注：

    1.  1.双向绑定一般都应用在表单元素上（如：input、select等）

    2.  2.v-model:value 可以简写为v-model，因为v-model默认收集的就是value值

## 3.4data与el的两种写法

1.  el有两种写法

    1.  new Vue时配置el属性

    2.  先创建Vue实例，随后再通过vm.\$mount('\#root')指定el的值。&nbsp;

2.  data的两种写法

    1.  对象式

    2.  函数式&nbsp;

    3.  如何选择，目前哪种写法都可以，以后学习到组件时，data必须使用函数式，否则会报错。&nbsp;

3.  一个重要的原则：

    1.  由Vue管理的函数，一定不要写箭头函数，一旦写了箭头函数，this就不在是Vue的实例了

## 3.5MVVM模型

1.  M：模型（Model）：data中的数据

2.  V：视图（View）：模板代码&nbsp;

3.  VM：视图模型（ViewModel）：Vue实例

4.  观察发现：

    1.  data中所有的属性，最后都出现在vm身上

    2.  vm身上所有的属性，及Vue原型上所有的属性，在Vue模板中都可以直接使用。
