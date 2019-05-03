---
title: JS老代码按需加载业务组件改造
date: 2018-12-20 13:11:52
tags:
    - js
    - webpack
clearReading: true
metaAlignment: center
categories: 技术
---

## 背景
部门有个老项目，使用的是原生js自研框架，是那种spa架构的产品，比较多的一个场景是列表或者表单，点击某个按钮有一些弹框的操作。目前的打包处理方式，这类弹框业务组件和一些基础的通用函数放置在一块打包成chunk，再根据一级路由划分打包成不同业务模块的js文件。所以打开页面时，会去加载某个路由js文件以及chunk文件。

chunk里面包含了一大堆弹框业务组件导致文件变大，影响首屏加载的速度，怎么去优化这个文件大小比较合适？

<!-- excerpt -->

## 方案

- 方法一：从代码角度出发，重构一下业务组件，抽离这些弹框操作的共有部分，节约一大波重复代码。
- 方法二：改变业务组件的引入方式，根据各路由模块拆分业务组件，然后再按需加载文件。

方法一比较花时间，并且需要对业务特别熟悉。团队人员变更较多，并且业务紧急的情况下不太可取。方法二按需加载时花点时间去加载js文件，但交互上感知轻微，并且相比方案一，方案二更可执行。所以采取方案二，那么怎么做到按需加载呢？

## 使用import（）按需加载业务组件
配合webpack使用符合[ECMAScript提案](https://github.com/tc39/proposal-dynamic-import)的**import()语法**来实现动态导入。

之前一般都是业务组件和基础函数一类打包成chunk文件，首屏加载时引入，相应的引入方式如下
```
import { initXXXModal } from 'COMMON/xxxModals'

//当监听中调用时
button.addEventListener('click', event => {
    initXXXModal(data);
});
```

这在文件比较小的时候还行，但项目迭代了，文件大了，就会影响加载速度，使用import()改进后如下，
```
//当监听中调用时
button.addEventListener('click', event => {
    import(/* webpackChunkName: "xxxModals" */ 'COMMON/xxxModals/index).then(({ initXXXModal }) => {
        initXXXModal(data);
    });
});
```

打包后会形成一个xxxModals.js文件，只有点击button时才会去加载。

xxxModals只涉及到某个人编写的某几处弹框组件，那么整个项目中各路由下的各自的组件如果都以这样的方式打包的话，那么最后会形成多个组件的js文件。

目前以一级路由来划分所有业务组件，某个路由下的组件统一一个chunkFile，在使用时再加载这个文件。

<!-- ## 总结
这只是一次简单的利用import()特性优化的过程。 -->