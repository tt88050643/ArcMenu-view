#自定义控件-卫星式菜单
1. 我们ArcMenu 卫星控件的效果如下，Android动画有三种，Tween动画、Frame动画、property动画，这里我们用到了Tween动画，实现view的平移、旋转效果。
控件有一个主ImageView 和若干个自ImageView组成，通过代码setVisibility(View.GONE)来动态的控制view的状态，效果：
![](http://i4.tietuku.com/1fc544678e9eda95.gif)
2. 首先看卫星控件的效果图，可以观察到
	- 一个主ImageView 和若干个自ImageView
	- 在layout的布局文件中：
	-
		<com.example.zhaimeng.imooc_arcmenu.view.ArcMenu
        android:id="@+id/id_menu"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        zhaimeng:position="right_bottom"
        zhaimeng:radius="150dp">
        <RelativeLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@drawable/composer_button">
            <ImageView
                android:id="@+id/id_button"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_centerInParent="true"
                android:src="@drawable/composer_icn_plus" />
        </RelativeLayout>
        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/composer_camera"
            android:tag="Camera" />
        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/composer_place"
            android:tag="Place" />
        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/composer_sleep"
            android:tag="Sleep" />
        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/composer_thought"
            android:tag="Sun" />
        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/composer_with"
            android:tag="People" />
    	</com.example.zhaimeng.imooc_arcmenu.view.ArcMenu>

	- 编写自定义控件的属性，在value文件夹下创建xml文件attr.xml
	-
		<?xml version="1.0" encoding="utf-8"?>
		<resources>
	    <attr name="position">
	        <enum name="left_top" value="0" />
	        <enum name="left_bottom" value="1" />
	        <enum name="right_top" value="2" />
	        <enum name="right_bottom" value="3" />
	    </attr>
	    <attr name="radius" format="dimension" />
	    <declare-styleable name="ArcMenu">
	        <attr name="position" />
	        <attr name="radius" />
	    </declare-styleable>
		</resources>
	我们自定义了两个属性，position和ardius
3. 创建自定义控件的类，ArcMenu.java，继承自ViewGroup
	- 创建构造方法，在构造方法中拿到控件的属性
	-
		TypedArray a = context.getTheme().obtainStyledAttributes(attrs, R.styleable.ArcMenu, defStyleAttr, 0);
        int pos = a.getInt(R.styleable.ArcMenu_position, 3);
        switch (pos) {
            case POS_LEFT_TOP:
                mPosition = Position.LEFT_TOP;
                break;
            case POS_LEFT_BOTTOM:
                mPosition = Position.LEFT_BOTTOM;
                break;
            case POS_RIGHT_TOP:
                mPosition = Position.RIGHT_TOP;
                break;
            case POS_RIGHT_BOTTOM:
                mPosition = Position.RIGHT_BOTTOM;
                break;
        }
        mRadius = (int) a.getDimension(R.styleable.ArcMenu_radius, TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, 100, getResources().getDisplayMetrics()));
        Log.i("cool", "position = " + mPosition + ", radius = " + mRadius);
        a.recycle();
	最后别忘了调用a.recycle方法
4. 我们想让自定义控件按照我们自己的想法展示出来就要重写viewGroup的onMeasure，onLayout，有的甚至是onDraw方法
	- onMeasure：对于viewGroup来说，onMeasure方法用于测量每个子view的大小，调用measureChild方法实现
	- 
		protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	        int count = getChildCount();
	        for (int i = 0; i < count; i++) {
	            measureChild(getChildAt(i), widthMeasureSpec, heightMeasureSpec);//测量child
	        }
	        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    	}
	- onLayout：我们拿住button为例。最重要的就是要调用mButton.layout(l, t, l + width, t + height); 方法，就是子view的layout方法，参数为左、上、右、下边位置，这样就决定了控件在屏幕的什么位置
	- 
		private void layoutCButton() {
	        mButton = getChildAt(0);
	        mButton.setOnClickListener(this);
	        int l = 0;
	        int t = 0;
	        int width = l + mButton.getMeasuredWidth();
	        int height = t + mButton.getMeasuredHeight();
	        switch (mPosition) {
	            case LEFT_TOP:
	                l = 0;
	                t = 0;
	                break;
	            case LEFT_BOTTOM:
	                l = 0;
	                t = getMeasuredHeight() - height;
	                break;
	            case RIGHT_TOP:
	                l = getMeasuredWidth() - width;
	                t = 0;
	                break;
	            case RIGHT_BOTTOM:
	                l = getMeasuredWidth() - width;
	                t = getMeasuredHeight() - height;
	                break;
	        }
	        mButton.layout(l, t, l + width, t + height);
    	}
4. 经过以上两个步骤，控件就会显示在屏幕上，下面就是要给控件添加点击监听
	-
		public class ArcMenu extends ViewGroup implements View.OnClickListener
		//重写其onClick方法
   		 @Override
    		public void onClick(View v) {
        	//mButton = findViewById(R.id.id_button);
        	rotateButton(v, 0f, 360f, 300);
        	toggleMenu(300);
    	}
	- 剩下的内容为实现动画部分了，此控件主要用了平移和旋转动画，注意这里并没有使用属性动画
	- 
		AnimationSet animationSet = new AnimationSet(true);//动画集，让一个view同一时间实现两个动画效果
		//平移动画
		tranAnim = new TranslateAnimation(xflag * cl, 0, yflag * ct, 0);
            tranAnim.setFillAfter(true);//设置此动画结束后view所在位置，true为在动画结束位置
            tranAnim.setDuration(duration);//动画持续时间
            tranAnim.setStartOffset((i * 150) / count);//延迟一点时间再启动动画
            tranAnim.setAnimationListener(new Animation.AnimationListener() {
                @Override
                public void onAnimationStart(Animation animation) {
                }
                @Override//动画结束时调用
                public void onAnimationEnd(Animation animation) {
                    if (mCurrentStatus == Status.CLOSE) {
                        childView.setVisibility(View.GONE);
                    }
                }
                @Override
                public void onAnimationRepeat(Animation animation) {
                }
            });
			//旋转动画
			RotateAnimation rotateAnim = new RotateAnimation(0, 720, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);//设置旋转起点、终点、旋转中心等
            tranAnim.setFillAfter(true);
            tranAnim.setDuration(duration);
            animationSet.addAnimation(rotateAnim);
            animationSet.addAnimation(tranAnim);
            childView.startAnimation(animationSet);//开始动画集

5. 最后一步，给自定义控件添加点击事件
我们先明确三个事情，1是要有一个接口，接口中有onClick方法；2是在控件类中我们控制什么时候回调这个接口的onClick方法；3是在监听这个点击事件的类中去实现具体的onClick方法
	- 定义一个接口
	-
		public interface OnMenuItemClickListener {
        	void onClick(View view, int pos);
    	}

	- 控件类ArcMenu中，OnMenuItemClickListener将拿到接口对象，并在自己的逻辑中去执行mMenuItemClickListener.onClick()方法
	- 
		private OnMenuItemClickListener mMenuItemClickListener;
			public void setOnMenuItemClickListener(OnMenuItemClickListener mMenuItemClickListener) {
        	this.mMenuItemClickListener = mMenuItemClickListener;
    	}
	- 在MainActivity中
	- 
		mArcMenu.setOnMenuItemClickListener(new ArcMenu.OnMenuItemClickListener() {
            @Override
            public void onClick(View view, int pos) {
                Toast.makeText(MainActivity.this, "" + view.getTag(), Toast.LENGTH_SHORT).show();
            }
        });

##总结
1.先写xml布局文件，这个布局是个viewgroup，里面包含了imageview  
2.然后写一个控件类，实现构造方法，重写onMeasure，onLayout方法  
3.onMeasure方法中，我们在for循环中measure所有的子view，以便可以在onLayout中使用子view的宽和高，在`onMeasure(int widthMeasureSpec, int heightMeasureSpec)`方法中，两个参数是由包含view的viewgroup传进来的，由ViewGroup中的layout_width，layout_height和padding以及View自身的layout_margin共同决定。所以widthMeasureSpec和heightMeasureSpec是一个复合值，它的高两位是模式（模式有三种：UNSPECIFIED（未确定尺寸如listview），EXACTLY（精确尺寸），AT_MOST（WRAP_CONTENT下的最大尺寸）），低30位是size大小。  
4.onLayout方法中，需要定位一个控件，需要上下左右边在屏幕中位置的数值，而这就需要onMeasure中测量出的子view的大小。  
5.设置viewgroup的onclick事件（一开始 子Button是设置为GONE，这样整个viewgroup就只有主Button才可以点击），点击后开始执行子 Button的旋转和平移的动画，并设置了子 Button的点击监听  
6.监听到点击事件后子 Button会消失




