# 1 引言 
#### 原作：  Mr_兔子先生

分享一下如何仅用CSS3，实现单标签的动态晴阴雨雪。技术关键点就是“单标签”和“纯CSS”。先看下最终效果：

![](https://user-gold-cdn.xitu.io/2019/7/11/16be0b134bc8088f?imageslim)

再看看HTML代码：

```html
<!--晴-->
<div class="weather sunny"></div>
<!--阴-->
<div class="weather cloudy"></div>
<!--雨-->
<div class="weather rainy"></div>
<!--雪-->
<div class="weather snowy"></div>
```
  
没错，就是这么任性，每个动图就一个标签，而且无图无JS！下面就来详细介绍下技术实现。

涉及到的关键CSS3属性：
> 1.  transform：用于移位、旋转、缩放效果
> 2.  box-shadow：利用投影实现图像的复制(关键！)
> 3.  clip-path：基于绘制的形状对元素进行遮罩处理
> 4.  animation：设置元素的动画

以及实现单标签最关键的:before、:after伪元素运用。

通过这次分享，最大的能学习到的一点就是：box-shadow的另类玩法——“影分身”。

下面开始逐个讲解。

# 2 基础背景

图中的蓝块背景区域，很基础了，不用讲了。
设置了区域的宽高、背景色和圆角效果。
```css
.weather {
    position: relative;
    display: inline-block;
    width: 180px;
    height: 240px;
    background: #23b7e5;
    border-radius: 8px;
}
```

# 3 晴天

晴天图标由两个元素组成：太阳和内六角形阳光。
:before、:after 两个伪元素可以在元素内部分别“添加”一个元素，正好都利用上了。

### 3.1 绘制太阳
首先，用 :before实现太阳
```css
.sunny:before {
    content: "";
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 60px;
    height: 60px;
    background: #F6D963;
    border-radius: 50%;
    box-shadow: 0 0 20px #ff0;
    z-index: 2;
}
```
content用来生成一个元素。
position、top、left、transform用来实现中心居中。
box-shadow实现外发光效果，这只是box-shadow最基本最常用的使用方式。

![](https://user-gold-cdn.xitu.io/2019/7/11/16be0beb391e9845?imageslim)

### 3.2 绘制内六角形
用 :after实现内六角形。
实现的关键就是使用遮罩。通过clip-path绘制一个内六角形。这就变成了一个简单的初中几何问题。
内六角形由两个等边三角形拼合而成。

![](https://user-gold-cdn.xitu.io/2019/7/11/16be0b2bc5ffeba6?imageslim)

合并之后，我们可以把整体划分为若干个完全相同的小等边三角形。

![](https://user-gold-cdn.xitu.io/2019/7/11/16be0b2f912a9050?imageslim)

在垂直方向做个辅助线，连接中间顶部和底部两点。不难发现，“垂直方向的最大长度”要大于“水平方向的最大长度”。

![](https://user-gold-cdn.xitu.io/2019/7/11/16be0b363c3390c4?imageslim)

设小等边三角形的边长为1，以内六角形中心为坐标原点，可以计算出每个点的坐标，如下：

![](https://user-gold-cdn.xitu.io/2019/7/11/16be0b3a7569a523?imageslim)

为了使用clip-path的百分比定位来绘制图像，下一步需要把长度坐标转换为百分比坐标。

设垂直方向最大长度为100%，仍以内六角形中心为坐标原点，每个点的坐标值转换如下：

![](https://user-gold-cdn.xitu.io/2019/7/11/16be0b3e77810f7b?imageslim)

由于clip-path绘制原点是在左上角，x轴右侧为正值，y轴下方为正值。需要做下坐标系转换。即：

>新x轴坐标值 = 旧x轴坐标值 + 50%  
>新y轴坐标值 = (旧y轴坐标值 - 50%) * -1

![](https://user-gold-cdn.xitu.io/2019/7/12/16be6b9aed72962d?imageslim)

使用clip-path的polygon方法绘制内六角形，坐标已通过上面的步骤计算出来了。

样式代码如下：
```css
.sunny:after {
    content: "";
    position: absolute;
    top: 50%;
    left: 50%;
    margin: -45px 0 0 -45px; 
    width: 90px;
    height: 90px;
    background: #FFEB3B;
    clip-path: polygon(
    	50% 0%,
        65.43% 25%,
        93.3% 25%,
        78.87% 50%,
        93.3% 75%,
        64.43% 75%,
        50% 100%,
        35.57% 75%,
        6.7% 75%,
        21.13% 50%,
        6.7% 25%,
        35.57% 25%);
    z-index: 1;
    animation: sunScale 2s linear infinite;
}
@keyframes sunScale {
    0% {
        transform: scale(1);
    }
    50% {
    	transform: scale(1.1);
    }
    100% {
        transform: scale(1);
    }
}
```

>※注：safari需要将clip-path改为-webkit-clip-path。由于代码太占篇幅，这里就不重复写两遍了。

![](https://user-gold-cdn.xitu.io/2019/7/11/16be0be3cd63fbae?imageslim)

实现原理就是通过clip-path绘制了一个内六角形遮罩，把黄颜色背景通过遮罩变成了最终的内六角形。

animation通过关键帧动画实现了“放大缩小”交替动效。

最终效果：

![](https://user-gold-cdn.xitu.io/2019/7/11/16be0ba7e0967074?imageslim)

# 4 阴天

观察图形发现，有两个云朵：前面的白云和后面的乌云。貌似需要分别用 :before和 :after实现。如果这样做的话，后续章节的雨天和雪天的雨雪元素就没有多余的伪元素可用了。所以只能用一个伪元素实现两朵云。 这里就用到了box-shadow的“影分身”了！

由于后续章节的雨天和雪天都复用了云的样式，所以写在一起了，代码如下：
```css
.cloudy:before,
.rainy:before,
.snowy:before {
    content: "";
    position: absolute;
    top: 50%;
    left: 25%;
    transform: translate(-50%, -50%);
    width: 36px;
    height: 36px;
    background: #fff;
    border-radius: 50%;
    z-index: 2;
}
```

![](https://user-gold-cdn.xitu.io/2019/7/11/16be0c4f3bc5ddf8?imageslim)

真实的元素（真身）就是一个圆。通过box-shodow来把投影作为“分身”。

先来看看box-shadow的属性：

>box-shadow: h-shadow v-shadow blur spread color inset;  
参数详解：  
h-shadow: 阴影的水平偏移量。  
v-shadow: 阴影的垂直偏移量。  
blur: 模糊距离(就是渐变的距离，设为0就没有渐变)。  
spread: 投影的尺寸，通过这个控制“影分身”的大小。  
color: 投影颜色，通过这个实现后方的乌云。  
inset: 改为内阴影。这里用不到。

先复制一个影分身试试：

```css
box-shadow: #fff 22px -15px 0 6px;
```
![](https://user-gold-cdn.xitu.io/2019/7/11/16be0c575fb98168?imageslim)

继续复制多个影分身，带全部影分身的完整代码如下：
```css
.cloudy:before,
.rainy:before,
.snowy:before {
    content: "";
    position: absolute;
    top: 50%;
    left: 25%;
    transform: translate(-50%, -50%);
    width: 36px;
    height: 36px;
    background: #fff;
    border-radius: 50%;
    box-shadow: 
    	#fff 22px -15px 0 6px,
    	#fff 57px -6px 0 2px, 
    	#fff 87px 4px 0 -4px,
    	#fff 33px 6px 0 6px,
    	#fff 61px 6px 0 2px,
    	#ccc 29px -23px 0 6px,
    	#ccc 64px -14px 0 2px,
    	#ccc 94px -4px 0 -4px;
    z-index: 2;
}
```
五个分身的白圆（#fff）,三个分身的灰圆（#ccc）拼成了两朵云。

再给云朵加上“上下浮动”的动效：
```css
.cloudy:before {
    animation: cloudMove 2s linear infinite;
}
@keyframes cloudMove {
    0% {
        transform: translate(-50%, -50%);
    }
    50% {
        transform: translate(-50%, -60%);
    }
    100% {
        transform: translate(-50%, -50%);
    }
}
```
![](https://user-gold-cdn.xitu.io/2019/7/11/16be0c61044c0ff9?imageslim)

# 5 雨天

云朵的代码直接复用第4章的阴天。这里使用 :after 伪元素实现雨滴。

先实现一个雨滴（为方便观看，暂时隐藏云朵）：
```css
.rainy:after {
	content: "";
    position: absolute;
    top:50%;
    left: 25%;
    width: 4px;
    height: 14px;
    background: #fff;
    border-radius: 2px;
}
```
![](https://user-gold-cdn.xitu.io/2019/7/11/16be0c65ddc32a58?imageslim)

```diff
    .rainy:after {
	    content: "";
        position: absolute;
        top:50%;
        left: 25%;
        width: 4px;
        height: 14px;
        background: #fff;
        border-radius: 2px;
+       box-shadow:
+    	    #fff 25px -10px 0,
+    	    #fff 50px 0 0,
+    	    #fff 75px -10px 0,
+    	    #fff 0 25px 0,
+    	    #fff 25px 15px 0,
+    	    #fff 50px 25px 0,
+    	    #fff 75px 15px 0,
+    	    #fff 0 50px 0,
+    	    #fff 25px 40px 0,
+    	    #fff 50px 50px 0,
+    	    #fff 75px 40px 0;
    }
```
![](https://user-gold-cdn.xitu.io/2019/7/11/16be0c6c1f36abae?imageslim)
再加入下雨的移动动效，修改如下：
```diff
    .rainy:after {
        ...(略)
+        animation: rainDrop 2s linear infinite; 
    }
+   @keyframes rainDrop {
+       0% {
+           transform: translate(0, 0) rotate(10deg);
+       }
+       100% {
+           transform: translate(-4px, 24px) rotate(10deg);
+           box-shadow:
+           #fff 25px -10px 0,
+           #fff 50px 0 0,
+           #fff 75px -10px 0,
+           #fff 0 25px 0,
+           #fff 25px 15px 0,
+           #fff 50px 25px 0,
+           #fff 75px 15px 0,
+           rgba(255, 255, 255, 0) 0 50px 0,
+           rgba(255, 255, 255, 0) 25px 40px 0,
+           rgba(255, 255, 255, 0) 50px 50px 0,
+           rgba(255, 255, 255, 0) 75px 40px 0;
+       }
+   }
```
![](https://user-gold-cdn.xitu.io/2019/7/11/16be0c763481102b?imageslim)

动画添加了10度的旋转，让雨滴倾斜，以及垂直方向的移动。

这里的关键就是：虽然本质是垂直移动，但为了看上去是“循环”效果，需要将最下面的雨滴进行透明渐变，同时调节X和Y轴的值，让最终位置正好跟初始位置重合，就不会显得“断开”。

我们生成的是三行雨滴，第一行被云朵挡住了，实际能看到的是下面两行。在第一行移动到第二行位置的时候，原第三行已经透明看不见了，正好与初始状态一样，实现了无缝循环拼接。

# 6 雪天
雪天与雨天的区别就是把雨滴换成圆形，取消旋转角度。 代码如下：
```css
.snowy:after {
    content: "";
    position: absolute;
    top:50%;
    left: 25%;
    width: 8px;
    height: 8px;
    background: #fff;
    border-radius: 50%;
    box-shadow:
        #fff 25px -10px 0,
        #fff 50px 0 0,
        #fff 75px -10px 0,
        #fff 0 25px 0,
        #fff 25px 15px 0,
        #fff 50px 25px 0,
        #fff 75px 15px 0,
        #fff 0 50px 0,
        #fff 25px 40px 0,
        #fff 50px 50px 0,
        #fff 75px 40px 0;
    animation: snowDrop 2s linear infinite; 
}
@keyframes snowDrop {
    0% {
        transform: translateY(0);
    }
    100% {
        transform: translateY(25px);
        box-shadow:
        #fff 25px -10px 0,
        #fff 50px 0 0,
        #fff 75px -10px 0,
        #fff 0 25px 0,
        #fff 25px 15px 0,
        #fff 50px 25px 0,
        #fff 75px 15px 0,
        rgba(255, 255, 255, 0) 0 50px 0,
        rgba(255, 255, 255, 0) 25px 40px 0,
        rgba(255, 255, 255, 0) 50px 50px 0,
        rgba(255, 255, 255, 0) 75px 40px 0;
    }
}
```
![](https://user-gold-cdn.xitu.io/2019/7/11/16be0c7c4ca14f02?imageslim)

# 7 全部源码

见文件 Weather.html。








