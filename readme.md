目标： 仿造beoplay这款耳机[主页的宣传视频](http://www.beoplay.com/products/beoplayh7?_ga=1.127614725.969767543.1461077943#video)来实现一个类似的视频分屏的效果。

版本1.0（不加视频自适应居中）:

![demo1.0](https://dl.dropboxusercontent.com/s/zppojylzfsyk6in/screensplit1.0.gif?dl=0)

right video是静止的不动的， 同时位于最底层。 目前所有的伸缩都是在控制left video， 同时由于我们设置了left video的`z-index`为3， 那么左边的视频会覆盖在右边的视频上。接下来的目标就很明确了， 我们需要追踪鼠标在整个container里面的位置(我们会用占宽度的百分比来表示)， 然后， 通过改变左边视频的宽度， 同时也就把位于下面的右边视频暴露出来了， 来达到切换的效果。

理解到这一步之后，目标就是**如何改变视频宽度**了。这里其实隐藏着一个陷阱， 就是因为我们不可以直接改变**视频本身的宽度**，因为视频的高宽比在拍摄的时候就决定了， 我们如果只拉长视频而不同时提高高度的话， 就会使得在纵轴上部分内容被遮盖到， 大致意思如图：

![oversizing](http://ww2.sinaimg.cn/large/c5ee78b5gw1f37ixae23nj219e0oe77a.jpg)

而这并不是我们想要的效果， 我们希望， 视频可以维持在同一个大小，而改变的只是**视窗的横向移动**，
这一步我用到了一个trick， 就是`overflow: hidden;`， 我们在左边的视频外面包住了一层`div`， 因为我们控制的就是左边的视频， 然后我们来改变这个“wrapper”的宽度（同时保证保证里面的left video的宽度占比仍然占最外面container的100%）， 来改变我们的视窗， 所以这个wrapper在网页上所包住的部分， 加上了`overflow: hidden;`遮去了超出部分后， 显示的就是左边的视频了。

比如一开始， 我们希望两个视频都各占一半， 那么左边的视窗一开始就会是占比原container的宽度的50%， 而原视频本身的宽度应该不变， 即container的宽度的100%， 所以我们需要给left video赋上200%的宽度， 理由是这个属性是对其直接父级的div起作用的。

```css
/* clipper 即左侧视频的视窗 */
#clipper {
	width: 50%;
	position: absolute;
	top: 0;
	bottom: 0;
	overflow: hidden;
}

/* 该属性作用于左侧视频  */
#clipper video {
	width: 200%;
	position: absolute;
	z-index: 10;
}
```

效果如下：

![init](http://ww2.sinaimg.cn/large/c5ee78b5gw1f37j53tshej21a20o643j.jpg)

之后的事情就清楚了， 我们只需要改变**左边视窗的宽度， 并同时保证视频本身相对原容器的宽度不变**。写的js代码如下：

```javascript
var rect = videoContainer.getBoundingClientRect(),
offset = e.pageX - videoContainer.offsetLeft,
position = ((e.pageX - videoContainer.offsetLeft) / videoContainer.offsetWidth) * 100;

if (position <= 100) {
	videoClipper.style.width = position+"%";
	leftVideo.style.width = ((100/position)*100)+"%";
	leftVideo.style.zIndex = 3;
｝
```

> 简单地说， 只需保证`videoClipper.style.width * leftVideo.style.width = 100%`即可。

那最后一个改变`z-index`的目的， 就是确保左边的视频一直叠加在右边的视频之上。

版本2.0（增加了自适应居中）：

![demo2.0](https://dl.dropboxusercontent.com/s/8y6fr3ib87yz9h7/screensplit2.0.gif?dl=0)

初衷： 因为从拍摄的角度， 大部分的时间， 我们都会把拍摄的主体放在视频的中间位置， 而如果我们只是简单地移动视窗而不改变视窗的主角位置的话， 我们在鼠标移动的时候， 就只能看到边边角的视频内容了， 这违反设计直觉。

因此我们希望在移动视窗的同时， **同时移动视频本身的位置来弥补视窗的偏差**。相应的js代码是：

```javascript
offset = e.pageX - videoContainer.offsetLeft,
offsetRight = videoContainer.offsetWidth - offset,

// for adaptive resizing:
rightVideo.style.webkitTransform = "translate(" + offset / 2 + "px, 0)";
leftVideo.style.webkitTransform = "translate(-" + offsetRight / 2 + "px, 0)";
```

> 画外音： 使用`transform`来改变DOM元素的位置相比直接改写他们的”定位属性“有很多好处， 其中最突出的就是， transform使用的是GPU， 而那些`top\left`等的定位元素使用的是CPU， 我们希望充分利用GPU和它提供的硬件加速， 同时`transform`也不会触发网页的repaint， 从而在渲染上更加的smooth和fast。

那这个地方，`offset`就是鼠标在container里面距离左边边框的距离， 那么我们同时根据这个距离， 让左边的视频在视窗移动时， **往相反的方向以一半的速度移动来弥补视窗偏差**。使得可以在视窗移动的时候， 始终保持视频本身拍摄的主体视角也处于该视窗的主体视角! 原理可以参考这个图：

![demo3.0](https://dl.dropboxusercontent.com/s/t99wbc4n3s8c4r7/screensplit3.0.gif?dl=0)

那么也就大功告成啦!

> 总结： HTML5对media文件更多功能上的支持使得我们可以更好的操作media文件， 包括视频的开始暂停， 以及各种音量的调节， 而这个程序实例， 是一个很好的对相关API的使用的一个示范和一个小小的创意细节。这个项目， 也是饭后和朋友聊天聊出来的实现方案，也希望看到更多的创意脑洞和美的设计!

这个是这个项目的[github repo](https://github.com/chocoluffy/screen-split-video), 这个是项目的展示[demo](http://chocoluffy.com/screen-split-video/) 欢迎评论和star~
