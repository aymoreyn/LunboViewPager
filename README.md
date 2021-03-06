﻿# LunboViewPager
### 对ViewPager和ViewPagerIndicator的简单封装，实现轮播图效果。

- 轮播图可以直接在Activity/Fragment中，也可以是RecyclerView的一个Item。

- 该有的基本都有，页码指示，左右无限循环翻页，定时自动翻页，用手指翻页时暂停自动翻页，只有一张图片时变为静态相框，解决原生ViewPager当页数为2时的滑动Bug。

- 提供CirclePageIndicator（圆点）指示器

![Alt text](https://raw.githubusercontent.com/xingda920813/LunboViewPager/master/video.gif)

## 引入
### 1.添加二进制

build.gradle中添加

    compile 'com.xdandroid:lunboviewpager:+'

### 2.布局文件(Activity/Fragment/RecyclerView的Item)

将ViewPager与CirclePageIndicator元素并列。

	<FrameLayout
    	android:layout_width="match_parent"
    	android:layout_height="match_parent"
    	android:background="@android:color/white">

    	<android.support.v4.view.ViewPager
    		android:id="@+id/vp_lunbo"
    		android:layout_width="352dp"
    		android:layout_height="176dp"/>

    	<com.xdandroid.lunboviewpager.CirclePageIndicator
    		android:id="@+id/indicator_lunbo"
    		android:layout_width="wrap_content"
    		android:layout_height="wrap_content"
    		android:layout_gravity="bottom|right"
    		android:layout_marginBottom="6dp"
    		android:layout_marginRight="2dp"/>
    </FrameLayout>

### 4. 布局文件(ViewPager的每一页，通常由一个图片控件和少量其他控件组成)

	<?xml version="1.0" encoding="utf-8"?>
	<ImageView
   		xmlns:android="http://schemas.android.com/apk/res/android"
  		android:id="@+id/iv_lunbo"
    	android:scaleType="centerCrop"
    	android:layout_height="176dp"
    	android:layout_width="350dp"/>

使用方法详见Sample。

## 直接在Activity/Fragment内使用

#### 1.给ViewPager和Indicator设置需要的自定义属性（OnPageChangeListener, Indicator的填充颜色, etc.）

Indicator的可设置项参照JakeWharton/ViewPagerIndicator提供的API。

[https://github.com/JakeWharton/ViewPagerIndicator](https://github.com/JakeWharton/ViewPagerIndicator "JakeWharton/ViewPagerIndicator")

相对于JakeWharton的版本, 增加了对wrap_content的支持;

增加了一个API, 用于设置和得到圆点之间的距离相对于圆点半径的倍数 :

```
void CirclePageIndicator.setDistanceBetweenCircles(double timesToRadius_multiplier);

double CirclePageIndicator.getDistanceBetweenCirclesMultiplier()
```

```
//设置OnPageChangeListener
vp_lunbo.setOnPageChangeListener(onPageChangeListener);

//设置当前被选中的圆点的填充颜色
indicator_lunbo.setFillColor(getResources().getColor(R.color.colorAccent));

//设置圆点半径
indicator_lunbo.setRadius(UIUtils.dp2px(this, 3.25f));

//设置圆点之间的距离相对于圆点半径的倍数
indicator_lunbo.setDistanceBetweenCircles(3.0);
```

#### 2.构建Adapter

public Adapter(Context context);

需实现Adapter里的4个方法：

- protected abstract int getViewPagerItemLayoutResId();		//指定ViewPager的每一页的Layout资源ID

- protected abstract View showImage(View inflatedLayout, int position);

inflatedLayout即根据上面的资源ID渲染出来的View，开发者需要通过findViewById找到布局中的图片控件，再使用图片加载框架（UIL、Picasso、Fresco等）把图片加载进去，最后返回图片控件。

- protected abstract int getImageCount();	//总页数

- protected abstract void onImageClick(View view, int position);		//OnClickListener

示例：

	new Adapter(MainActivity.this) {
        protected View showImage(View inflatedLayout, int position) {
            ImageView imageView = (ImageView) inflatedLayout.findViewById(R.id.iv_lunbo);
            Picasso.with(MainActivity.this).load(list.get(position).getImageResId()).into(imageView);
            return imageView;
        }
        protected int getViewPagerItemLayoutResId() {return R.layout.item_in_viewpager;}
        protected int getImageCount() {return list.size();}
        protected void onImageClick(View view, int position) {
			Snackbar.make(view,list.get(position).getMessage(),Snackbar.LENGTH_SHORT).show();
		}
    }

#### 3.构建Proxy

public Proxy(List<${JavaBean}> list, int interval, Adapter adapter);

interval为轮播时间间隔。

#### 4.开始轮播

void Proxy.onBindView(ViewPager viewPager,View indicator);

## 轮播图ViewPager作为RecyclerView的一个Item来使用

#### 1.在RecyclerView.Adapter的构造方法里构建com.xdandroid.lunboviewpager.Adapter

public com.xdandroid.lunboviewpager.Adapter(Context context);

需实现com.xdandroid.lunboviewpager.Adapter里的4个方法：

- protected abstract int getViewPagerItemLayoutResId();		//指定ViewPager的每一页的Layout资源ID

- protected abstract View showImage(View inflatedLayout, int position);

inflatedLayout即根据上面的资源ID渲染出来的View，开发者需要通过findViewById找到布局中的图片控件，再使用图片加载框架（UIL、Picasso、Fresco等）把图片加载进去，最后返回图片控件。

- protected abstract int getImageCount();	//总页数

- protected abstract void onImageClick(View view, int position);		//OnClickListener

示例：

	new com.xdandroid.lunboviewpager.Adapter(context) {
        protected View showImage(View inflatedLayout, int position) {
            ImageView imageView = (ImageView) inflatedLayout.findViewById(R.id.iv_lunbo);
            Picasso.with(context).load(list.get(position).getImageResId()).into(imageView);
            return imageView;
        }
        protected int getViewPagerItemLayoutResId() {return R.layout.item_in_viewpager;}
        protected int getImageCount() {return list.size();}
        protected void onImageClick(View view, int position) {
			Snackbar.make(view,list.get(position).getMessage(),Snackbar.LENGTH_SHORT).show();
		}
    }

#### 2.在RecyclerView.Adapter的构造方法里构建Proxy

public Proxy(List<${JavaBean}> list, int interval, com.xdandroid.lunboviewpager.Adapter adapter);

interval为轮播时间间隔。

#### 3.onBindViewHolder

先给ViewPager和Indicator设置需要的自定义属性（OnPageChangeListener, Indicator的填充颜色, etc.）

Indicator的可设置项参照JakeWharton/ViewPagerIndicator提供的API。

[https://github.com/JakeWharton/ViewPagerIndicator](https://github.com/JakeWharton/ViewPagerIndicator "JakeWharton/ViewPagerIndicator")

相对于JakeWharton的版本, 增加了对wrap_content的支持;

增加了一个API, 用于设置和得到圆点之间的距离相对于圆点半径的倍数 :

```
void CirclePageIndicator.setDistanceBetweenCircles(double timesToRadius_multiplier);

double CirclePageIndicator.getDistanceBetweenCirclesMultiplier()
```

最后，添加proxy.onBindView(holder.viewPager,holder.indicator);

示例：

```
  @Override
  public void onBindViewHolder(final MainAdapter.ViewHolder holder, int position) {
      //设置当前被选中的圆点的填充颜色
      holder.indicator_lunbo.setFillColor(context.getResources().getColor(R.color.colorAccent));

      //设置圆点半径
      holder.indicator_lunbo.setRadius(UIUtils.dp2px(holder.indicator_lunbo.getContext(), 3.25f));

      //设置圆点之间的距离相对于圆点半径的倍数
      holder.indicator_lunbo.setDistanceBetweenCircles(3.0);

      proxy.onBindView(holder.vp_lunbo,holder.indicator_lunbo);
  }
```
