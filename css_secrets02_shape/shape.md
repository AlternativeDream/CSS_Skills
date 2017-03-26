# 形状

#### 1. 自适应的椭圆
> 背景知识：`border-radius`属性的基本用法

使用border-radius制作一个圆很简单，只要给任何正方形元素设置固定宽高及一半长度以上的border-radius，就可以得到圆形

``` css
background: #fb3;
width: 200px;
height: 200px;
border-radius: 100px;  /*>=正方形边长的一半*/
```

![enter description here][1]

#### 1.1 椭圆

在实际开发中可能更多的是让元素根据内容自动调增，而不是事先设置好宽高，因为很多时候内容是不定的。我们期望：如果宽高相等，就显示一个圆，不相等就显示一个椭圆。

上面代码实现不了，当宽度大于高度的时候，便会出现下面这种情况：

![enter description here][2]

解决方案：

`border-radius`可以单独指定水平和垂直半径，只要用一个(/)分隔这两个值就行。还一个特性是，它可以接受长度值，还可以接受百分比，两种特性结合，就可以自适应了。

``` css
border-radius: 50% / 50%;
```

![enter description here][3]

由于斜杠前后的两个值现在是一致的，可以简写

``` css
border-radius: 50%;
```

#### 1.2 半椭圆

border-radius是一个简写属性，包含四个展开式属性，这样一来可以通过设置四个角的不同半径来达到效果。

* `border-top-left-radius`
* `border-top-right-radius`
* `border-bottom-right-radius`
* `border-bottom-left-radius`

通过属性名字可以很清楚的知道他们用于设置哪个角。

不过还是推荐使用简洁写法，因为可以向它一次性提供用空格分开的多个值。

* 传四个值，这四个值就会被分别从左上角开始以顺时针顺序应用到各个角
* 传三个值，则意味着第四个值与第二个值相同
* 传两个值，意味着第三个值与第一个值相同，第四个值与第二个值相同。
* 传一个值，全部相同。

我们甚至可以为所有四个角提供完全不同的水平和垂直半径，方法就是在斜杠前指定1~4个值，斜杠后指定1~4个值。这两组值是单独展开为四个值。

比如`border-radius：10px / 5px 20px`, 
相当于10px 10px 10px 10px / 5px 20px 5px 20px;

两个例子：

``` css
border-radius:2em;
```

等同于

``` css
border-top-left-radius:2em;
border-top-right-radius:2em;
border-bottom-right-radius:2em;
border-bottom-left-radius:2em;
```

```css
border-radius: 2em 1em 4em / 0.5em 3em;
```

等同于

``` css
border-top-left-radius: 2em 0.5em;
border-top-right-radius: 1em 3em;
border-bottom-right-radius: 4em 0.5em;
border-bottom-left-radius: 1em 3em;
```

我们要实现的效果

![enter description here][4]


弄清了，border-radius的用法，开始分析上面半椭圆的实现

* 这个形状是垂直对称的，意味着左上角和右上角的半径值应该相同，右下角和左下角半径也是相同
* 顶部边缘没有平直的部分，意味着左上角和右上角半径之和应该等于整个形状的宽度，结合上一条，左半径和右半径在水平方向上是`50%`。
* 垂直方向，顶部的两个圆角占据了整个元素的高度，而且底部没有任何圆角，因此在垂直方向上值应该好似`100% 100% 0 0`。
* 因为底部两个角的垂直圆角是零，那么他们的水平圆角是多少就完全不重要了


``` css
border-radius: 50% / 100% 100% 0 0;
```

![enter description here][5]

沿纵轴劈开的半椭圆

``` css
border-radius: 100% 0 0 100% / 50%;
```

![enter description here][6]


#### 1.3 四分之一椭圆

创建一个四分之一椭圆，其中一个角水平和垂直半径值都需要100%，而其他三个角都不能设为圆角

``` css
border-radius: 100% 0 0 0;
```


![enter description here][7]



### 2. 平行四边形
> 背景知识：基本变形属性`transform`，属性很多，用到哪个讲哪个。

我们一般用skew属性来对矩形进行斜向拉伸，变成平行四边形，但是如果直接对元素使用的话，它里面的内容也会跟着斜向变动。

![enter description here][8]



#### 2.1 嵌套元素方案

对容器内容再应用一次反向的skewX()变形，从而抵消容器的变形效果

```html
<a href="#" class="button">
	<div>
		click me 
	</div>
</a>
```

css

``` css
.button {
	transform: skewX(-45deg);
}

.button > div {
	transform: skewX(45deg);
}
```

> `skewX()`	定义沿着 X 轴的 2D 倾斜转换。对应还有`skewY(angle)，skew(x-angle,y-angle)`

#### 2.2 伪元素方法
上面方法需要添加额外的HTML元素。这里提供一个不用添加额外的HTML元素的。

这种方法的思路是把所有的样式(背景、边框等)应用到伪元素上，然后对伪元素进行变形。


``` css
 .button {
		position: relative;
		
		/* 其他的文字颜色、内外变局等样式*/
	}

.button::before {
	content: '';
	position: absolute;
	top: 0;right: 0;bottom: 0;left: 0;

	z-index: -1;

	background: #58a;
	transform: skew(-45deg);
}
```


![enter description here][9]


要注意几点，一是要给宿主元素设置`position: relative`，并给伪类元素设置`position: absolute`，且偏移量都为0，以便让它在水平和垂直方向上都被拉伸至宿主元素的尺寸。二是伪元素生成的方块是重叠在内容之上的，一旦设置背景，就会遮住内容，应该设置`z-index: -1`或者更小的数，宿主没有设置`z-index`默认为0；

### 3. 菱形图片

> 背景知识：css变形、“平行四边形”

#### 3.1 基于变形的方案

思路：需要把图片用一个<div>包起来，然后对其应用相反的rotate()变形样式。


```html
<div class="picture">
  <img src="dog.jpg">
</div>
```

样式

``` css
.picture {
	width: 200px;
	height: 200px;
	transform: rotate(45deg);
	overflow: hidden;
}

.picture img {
	max-width: 100%;
	transform: rotate(-45deg);
}
```	

并没有发生我们想象的那样，得到一个八角形，还挺好看的，哈啊。

![enter description here][10]

> `totate()`定义 2D 旋转，在参数中规定角度。

为了观察清楚，为容器添加边框

![enter description here][11]


出现这样的情况主要是`max-width：100%`，`100%`会被解析为容器(.picture)的边长，但是，我们想要图片的宽度与容器的对角线相等。通过计算可以很快计算出正方形的对角线的长度为`根号2` 乘以正方形的边长。因为`根号2`约等于`1.414`，那么可以设置max-width为`1.141 * 100%`，向上取整为142%。

但是更推荐使用scale属性把图片放大，更合理。

* 我们希望图片的尺寸保留100%这个值，这样当浏览器不支持变形样式时仍然可以得到一个合理的布局
* 通过scale变形样式来缩放图片，是以它的中心点进行缩放的。通过width属来放大图片时，只会以它的左上角为原点进行缩放，从容迫使我们动用额外的负边距来调整图片。

``` css
.picture {
	width: 200px;
	height: 200px;
	margin: 100px;
	transform: rotate(45deg);
	overflow: hidden;
}

.picture img {
	max-width: 100%;
	transform: rotate(-45deg) scale(1.42);
}
```

![enter description here][12]


#### 3.2 裁剪路径方案

上面方案需要一层额外的HTML，不够简洁，还有就是如果处理一张非正方形的图片，就不会有这样的效果。

使用的属性为`clip-path`，裁剪路径。我们使用其中的一个属性值`polygon()`（多边形）来指定一个菱形。语法从SVG那借鉴过来的

> clip-path还有很多强大的功能，可以自行研究，还有就是目前为止，IE11和Edge完全不支持，不过可以平稳退化。

``` css
clip-path: polygon(50% 0, 100% 50%, 50% 100%, 0 50%);
```

上面用逗号分隔，每组数据代表一个坐标点，最后连接起来，就组成一个菱形。效果和上面一样。

还可以增加动画，比如悬停的时候，让图片扩展为完整的面积。

``` css
img {
	max-width: 250px;
	margin: 20px;

	clip-path: polygon(50% 0, 100% 50%, 50% 100%, 0 50%);

	transition: 1s clip-path;
}

img:hover {
	clip-path: polygon(0 0, 100% 0, 100% 100%, 0 100%);
}
```


### 4.切角效果

> 背景知识：css渐变、background-size、条纹背景

#### 4.1 直角切角

先实现一个简单的，比如让一个矩形，右下角切掉。实现该功能可以使用强大的渐变功能，有了渐变基础，应该不难理解。

``` css
 background: #58a;
 background: linear-gradient(-45deg, transparent 15px, #58a 0);
```

![enter description here][13]


`background: #58a`不是必须的，加上它是为了将其作为回退机制。


实现两个切角，左右下角各一个。一层渐变肯定不行，需要两层。按想法一步步实现，首先可能会这样写

``` css
background: #58a;
background: linear-gradient(-45deg, transparent 15px, #58a 0),
	linear-gradient(45deg, transparent 15px, #655 0);
```

![enter description here][14]

可以看到，效果并没有实现，原因是两层渐变都会填满整个元素，因此它们会相互覆盖。需要让它们缩小一点，使用background-size让每层渐变分别只占据整个元素的一半。

``` css
 background: #58a;
background: linear-gradient(-45deg, transparent 15px, #58a 0) right,
	linear-gradient(45deg, transparent 15px, #655 0) left;
background-size: 50% 100%;
```

![enter description here][15]

依然没有达到效果，这是因为没有添加`background-repeat`属性，因而每层渐变图案各自平铺了两次。

``` css
 background: #58a;
background: linear-gradient(-45deg, transparent 15px, #58a 0) right,
	linear-gradient(45deg, transparent 15px, #655 0) left;
background-size: 50% 100%;
background-repeat: no-repeat;
```

![enter description here][16]

好了，现在实现了。如果要四个角的话，就要四层渐变了。

``` css
background: #58a;
background: linear-gradient(135deg, transparent 15px, #58a 0) top left,
	 linear-gradient(-135deg, transparent 15px, #58a 0) top right,
	 linear-gradient(-45deg, transparent 15px, #58a 0) bottom right,
	 linear-gradient(45deg, transparent 15px, #58a 0) bottom left;
background-size: 50% 50%;
background-repeat: no-repeat;
```

![enter description here][17]

> 注： 在chrome中运行，发现一个问题，中间会出现一条白线。其他浏览器没有，还没有找到原因，知道的话请点明一下。

![enter description here][18]


上面代码的可维护性差，要改变背景的时候，需要修改好多处，可以使用预处理器。这里使用`scss`。

``` scss
@mixin beveled-corners($bg, $tl:0, $tr:$tl, $br:$tl, $bl:$tr) {
	background: $bg;
	background: linear-gradient(135deg, transparent $tl, $bg 0) top left,
			linear-gradient(-135deg, transparent $tr, $bg 0) top right,
			linear-gradient(-45deg, transparent $br, $bg 0) bottom right,
			linear-gradient(45deg, transparent $bl, $bg 0) bottom left;
	background-size: 50% 50%;
	background-repeat: no-repeat;
}
```

调用的时候传入2~5个参数

``` scss
@include beveled-corners(#58a, 15px, 5px);
```

#### 4.2 弧形切角

还可以创建弧形切角（“内凹圆角”），使用径向渐变属性代替上面线性渐变。

``` css
background: #58a;
            background:radial-gradient(circle at top left, transparent 15px, #58a 0) top left,
                radial-gradient(circle at top right, transparent 15px, #58a 0) top right,
                radial-gradient(circle at bottom  right, transparent 15px, #58a 0) bottom right,
                radial-gradient(circle at bottom left, transparent 15px, #58a 0) bottom left;
            background-size: 50% 50%;
            background-repeat: no-repeat;
```

上面代码中`circle`指定圆形的径向渐变以及位置。

效果

![enter description here][19]

这里同样可以使用预处理器来封装，类似。


#### 4.3 内联SVG与border-image方案

基于渐变的方法有几个缺点：
* 代码繁琐冗长
* 不能让各个切角的尺寸以动画的方式发生变化

这里提供另一种方案，使用border-image，之前有提到过基本用。

![enter description here][20]

根据`border-image`工作原理，使用上面SVG图形就可以产生带有切角的边框。黑线是增加的辅助线，描述`border-image`工作原理，将背景图分成九宫格。

由于尺寸无关紧要（border-image会解决缩放问题，而SVG可以实现与尺寸完全无关的完美缩放--矢量图的好处）。每个切片的尺寸都可以设置为1，便于书写。上面就是放大的图。

``` css
border: 15px solid transparent;
	        border-image: 1 url('data:image/svg+xml,\
	                      <svg xmlns="http://www.w3.org/2000/svg" width="3" height="3" fill="%2358a">\
	                      <polygon points="0,1 1,0 2,0 3,1 3,2 2,3 1,3 0,2" />\
	                      </svg>');
```

![enter description here][21]

> 注意：使用的切面尺寸为1，这并不代表1像素，所对应的是SVG文件的坐标系统，所以也不同单位。


切角效果出来了，但还缺少整片背景，可以指定一个背景色。还有就是切角没有之前大。原因是在渐变中，这个15px是沿着渐变轴来度量的，它的水平方向和渐变推进的方向一致。边框宽度并不是斜向度量的，而是以水平或垂直方向度量的。

和之前“条纹背景类似”，为了得到相同的尺寸，我们需要把渐变中的尺寸乘以`根号2`，然后才能用在边框宽度属性中。`15*根号2`约等于21px。

``` css
border: 21px solid transparent;
border-image: 1 url('data:image/svg+xml,\
			  <svg xmlns="http://www.w3.org/2000/svg" width="3" height="3" fill="%2358a">\
			  <polygon points="0,1 1,0 2,0 3,1 3,2 2,3 1,3 0,2" />\
			  </svg>');
background: #58a;
```

![enter description here][22]

啊偶，切角效果没了，换个背景色看看。

![enter description here][23]

还挺好看的，切角还在。原来是因为背景色和切角边框混成一团了，使用`background-clip`修复。

``` css
border: 21px solid transparent;
border-image: 1 url('data:image/svg+xml,\
			  <svg xmlns="http://www.w3.org/2000/svg" width="3" height="3" fill="%2358a">\
			  <polygon points="0,1 1,0 2,0 3,1 3,2 2,3 1,3 0,2" />\
			  </svg>');
background: #58a;
background-clip: padding-box;
```

![enter description here][24]


效果出来了，还可以增加其他效果，比如背景渐变，切角动画。

![enter description here][25]


#### 4.3 裁切路径方案

`boder-image`也有局限。比如我们要么指定某个实色背景，要么指定一个边缘接近某个实色的背景图案。假如我们设置其他类型的背景（比如纹理、平铺图案或一道线性渐变）。

`clip-path`可以打破这种局限。

``` css
 background: #58a;
clip-path: polygon(20px 0, calc(100% - 20px) 0, 100% 20px,
		100% calc(100% - 20px), calc(100% - 20px) 100%,
		20px 100%, 0 calc(100% - 20px), 0 20px);
```

可以使用任意背景，还可以是图片

![enter description here][26]

### 5. 梯形标签页
> 背景知识：基本的3D变形，平行四边形

现实的三维世界中旋转一个矩形，由于透视的关系，我们最终看到的二维视图往往是一个梯形。

``` css
transform: perspective(.5em) rotateX(5deg);
```


* perspective(.5em)为 3D 转换元素定义透视视图,不设置的话看不到3D效果
*  rotateX(5deg) 定义沿着 X 轴的 3D 旋转。

![enter description here][27]


可以生成一个梯形，但是内部的内容不可逆转，不像2D变形。可以使用伪元素。

``` css
.tab {
   margin: 3em;
   position: relative;
   display: inline-block;
   padding: .5em 1em .35em;
   color: white;
}

.tab::before {
	content: '';
	position: absolute;
	top: 0;right: 0;bottom: 0;left: 0;
	z-index: -1;
	background: #58a;
	transform: perspective(.5em) rotateX(5deg);
}
```

![enter description here][28]

现在生成了一个基本的梯形，但是还有许多问题。当我们没有设置`transform-origin`属性时，应用变形效果会让这个元素以它自身的中心线为轴进行空间上的旋转。因此，元素投射到2D屏幕上的尺寸会发生变化。比如宽度会增加，所占位置会稍稍下移，高度会有少许缩减。

我们可以为它指定`transform-origin:bottom`，当在3D空间旋转的时候，可以把它的底边固定住。

``` css
transform: perspective(.5em) rotateX(5deg);
transform-origin: bottom;
```

![enter description here][29]

这样一来只有高度会发生变化，但是高度缩水也非常明显，那么可以在垂直方向使用scaleY()进行放大。

``` css
transform:scaleY(1.3) perspective(.5em) rotateX(5deg);
transform-origin: bottom;
```

![enter description here][30]

使用这个技巧随后为标签页添加样式就变得很方便，

``` css
nav > a {
	position: relative;
	display: inline-block;
	padding: .3em 1em 0;
}

nav > a::before {
	content: ''; 
	position: absolute;
	top: 0; right: 0; bottom: 0; left: 0;
	z-index: -1;
	border-bottom: none;
	border-radius: .5em .5em 0 0;
	background: #ccc linear-gradient(hsla(0,0%,100%,.6), hsla(0,0%,100%,0));
	box-shadow: 0 .15em white inset;
	transform: scale(1.1, 1.3) perspective(.5em) rotateX(5deg);
	transform-origin: bottom;
}
```

我们给它添加了背景、边框、圆角、投影等样式，都可以完美生效，我们还可以通过设置`background-origin`为bottom left或bottom right，就可以得到左倾斜或右倾斜的标签页。具体代码可以查看[仓库][31]。

![enter description here][32]


  [1]: ./images/01-1.png "01-1.png"
  [2]: ./images/01-2.png "01-2.png"
  [3]: ./images/01-3.png "01-3.png"
  [4]: ./images/01-4.png "半椭圆"
  [5]: ./images/01-4.png "01-4.png"
  [6]: ./images/01-5.png "01-5.png"
  [7]: ./images/01-6.png "01-6.png"
  [8]: ./images/02-1.png "02-1.png"
  [9]: ./images/02-3.png "02-3.png"
  [10]: ./images/03-1.png "03-1.png"
  [11]: ./images/03-2.png "03-2.png"
  [12]: ./images/03-3.png "03-3.png"
  [13]: ./images/04-1.png "04-1.png"
  [14]: ./images/04-2.png "04-2.png"
  [15]: ./images/04-3.png "04-3.png"
  [16]: ./images/04-4.png "04-4.png"
  [17]: ./images/04-5.png "04-5.png"
  [18]: ./images/04-6.png "04-6.png"
  [19]: ./images/04-7.png "04-7.png"
  [20]: ./images/04-8.png "04-8.png"
  [21]: ./images/04-9.png "04-9.png"
  [22]: ./images/04-10.png "04-10.png"
  [23]: ./images/04-11.png "04-11.png"
  [24]: ./images/04-12.png "04-12.png"
  [25]: ./images/04-13.png "04-13.png"
  [26]: ./images/04-14.png "04-14.png"
  [27]: ./images/05-1.png "05-1.png"
  [28]: ./images/05-2.png "05-2.png"
  [29]: ./images/05-3.png "05-3.png"
  [30]: ./images/05-4.png "05-4.png"
  [31]: https://github.com/xxxgitone/CSS_Skills
  [32]: ./images/05-5.png "05-5.png"