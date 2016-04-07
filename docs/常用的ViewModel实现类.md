# 常用的ViewModel实现类

## GeneralViewModel

ViewModel理论上都可以直接继承[GeneralViewModel](javadoc/cn/campusapp/pan/GeneralViewModel.html)，完全遵循[ViewModel的生命周期](Components/#viewmodelcontroller)。

GeneralViewModel需要有一个无参的构造函数，以使工厂可以进行实例化。在作为非静态内部类的情况，工厂无法实例化该对象，此时可以自行实例化后，直接将对象传入工厂方法[with的重载](javadoc/cn/campusapp/pan/Pan.html#with-cn.campusapp.pan.lifecycle.LifecycleObserved-S-)。

GeneralViewModel可以绑定一个GeneralController，且要求GeneralController的泛型参数为此ViewModel。

GeneralViewModel的Controller必须负责调用ViewModel的render方法。即在特定的时机更新界面。

如果是简单的界面，在工厂方法的结尾可以直接进行第一次render即可，例如：
```Java
Pan.with(getActivity(), MainViewModel.class)
	.getViewModel()
	.render();
```

一个不错的做法是，在您的项目中继承GeneralViewModel实现一个CustomGeneralViewModel，所有的ViewModel继承这个订制的ViewModel，以方便后期有定制化需求时，可以方便的订制。

## AutoRenderViewModel

[AutoRenderViewModel](javadoc/cn/campusapp/pan/autorender/AutoRenderViewModel.html)也是一个常用的基类，其特点是会自动调用render进行渲染。自动渲染的时机如下：

1. Activity的OnResume（和Fragment的OnResume其实是同时机）
2. Fragment的setVisibleHint(true)的情况，即OnVisible接口方法得到参数true

AutoRenderViewModel的子类需要额外实现[loadDataQuickly](javadoc/cn/campusapp/pan/autorender/AutoRender.html#loadDataQuickly--)方法，会在每次render前调用，以适当的更新数据。注意，此方法会在主进程被调用，需要保证其不会阻塞。

当没有复杂的用户交互时，AutoRenderViewModel可以满足大多数情况下的页面重绘。

在一些场景下，AutoRenderViewModel的子类可能不希望自动绘制，可以通过重载[shouldRenderOnTrigger](javadoc/cn/campusapp/pan/autorender/AutoRender.html#shouldRenderOnTrigger--)以订制自动绘制的行为。

自动绘制的功能实现是通过[AutoRenderControllerLifecyclePlugin](javadoc/cn/campusapp/pan/autorender/AutoRenderControllerLifecyclePlugin.html)实现的，有兴趣可以看下源码。

## RecyclerViewModel

RecyclerViewModel和GeneralViewModel的实现完全一致，同时继承了RecyclerView.ViewHolder，可以方便的和RecyclerViewModel搭配使用。

## FactoryViewModel:订制Pan框架

Pan框架本身并不依赖GeneralViewModel的实现，因此，如果想深度订制Pan框架，例如订制ViewModel的生命周期，可以自行实现FactoryViewModel作为基类，Pan的用法可以保持一致。当然，推荐的做法仍然是继承GeneralViewModel来订制一个基类，并增加自己的额外的hook，从而让ViewModel符合文档中描述的生命周期。
