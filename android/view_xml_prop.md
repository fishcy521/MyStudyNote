# View 类的XML属性、相关方法和说明
###对齐
表格可以指定对齐方式

|           XML属性           |         相关方法                       |               说明                                 |
| :-------------------------- |:-------------------------------------- | :--------------------------------------------------|
| android:alpha               | setAlpha(float)                        | 设置该证件的透明度                                 |
| android:background          | setBackgroundResource(int)             | 设置该组件的背景颜色                               |
| android:clickable           | setClickable(boolean)                  | 设置该组件是否可以激发单击事件                     |
| android:contentDescription  | setContentDescription(CharSequence)    | 设置该组件的内容描述信息                           |
| android:drawingCacheQuality | setContentCacheQuality(int)            | 设置该组件所使用的绘制缓存的质量                   |
| android:elevation           | setElevation(float)                    | 设置该组件“浮”起来的高度，通过设置该属性可以让组件呈现3D效果，5.0Material Design中的功能，组件垂直屏幕"浮"起来|
| android:fadeScrollbars      | setScrollbarFadingEnable(boolean)      | 当不使用该组件的滚动条时,是否淡出显示滚动条        |
| android:fadingEdge          | setVerticalFadingEdgeEnable(boolean)   | 设置滚动该组件的组件边界是否使用淡出效果           |
| android:fadingEdgeLength    | getVerticalFadingEdgeLength()          | 设置淡出边界的长度                                 |
| android:focusable           | setFocusable(boolean)                  | 设置该组件是否可以得到焦点                         |
| android:focusableInTouchMode| setFocusableInTouchMode(boolean)       | 设置该组件在触摸模式下是否可以得到焦点             |
| android:id                  | setId(int)                             | 设置该组件的唯一标识。在java代码中通过findViewById获取|
| android:isScrollContainer   | setScrollContainer(boolean)            | 设置该组件是否会强制手机屏幕一直打开               |
| android:keepScreenOn        | setKeepScreenOn(boolean)               | 设置该组件是否会强制手机屏幕一直打开               |
| android:longClickable       | setLongClickable(boolean)              | 设置该组件是否可以相应长单击事件                   |
| android:minHeight           | setMinimumHeight(int)                  | 设置该组件的最小高度                               |
| android:minWidth            | setminimumWidth(int)                   | 设置该组件的最小宽度                               |
| android:nextFocusDown       | setNextFocusDownId(int)                | 设置焦点在该组件上，且单击向下键时获得焦点的组件Id |
| android:nextFocusLeft       | setNextFocusLeftId(int)                | 设置焦点在该组件上，且单击向左键时获得焦点的组件Id |
| android:nextFocusRight      | setNextFocusRight(int)                 | 设置焦点在该组件上，且单击向右时获得焦点的组件Id   |
| android:nextFocusUp         | setNextFocusUp(int)                    | 设置焦点在该组件上，且单击向上时获得焦点组件id     |
| android:onClick         |                     | 为该组件的单击事件绑定监听器     |
| android:padding         | setPadding(int,int,int,int)                    | 在附件的四边设置填充区域     |
| android:paddingBottom         | setPadding(int,int,int,int) | 在组件的下边设置填充区域     |
| android:paddingLeft         | setPadding(int,int,int,int) | 在组件的左边设置填充区域     |
| android:paddingRight         | setPadding(int,int,int,int) | 在组件的右边设置填充区域     |
| android:paddingTop         | setPadding(int,int,int,int) | 在组件的上边设置填充区域     |
| android:rotation         | setRotation(float) | 设置组件旋转的角度    |
| android:rotationX         | setRotationX(float) | 设置该组件绕X轴旋转的角度     |
| android:rotationY         | setRotationY(float) | 设置该组件绕Y轴旋转的角度     |
| android:saveEnabled         | setSaveEnabled(float) | 如果设置为false，那么当该组件被冻结时不会保存它的状态    |
| android:scaleX         | setScaleX(float) | 设置该组件在水平方向的缩放比     |
| android:scaleY         | setScaleY(float) | 设置该组件在垂直方向的缩放比     |
| android:scrollX         |  | 该组件初始化后的水平滚动便宜     |
| android:scrollY         |  | 该组件初始化后的垂直滚动便宜     |
| android:scrollbarAlwaysDrawHorizontalTrack         |  | 设置组件是否显示水平滚动条的轨道     |
| android:scrollbarAlwaysDrawVerticalTrack         |  | 设置组件是否显示垂直滚动条的轨道     |
| android:scrollbarDefaultDelayBeforeFade         | setScrollBarDefaultDelayBeforeFade(int) | 设置滚动条在淡出隐藏之前延迟多少毫秒     |
| android:scrollbarFadeDuration         | setScrollBarFadeDuration(int) | 设置滚动条在淡出隐藏过程需要多少秒     |
| android:scrollbarSize         | setScrollBarSize(int) | 设置垂直滚动条的宽度和水平滚动条的高度     |
| android:scrollbarStyle  | setScrollBarStyle(int) | 设置滚动条的风格和位置。属性有以下值 insideOverlay,insideInset,outsideOverlay,outsideInset	|
| android:scrollbarThumbHorizontal |  | 设置该组件的水平滚动条的滑块对应的Drawable对象 |
| android:scrollbarThumbVertial |  | 设置该组件的垂直滚动条的滑块对应的Drawable对象 |
| android:scrollbarTrackHorizontal |  | 设置该组件的水平滚动条的轨道对应的Drawable对象 |
| android:scrollbarTrackVertical |  | 设置该组件的垂直滚动条的轨道对应的Drawable对象 |
| android:scrollbars |  | 定义该组件滚动时显示几个滚动条。有以下值   none: 不显示滚动条  horizontal:显示水平滚动条 vertical: 显示垂直滚动条 |
| android:soundEffectsEnable | setSoundEffectsEnabled(boolean) | 设置该组件被单击时是否使用音效 |
| android:tag | | 为该组件设置一个字符串类型的tag值。接下来可通过View的getTag()获取该字符串，或通过findViewWithTag()查找该组件 |
| android:transformPivotX | setPivotX(float) | 设置该组件旋转时旋转中心的X坐标 |
| android:transformPivotY | setPivotY(float) | 设置该组件旋转时旋转中心的Y坐标 |
| android:translationX | setTranslationX(float) | 设置该组件在X方向上的位移 |
| android:translationY | setTranslationY(float) | 设置该组件在Y方向上的位移 |
| android:translationZ | setTranslationZ(float) | 设置该组件在Z方向(垂直屏幕方向)上的位移 |
| android:visibility | setVisibility(int) | 设置该组件是否可见 |