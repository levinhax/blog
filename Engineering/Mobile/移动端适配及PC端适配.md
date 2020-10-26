## 相关知识

### 像素

像素就是构成图像的最小单位,指显示屏上的最小单位, 图像由像素组成,单位面积内的像素越多 效果就越好 **像素的大小不是绝对的,是根据设备的分辨率决定的**

### 分辨率

屏幕分辨率 : 屏幕横向和纵向的像素点数,单位为px

相同大小的屏幕 分辨率越低,单位像素尺寸越大,分辨率越高,单位像素尺寸越小

图像分辨率 : 指图片含有的像素数 , 表示图片分别在垂直和水平上所具有的像素点数

同一尺寸的图片，分辨率越高，图片越清晰。

### PPI

每英寸包括的像素数

PPI可以用于描述屏幕的清晰度以及一张图片的质量,PPI越高，屏幕越清晰。

### 设备物理像素 (物理分辨率)

设备的真实分辨率 屏幕有多少个像素点 就是多少分辨率

以iphone6 为例 物理分辨率为750*1334,也就是显示屏内部led灯的个数

### 设备独立像素dips (逻辑分辨率)

一种单位来告诉不同分辨率的手机，它们在界面上显示元素的大小是多少 即设备几个像素当成一个像素使用
```
以iphone6为例, dips为375* 667 即是2*2个物理像素当做一个设备独立像素
```

```
各种设备，手机，平板，笔记本等逻辑像素

手机：逻辑像素在3xx-4xx（短边）之间

平板：10寸平板7xx-8xx（短边）

笔记本：13寸 1280 （长边）

24寸显示屏：1920（长边）
```

### 设备像素比dpr

物理像素与设备独立像素的比值

css中的像素只是一个抽象的单位，在不同的设备或不同的环境中，css中的1px所代表的设备物理像素是不同的,1px并不是绝对的,它只代表了当前设备像素的最小单位.

在pc端 1px等于一个设备的物理像素,但是移动设备的屏幕像素密度越来越高 ,iphone6上一个css像素是等于两个物理像素,通过dpr，我们可以知道该设备上一个css像素代表多少个物理像素

获取dpr : js:
```
window.devicePixelRatio
```

css:
```
-webkit-min-device-pixel-ratio
React Native: PixelRatio.get()
```

为什么iphone6的逻辑像素是375 ,设计稿为750宽?

这种设计图秤为2倍图,因为切图时获取的图片是页面的两倍大, 但是视网膜屏幕会把图片缩小一倍,这样会更加细腻 更清晰

### css像素

CSS像素，当页面缩放比例为100%时，一个CSS像素等于一个设备独立像素。

但是CSS像素是很容易被改变的，当用户对浏览器进行了放大，CSS像素会被放大，这时一个CSS像素会跨越更多的物理像素。

页面的缩放系数 = CSS像素 / 设备独立像素。

## 视口

### 布局视口layout viewport

布局视口:当前浏览器的可视区域,不包括菜单栏及浏览器的ui不包含滚动条等

在PC端上，设置viewport不生效,布局视口永远等于浏览器窗口的宽度。

在移动端上，由于要使为PC端浏览器设计的网站能够完全显示在移动端的小屏幕里 布局视口默认是980px,会出现滚动条

*但是在iphone和ipad上没有指定初始的缩放值的话，那么iphone和ipad会自动计算initial-scale这个值，以保证当前layout viewport的宽度在缩放后就是浏览器可视区域的宽度*

```
//获取布局视口宽度
document.documentElement.clientWidth / Height
```

### 视觉视口visual viewport

视觉视口:用户当前看到的区域,包含滚动条等.默认等于当前浏览器的窗口大小

用户通过缩放放大网站，CSS像素增大,一个CSS像素可以跨越更多的设备像素,我们能看到的网站区域将缩小，此时视觉视口变小

假设屏幕上本来需要200个CSS像素才能占满屏幕，由于放大，现在只需要100个CSS像素就能占满，所以视觉视口的宽就变成100px。

```
//获取视觉窗口宽度
window.innerWidth
```

### 理想视口 ideal viewport

布局视口的一个理想尺寸，只有当布局视口的尺寸等于设备屏幕的尺寸时，才是理想视口。

ideal viewport 的意义在于，无论在何种分辨率的屏幕下，那些针对ideal viewport 而设计的网站，不需要用户手动缩放，也不需要出现横向滚动条，都可以完美的呈现给用户。
比如一段14px大小的文字，不会因为在一个高密度像素的屏幕里显示得太小而无法看清，理想的情况是这段14px的文字无论是在何种密度屏幕，何种分辨率下，显示出来的大小都是差不多的。

```
//获取理想视口宽度
window.screen.width
```

### 移动端如何得到理想视口

```
<meta name="viewport" content="width=device-width; initial-scale=1; maximum-scale=1; minimum-scale=1; user-scalable=no;">
```

该meta标签的作用是让当前布局视口的宽度等于设备的宽度，同时不允许用户手动缩放

视觉视口等于理想视口这时，1个CSS像素就等于1个设备独立像素，而且我们也是基于理想视口来进行布局的，所以呈现出来的页面布局在各种设备上都能大致相似。

*要把当前的viewport宽度设为ideal viewport的宽度，既可以设置 width=device-width，也可以设置 initial-scale=1，但这两者各有一个小缺陷，就是iphone、ipad以及IE 会横竖屏不分，通通以竖屏的ideal viewport宽度为准。所以，最完美的写法应该两者都写上去，这样就 initial-scale=1 解决了 iphone、ipad的毛病，width=device-width则解决了IE的毛病*

页面的缩放系数 = 理想视口宽度 / 布局视口宽度

如果meta同时设置了 initial-scale和width,布局视口 讲取两者间的最大值

### 常见获取窗口大小

- window.innerHeight：获取浏览器视觉视口高度（包括垂直滚动条）。
- window.outerHeight：获取浏览器窗口外部的高度。表示整个浏览器窗口的高度，包括侧边栏、窗口镶边和调正窗口大小的边框。
- window.screen.Height：获取获屏幕取理想视口高度，这个数值是固定的，设备的分辨率/设备像素比
- window.screen.availHeight：浏览器窗口可用的高度。
- document.documentElement.clientHeight：获取浏览器布局视口高度，包括内边距，但不包括垂直滚动条、边框和外边距。
- document.documentElement.offsetHeight：包括内边距、滚动条、边框和外边距。
- document.documentElement.scrollHeight：在不使用滚动条的情况下适合视口中的所有内容所需的最小宽度。测量方式与clientHeight相同：它包含元素的内边距，但不包括边框，外边距或垂直滚动条。

## 常见的布局方案

### 固定布局

**以像素作为页面的基本单位**，不管设备屏幕及浏览器宽度，只设计一套尺寸； 可切换的固定布局：同样以像素作为页面单位，参考主流设备尺寸，设计几套不同宽度的布局。通过识别的屏幕尺寸或浏览器宽度，选择最合适的那套宽度布局；

### 弹性布局

**以百分比作为页面的基本单位**，可以适应一定范围内所有尺寸的设备屏幕及浏览器宽度

## web适配产生的问题

### 浏览器mate指定内核

识别为chrome内核：doctype 为标准 + meta 标签元素

识别为IE内核：doctype 为非标准 + meta 元素检测

### 1px边框问题

设计图的1px和实现出来的不一样的原因：

按照iPhone6的尺寸,一张750px宽的设计图，这个750px其实就是iPhone6的设备像素，在测量设计图时量到的1px其实是1设备像素，而当我们设置布局视口等于理想视口等于375px，并且由于iPhone6的DPR为2，写css时的1px对应的是2设备像素，所以看起来会粗一点。

解决方案：

1. border-image

基于media查询判断不同的设备像素比给定不同的border-image：
```
.border_1px{
    border-bottom: 1px solid #000;
}

@media only screen and (-webkit-min-device-pixel-ratio:2) {
    .border_1px{
        border-bottom: none;
        border-width: 0 0 1px 0;
        border-image: url(../img/1pxline.png) 0 0 2 0 stretch;
    }
}
```

2. background-image

和border-image类似，准备一张符合条件的边框背景图，模拟在背景上
```
.border_1px{
    border-bottom: 1px solid #000;
}

@media only screen and (-webkit-min-device-pixel-ratio:2) {
    .border_1px{
        background: url(../img/1pxline.png) repeat-x left bottom;
        background-size: 100% 1px;
    }
}
```
上面两种都需要单独准备图片，而且圆角不是很好处理，但是可以应对大部分场景。

3. 伪类 + transform

基于media查询判断不同的设备像素比对线条进行缩放：
```
.border_1px:before {
    content: '';
    position: absolute;
    top: 0;
    height: 1px;
    width: 100%;
    background-color: #000;
    transform-origin: 50% 0%;
}

@media only screen and (-webkit-min-device-pixel-ratio:2) {
    .border_1px:before{
        transform: scaleY(0.5);
    }
}

@media only screen and (-webkit-min-device-pixel-ratio:3){
    .border_1px:before{
        transform: scaleY(0.33);
    }
}
```
这种方式可以满足各种场景，如果需要满足圆角，只需要给伪类也加上border-radius即可。

4. 使用svg(插件帮助postcss-write-svg)

借助PostCSS的postcss-write-svg我们能直接使用border-image和background-image创建svg的1px边框
```
@svg border_1px {
    height: 2px;

    @rect {
        fill: var(--color, black);
        width: 100%;
        height: 50%;
    }
}

.example {
    border: 1px solid transparent;
    border-image: svg(border_1px param(--color #00b1ff)) 2 2 stretch;
}
```

编译后：
```
.example { border: 1px solid transparent; border-image: url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg' height='2px'%3E%3Crect fill='%2300b1ff' width='100%25' height='50%25'/%3E%3C/svg%3E") 2 2 stretch; }
```

### 适配iPhoneX

我们需要将顶部和底部合理的摆放在安全区域内，iOS11新增了两个CSS函数env、constant，用于设定安全区域与边界的距离。

函数内部可以是四个常量：

- safe-area-inset-left：安全区域距离左边边界距离
- safe-area-inset-right：安全区域距离右边边界距离
- safe-area-inset-top：安全区域距离顶部边界距离
- safe-area-inset-bottom：安全区域距离底部边界距离

*我们必须指定viweport-fit后才能使用这两个函数*

```
<meta name="viewport" content="width=device-width, viewport-fit=cover">
```

constant在iOS < 11.2的版本中生效，env在iOS >= 11.2的版本中生效，这意味着我们往往要同时设置他们，将页面限制在安全区域内：
```
body {
    padding-bottom: constant(safe-area-inset-bottom);
    padding-bottom: env(safe-area-inset-bottom);
}
```

当使用底部固定导航栏时，我们要为他们设置padding值：
```
{
    padding-bottom: constant(safe-area-inset-bottom);
    padding-bottom: env(safe-area-inset-bottom);
}
```

### 关于横屏

js识别
```
window.addEventListener("resize", ()=>{
    if (window.orientation === 180 || window.orientation === 0) { 
      // 正常方向或屏幕旋转180度
        console.log('竖屏');
    };
    if (window.orientation === 90 || window.orientation === -90 ){ 
       // 屏幕顺时钟旋转90度或屏幕逆时针旋转90度
        console.log('横屏');
    }  
});
```

css识别
```
@media screen and (orientation: portrait) {
  /*竖屏...*/
}

@media screen and (orientation: landscape) {
  /*横屏...*/
}
```

### 图片模糊问题

产生原因：我们平时使用的图片大多数都属于位图（png、jpg..），位图由一个个像素点构成的，每个像素都具有特定的位置和颜色值，理论上，位图的每个像素对应在屏幕上使用一个物理像素来渲染，才能达到最佳的显示效果。

而在dpr > 1的屏幕上，位图的一个像素可能由多个物理像素来渲染，然而这些物理像素点并不能被准确的分配上对应位图像素的颜色，只能取近似值，所以相同的图片在dpr > 1的屏幕上就会模糊。

解决方案：

为了保证图片质量，我们应该尽可能让一个屏幕像素来渲染一个图片像素，所以，针对不同DPR的屏幕，我们需要展示不同分辨率的图片。

如：在dpr=2的屏幕上展示两倍图(@2x)，在dpr=3的屏幕上展示三倍图(@3x)。

1. media查询

使用media查询判断不同的设备像素比来显示不同精度的图片
```
.avatar{
    background-image: url(conardLi_1x.png);
}
@media only screen and (-webkit-min-device-pixel-ratio:2){
    .avatar {
        background-image: url(conardLi_2x.png);
    }
}

@media only screen and (-webkit-min-device-pixel-ratio:3){
    .avatar{
        background-image: url(conardLi_3x.png);
    }
}
```
只适用于背景图

2. image-set

```
.avatar {
    background-image: -webkit-image-set( "conardLi_1x.png" 1x, "conardLi_2x.png" 2x );
}
```
只适用于背景图

3. srcset

使用img标签的srcset属性，浏览器会自动根据像素密度匹配最佳显示图片
```
<img src="conardLi_1x.png"
     srcset="conardLi_2x.png 2x, conardLi_3x.png 3x">
```

4. JavaScript拼接图片url

使用window.devicePixelRatio获取设备像素比，遍历所有图片，替换图片地址：
```
const dpr = window.devicePixelRatio;
const images =  document.querySelectorAll('img');
images.forEach((img)=>{
  img.src.replace(".", `@${dpr}x.`);
})
```

5. 使用svg

SVG的全称是可缩放矢量图,不同于位图的基于像素，SVG 则是属于对图像的形状描述，所以它本质上是文本文件，体积较小，且不管放大多少倍都不会失真。
```
<img src="conardLi.svg">
<img src="data:image/svg+xml;base64,[data]">
.avatar {
    background: url(conardLi.svg);
}
```

## h5相关

H5 技术本身是用于移动端的 web 网页。由于 App 本身有个 webview 的容器，在容器里可以运行 Web 前端相关代码，由此 H5 和原生 App 结合又衍生出来了 Hybrid App 技术。

### 移动端响应式布局

#### 方案1: rem + pxToRem

1. 监听屏幕视窗的宽度，通过一定比例换算赋值给html的font-size。此时，根字体大小就会随屏幕宽度而变化。
2. 将 px 转换成 rem, 常规方案有两种，一种是利用sass/less中的自定义函数 pxToRem，写px时，利用pxToRem函数转换成 rem。另外一种是直接写px，编译过程利用插件全部转成rem。这样 dom 中元素的大小，就会随屏幕宽度变化而变化了。

```
const MAX_FONT_SIZE = 420
// 定义最大的屏幕宽度
document.addEventListener('DOMContentLoaded', () => {
  const html = document.querySelector('html')
  let fontSize = window.innerWidth / 10
  fontSize = fontSize > MAX_FONT_SIZE ? MAX_FONT_SIZE : fontSize
  html.style.fontSize = fontSize + 'px'
})
```

#### 方案2: px 转 rem

pxToRem(方法1)
```
$rootFontSize: 375 / 10;
// 定义 px 转化为 rem 的函数
@function px2rem ($px) {
    @return $px / $rootFontSize + rem;
}
.demo {
    width: px2rem(100);
    height: px2rem(100);
}
```

pxToRem(方法2)
在vue-cli3 中装 postcss-pxtorem 插件就可以了，其他平台也是大致差不多的思路。
```
const autoprefixer = require('autoprefixer')
const pxtorem = require('postcss-pxtorem')
module.exports = { 
  // ...
  css: {
    sourceMap: true,
    loaderOptions: {
      postcss: {
        plugins: [
          autoprefixer(),
          pxtorem({
            rootValue: 37.5,
            propList: ['*']
          })
        ]
      }
    }
  }
}
```

实现原理：
```
function createPxReplace (rootValue, unitPrecision, minPixelValue) {
    return function (m, $1) {
        if (!$1) return m;
        var pixels = parseFloat($1);
        if (pixels < minPixelValue) return m;
        var fixedVal = toFixed((pixels / rootValue), unitPrecision);
        return (fixedVal === 0) ? '0' : fixedVal + 'rem';
    };
}
```

#### vw + vh

与 rem 类似做法，直接使用 postcss-px-to-viewport 插件进行配置, 配置方式也是和 postcss-pxtorem 大同小异。

**推荐flex 与 rem 结合使用更佳**

[vue移动端模板](https://github.com/suoyuesmile/vue-h5-awsome)

## h5遇到的问题汇总

### ios滑动不流畅

表现: 上下滑动页面会产生卡顿，手指离开页面，页面立即停止运动。整体表现就是滑动不流畅，没有滑动惯性。

产生原因: 为什么 iOS 的 webview 中 滑动不流畅，它是如何定义的？

原来在 iOS 5.0 以及之后的版本，滑动有定义有两个值 auto 和 touch，默认值为 auto。
```
-webkit-overflow-scrolling: touch; /* 当手指从触摸屏上移开，会保持一段时间的滚动 */
-webkit-overflow-scrolling: auto; /* 当手指从触摸屏上移开，滚动会立即停止 */
```

解决方案

1. 在滚动容器上增加滚动 touch 方法
将-webkit-overflow-scrolling 值设置为 touch
```
.wrapper {
    -webkit-overflow-scrolling: touch;
}
```
设置滚动条隐藏： .container ::-webkit-scrollbar {display: none;}

可能会导致使用position:fixed; 固定定位的元素，随着页面一起滚动

2. 设置 overflow

设置外部 overflow 为 hidden,设置内容元素 overflow 为 auto。内部元素超出 body 即产生滚动，超出的部分 body 隐藏。
```
body {
    overflow-y: hidden;
}
.wrapper {
    overflow-y: auto;
}
```
两者结合使用更佳！

### iOS 上拉边界下拉出现白色空白

表现：手指按住屏幕下拉，屏幕顶部会多出一块白色区域。手指按住屏幕上拉，底部多出一块白色区域。

产生原因：在 iOS 中，手指按住屏幕上下拖动，会触发 touchmove 事件。这个事件触发的对象是整个 webview 容器，容器自然会被拖动，剩下的部分会成空白。

解决方案

1. 监听事件禁止滑动

移动端触摸事件有三个，分别定义为

- touchstart ：手指放在一个DOM元素上。
- touchmove ：手指拖曳一个DOM元素。
- touchend ：手指从一个DOM元素上移开。

显然我们需要控制的是 touchmove 事件。

touchmove 事件的速度是可以实现定义的，取决于硬件性能和其他实现细节。

preventDefault 方法，阻止同一触点上所有默认行为，比如滚动。

我们可以通过监听 touchmove，让需要滑动的地方滑动，不需要滑动的地方禁止滑动。
*值得注意的是我们要过滤掉具有滚动容器的元素。*

```
document.body.addEventListener('touchmove', function(e) {
    if(e._isScroller) return;
    // 阻止默认事件
    e.preventDefault();
}, {
    passive: false
});
```

2. 滚动妥协填充空白，装饰成其他功能

在很多时候，我们可以不去解决这个问题，换一直思路。根据场景，我们可以将下拉作为一个功能性的操作。

比如： 下拉后刷新页面

3. 页面放大或缩小不确定性行为

表现：双击或者双指张开手指页面元素，页面会放大或缩小。

产生原因：HTML 本身会产生放大或缩小的行为，比如在 PC 浏览器上，可以自由控制页面的放大缩小。但是在移动端，我们是不需要这个行为的。所以，我们需要禁止该不确定性行为，来提升用户体验。

原理与解决方案

HTML meta 元标签标准中有个 中 viewport 属性，用来控制页面的缩放，一般用于移动端。可以设置 maximum-scale、minimum-scale 与 user-scalable=no 用来避免这个问题
```
<meta name=viewport
  content="width=device-width, initial-scale=1.0, minimum-scale=1.0 maximum-scale=1.0, user-scalable=no">
```

阻止页面放大(meta不起作用时)
```
  window.addEventListener(
    "touchmove",
    function (event) {
      if (event.scale !== 1) {
        event.preventDefault();
      }
    },
    { passive: false }
  );
```

### click 点击事件延时与穿透

表现: 监听元素 click 事件，点击元素触发时间延迟约 300ms; 点击蒙层，蒙层消失后，下层元素点击触发。

产生原因: 为什么会产生 click 延时？

iOS 中的 safari，为了实现双击缩放操作，在单击 300ms 之后，如果未进行第二次点击，则执行 click 单击操作。也就是说来判断用户行为是否为双击产生的。但是，在 App 中，无论是否需要双击缩放这种行为，click 单击都会产生 300ms 延迟。

为什么会产生 click 点击穿透？

双层元素叠加时，在上层元素上绑定 touch 事件，下层元素绑定 click 事件。由于 click 发生在 touch 之后，点击上层元素，元素消失，下层元素会触发 click 事件，由此产生了点击穿透的效果。

原理与解决方案：

方案一：使用 touchstart 替换 click

将 click 替换成 touchstart 不仅解决了 click 事件都延时问题，还解决了穿透问题。因为穿透问题是在 touch 和 click 混用时产生。
```
el.addEventListener("touchstart", () => { console.log("ok"); }, false);


<button @touchstart="handleTouchstart()">点击</button>
```

方案二： 使用 fastclick 库

```
import FastClick from 'fastclick';
FastClick.attach(document.body, options);
```

同样，使用fastclick库后，click 延时和穿透问题都没了

### 软键盘将页面顶起来、收起未回落问题

表现: Android 手机中，点击 input 框时，键盘弹出，将页面顶起来，导致页面样式错乱; 移开焦点时，键盘收起，键盘区域空白，未回落。

产生原因:
我们在app 布局中会有个固定的底部。安卓一些版本中，输入弹窗出来，会将解压 absolute 和 fixed 定位的元素。导致可视区域变小，布局错乱。

原理与解决方案

方案一：监听input失焦事件,失焦即回落
```
/iphone|ipod|ipad/i.test(navigator.appVersion) &&
  document.addEventListener(
    "blur",
    (event) => {
      // 当页面没出现滚动条时才执行，因为有滚动条时，不会出现这问题
      // input textarea 标签才执行，因为 a 等标签也会触发 blur 事件
      if (
        document.documentElement.offsetHeight <= document.documentElement.clientHeight &&
        ["input", "textarea"].includes(event.target.localName)
      ) {
        document.body.scrollIntoView(); // 回顶部
      }
    },
    true
  );
```

方案二：软键盘将页面顶起来的解决方案，主要是通过监听页面高度变化，强制恢复成弹出前的高度。
```
// 记录原有的视口高度
const originalHeight = document.body.clientHeight || document.documentElement.clientHeight;
window.onresize = function(){
  var resizeHeight = document.documentElement.clientHeight || document.body.clientHeight;
  if(resizeHeight < originalHeight ){
    // 恢复内容区域高度
    // const container = document.getElementById("container")
    // 例如 container.style.height = originalHeight;
  }
}
```

键盘不能回落问题出现在 iOS 12+ 和 wechat 6.7.4+ 中，而在微信 H5 开发中是比较常见的 Bug。

兼容原理，1.判断版本类型 2.更改滚动的可视区域
```
const isWechat = window.navigator.userAgent.match(/MicroMessenger\/([\d\.]+)/i);
if (!isWechat) return;
const wechatVersion = wechatInfo[1];
const version = (navigator.appVersion).match(/OS (\d+)_(\d+)_?(\d+)?/);

 // 如果设备类型为iOS 12+ 和wechat 6.7.4+，恢复成原来的视口
if (+wechatVersion.replace(/\./g, '') >= 674 && +version[1] >= 12) {
  window.scrollTo(0, Math.max(document.body.clientHeight, document.documentElement.clientHeight));
}
```
window.scrollTo(x-coord, y-coord)，其中window.scrollTo(0, clientHeight)恢复成原来的视口

### iPhone X系列安全区域适配问题

表现：头部刘海两侧区域或者底部区域，出现刘海遮挡文字，或者呈现黑底或白底空白区域。

产生原因：iPhone X 以及它以上的系列，都采用刘海屏设计和全面屏手势。头部、底部、侧边都需要做特殊处理。才能适配 iPhone X 的特殊情况。

解决方案：

设置安全区域，填充危险区域，危险区域不做操作和内容展示。

viewport-fit ,meta 标签设置为 cover，获取所有区域填充。 判断设备是否属于 iPhone X，给头部底部增加适配层
```
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes, viewport-fit=cover">
```

增加适配层

使用 safe area inset 变量
```
/* 适配 iPhone X 顶部填充*/
@supports (top: env(safe-area-inset-top)){
  body,
  .header{
      padding-top: constant(safe-area-inset-top, 40px);
      padding-top: env(safe-area-inset-top, 40px);
      padding-top: var(safe-area-inset-top, 40px);
  }
}
/* 判断iPhoneX 将 footer 的 padding-bottom 填充到最底部 */
@supports (bottom: env(safe-area-inset-bottom)){
    body,
    .footer{
        padding-bottom: constant(safe-area-inset-bottom, 20px);
        padding-bottom: env(safe-area-inset-bottom, 20px);
        padding-top: var(safe-area-inset-bottom, 20px);
    }
}
```

### 页面生成为图片和二维码问题

表现：在工作中有需要将页面生成图片或者二维码的需求。可能我们第一想到的，交给后端来生成更简单。但是这样我们需要把页面代码全部传给后端，网络性能消耗太大。

解决方案：

生产二维码

使用 QRCode 生成二维码
```
import QRCode from 'qrcode';
// 使用 async 生成图片
const options = {};
const url = window.location.href;
async url => {
  try {
    console.log(await QRCode.toDataURL(url, options))
  } catch (err) {
    console.error(err);
  }
}
```
将 await QRCode.toDataURL(url, options) 赋值给 图片 url 即可

生成图片

主要是使用 htmlToCanvas 生成 canvas 画布
```
import html2canvas from 'html2canvas';
html2canvas(document.body).then(function(canvas) {
    document.body.appendChild(canvas);
});
```

由于是 canvas 的原因。移动端生成出来的图片比较模糊，我们使用一个新的 canvas 方法多倍生成，放入一倍容器里面，达到更加清晰的效果，通过超链接下载图片 
```
const scaleSize = 2;
const newCanvas = document.createElement("canvas");
const target = document.querySelector('div');
const width = parseInt(window.getComputedStyle(target).width);
const height = parseInt(window.getComputedStyle(target).height);
newCanvas.width = width * scaleSize;
newCanvas.height = widthh * scaleSize;
newCanvas.style.width = width + "px";
newCanvas.style.height =width + "px";
const context = newCanvas.getContext("2d");
context.scale(scaleSize, scaleSize);
html2canvas(document.querySelector('.demo'), { canvas: newCanvas }).then(function(canvas) {
  // 简单的通过超链接设置下载功能
  document.querySelector(".btn").setAttribute('href', canvas.toDataURL());
}
```
根据需要设置 scaleSize 大小

### 微信公众号分享问题

表现：在微信公众号 H5 开发中，页面内部点击分享按钮调用 SDK，方法不生效。

解决方案

添加一层蒙层，做分享引导。

因为页面内部点击分享按钮无法直接调用，而分享功能需要点击右上角更多来操作。

### H5 调用 SDK 相关解决方案

原生与 H5 的通信：使用 DSBridge 同时支持 iOS 与 Android

SDK小组 提供方法

1.注册方法 bridge.register
```
bridge.register('enterApp', function() {
  broadcast.emit('ENTER_APP')
})
```

2.回调方法bridge.call
```
export const getSDKVersion = () => bridge.call('BLT.getSDKVersion')
```

事件监听与触发法
```
const broadcast = {
  on: function(name, fn, pluralable) {
    this._on(name, fn, pluralable, false)
  },
  once: function(name, fn, pluralable) {
    this._on(name, fn, pluralable, true)
  },
  _on: function(name, fn, pluralable, once) {
    let eventData = broadcast.data
    let fnObj = { fn: fn, once: once }
    if (pluralable && Object.prototype.hasOwnProperty.call(eventData, 'name')) {
      eventData[name].push(fnObj)
    } else {
      eventData[name] = [fnObj]
    }
    return this
  },
  emit: function(name, data, thisArg) {
    let fn, fnList, i, len
    thisArg = thisArg || null
    fnList = broadcast.data[name] || []
    for (i = 0, len = fnList.length; i < len; i++) {
      fn = fnList[i].fn
      fn.apply(thisArg, [data, name])
      if (fnList[i].once) {
        fnList.splice(i, 1)
        i--
        len--
      }
    }
    return this
  },
  data: {}
}
export default broadcast
```

一定要判断 SDK 是否提供该方法 如果 Android 提供该方法，iOS 上调用就会出现一个方法调用失败等弹窗。 怎么解决呢？
提供一个判断是否 Android、iOS。根据设备进行判断
```
export const hasNativeMethod = (name) =>
  return bridge.hasNativeMethod('BYJ.' + name)
}

export const getSDKVersion = function() {
  if (hasNativeMethod('getSDKVersion')) {
    bridge.call('BYJ.getSDKVersion')
  }
}
```

### H5 调试相关方案策略

1. vconsole 控制台插件
2. 代理 + spy-debugger

## 原文地址

https://juejin.im/post/6884042902587047943
