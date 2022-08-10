---
layout: post
title: ECharts 自定义立体柱状图
date: 2022-08-10 08:54:24.000000000 +08:00
tag: ECharts
---

## 最终效果
---
![a表](/assets/images/2022/20220810_001.png)

## 配置项
``` js
var categoryData = [];
var errorData = [];
var barData = [];
var dataCount = 100;
for (var i = 0; i < dataCount; i++) {
    var val = Math.random() * 1000;
    categoryData.push('category' + i);
    errorData.push([
        i,
        echarts.number.round(Math.max(0, val - Math.random() * 100)),
        echarts.number.round(val + Math.random() * 80)
    ]);
    barData.push(echarts.number.round(val, 2));
}


// 绘制左侧面
const CubeLeft = echarts.graphic.extendShape({
    shape: {
        x: 0,
        y: 0
    },
    buildPath: function (ctx, shape) {
        // 会canvas的应该都能看得懂，shape是从custom传入的
        const xAxisPoint = shape.xAxisPoint
        const c0 = [shape.x, shape.y]
        const c1 = [shape.x - 13, shape.y - 13]
        const c2 = [xAxisPoint[0] - 13, xAxisPoint[1] - 13]
        const c3 = [xAxisPoint[0], xAxisPoint[1]]
        ctx.moveTo(c0[0], c0[1]).lineTo(c1[0], c1[1]).lineTo(c2[0], c2[1]).lineTo(c3[0], c3[1]).closePath()
    }
})
// 绘制右侧面
const CubeRight = echarts.graphic.extendShape({
    shape: {
        x: 0,
        y: 0
    },
    buildPath: function (ctx, shape) {
        const xAxisPoint = shape.xAxisPoint
        const c1 = [shape.x, shape.y]
        const c2 = [xAxisPoint[0], xAxisPoint[1]]
        const c3 = [xAxisPoint[0] + 18, xAxisPoint[1] - 9]
        const c4 = [shape.x + 18, shape.y - 9]
        ctx.moveTo(c1[0], c1[1]).lineTo(c2[0], c2[1]).lineTo(c3[0], c3[1]).lineTo(c4[0], c4[1]).closePath()
    }
})
// 绘制顶面
const CubeTop = echarts.graphic.extendShape({
    shape: {
        x: 0,
        y: 0
    },
    buildPath: function (ctx, shape) {
        const c1 = [shape.x, shape.y]
        const c2 = [shape.x + 18, shape.y - 9]
        const c3 = [shape.x + 5, shape.y - 22]
        const c4 = [shape.x - 13, shape.y - 13]
        ctx.moveTo(c1[0], c1[1]).lineTo(c2[0], c2[1]).lineTo(c3[0], c3[1]).lineTo(c4[0], c4[1]).closePath()
    }
})
// 注册三个面图形
echarts.graphic.registerShape('CubeLeft', CubeLeft)
echarts.graphic.registerShape('CubeRight', CubeRight)
echarts.graphic.registerShape('CubeTop', CubeTop)


option = {
    tooltip: {
        trigger: 'axis',
        axisPointer: {
            type: 'shadow'
        }
    },
    title: {
        text: 'Error bar chart'
    },
    legend: {
        data: ['bar', 'error']
    },
    dataZoom: [
        {
            type: 'slider',
            start: 0,
            end: 20,
            height: 12,
            brushSelect: false,
            fillerColor: '#a4b5eb',
            borderColor: "#ffffff00"
        },
        {
            type: 'inside',
            start: 0,
            end: 20,
            zoomLock: true,
            moveOnMouseWheel: true
        }
    ],
    xAxis: {
        data: categoryData
    },
    yAxis: {},
    series: [
        {
            type: 'custom',
            name: 'error',
            label: {
                show: true,
            },
            renderItem: (params, api) => {
                let location = api.coord([api.value(0), api.value(1) - 12]);
                return {
                    type: 'group',
                    children: [{
                        type: 'CubeLeft',
                        shape: {
                            api,
                            x: location[0],
                            y: location[1],
                            xAxisPoint: api.coord([api.value(0), 0])
                        },
                        style: { fill: api.style().fill, text: null }
                    }, {
                        type: 'CubeRight',
                        shape: {
                            api,
                            x: location[0],
                            y: location[1],
                            xAxisPoint: api.coord([api.value(0), 0])
                        },
                        style: { fill: api.style().fill, text: null }
                    }, {
                        type: 'CubeTop',
                        shape: {
                            api,
                            x: location[0],
                            y: location[1],
                            xAxisPoint: api.coord([api.value(0), 0])
                        },
                        style: api.style()
                    }]
                }
            },
            itemStyle: {
                color: (params) => {
                    // 使每根柱子颜色都不一样
                    let colorList = ['#33b56a', '#fdcf5c', '#4c90ff', '#fe7b7a', '#69ccf6', '#a38bf8', '#ff9561', '#8cb0ea', '#fe81b4', '#ffb258'];

                    return new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                        offset: 0,
                        color: "#6CEDFC"
                    }, {
                        offset: 1,
                        color: "#225458"
                    }], false)
                }
            },
            encode: {
                x: 0,
                y: [1, 2],
                label: 0
            },
            data: errorData,
            z: 100
        }
    ]
};

```


