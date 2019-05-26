# 动态线性渐变的实现

![headimg](./img/headImg.png)

> 本文主要探究双色动态线性渐变的效果如何实现。

## 前端：静态的颜色渐变

渐变是在色彩上的一个相对缓慢的过度，我们的视觉会随着这个渐变的过度而产生一种流动感，而这种流动感全凭在色彩上发生的种种变化。

渐变主要分成两种主要形式，分别是线性渐变和径向渐变。
线性渐变值指的是是沿着一根轴线（水平或垂直）改变颜色，从起点到终点颜色进行顺序渐变（从一边拉向另一边）；径向渐变则是从起点到终点颜色从内到外进行圆形渐变。（从中间向外拉）。

在前端，我们也完全可以用代码实现这些色彩效果，而实现的方式有三种：css，canvas以及svg。
今天我们只讨论线性渐变的实现方法。接下来，先来看一下这三种方式的实现方法：

### css
我们首先来看用css怎么去实现一个简单的渐变效果：

```css
.linear-gradient{
    background-image:linear-gradient(0deg, #08AEEA 0%, #2AF598 100%);
}
```
效果是这样的：

![css-linear-bg](./img/cssLinearBg.png)

css主要使用两个函数`linear-gradient`(线性)和`radial-gradient`(径向)来创建颜色渐变，基本语法如下：

`linear-gradient([方向/角度], 颜色1 [位置/百分比], 颜色2 [位置/百分比], ...)`

生成的渐变图层会被视为css中的一个image(图片)对象，所以可以用在以下css属性上：
* `background-image`
* `list-style-image`
* `border-image`
* `cursor`
* 用 CSS `content`属性，和CSS伪元素 `::after` 和 `::before`替换元素内容
* `mask`(遮罩属性，webkit内核浏览器需要加`-webkit-`前缀)

下面是一个渐变用作边框的例子
```css
.linear-border{
            border: 30px solid ;
            border-image: linear-gradient(0deg, #08AEEA 0%, #2AF598 100%) 10;
}
```
效果如下：

![linear-border](./img/linearBorder.png)

(附：`border-image`的基本用法：[MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-image))

另外，只要css渐变的填充颜色不是完全不透明的话，渐变图层还可以进行多个叠加。

基本用法说到这里差不多了，更多的详细说明请看[MDN](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Using_CSS_gradients)上的说明.

接下来是canvas中渐变的实现方法：

```javascript
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
const linearGradient = ctx.createLinearGradient(canvas.width / 2 , 0, canvas.width / 2, canvas.height);
linearGradient.addColorStop(0, '#08AEEA');
linearGradient.addColorStop(1, '#2AF598');
ctx.fillStyle = linearGradient;
ctx.fillRect(0, 0, canvas.width, canvas.height);
```

效果如下：



