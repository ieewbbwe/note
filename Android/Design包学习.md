#Design 包下的新控件

Design 和5.0一起推出很久了，项目中也用了许多该包下的资源，并且现在的设计师也在依照Material Design进行设计，今天来总结一些Design包中一些控件的用法和使用效果。
https://developer.android.com/reference/android/support/design/widget/package-summary.html

##1 AppBarLayout

继承自LinearLayout,布局方向为垂直，它可以让你定制当某个可滚动View的滚动手势发生变化时，其内部的子View实现何种动作。

内部的子View可以通过添加 app:layout_scrollFlags 来设置动作。下面列举layout_scrollFlags有哪些值可以设置。

1. scroll:值设为scroll的View会跟随滚动事件一起发生移动。（但是发现当内部是ListView 的时候不执行滑动）
2. enterAlways:值设为enterAlways的View,当ScrollView往下滚动时，该View会直接往下滚动。而不用考虑ScrollView是否在滚动。
3. exitUntilCollapsed：值设为exitUntilCollapsed的View，当这个View要往上逐渐“消逝”时，会一直往上滑动，直到剩下的的高度达到它的最小高度后，再响应ScrollView的内部滑动事件。
4. enterAlwaysCollapsed：是enterAlways的附加选项，一般跟enterAlways一起使用，它是指，View在往下“出现”的时候，首先是enterAlways效果，当View的高度达到最小高度时，View就暂时不去往下滚动，直到ScrollView滑动到顶部不再滑动时，View再继续往下滑动，直到滑到View的顶部结束。

3 BottomSheetDialog
##4 CollapsingToolbarLayout

CollapsingToolbarLayout是用来对Toolbar进行再次包装的ViewGroup，主要是用于实现折叠（其实就是看起来像伸缩~）的App Bar效果。它需要放在AppBarLayout布局里面，并且作为AppBarLayout的直接子View。

5 CoordinatorLayout
6 FloatingActionButton
7 NavigationView
8 Snackbar
9 TabItem&TabLayout
10 TextInputEditText&TextInputLayout
