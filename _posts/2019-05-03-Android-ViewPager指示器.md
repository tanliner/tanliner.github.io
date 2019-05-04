---
layout:     post
title:      Android-ViewPager指示器
subtitle:   ViewPager Indicator
date:       2019-05-03
header-img: img/post-bg-desk.jpg
catalog:    true
tags:
    - Android
    - ViewPager
    - Note

---

### 背景
ViewPager 在应用中使用较为广泛，诸如页面轮播图，App引导页，大图预览等。其中指示器页比较重要
它用于提示用户当前的页面状态，使得应用更具交互性。

### PagerTitleStrip
PagerTitleStrip 内置的指示器，布局作为ViewPager的一个子View，需要设置android:layout_gravity=TOP/BOTTOM
但效果总不是太理想，多数情况，大都自定义一个Indicator，放在ViewPager的外部，通过ViewPager的onScroll方法，更新Indicator的位置效果等


### 自定义 Indicator
初步归纳为3步，自定义View，布局文件引用自定义View，关联自定义View与ViewPager

##### 自定义 PageIndicator
```
class PageIndicator extends RelativeLayout {
    @Override
    PageIndicator(context, attrs, defStyleAttr) {}
    // ...
}
```

##### 布局文件引用
```
    <com.ltan.music.view.PageIndicator
            android:id="@+id/music_indicator"
            android:layout_width="match_parent"
            android:layout_height="48dp" />
    <androidx.viewpager.widget.ViewPager
            android:id="@+id/music_view_pager"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
```

##### 关联ViewPager
```
mPageIndicator.setViewPager(mViewPager)
// 并设置指示器的内容
mPageIndicator.addIndicators(getIndicators())


private fun getIndicators(): ArrayList<Item> {
    val items = ArrayList<Item>()
    return items
}
```

### 控制Indicator行为
重写onPageScrolled，点击Indicator实现ViewPager翻页，跨页跳转时的属性动画

##### 关键方法：onPageScrolled()
ViewPager.OnPageChangeListener.onPageScrolled()
```
override fun onPageScrolled(position: Int, positionOffset: Float, positionOffsetPixels: Int) {       
    val litem = getIndicator(position)
    val ritem = getIndicator(position + 1)
    litem?.setActiveState(1 - positionOffset)
    ritem?.setActiveState(positionOffset)
}
```
##### `跨页切换 ViewPager 动画`
当前页面个数为4，从第一页直接跳转到第4页时，能明显感觉到2、3 两页页显示出来了。
```
mViewPager.setCurrentItem(toItemIndex, true)
```
其原因就是直接在onPageScrolled在这种场景下发生了回调，从而导致Indicator的行为改变。所以，应该在点击顶部指示器的时候，
屏蔽掉onPageScrolled里对指示器的控制，在点击时添加指示器的动画
```
animF.addListener(object : SimpleAnimator() {
    override fun onAnimationEnd(animation: Animator?) {
        indicator.isMenuItemClicked = false
    }
})
animF.start()

// onPageScrolled() 里
if (isMenuItemClicked) {
    // click the menu item, do animation directly
    return
}
litem?.setActiveState(1 - positionOffset)
ritem?.setActiveState(positionOffset)
```
##### `属性动画`
```
ValueAnimator.ofFloat(0, 1)
ValueAnimator.setDuration(250) // 默认的页面切换时间
ValueAnimator.addAmimator(onAnimationEnd())
```

### 如何获取进入界面和退出界面

由于ViewPager的currentItem在滑动过程中是变化的，不能作为参考。
`从第0页滑动到第1页`position:0 , offset: 0.0->1.0f, currentItem: 当offset>0.5f的时候，会被更新为1<br>
`从第1页滑动到第0页`position:0 , offset: 1.0->0.0f, currentItem: 当offset<0.5f的时候，会被更新为0<br>

`从第1页滑动到第2页`position:1 , offset: 0.0->1.0f, currentItem: 当offset>0.5f的时候，会被更新为2<br>
`从第2页滑动到第1页`position:1 , offset: 1.0->0.0f, currentItem: 当offset<0.5f的时候，会被更新为1<br>

`从第2页滑动到第3页`position:2 , offset: 0.0->1.0f, currentItem: 当offset>0.5f的时候，会被更新为3<br>
`从第3页滑动到第2页`position:2 , offset: 1.0->0.0f, currentItem: 当offset<0.5f的时候，会被更新为2<br>

可以看出，position在左滑或者右滑时，始终保持不变，offset的变化值始终是滑动过程中的右边页面。<br>
即：能定位到当前滑动页的左边和右边页，从而有效的控制指示器的行为

以`position = 1`举例：
如果offset从0到1，说明是从第1页滑向第2页。本次滑动过程中: 左边页是第1页，右边是第2页，offset从0~1，进入动画
如果offset从1到0，说明是从第2页滑向第1页。本次滑动过程中: 左边页是第1页，右边是第2页，offset从1~0，退出动画

### 总结
关键数据 position，是onPageScrolled的第一个参数`onPageScrolled(int position, *, **)`

滑动过程中的左边页的需要即为 `position`, 而右边页对应就是 `position + 1`
所以才有下面的写法
```
val litem = getIndicator(position)
val ritem = getIndicator(position + 1)
```
