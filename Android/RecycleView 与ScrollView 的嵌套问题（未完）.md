## RecycleView 与ScrollView 的嵌套问题
2.1 ScrollView嵌套RecycleView 问题

不要嵌套！不要嵌套！不要嵌套！ScrollView 的高度测量机制会引发很多问题！！宁可自定义View也不要嵌套！因为会出现RecycleView 的高度被错误测量的问题，如果你要需要加上item间距，然后item在用上CardView 的话，相信我，你会调UI欲仙欲死的。

如果你一定要嵌套，也不是没有解决办法。相信大家遇到过ScrollView嵌套ListView的情况，只需要自定义一个ListView，并重写onMeasure 方法，返回最大高度。

但是在RecycleView 中这样做是无效的，RecycleView把布局交给布局管理器去维护，所以这里我们要自定义布局管理器，测量每一个Item高，此处参考了鸿阳大神的写法，链接在本文最上面有。
2.2 RecycleView 添加item间距问题
2.3 notify 更新数据集问题
2.4 添加 头 | 脚 问题 - 添加头\脚View parent问题2.5 下拉\上拉刷新问题