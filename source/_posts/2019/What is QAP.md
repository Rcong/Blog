---
title: What is QAP
date: 2019-07-25 15:48:41
tags:
    - QAP
clearReading: true
metaAlignment: center
categories: 技术
---

QAP —— Qianniu Application Platform，是阿里千牛官方推出的开发者套件。底层使用了weex渲染，上层有SDK接口、rax库、基于Rax的ui组件库Nuke以及命令行工具QAP-CLI。

<!-- excerpt -->

## 背景

2017年以前，千牛官方使用插件的方案还是基于webview展现H5页面的形式，这样的方案下有几个问题:

- H5编写的Web App插件使用Webview容器加载后，会导致内存占用量飙升从而使千牛容易被系统回收。
- 一些运营商的劫持问题会骚扰用户和ISV。
- ISV直接在自己的服务器端进行发布不收千牛平台方的约束和管控。

## 方案

针对以上问题，以2017年阿里千牛将整个技术方向切换到了QAP平台，使用QAP相关的开发方案，去解决了一些的问题。

- 经过编译的QAP工程代码会以Native方式解析JS并运行。缩短了插件打开时间并提高运行时的效率。提供了良好的用户体验。
- 页面的渲染都是请求平台方的JS Bundle文件，然后在客户端JS解析与UI绘制，避免了运营商劫持的问题，虽然可以使用https解决这个问题，但是对于一些小型isv来说，QAP的方案替他们低成本的解决了问题。
- 发布QAP应用需要在QAP相关发布平台进行发布，千牛可以很好的约束和管控isv，在临近一些双十一的重大节日，可以封网发布。

## QAP的体系
上述说道isv的工程会以native的方式解析并去运行，这样就需要两部分的代码
- isv业务代码
- 运行环境的framework代码

这样就引出了一个问题，一般isv的插件是一个多页面的应用，加上framework的底层代码，打包成JS Bundle就会大大增加了包的大小，从而在访问插件下发Bundle文件的时候影响插件的性能和用户体验。

对于这个问题，千牛官方给出的解决方案是抽离出公共的运行环境framework代码（Rax、Nuke组件以及SDK）放入Main.js里面，Main.js是Weex的运行环境，这样Bundle包里面就只剩下了isv的业务代码了，这样使得包的大小能够很好地控制。所以引出了下图这样的体系：

![QAP体系](https://cdn.nlark.com/yuque/0/2019/png/103782/1563981630054-7e7cccc3-87a0-4715-8a2e-0d8b65171bf3.png?x-oss-process=image/resize,w_555)

- 顶层JS Bundle是isv的业务代码
- Rax、Nuke组件以及SDK则是被抽离出内置到了Weex的运行环境中去了，这部分负责JS与natvie 之间的交互。数据绑定、事件逻辑处理等。


基于这样的体系架构，整体的开发与访问流程就是：

![流程](https://cdn.nlark.com/yuque/0/2019/png/103782/1564047395539-aadd38ec-79c6-4ee4-a91d-71102d5bd29b.png?x-oss-process=image/resize,w_646)

- isv使用Nuke（Rax）、Rax来编写业务代码（即Weex DSL）
- 使用QAP-CLI执行打包命令，经过transformer得到了纯JS Bundle代码
- 利用QAP的发布平台，发布Bundle文件到服务端/CDN
- 当用户访问插件时，就会去服务端/CDN上请求JS Bundle文件
- 客户端JS解析与UI绘制




