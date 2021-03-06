# 视觉效果

### 1. 单侧投影

#### 1.1 单侧投影

大多数人使用box-shadow的方法是，制定三个长度和一个颜色值。

``` css
box-shadow: 2px 3px 4px rgba(0, 0, 0, .5);
```

![enter description here][1]

> box-shadow第一个长度表示水平方向投影，第二个为垂直方向投影，第三个为模糊半径。

投影绘制步骤：
* 以该元素的尺寸和位置画一个`rgba（0,0,0，.5）`的矩形
* 把它向右移`2px`，向下移3px
* 使用高斯模糊算法将它进行4px的模糊处理。这在本质上表示在阴影边缘发生阴影色和纯透明色之间的颜色过度长度近似模糊半径的两倍（这里为8px）
* 接下来，模糊后的矩形与原始元素的交集部分会被剪切掉，因此看起来像是在该元素后面。所以这里跟我们所理解的元素叠在模糊后矩形上层不同。所以说如果我们给元素设置一层半透明的背景，就看不到任何投影

使用`4px`的模糊半径意味着投影的尺寸会比元素本身的尺寸大约8px，因此投影的最外圈会从元素的四面向外显露出来。

* 设置偏移量，就可以把投影的顶部和左边隐藏起来，只要这两个方向上的偏移量不小于4px就行，不够会导致外漏的投影过于浓重

* 最终解决方案是使用`box-shadow`的第四个长度参数，排在模糊半径参数之后，称作扩张半径。这个参数会根据你指定的值去扩张或缩小（负值）投影的尺寸。比如一个-5px的扩张半径会把投影宽度和高度各减少10px（每边5px）

所以我们使用一个负的扩张半径，而他的值正好等于模糊半径

``` css
box-shadow: 0 5px 4px -4px black;
```

![enter description here][2]

现在只有底部才有一道投影了。

#### 1.2 邻边投影
可以使用之前说的设置偏移量，要么使用设置第四个长度参数

* 我们不应该把阴影缩的太小，而是把阴影藏进一侧，另一侧自然露出就好，因此，扩张半径不应该设为模糊半径的相反值，而是这个相反值的一半
* 需要指定两个偏移量，因为希望投影在水平和垂直方向上同时移动。它们的值需要大于或等于模糊半径的一半

``` css
box-shadow: 3px 3px 6px -3px black;
```

![enter description here][3]

#### 1.3 双侧投影
用来那个卡U呢投影（每边个一块）来达到目的。基本上就是把"单侧投影"中的技巧运用两次

``` css
box-shadow: 5px 0 5px -5px black,
                -5px 0 5px -5px black;
```

![enter description here][4]


### 2. 不规则投影

当元素添加了伪元素或半透明背景的时候，box-shadow会变得力不从心，出现一些问题，有些原理之前已经讲到过。

对下面三种情况的元素都使用了`box-shadow: .1em .1em .3em rgba(0, 0, 0, .5);`

比如，用伪元素生成气泡的小尾巴后，再对气泡使用投影，投影效果不会在小尾巴上显现。

![enter description here][5]

对元素使用了透明背景，投影会忽视透明部分。

![enter description here][6]

切角效果

![enter description here][7]

等等。

使用filter属性来解决，目前为止IE11还不支持，其他基本支持。filter实行从SVG借鉴过来的。我们使用其中一个函数`drop-down()`，跟`box-shadow`很相像，但不包括扩张半径和`inset`值。

可以这样改写

``` css
filter: drop-shadow(.1em .1em .1em rgba(0,0,0,.5));
```

最终效果

![enter description here][8]

### 3. 染色效果

使用`<canvas>`也可以实现，这里探讨css方法的实现。

#### 3.1 基于滤镜的方案
由于没有一种现成的滤镜是专门为这个效果而设计的，所以需要使用多个滤镜组合起来。

第一个滤镜是`sepia()`,它会给图片增加一种饱和度的橙黄色染色效果，几乎所有的色相值都被收敛在35~40之间。

``` css
filter: sepia();
```

![enter description here][9]

如果想要主色调的饱和度比这个高，可以用`staturate()`l滤镜来给每个像素提升饱和度。具体多少可以根据实际情况。

``` css
filter: sepia() saturate(4);
```

![enter description here][10]

但是我们想要一种亮粉色的，则还需要再添加一个`hue-rotate()`滤镜，把每个像素的色相以指定的度数进行偏移。为了把原来的色相值40提升至335，需要增大（335 - 40）= 295度

``` css
filter: sepia() saturate(4) hue-rotate(295deg);
```

![enter description here][11]

添加一个hover来触发切换

``` css
img {
	max-width: 250px;
	filter: sepia() saturate(4) hue-rotate(295deg);
	transition: 1s filter;
}

img:hover,img:focus {
	filter:none
}
```

#### 3.2 基于混合模式的方案

当两个元素叠加时，“混合模式”控制了上层元素的颜色与下层颜色进行混合的方式。用它来实现染色效果，需要用到的混合模式是`luminosity`，这种模式会保留上层元素的HSL亮度信息，并从它的下层吸取色相的饱和度信息。所以如果我们将主色调放在下层，待处理的图片放在上层，就相当于进行染色了。当然IE11肯定是不支持的

对一个元素设置混合模式，有两个属性可以用：`mix-blend-mode`可以为整个元素设置混合模式，`background-blend-mode`可以为每层背景单独指定混合模式。所以有两种选择

* 需要把图片包裹在一个容器中，并把容器的背景设置为我们想要的主色调
* 不用图片元素，而用div元素，把这个元素的第一层背景设置为要染色的图片第二层的背景设置为我们想要的主色调

``` html
 <a href="#">
	<img src="dog.jpg">
</a>
```

几行声明就可以解决

``` css
a {
		display: block;
		max-width: 250px;
		background: hsl(335, 100%, 50%);
	}

	img {
		width: 100%;
		mix-blend-mode: luminosity;
	}
```

mix-blend-mode是把整个元素向下进行混合，而不管下面是什么。因此只要把这个属性设置为luminosity混合模式，那图片总会跟某些东西进行混合。此外，使用background-blend-mode属性则可以让每层背景跟它的下层背景进行混合，但不关心元素之外是什么情况。所以，当我们只有一个背景图像以及一个透明的背景色时，就不会出现任何混合效果。利用此特点可以制作动画

``` html
<div class="dog" style="background-image: url('dog.jpg')">
    </div>
```

```css
.dog {
		width: 640px; height: 440px;
		background-size: cover;
		background-color: hsl(335, 100%, 50%);
		background-blend-mode: luminosity;
		transition: .5s background-color;
	}

	.dog:hover {
		background-color: transparent;
	}
```

两种方法都不够理想

* 图片的尺寸需要在css中写死
* 在语义上，这个元素不是一张图片













  [1]: ./images/01-1.png "01-1.png"
  [2]: ./images/01-2.png "01-2.png"
  [3]: ./images/01-3.png "01-3.png"
  [4]: ./images/01-4.png "01-4.png"
  [5]: ./images/02-1.png "02-1.png"
  [6]: ./images/02-2.png "02-2.png"
  [7]: ./images/02-3.png "02-3.png"
  [8]: ./images/02-4.png "02-4.png"
  [9]: ./images/03-1.png "03-1.png"
  [10]: ./images/03-2.png "03-2.png"
  [11]: ./images/03-3.png "03-3.png"