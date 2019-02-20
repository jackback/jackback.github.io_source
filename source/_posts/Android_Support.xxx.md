---
title: Android支持库(转)
date: 2017-04-13 07:42:30
tags: [Android Support]
categories: Android
---


随着 Android 5.0 Lollipop 的发布，Android 又为我们提供了更多的支持包，但是我相信大部分开发者都同我之前一样不知道这些包里究竟有些什么东西，我们应该在什么时候使用它。现在，我们就来逐个看看每一个版本的 Support 包中所拥有的东西，让用到他的时候不再迷茫。

首先，你需要了解每一个 Support 包版本后缀 vX 所代表的含义。当然我相信来看博客的诸位都一定知道 Android 对于每一个版本都有一个版本号，例如2.1是7，4.0是14，5.0是21。而这里，v 之后的数字，就代表着他能够被使用的最低版本等级，之所以无法在更低版本进行使用的原因，是因为随着版本的升级，在新版本中有很多之前不支持的特性或者 API，因此如果你在老版本中使用了这些支持包，就可能会导致应用崩溃。

<!-- more -->

现在，我们从头开始逐个浏览目前所支持的 support 包:
##### 1. support-v4
	support-v4包算是 Android 最低等级的支持包。所谓的 v4,代表着它最低支持 Android1.6(API Level4)，这个版本算得上是一个真正意义上比较成熟的 Android版本，更何况现在我们写应用的时候一般都只最低支持到 Android 2.x 系统，对于1.x 的系统基本已经已经完全抛弃了，因此你可已经他作为最基本的系统组件使用。
	
	在 support-v4包中，它所拥有的类还是很多的，主要包含了对应用组件的支持，用户交互体验的一些工具类，一些数据网络方面的工具类，相面我们将详细来看看它里面具体的一些类。
	
	1. 系统组件部分
	Fragment:其实 Fragment 是直到 Android3.0才正式进入 Android 框架体系的，但是 Android 为了低版本的兼容，因此他帮我们在低版本也适配了 Fragment 框架
	NotificationCompat:这是通知栏的一些适配，可以帮助你在低版本的通知栏显示更加丰富的信息
	LocalBroadcastManager:这个是用于本地广播通知的，当你希望发送的通知只被本应用接收时，你就应该使用它
	
	2. 用户界面交互部分
	ViewPager，这个相信我不用怎么说了，他主要用于帮助我们进行界面间的滑动交互
	PagerTitleStrip,PagerTabStrip 这两个算是 ViewPager 的帮助类吧，他们的作用是进行 Tab 栏的切换辅助显示
	DrawerLayout，主要用于侧滑栏的实现
	SlidingPaneLayout,这个类也是用于侧滑栏的实现，和 DrawerLayout 不同的是，DrawerLayout 侧滑栏出来的时候，默认是覆盖在当前页面上，而 SlidingPaneLayout 则是会将当前页面移走。
	
	3. Accessbility访问的帮助类
	ExploreByTouchHelper，帮助自定义 View 实现 Accessibility 的工具类
	AccessbilityEventCompat, AccessbilityNodeInfoCompat, AccessbilityNodeProviderCompat, AccessbilityDelegateCompat，这几个都是用作 Accessibility 功能适配的类
	
	4. 数据访问帮助类
	Loader，主要用于异步加载数据
	FileProvider，提供应用间的文件分享功能

 

##### 2. support-v7
	1.Appcompat
	这个包的主要作用是为了在低版本实现 Android 的 Holo 风格界面而引入的，与之类似的有一个开源项目叫做 SherlockActionbar
	
	2.CardView
	卡片布局是最近在 android5.0发布的时候才引入的新包，在我看来，他主要效果是让应用进行卡片花显示
	
	3.GridLayout
	网格布局能够帮助你将整个布局按照一格两格的格子形式进行排列
	
	4.MediaRouter
	这个布局主要是用来支持 GoogleCast 的，主要用于进行设备间的音频，视频交换显示
	
	5.Palette
	这个包也是最新出来的，他的作用是帮助 Android 实现他的 MaterialDesign，让你的 Actionbar 能够根据界面进行对应的颜色改变
	
	7.RecyclerView
	这个包同样也是刚出来的，他的作用是替换 ListView 和 GridView，但是可惜是没有实现 OnItemClick 这些接口，你需要自己处理它


##### 3. support-v8
	support-v8中其实只有一格特性，就是用来渲染脚本

##### 4. support-v13
	这个包的作用主要是为 Android3.2级以上的系统提供更多地 Framgnet 特性支持，使用它的原因在于，android-support-v4包中虽然也对 Fragment 做了支持，由于要兼容低版本，导致他是自行实现的 Fragment 效果，在高版本的 Fragment 的一些特性丢失了，而对于 v13以上的 sdk 版本，我们可以使用更加有效，特性更多的代码


##### 5. support-v17
	这个包得主要作用是用于支持电视设备，并为电视设备提供了很多组件
	例如下面的:
	BrowseFragment， DetailFragment， PlaybasckOverlayFragment， SearchFragment
	但是原谅我没有做过 Android TV 开发，我也不知道他们的用处是什么，如果真的想要查看，请去官网看看吧

[https://developer.android.com/topic/libraries/support-library/index.html](https://developer.android.com/topic/libraries/support-library/index.html "Android支持库")



转自[http://www.2cto.com/kf/201411/350928.html](http://www.2cto.com/kf/201411/350928.html)



## 附：API与Android版本对应表：
	Code name	        Version	         API level
	Nougat	            7.1	             API level 25
	Nougat	            7.0	             API level 24
	Marshmallow	        6.0	             API level 23
	Lollipop	        5.1	             API level 22
	Lollipop	        5.0	             API level 21, NDK 10
	KitKat	            4.4 - 4.4.4	     API level 19
	Jelly Bean	        4.3.x	         API level 18
	Jelly Bean	        4.2.x	         API level 17
	Jelly Bean	        4.1.x	         API level 16
	Ice Cream Sandwich	4.0.3 - 4.0.4	 API level 15, NDK 8
	Ice Cream Sandwich	4.0.1 - 4.0.2	 API level 14, NDK 7
	Honeycomb	        3.2.x	         API level 13
	Honeycomb	        3.1	             API level 12, NDK 6
	Honeycomb	        3.0	             API level 11
	Gingerbread	        2.3.3 - 2.3.7	 API level 10
	Gingerbread	        2.3 - 2.3.2	     API level 9, NDK 5
	Froyo	            2.2.x	         API level 8, NDK 4
	Eclair	            2.1	             API level 7, NDK 3
	Eclair	            2.0.1	         API level 6
	Eclair	            2.0	             API level 5
	Donut	            1.6	             API level 4, NDK 2
	Cupcake	            1.5	             API level 3, NDK 1
	(no code name)	    1.1	             API level 2
	(no code name)	    1.0              API level 1

