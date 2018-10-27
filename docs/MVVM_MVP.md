# MVP&MVVM: 让Pan按照你喜欢的模型工作

Pan的模型是MV*，意味着，只要达到了Pan的初心：分离Activity的代码，Pan可以被用来以任何你喜欢的模型来编写代码。例如，你可以把Pan当做一个MVP框架来使用，也可以当做一个MVVM的框架来使用，只需要很少、甚至不需要代码改动。

## Pan当做MVP框架(复刻[Mosby](http://hannesdorfmann.com/mosby/))

ViewModel本身理解为MVP中的View。ViewModel的render方法相当于全量刷新界面，而更局部刷新可以在子类提供诸如renderUser等方法。而Controller则可以类比到Presenter。因此，符合MVP框架的代码可以直接迁移成Pan框架的代码，你会发现是等价的。不仅如此，Pan框架由于强制代码分离，会使得原先仍然有很多代码的Activity，变的只需要寥寥几行。

**例如，我们可以直接迁移最近比较火的MVP框架[Mosby的Getting Started](http://hannesdorfmann.com/mosby/getting-started/)，你可以类似的写出下面的代码：**

首先是View接口，定义View的渲染行为
```Java
// View interface
public interface HelloWorldView {

  // displays "Hello" greeting text in red text color
  void showHello(String greetingText);

  // displays "Goodbye" greeting text in blue text color
  void showGoodbye(String greetingText);
}
```

然后是View接口的实现。Mosby是把实现交给了Activity，而在Pan中，对应的渲染逻辑实现则是ViewModel，从而使得Activity没有更多的代码。
当然Mosby也是支持不交给Activity的，用别的实现类；而Pan则是强制分离。
```Java
public class HelloWorldViewImpl extends GeneralViewModel implements HelloWorldView{

	@BindView(R.id.greetingTextView) TextView vGreetingTextView;
	@BindView(R.id.helloButton) Button vHelloBtn;

	@Override void showHello(String greetingText){
		greetingTextView.setTextColor(Color.RED);
    	greetingTextView.setText(greetingText);
	}

	@Override public void showGoodbye(String greetingText){
		greetingTextView.setTextColor(Color.BLUE);
		greetingTextView.setText(greetingText);
	}	

	@Override public void render(){
		//It is more clear that put 'greetingText' and 'isHelloOrGoodbye' as a member of ViewModel in Pan's philosophy,
		//and render method will use current state to choose show which color.

		//But anyway, this is the example for Mosby.
	}
}
```

接下来是Presenter，实际上Mosby的Presenter处理的是将异步请求的结果绑定到View的渲染方法上的作用，而这个作用在Pan中由Controller实现。这里可以直接将Mosby的Presenter的代码拷贝过来。
Mosby的点击事件绑定也交给了Activity，如果迁移到Pan，就需要放到Controller的bindEvents方法中来。你可以在bindEvents中使用Butterknife，这样就更像Mosby的做法。
```Java
public class HellowWorldPresenter extends GeneralController<HelloWorldViewImpl>{

	// Greeting Task is "business logic"
	private GreetingGeneratorTask greetingTask;

	private void cancelGreetingTaskIfRunning(){
		if (greetingTask != null){
			greetingTask.cancel(true);
	    }
	}

	public void greetHello(){
		cancelGreetingTaskIfRunning();

		greetingTask = new GreetingGeneratorTask("Hello", new GreetingTaskListener(){
			public void onGreetingGenerated(String greetingText){
				if (isViewAttached())
				getView().showHello(greetingText);
			}
		});
		greetingTask.execute();	
	}

	@Override public void bindEvents(){
		$vm.vHelloBtn.setOnClickListener(v -> {
			greetHello();
		});
	}

	//...more logic can be same way ported.
}
```
最后，在Activity中调用工厂方法。 
```Java
Pan.with(this, HelloWorldViewImpl.class)
	.controlledBy(HelloWorldPresenter.class)
	.getViewModel();
```

## Pan当做MVVM
 当然，你也可以完全可以把Pan改造成一个MVVM框架，只需很少的代码。如果你阅读过[AutoRenderViewModel的介绍](常用的ViewModel实现类/#autorenderviewmodel)，会发现这个类的用法已经在向MVVM上靠拢了。

MVVM的核心在于VM-V双向绑定。所以我们先搞定VM->V，意味着我们要限定ViewModel中使用的数据对象：

```Java
public interface ViewModelData{
	void notifyRender();
	void addViewModel(ViewModel viewModel)
}

public abstract class ViewModelDataBase implements ViewModelData{
	Set<ViewModel> mViewModels = Collections.newSetFromMap(new WeakHashMap<ViewModel, Boolean>());

	public void notifyRender(){
		for(ViewModel vm: mViewModels){
			if(vm != null) vm.render();
		}
	}

	public void addViewModel(ViewModel viewModel){
		mViewModels.add(new WeakReference(viewModel));
	}
}
```

上面的代码提供了一个数据类型的基类，他在合适的时机通知ViewModel去刷新界面，一个可能的实现类如下：

```Java
public class WeiboData extends ViewModelDataBase{
	String weiboText;

	public void setWeiboText(String weiboText){
		this.weiboText = weiboText;
		notifyRender();
	}
	//...
}
```

接下来编写一个ViewModel的基类，ViewModel需要至少提供单向的自动刷新，所以会有如下代码：

```Java
public abstract class TwoWayBindingViewModel<D extends ViewModelDataBase> extends GeneralViewModel{
	
	protected D mData;

	public TwoWayBindingViewModel setData(D data){
		mData = data;
		if(mData != null){
			mData.addViewModel(this);
		}
		return this;
	}

	public TwoWayBindingViewModel setDataAndRender(D data){
		setData(data);
		render();
		return this;
	}

}
```

在具体的场景中，直接使用TwoWayBindingViewModel作为基类，并传入ViewModelDataBase的子类作为数据，就可以实现VM->V自动刷新功能。

接下来，我们通过Controller来实现V->VM的绑定。首先需要一个注解标记绑定的field。

```Java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Binding{
	
}

public class WeiboViewModel extends TwoWayBindingViewModel<Weibo>{
	
	@Binding("weiboText")
	protected EditText vEditText;
	
}
```

接下来，是Controller的具体实现，为了避免反射代码影响阅读性，这里用伪代码编写：
```
public abstract class TwoWayBindingController<T extends TwoWayBindingViewModel> extends GeneralController<T>{
	
	protected void bindEvents(){
		super.bindEvents();

		for each field in $vm.getClass() {
			if(f.getName().startsWith("v")){
				bindField(f);
			}
		}
	}

	protected void bindField(Field f){

		final Method setter = find the setter binded, e.g. setWeiboText;

		switch f.getType():
		    case EditText.class:
		         f.get($vm).addTextChangeListener(new TextWatcher(){
		         		public void afterTextChanged(Editable s){

		         			setter.invoke(mData, s.toString());
		         		}
		         	})
		         break;
		    case ... //other controls
	}
	
}
```

这里用的是反射，考虑到一个界面上一般只需要绑定一次，以及需要绑定的控件其实有限，这样做不会对性能有太大的影响。当然，用Annotation Processing可以实现相同的效果，只是编写/测试起来相对麻烦。

## Pan融合Android官方的Data-Binding

如果你对上面自己实现MVVM的方案不满意，官方的Data-Binding是避免了反射，而且可以直接使用。Pan也同样支持，接入后让使用DataBinding的也更加舒适，虽然这样做并不被推荐。可以直接参考[集成Data-binding](集成Data-binding)。















