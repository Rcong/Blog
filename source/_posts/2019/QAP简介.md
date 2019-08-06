---
title: QAP简介
date: 2019-07-25 15:48:41
tags:
    - QAP
clearReading: true
metaAlignment: center
categories: 技术
---

QAP —— Qianniu Application Platform，是阿里千牛官方推出的开发者套件。底层使用了 weex 渲染，上层有 SDK 接口、Rax 库、基于 Rax 的 UI 组件库 Nuke 以及命令行工具 QAP-CLI。

<!-- excerpt -->

QAP —— Qianniu Application Platform，是阿里千牛官方推出的开发者套件。底层使用了 weex 渲染，上层有 SDK 接口、Rax 库、基于 Rax 的 UI 组件库 Nuke 以及命令行工具 QAP-CLI。

## 背景

2017年以前，千牛官方使用插件的方案还是基于 webview 展现 H5 页面的形式，这样的方案下有几个问题:

- H5 编写的 Web App 插件使用 webview 容器加载后，会导致内存占用量飙升从而使千牛容易被系统回收。
- 一些运营商的劫持问题会骚扰用户和 ISV。
- ISV 直接在自己的服务器端进行发布不收千牛平台方的约束和管控。

## 方案

针对以上问题，以2017年阿里千牛将整个技术方向切换到了 QAP 平台，使用 QAP 相关的开发方案，去解决了一些的问题。

- 经过编译的 QAP 工程代码会以 native 方式解析 JS 并运行。缩短了插件打开时间并提高运行时的效率。提供了良好的用户体验。
- 页面的渲染都是请求平台方的 JS Bundle 文件，然后在客户端 JS 解析与 UI 绘制，避免了运营商劫持的问题，虽然可以使用 https 解决这个问题，但是对于一些小型 ISV 来说，QAP 的方案替他们低成本的解决了问题。
- 发布 QAP 应用需要在 QAP 相关发布平台进行发布，千牛可以很好的约束和管控 ISV，在临近一些双十一的重大节日，可以封网发布。

## QAP的体系

上述说道 ISV 的工程会以 native 的方式解析并去运行，这样就需要两部分的代码
- ISV 业务代码
- 运行环境的 framework 代码

这样就引出了一个问题，一般 ISV 的插件是一个多页面的应用，加上 framework 的底层代码，打包成 JS Bundle 就会大大增加了包的大小，从而在访问插件下发 Bundle 文件的时候影响插件的性能和用户体验。


对于这个问题，千牛官方给出的解决方案是抽离出公共的运行环境 framework 代码（ Rax、Nuke 组件以及 SDK ）放入 Main.js 里面， Main.js 是Weex的运行环境，这样 Bundle 包里面就只剩下了 ISV 的业务代码了，这样使得包的大小能够很好地控制。所以引出了下图这样的体系：

![QAP体系](https:////wx1.sinaimg.cn/mw1024/8e70eab6ly1g5qa4k5fwlj20tw0qa1kx.jpg)

- 顶层 JS Bundle 是 ISV 的业务代码
- Rax、Nuke 组件以及 SDK 则是被抽离出内置到了 weex 的运行环境中去了，这部分负责 JS 与 natvie 之间的交互。数据绑定、事件逻辑处理等。


那么在这样的体系中，整个开发流程是怎样，体验是怎么样？

## 开发流程

这是根据一些淘系 weex 的 PPT 资料整理出来的图:

![流程](https://wx1.sinaimg.cn/mw1024/8e70eab6ly1g5qa4jz2dwj20wo0u0q4t.jpg)

1. 在这里开发者使用 weex 的 DSL —— 即 Nuke（基于 Rax 的组件库）和 Rax 来编写整个项目工程， Rax 的使用方式和 RN 类似。
2. 相关的开发体验都会基于[ QAP-CLI ](https://www.npmjs.com/package/qap-cli)这个脚手架，功能涉及到了初始化项目、调试、工程打包等，当开发者完成了项目，使用脚手架的打包命令```qap package```，即可以将 Rax 语法的代码 transformer 得到了纯 JS Bundle 代码。
3. 利用 QAP 的发布平台，发布 Bundle 包到服务端/CDN。
4. 当用户打开千牛的时候， weex runtime 就会初始化，然后去执行 JS Framework 的代码
5. 访问 ISV 插件时，就会去服务端/CDN上请求 JS Bundle 文件
6. 客户端 JS 解析与 UI 绘制。


体验：在开发者层面的还是集中在第一点，业务层的开发，开发体验和在 Web 界面写 React 还是很接近。

## 结语

通过 QAP 开发，在千牛插件可以做到一次编写，两端（ iOS、Android ）运行，提高运行时的效率，提供了良好的用户体验。但与传统 Web 开发还是略有不同，文章简介了 QAP 相关，让不熟悉 QAP 的相关人员略有了解。文章有遗漏之处或者不准确的地方欢迎指出。



