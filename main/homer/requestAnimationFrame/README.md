# 9012年JavaScript还能写动画吗?
> JavaScript日益强大的能力使得我们有理由期待它可以去做更多的事情。现在的web页面已经跳脱了静态展示页面的范畴，我们不仅可以在页面上交互了，还可以在页面上编写出令用户眼花缭乱的展示效果，没错，我要说的就是动画。

## JavaScript动画本来的写法

人的眼睛看到一幅画或一个物体后，在0.34秒内不会消失。这叫做人类视觉暂留原理。利用这一原理，在一幅画还没有消失前播放下一幅画，就会给人造成一种流畅的视觉变化效果。

我们所要做的就是使用这个原理，在一定时间内，去一步步的操作dom的样式属性（比如颜色，大小，位置等），使得dom在页面上动起来。抽象一下，其实就是每隔多少时间去做一件什么样的事情。我们就非常容易想到JavaScript里的两个非常有用的全局函数
`setTimeout`和`setInterval`。这两个都是作为一个定时器的作用，`setTimeout`允许我们向浏览器约定一个时间，在过了这段时间之后，执行回调函数的内容。而`setInterval`则是跟浏览器规定，每隔多少时间后就要执行一次回调函数的内容。这两个函数都可以利用起来写动画效果。
例如：
```javascript
// 使用setTimeout编写动画
const finalLeftPostion = 200;
let dom = document.getElementById('animation');
function moveDom() {
  setTimeout(() => {
      dom.style.left = (parseInt(dom.style.left, 10) + 5) + 'px';
      if (parseInt(dom.style.left, 10) < finalLeftPostion) { // 判断是否达到动画停止的条件
          moveDom();
      }
  }, 16);
}
moveDom();

// 使用setInerval编写动画
function moveDom2() {
    const timer  = setInterval(() => {
        dom.style.left = (parseInt(dom.style.left, 10) + 5) + 'px';
        if (parseInt(dom.style.left, 10) > finalLeftPostion) { // 判断是否达到动画停止的条件
                  clearInterval(timer);
              }
    }, 16)
}
moveDom2();
```
使用`setTimeout`编写动画的思路就是，如果还没有达到动画结束条件，则会一直调用`setTimeout`来执行动画操作，否则不再调用`setTimeout`。而使用`setInterval`的编写动画的思路则为，调用`setInterval`来重复执行的动画操作，当达到动画结束条件时，则取消掉这个定时器。

乍一看这个两个实现思路完全没有问题，但是，你万万想不到的是，恰恰是`setTimeout`和`setInterval`这两个函数有着最大的问题。首先从浏览器的底层来说，浏览器的维护着一条task队列，它掌握着浏览器各种操作执行比如js代码的执行、UI渲染等的先后顺序。而`setTimeout`和`setInterval`这两个函数则会粗暴的向这条task队列插入新的task，一旦遇到需要构建复杂的动画场景的时候，对浏览器的性能会造成一定的影响。
退一步来说，`setTimeout`和`setInterval`这两个函数使用不当也会出现问题，比如如果声明了太多的定时器不去有计划地去清除的话会导致定时器越来越多，有可能导致内存泄漏（在定时器中使用过多闭包所致），或者导致浏览器卡死（定时器一直在执行导致UI没有空闲更新）等等。另外，`setInterval`执行的回调操作并不会关心上一次操作是否完成，如果回调操纵时间比间隔时间长的话，回调操作就会出现错乱。
综上，这其实并不是web前端编写动画的最好方法。

## css3 前端动画的救星？

JavaScript在进步，css也不例外。css3推出了很多激动人心的新特性，其中就有animation模块。CSS Animations 模块定义了如何用关键帧来随时间推移对CSS属性的值进行动画处理。关键帧动画的行为可以通过指定它们的持续时间，它们的重复次数以及它们如何重复来控制。
animation模块主要包含两方面的内容，第一个是animation系列属性，可以在css中声明动画所要执行规则、动画持续时间、动画执行的延迟等参数；第二个就是`@keyframe`的动画执行规则。css3动画有着非常多的优点，首先，相比于JavaScript需要通过获取dom的引用来操作样式，css直接使用浏览器渲染引擎来操作样式有着天然的优势；其次css3进行了相应的优化，升级可以使用GPU去渲染页面，大大减少了浏览器的渲染压力；再加上`@keyframe`规则可以详细的去一个动画周期规定各个阶段中要达到的状态，剩下过渡动画的过程、动画时间函数等复杂的东西就直接交给浏览器去搞定就可以了。
比如：
```css
.animation-div{
    animation: slidein 3s linear 1s infinite alternate;
}
@keyframes slidein  {
    0% {
        margin-left: -20%;
    }
    100% {
        margin-left: 100%;
    }
}
```

从上面的代码可以看到，相比于JavaScript动画代码，css3的代码看起更加清晰精简，需要计算的东西也更少了。很多特性简单的声明一下就了事。

那这就是前端动画的最终解决方案了吗？

那倒不一定。虽然css3编写足够简单，但是在有些时候却会抓襟见肘。由于css的不可编程，在编写动画上就少了一部分可能性和灵活性。而且css是静态的，单单靠css是无法检测到交互再条件性地执行动画。
这些虽然不是很大的缺点，但是也足以限制css3动画的在一些前端业务场景上的使用。

所以，有没有东西可以一下弥补css3的不足呢？

## `requestAnimationFrame`， JavaScript动画的最后武器

那么JavaScript是不是真的不用于前端动画编写呢？现在，拥有了`requestAnimationFrame`的JavaScript可以自豪的说，可以了。
`window.requestAnimationFrame()`是一个全局的函数，告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次重绘之前执行。
由于动画的更新时间的是交给浏览器的来决定，所以不会再有类似`setTimeout`和`setInterval`无限加入task队列的问题出现。而且只要传入回调函数，就可以充分利用JavaScript的编程能力。
除了JavaScript和dom无法避免的访问代价，`requestAnimationFrame`堪称完美。

接下来，我们去详细了解`requestAnimationFrame`。先来看一下代码示例：

```javascript
let dom = document.getElementById('Animate');
const finalLeftPostion = 200;
function animateDom() {
  dom.style.left = (parseInt(dom.style.left, 10) + 5) + 'px';
  if (parseInt(dom.style.left, 10) < finalLeftPostion) {
           requestAnimationFrame(animateDom);
  }
}
requestAnimationFrame(animateDom);
```
从上面代码中可以发现，`requestAnimationFrame`（后面简称rAF）的思路其实和`setTimeout`编写动画的写法其实一样的。只要还没达到动画停止的条件，就会不停的去调用动画函数，直到符合条件为止。
唯一不同的是，rAF把更新动画的时间交给了浏览器去处理，更确切的说，浏览器在下一次重绘之前会调用你传入给该方法的动画函数(即你的回调函数)。回调函数执行次数通常是每秒60次，但在大多数遵循W3C建议的浏览器中，回调函数执行次数通常与浏览器屏幕刷新次数相匹配。
为了提高性能和电池寿命，因此在大多数浏览器里，当rAF 运行在后台标签页或者隐藏的`<iframe> `里时，rAF 会被暂停调用以提升性能和电池寿命。

rAF接受一个回调函数作为参数，这个函数的内容就是我们动画操作主要执行内容，而函数会获得一个时间戳（DOMHighResTimeStamp，一个双精度浮点数的时间值，performance.now()的返回值相同）作为参数。
rAF函数本身会返回一个id值，这个值可以用来作为`cancelAnimationFrame()`的入参来取消对应的动画。

利用rAF的一些特性我们可以定制封装出很多动画的函数，比如我们需要做一个元素的旋转效果，但是这个效果的持续时间我们需要在调用动画的时候才能决定，换句话说就是持续时间是可定制的。
那我们就可以灵活运用rAF传入的时间戳参数，来达到这个效果。
示例(为了简单易懂将直接用函数来实现)： 
```javascript
/**
* @param dom 传入需要制作动画的dom元素
* @param duration 动画所要持续的时间
*/
function rotateDom(dom, duration) {
    let startTime = 0;
    function stepAnimation (timestamp) {
      if (!startTime) {
          startTime = timestamp;
      }
      dom.style.transform = 'rotate(' + parseInt(dom.style.transform.split('(')[1], 10) + 'deg)';
      if (timestamp - startTime <= duration) {
          requestAnimationFrame(stepAnimation);
      }
    }
    requestAnimationFrame(stepAnimation);
}
```
再比如我们需要在用户触发一些事件的时候把动画中断掉，然后在某些条件下又把中断掉的动画继续进行下去：
```javascript
/**
* @param dom 需要执行动画的dom元素
* @returns {{abortAnimation(): void, resumeAnimation(): void}}
*/
    function animationCanBeCanceled(dom) {
        let rAF = null;
        function stepAnimation() {
            dom.style.left = (parseInt(dom.style.left, 10) + 5) + 'px';
            if (parseInt(dom.style.left, 10) <= 900) {
                rAF = requestAnimationFrame(stepAnimation)
            }
        }
        rAF = requestAnimationFrame(stepAnimation);
        return {
            abortAnimation() {
                cancelAnimationFrame(rAF);
            },
            resumeAnimation() {
                requestAnimationFrame(stepAnimation);
            }
        }
    }
    const adom = document.getElementById('animation');
    const animateController = animationCanBeCanceled(adom);
    setTimeout(() => {
        animateController.abortAnimation();
        setTimeout(() => {
            animateController.resumeAnimation();
        }, 5000)
    }, 2000)
```
以上就是rAF的两个最主要的用法，关于rAF编写动画的更深入一层的写法，将在下一篇文章[《rAF的好伙伴补间算法TWEEN》](../TWEENjs/README.md)中进行详细描述。

## 结语

对于动画编写的这件事情上，看来JavaScript和css各有秋千，都有其擅长的应用场景。我们要根据实际业务场景去做符合情况的选择。最重要的一点就是，前端技术是没有银弹的，抛开应用场景谈优劣是不行的。


*Homer 2019.3.29*

*参考：*

[*Tasks, microtasks, queues and schedules*](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

[*CSS3 动画*](http://www.w3school.com.cn/css3/css3_animation.asp)

[*window.requestAnimationFrame*](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame)
