# MVP&MVVM: 让Pan按照你喜欢的模型工作

Pan的模型是MV*，意味着，只要达到了Pan的初心：分离Activity的代码，Pan可以被用来以任何你喜欢的模型来编写代码。例如，你可以把Pan当做一个MVP框架来使用，也可以当做一个MVVM的框架来使用，只需要很少、甚至不需要代码改动。

## Pan当做MVP框架使用([Mosby](http://hannesdorfmann.com/mosby/))

ViewModel本身理解为MVP中的View。ViewModel的render方法相当于全量刷新界面，而更局部刷新可以在子类提供诸如renderUser等方法。而Controller则可以类比到Presenter。因此，符合MVP框架的代码可以直接迁移成Pan框架的代码，你会发现是等价的。不仅如此，Pan框架由于强制代码分离，会使得原先仍然有很多代码的Activity，变的只需要寥寥几行。

例如，我们可以直接迁移最近比较火的MVP框架[Mosby的Getting Started](http://hannesdorfmann.com/mosby/getting-started/)，你可以类似的写出下面的代码：

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

	@Bind(R.id.greetingTextView) TextView vGreetingTextView;
	@Bind(R.id.helloButton) Button vHelloBtn;

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

## MVVM

//TODO