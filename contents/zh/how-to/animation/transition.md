# 过渡动画

Apache ECharts<sup>TM</sup> 中使用了平移，缩放，变形等形式的过渡动画让数据的添加更新删除，以及用户的交互变得更加顺滑。通常情况下开发者不需要操心该如何去使用动画，只需要按自己的需求使用`setOption`更新数据，ECharts 就会找出跟上一次数据之间的区别，然后自动应用最合适的过渡动画。

比如下面例子就是定时更新饼图数据（随机）的过渡动画效果。

```js live {layout: 'lr'}
function makeRandomData() {
  return [
    {
      value: Math.random(),
      name: 'A'
    },
    {
      value: Math.random(),
      name: 'B'
    },
    {
      value: Math.random(),
      name: 'C'
    }
  ];
}
option = {
  series: [
    {
      type: 'pie',
      radius: [0, '50%'],
      data: makeRandomData()
    }
  ]
};

setInterval(() => {
  myChart.setOption({
    series: {
      data: makeRandomData()
    }
  });
}, 2000);
```

## 过渡动画的配置

因为数据添加和数据更新往往会需要不一样的动画效果，比如我们会期望数据更新动画的时长更短，因此 ECharts 区分了这两者的动画配置：

- 对于新添加的数据，我们会应用入场动画，通过`animationDuration`, `animationEasing`, `animationDelay`三个配置项分别配置动画的时长，缓动以及延时。
- 对于数据更新，我们会应用更新动画，通过`animationDurationUpdate`, `animationEasingUpdate`, `animationDelayUpdate`三个配置项分别配置动画的时长，缓动以及延时。

可以看到，更新动画配置是入场动画配置加上了`Update`的后缀。

> 在 ECharts 中每次 setOption 的更新，数据会跟上一次更新的数据做对比，然后根据对比结果分别为数据执行三种状态：添加，更新以及移除。这个比对是根据数据的`name`来决定的，例如上一次更新数据有三个`name`为`'A'`, `'B'`, `'C'`的数据，而新更新的数据变为了`'B'`, `'C'`, `'D'`的数据，则数据`'B'`, `'C'`会被执行更新，数据`'A'`会被移除，而数据`'D'`会被添加。如果是第一次更新因为没有旧数据，所以所有数据都会被执行添加。根据这三种状态 ECharts 会分别应用相应的入场动画，更新动画以及移除动画。

所有这些配置都可以分别设置在`option`最顶层对所有系列和组件生效，也可以分别为每个系列配置。

如果我们想要关闭动画，可以直接设置`option.animation`为`false`。

### 动画时长

`animationDuration`和`animationDurationUpdate`用于设置动画的时长，单位为`ms`，设置较长的动画时长可以让用户更清晰的看到过渡动画的效果，但是我们也需要小心过长的时间会让用户再等待的过程中失去耐心。

设置为`0`会关闭动画，在我们只想要单独关闭入场动画或者更新动画的时候可以通过单独将相应的配置设置为`0`来实现。

### 动画缓动

`animationEasing`和`animationEasingUpdate`两个配置项用于设置动画的缓动函数，缓动函数是一个输入动画时间，输出动画进度的函数：

```ts
(t: number) => number;
```

在 ECharts 里内置了缓入`'cubicIn'`，缓出`'cubicOut'`等常见的动画缓动函数，我们可以直接通过名字来声明使用这些缓动函数。

内置缓动函数：

<md-example src="line-easing" width="100%" height="400" />

### 延时触发

`animationDelay`和`animationDelayUpdate`用于设置动画延迟开始的时间，通常我们会使用回调函数将不同数据设置不同的延时来实现交错动画的效果：

```ts live { layout: 'lr' }
var xAxisData = [];
var data1 = [];
var data2 = [];
for (var i = 0; i < 100; i++) {
  xAxisData.push('A' + i);
  data1.push((Math.sin(i / 5) * (i / 5 - 10) + i / 6) * 5);
  data2.push((Math.cos(i / 5) * (i / 5 - 10) + i / 6) * 5);
}
option = {
  legend: {
    data: ['bar', 'bar2']
  },
  xAxis: {
    data: xAxisData,
    splitLine: {
      show: false
    }
  },
  yAxis: {},
  series: [
    {
      name: 'bar',
      type: 'bar',
      data: data1,
      emphasis: {
        focus: 'series'
      },
      animationDelay: function(idx) {
        return idx * 10;
      }
    },
    {
      name: 'bar2',
      type: 'bar',
      data: data2,
      emphasis: {
        focus: 'series'
      },
      animationDelay: function(idx) {
        return idx * 10 + 100;
      }
    }
  ],
  animationEasing: 'elasticOut',
  animationDelayUpdate: function(idx) {
    return idx * 5;
  }
};
```

## 动画的性能优化

在数据量特别大的时候，为图形应用动画可能会导致应用的卡顿，这个时候我们可以设置`animation: false`关闭动画。

对于数据量会动态变化的图表，我们更推荐使用`animationThreshold`这个配置项，当画布中图形数量超过这个阈值的时候，ECharts 会自动关闭动画来提升绘制性能。这个配置往往是一个经验值，通常 ECharts 的性能足够实时渲染上千个图形的动画（我们默认值也是给了 2000），但是如果你的图表很复杂，或者你的用户环境比较恶劣，页面中又同时会运行很多其它复杂的代码，也可以适当的下调这个值保证整个应用的流畅性。

## 监听动画结束

有时候我们想要获取当前渲染的结果，如果没有使用动画，我们在`setOption`之后 ECharts 就会直接执行渲染，我们可以同步的通过`getDataURL`方法获取渲染得到的结果。

```ts
const chart = echarts.init(dom);
chart.setOption({
  animation: false
  //...
});
// 可以直接同步执行
const dataUrl = chart.getDataURL();
```

但是如果图表中有动画，马上执行`getDataURL`得到的是动画刚开始的画面，而非最终展示的结果。因此我们需要知道动画结束然后再执行`getDataURL`得到结果。

假如你确定动画的时长，一种比较简单粗暴的方式是根据动画时长来执行`setTimeout`延迟执行：

```ts
chart.setOption({
  animationDuration: 1000
  //...
});
setTimeout(() => {
  const dataUrl = chart.getDataURL();
}, 1000);
```

或者我们也可以使用 ECharts 提供的`rendered`事件来判断 ECharts 已经动画结束停止了渲染

```ts
chart.setOption({
  animationDuration: 1000
  //...
});

function onRendered() {
  const dataUrl = chart.getDataURL();
  // ...
  // 后续如果有交互，交互发生重绘也会触发该事件，因此使用完就需要移除
  chart.off('rendered', onRendered);
}
chart.on('rendered', onRendered);
```
