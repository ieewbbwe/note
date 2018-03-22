# ViewPager onPageSelected 的调用时机
## 前言

TabLayout + ViewPager + fragment 的情景下ClickTab和ScrollPage触发的onPageSelected 不一样。

ClickTab：先走预加载页面的生命周期在走onPageSelected
ScrollPage：先执行onPageSelected 在走的预加载页面的生命周期

## 内容

先抛了一个大坑出来，不信？我跑给你看。代码很简单不想自己实现的话就直接看结果吧。

### ClickTab 的方式
```
02-13 08:38:10.825 7928-7928/com.webber.demos D/picher: Fragment3------->>setUserVisibleHint:false
02-13 08:38:10.825 7928-7928/com.webber.demos D/picher: Fragment1------->>setUserVisibleHint:false
02-13 08:38:10.826 7928-7928/com.webber.demos D/picher: Fragment2------->>setUserVisibleHint:true
02-13 08:38:10.834 7928-7928/com.webber.demos D/picher: Fragment3------->>onViewCreated
02-13 08:38:10.834 7928-7928/com.webber.demos D/picher: Fragment3------->>onActivityCreated
02-13 08:38:10.835 7928-7928/com.webber.demos D/picher: Fragment3------->>onStart
02-13 08:38:10.835 7928-7928/com.webber.demos D/picher: Fragment3------->>onResume
02-13 08:38:10.835 7928-7928/com.webber.demos D/picher: onPageSelected:1

```
### SwipePage 的方式

```
02-13 08:39:25.746 7928-7928/com.webber.demos D/picher: onPageSelected:1
02-13 08:39:25.876 7928-7928/com.webber.demos D/picher: Fragment3------->>setUserVisibleHint:false
02-13 08:39:25.876 7928-7928/com.webber.demos D/picher: Fragment1------->>setUserVisibleHint:false
02-13 08:39:25.876 7928-7928/com.webber.demos D/picher: Fragment2------->>setUserVisibleHint:true
02-13 08:39:25.880 7928-7928/com.webber.demos D/picher: Fragment3------->>onViewCreated
02-13 08:39:25.880 7928-7928/com.webber.demos D/picher: Fragment3------->>onActivityCreated
02-13 08:39:25.880 7928-7928/com.webber.demos D/picher: Fragment3------->>onStart
02-13 08:39:25.880 7928-7928/com.webber.demos D/picher: Fragment3------->>onResume
``` 

看到了吧！！

点击tab的时候先加载了Fragment3，然后执行了onPageSelected；
swipePage 的时候先走了onPageSelected 然后加载Fragmnet3。

蛋疼的一批，都是切换页面，两种方式的逻辑竟然不一样！！这就坑大了！！这就说明在某些情况下swipe和click的处理逻辑必须要不同。

举个会出现问题的例子：
需求：

### 原因

找找原因吧，翻遍了stackOverFlow 也没找到答案，

## 总结