#自定义View系统学习笔记-第五期：Canvas的绘制流程

	前几期讲的是【术】，是用哪些api可以绘制什么内容。
	到上一期为止，【术】已讲完了，接下来要讲的是【道】，是怎么去安排这些绘制。
	这期是道的第一期：绘制顺序。
	
	Android里面的绘制都是按顺序的，先绘制的内容会被后绘制的盖住。
	比如你在重叠的位置先画圆再画方，和先画方再画圆所呈现出来的结果是不同的：


![](https://user-gold-cdn.xitu.io/2017/8/14/8d13f9d6661d6654775eef851499f2f5?imageView2/0/w/1280/h/960/ignore-error/1)

## 1，super.onDraw()前 or 后？

	自定义View的最基本的形态：继承View类。onDraw()在View类里是空实现，所以继承View的子类删除super.onDraw()没任何影响，放前放后也没影响，但是再继承的子类就不一样了，要根据实际情况放置。
![](https://user-gold-cdn.xitu.io/2017/8/14/6764913eb52c6ce7cfad6b20f414652f?imageView2/0/w/1280/h/960/ignore-error/1)
自定义View，就不能不考虑super.onDraw()了，要根据实际情况，判断出你绘制的内容需要盖住控件原有的内容还是需要被控件原有的内容盖住，从而确定你的绘制代码应该在super.onDraw()的上面还是下面。

### 1.1 写在super.onDraw()的下面
	由于绘制代码会在原有内容绘制结束后才执行，所以绘制内容就会盖住控件原来的内容。

	  Drawable drawable = getDrawable();
        if (drawable != null) {
            canvas.save();
            canvas.concat(getImageMatrix());
            Rect bounds = drawable.getBounds();
            canvas.drawText(getResources().getString(R.string.image_size, bounds.width(), bounds.height()), 20, 40, paint);
            canvas.restore();
        }
效果如下：
![](https://user-gold-cdn.xitu.io/2017/8/14/58244ea331636a82b60267f37d98e8f0?imageView2/0/w/1280/h/960/ignore-error/1)

### 1.2 写在super.onDraw()的上面
	
	由于绘制代码会执行在原有内容的绘制之前，所以绘制的内容会被控件的原内容盖住。
	
	 canvas.save();
        bounds.left=layout.getLineLeft(0);
        bounds.top=layout.getLineTop(0);
        bounds.right=layout.getLineRight(0);
        bounds.bottom=layout.getLineBottom(0);
        paint.setColor(Color.RED);
        canvas.drawRect(bounds,paint);
        canvas.restore();

        super.onDraw(canvas);
效果如下：
![](https://user-gold-cdn.xitu.io/2017/8/14/58f7bf33928eadc426753354b00f4dde?imageView2/0/w/1280/h/960/ignore-error/1)

## 2，dispatchDraw() 绘制子View的方法
	实际上，绘制方法不止一个，还有好几个，onDraw()是绘制主体，dispatchDraw()就是用来绘制子view的。
	有时候遮盖效果无法通过onDraw()来实现。
	例如：继承了ViewGroup，重写了它的onDraw()方法，在super.onDraw()中插入了你自己的绘制代码，使它能在内部绘制一些斑点点缀。
![](https://user-gold-cdn.xitu.io/2017/8/14/8fe7e00dc5fe7c4b0aade354a6ddf6d1?imageView2/0/w/1280/h/960/ignore-error/1)
但是效果不和期望一致，这是因为Android的绘制顺序，每一个ViewGroup会先调用自己的onDraw()来绘制完自己的主体之后再去绘制它的子View。所以上面的ViewGroup在绘制完自己的主题后再绘制斑点，再去绘制它的子View，结果就是斑点被遮盖了。
具体来讲，每一个View和ViewGroup都会先调用onDraw()方法来绘制主体，再调用dispatchDraw()方法来控制子View。
	
	注：
		虽然View和ViewGroup都有dispatchDraw()方法，不过由于View是没有子View的，所以一般来说，dispathc这个方法只对ViewGroup有意义。
![](https://user-gold-cdn.xitu.io/2017/8/14/804c043b21477927c248e4265b59d308?imageView2/0/w/1280/h/960/ignore-error/1)

所以，如果要让绘制的内容显示，就要在绘制子View完后。
### 2.1 写在super.dispathDraw()的下面
	放在后面就会在子view绘制完后执行。绘制内容就会盖住子View。

### 2.2 写在super.dispatchDraw()的上面
	放在上面就是绘制内容会出现在主体内容和子View之间。
	。。。。这。。就是1.1所说的。

## 3，绘制过程简述
	完整的绘制过程会按下面依次执行：
		1，背景
		2，主体
		3，子View
		4，滑动边缘渐变和滑动条
		5，前景	
	注意：
		前景的支持是在Android6.0才加入的，之前其实也有，不过只支持FrameLayout，而直到6.0才把这个支持放到了View里。
![](https://user-gold-cdn.xitu.io/2017/8/14/3faafa03aaf373b61c297cd619f9c101?imageView2/0/w/1280/h/960/ignore-error/1)

## 4，onDrawForeground()
	绘制滑动边缘渐变和滑动条
	这个方法是6.0才引入的

### 4.1 写在super.onDrawForeground()的下面
	如果你把绘制代码写在了super.onDrawForeground()的下面，绘制代码会在滑动边缘渐变，滑动条和前景之后被执行。
	那么绘制的内容将会盖住滑动边缘渐变、滑动条和前景。
![](https://user-gold-cdn.xitu.io/2017/8/14/7e5199890125adf8f3538003c6fc0122?imageView2/0/w/1280/h/960/ignore-error/1)

### 4.2 写在super.onDrawForeground()的上面
	绘制内容就会在dispatchDraw()和super.onDrawForeground()之间执行，那么绘制内容会盖住子View，但是被滑动边缘渐变、滑动条以及前景盖住。
![](https://user-gold-cdn.xitu.io/2017/8/14/2f38e3647de47d5d9b8c41429636820a?imageView2/0/w/1280/h/960/ignore-error/1)

### 4.3 想在滑动边缘渐变、滑动条和前景之间插入绘制代码？

很简单：不行。

## 5，draw()总调度方法
	除了onDraw() dispatchDraw()和onDrawForeground()之外，还有一个可以用来实现自定义绘制的方法：draw()。

	draw()是绘制过程的总调度方法。
	一个View的整个绘制过程都发生在draw()方法里。
	前面讲到的前景，主体，子View，滑动相关以及前景的绘制，它们其实都是在draw()方法里的。




	// View.java 的 draw() 方法的简化版大致结构（是大致结构，不是源码哦）：

	public void draw(Canvas canvas) {
    ...

    drawBackground(Canvas); // 绘制背景（不能重写）
    onDraw(Canvas); // 绘制主体
    dispatchDraw(Canvas); // 绘制子 View
    onDrawForeground(Canvas); // 绘制滑动相关和前景

    ...
	}

	从上面draw()方法就可以看出绘制的顺序了。
![](https://user-gold-cdn.xitu.io/2017/8/14/756f73b23df06b73b4c57b13f6e0de10?imageView2/0/w/1280/h/960/ignore-error/1)

### 5.1 写在super.draw()的下面
	由于draw是总调度方法，所以如果把绘制代码写在suoer.draw()的下面，那么这段代码会在所有绘制完成之后再执行，饿就是说，它的绘制内容会盖住其他所有绘制内容。
	
	它的效果和重写onDrawForeground()之后是一样的。
### 5.2写在super.draw()的上面
	同理，会在所有绘制之前被执行，所以这部分绘制内容会被所有其他内容盖住，包括背景。是的，背景也会盖住它。
如下，如果要给edittext换背景，就不能直接换了，因为那条横线就是背景，这时候我们就要用这种方法来实现了。所以还是有用的。
![](https://user-gold-cdn.xitu.io/2017/8/14/d25142aa4ff360e11e6c960276a2a92c?imageView2/0/w/1280/h/960/ignore-error/1)
![](https://user-gold-cdn.xitu.io/2017/8/14/04276a9df53fe995999deda03928c886?imageView2/0/w/1280/h/960/ignore-error/1)
![](https://user-gold-cdn.xitu.io/2017/8/14/a729d0604251269b1d120601f4adf163?imageView2/0/w/1280/h/960/ignore-error/1)

#注意：
	关于绘制方法，有两点需要注意一下：
		1，出于效率的考虑，ViewGroup默认会绕过draw()方法，换而直接执行dispatchDraw()，以此来简化绘制历程。
		所以如果定义了某个ViewGroup的子类并且需要在它的除dispatch()以外的任何一个绘制方法内绘制内容，需要调用【View.setWillNotDraw(false)】，例如scrollview。
		2，有的时候，一段绘制代码写在不同的绘制方法中效果是一样的，这时可以选一个自己喜欢或者习惯的绘制方法来重写。但是有一个例外：如果绘制代码既可以写在onDraw()里，也可以写在其他绘制方法里，那么优先写在onDraw()，因为Android有相关的优化，可以在不需要重绘的时候自动跳过onDraw()的重复执行，以提高开发效率。享受这种优化的只有onDraw()一个方法。

#总结：
![](https://user-gold-cdn.xitu.io/2017/8/14/756f73b23df06b73b4c57b13f6e0de10?imageView2/0/w/1280/h/960/ignore-error/1)
![](https://user-gold-cdn.xitu.io/2017/8/14/82e9383685be9ddae82b656176e69cf1?imageView2/0/w/1280/h/960/ignore-error/1)