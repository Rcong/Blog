---
title: QAP中的Web Chart需求
date: 2018-10-02 19:48:41
tags:
    - QAP
clearReading: true
metaAlignment: center
categories: 技术
---

## 背景
QAP的Chart库是基于weex + rax, 包含了线图、柱状图、饼图、雷达图等，由客户端插件实现，性能好，开发友好，直接引用就行。

- QAP原生的Chart

![Qap Chart](https:////wx4.sinaimg.cn/mw690/8e70eab6ly1g5q9yq5aexj20k40un0uj.jpg)

但是对于各插件商自己的业务需求，QAP的Chart库很难去满足不同的展现逻辑和需求。

<!-- excerpt -->

## 背景

QAP的Chart库是基于weex + rax, 包含了线图、柱状图、饼图、雷达图等，由客户端插件实现，性能好，开发友好，直接引用就行。

- QAP原生的Chart

![Qap Chart](https:////wx4.sinaimg.cn/mw690/8e70eab6ly1g5q9yq5aexj20k40un0uj.jpg)

但是对于各插件商自己的业务需求，QAP的Chart库很难去满足不同的展现逻辑和需求。

- 使用H5达到效果的Chart

![H5 Chart](https:////wx3.sinaimg.cn/mw690/8e70eab6ly1g5q9yq9iy3j20iz0pc119.jpg)

## 方案

H5中的Chart库相对于QAP的图表库拥有更丰富的自定义设置，可以满足日常业务方所需要的展示逻辑和需求，但是H5的Chart库怎么在以Weex渲染的QAP平台中去展现的?

- 在native端使用weex提供的Web网页容器

```
import {createElement, Component, render} from 'rax'; // eslint-disable-line no-unused-vars
import { Web, Page, Modal, Button, View } from 'nuke';
import QAP from 'QAP-SDK';

const styles = {
    content : {
        height:'500rem',
        margin:'30rem',
        padding:'10rem',
        backgroundColor:'#ffffff',
        justifyContent:'center',
        alignItems:'center'
    },
    btns:{
        margin:'30rem'
    }
}

class WebDemo extends Component {
    constructor() {
        super();
        this.eventsBind();
    }

    eventsBind(){
        QAP.on('App.WebChart.chartDemoLoaded', () => {
            Modal.toast('H5页面Loaded');
            this.getRandomData()
        });
    }

    getRandomData(){
        let date = ['09.06', '09.07', '09.08', '09.09', '09.10', '09.11', '09.12'],
            data = [Math.random() * 100, Math.random() * 100, Math.random() * 100, Math.random() * 100, Math.random() * 100, Math.random() * 100, Math.random() * 100];

        QAP.emit('App.WebChart.chartDemo', {date: date, data: data, util: 'QAP与H5图表通信随机数据'});
    }

    render() {
        return (
            <Page title="Web">
                <Page.Intro main="使用 web 组件载入图表页面,H5托管七牛云"></Page.Intro>
                <Web style={[styles.content]} src="http://puqqf48gp.bkt.clouddn.com/chartDemo.html"></Web>
                <View style={styles.btns}>
                    <Button onPress={this.getRandomData} block="true" type="primary">随机数据</Button>
                </View>
            </Page>
        );
    }
}

render(<WebDemo/>);
```

- H5页面
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>图表展示</title>
    <meta name="renderer" content="webkit">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <script src="https://g.alicdn.com/x-bridge/qap-sdk/1.0.10/qn.min.js"></script>
    <script src="https://g.alicdn.com/sj/lib/jquery.min.js"></script>
    <script src="https://code.highcharts.com.cn/highcharts/highcharts.js"></script>
</head>
<body>
    <div id="chartContainer">
    </div>
</body>
<script type="text/javascript">
    $(function() {
        QN.emit('App.WebChart.chartDemoLoaded', true);

        QN.on('App.WebChart.chartDemo', function(chartParam) {
            //图表展示逻辑
        });
    })
</script>
</html>
```

## 整体的逻辑

![整体的逻辑](https:////wx1.sinaimg.cn/mw690/8e70eab6ly1g5q9yqc3f0j210i0u0e11.jpg)

总的来说,通过QAP提供的事件机制起到Web页与QAP页面的通信。Web页面的展现逻辑相对于Weex渲染的页面和组件可以做到更丰富。