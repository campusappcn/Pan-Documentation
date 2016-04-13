# Components 组件

详细介绍Pan的组成和使用方法。

## ViewModel和Pan工厂

ViewModel是整个框架中的核心，接口定义如下：

```Java
public interface ViewModel {

    ViewModel render();

    View getRootView();
}
```

render方法负责完成渲染操作，而rootView是当前渲染的最顶层的View。ViewModel的基础实现是[GeneralViewModel](javadoc/cn/campusapp/pan/GeneralViewModel.html)。

[GeneralViewModel](javadoc/cn/campusapp/pan/GeneralViewModel.html)提供了以下功能：

1. @Bind注解自动注入View。（底层由[Butterknife](http://jakewharton.github.io/butterknife/)支持）
2. 持有当前的Context，一定有Activity的引用，如果在Fragment中，则有Fragment的引用
3. 支持绑定一个[GeneralController](javadoc/cn/campusapp/pan/GeneralController.html)

一般来说，ViewModel的实例化可以通过Pan工厂来完成，只需告诉Pan工厂使用何类型的ViewModel，Pan工厂会负责实例化ViewModel。当然，也可以自行实例化ViewModel，这在内部类的场景时可以使用，然后将ViewModel对象传入Pan工厂，以进一步绑定View和Controller。

## Pan：PanActivity中使用

```Java
@Override
protected void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    MainViewModel vm = Pan.with(this, MainViewModel.class)
        .getViewModel(); //直接使用Activity的decorView的第一个子View作为rootView

    vm.render();
}
```
在Activity中，需要在onCreate中完成工厂实例化过程。直接使用无参数的getViewModel()工厂方法获得ViewModel实例。此时，getRootView()返回的是Activity的decorView的第一个子View，即整个页面的根View。

注意，这里要求Activity实现LifecycleObserved接口，可以通过直接继承[PanActivity](javadoc/cn/campusapp/pan/PanActivity.html)完成，也可以在自定义的Activity中，仿写[PanActivity](javadoc/cn/campusapp/pan/PanActivity.html)的代码并实现[LifecycleObserved](javadoc/cn/campusapp/pan/lifecycle/LifecycleObserved.html)接口。参考[自定义PanActivity/PanFragment](#panactivitypanfragment)。

## Pan：PanFragment中使用（Adapter同理）

```Java
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {

    MainViewModel vm = Pan.with(this, MainViewModel.class)
        .getViewModel(container, null, false);

    vm.render();

    return vm.getRootView(); //工厂方法根据@Xml注解创建的View
}

@Xml(R.layout.activity_main) //此时@Xml注解为必须
public static class MainViewModel extends GeneralViewModel{
	//...
}

```
在Fragment的onCreateView方法中，完成工厂的实例化过程。Fragment的onCreateView需要返回一个新创建的View，因此，Pan工厂在创建ViewModel的同时，可以完成创建View的过程。使用```getViewModel(ViewGroup, View, attach)```的重载，可以像调用inflat方法一样，创建一个新的View。而这个View的Xml，则来自于MainViewModel的[@Xml](javadoc/cn/campusapp/pan/annotaions/Xml.html)注解。

注意，这里要求Fragment实现[LifecycleObserved](javadoc/cn/campusapp/pan/lifecycle/LifecycleObserved.html)接口，可以通过直接继承[PanFramgent](javadoc/cn/campusapp/pan/PanFragment.html)完成，也可以在自定义的Fragment中，仿写[PanFramgent](javadoc/cn/campusapp/pan/PanFragment.html)的代码并实现[LifecycleObserved](javadoc/cn/campusapp/pan/lifecycle/LifecycleObserved.html)接口。参考[自定义PanActivity/PanFragment](#panactivitypanfragment)。

安卓中，界面部件的重要场景是在ListView的Adapter中，或者RecyclerView的场景中，在这些Adapter中，也需要实例化新的View，以及使用ViewHolder模式重用现有的View。**Pan框架原生支持ViewHolder模式，使用者无需再自己保存View的实例。参考[ViewHolder模式和嵌套使用](ViewHolder模式和嵌套使用/)**因此，在Adapter的getView方法中，可以直接使用本方法。即：

```Java
@Override
public View getView(int position, View convertView, ViewGroup parent) {
    return Pan.with(getActivity(), MainViewModel.class)
              .getViewModel(parent, convertView, false);//convertView会自动重用绑定的ViewModel
}
```


## Pan：View中使用

除了上述两种主要场景外，其他任何通过已经有实例化好的View的情况，都可以将View直接绑定到ViewModel上，以进行渲染和控制。此时，ViewHolder模式同样会发生作用，如果View上面已经绑定了一个相同类型的ViewModel，将会直接重用。

同时，Pan也支持一个View被多次绑定到不同类型的ViewModel上，所以，不用担心不同类型的ViewModel会被覆盖绑定的情况。但注意，同类型的ViewModel的重复绑定，是无效的。

```Java
Pan.with(getActivity(), MainViewModel.class)
    .getViewModel(view);

```

## Controller

Controller的职责监控各类交互/生命周期事件，并决定何时调用ViewModel的render。

用法也很简单，可以直接继承GeneralController，并通过泛型指定绑定的ViewModel

```Java
public class MainController extends GeneralController<MainViewModel> {
    @Override protected void bindEvents() {
    }
}

//在工厂方法直接指定Controller即可
Pan.with(getActivity(), MainViewModel.class)
	.controlledBy(MainController.class)
	.getViewModel();

```

Controller可以通过$vm直接获取到绑定的ViewModel对象，从而对其进行事件的绑定

## Controller：Activity/Fragment生命周期

Controller的一个核心功能是监控Activity/Fragment的生命周期。Controller需要得到某个周期的回调，直接通过实现与生命周期同名的接口即可，例如：

```Java
public class MainController extends GeneralController<MainViewModel> 

        //实现接口以监听
	    implements OnResume, 
	               OnVisible, 
	               OnRestart{

    @Override protected void bindEvents() {
    }

    @Override protected void onResume(){
    	//do something
    }

    @Override protected void onVisible(boolean isVisible){
    	//相当于Fragment的setUserVisibleHint方法
    }

    @Override protected void onRestart(){
    	//Activity的专属生命周期
    }
}
```
Pan框架将保证在特定的生命周期中对这些方法进行调用。所有可以使用的生命周期方法可以参考[lifecycle包](javadoc/cn/campusapp/pan/lifecycle/package-summary.html)

Activity和Fragment有很多生命周期是重合的，例如OnResume、OnStart，在Pan的实现中，当该生命周期到达时，Controller只会被调用一次。

一个Controller可以同时监听只属于Activity的生命周期，和只属于Fragment的生命周期，从而让Controller的可复用性大大提高。在Activity的环境中使用时，Fragment的生命周期将不会被调用到。

早期的Pan曾今区分过Activity和Fragment的具体调用，但实际使用过程中发现，这个区分并无必要，因为Fragment的onResume在安卓源码中就是被Activity的onResume所调用的，时机完全相同，区分反而增加了代码的复杂度，因此，在现在的版本中，重合的生命周期不加以区分。

背后的实现，是由Pan工厂维护了一个全局的```ACTIVITY_CONTROLLER_MAP```和```FRAGMENT_CONTROLLER_MAP```完成，可以参考源码。

## Controller：用户交互

Controller的另外一个基本功能是监听View的事件，例如Click事件：
```Java
public class MainController extends GeneralController<MainViewModel> {

    @Override protected void bindEvents() {
    	$vm.vButton.setOnClickListener(v -> /*...*/);
    }
}
```

这里需要注意的是，Controller不会跟随ViewModel一起被重用，因此当ViewModel被重用时，会重新实例化一个Controller以提供控制功能。由于ViewModel被重用时，其包含的View的实例未曾改变，所以setOnClickListener将会覆盖原有的Listener实例。

这在大部分情况下没有问题，除了以下两种情况：

1. 在重用时，由于某些状态，没有设置新的Listener，此时将仍然在使用老的Listener而导致bug
2. bindEvents使用的是addEventListener这类模式，即增加一个Listener，由于原先的列表并没有重置，会造成重复绑定相同的Listener的问题导致bug。

因此，在具体重用时，需要考虑清楚这些Listener的生命周期，并在bindEvents的逻辑中加以区分。未来的Pan或许会考虑提供内部的click方法以实现自动unbind，考虑到Pan的使用者仍然可能会调用原生的方法，这种方式也并不能从根本上解决问题。


## ViewModel和Controller的生命周期

Pan工厂会保证传入的GeneralViewModel将按照下列顺序初始化：

<img style="max-width: 60%;" src="https://img.alicdn.com/imgextra/i4/56380417/TB2qwPDmpXXXXceXpXXXXXXXXXX_!!56380417.png" alt="Pan工厂的执行顺序">

## 自定义Activity/Fragment

由于Java的单继承特性，如果某个Activity已经继承了某个自定义的Activity，或者某个类库的Activity，将无法直接继承PanActivity/PanFragment。

此时，开发者可以继承基类Activity，并实现[LifecycleObserved](javadoc/cn/campusapp/pan/lifecycle/LifecycleObserved.html)接口，同时，将PanActivity中的代码，直接拷贝到CustomActivity中，即可使用。Fragment同理。

```Java
public class PanCustomActivityBase extends CustomActivity implements LifecycleObserved{
	
	//拷贝PanActivity的所有源码到这里。

}
```
