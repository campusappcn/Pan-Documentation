# 集成官方的Data-Binding

!!!info "可以直接参考 [sample-databinding](https://github.com/campusappcn/Pan/tree/master/sample-databinding)"

官方的Data-Binding是一个避免了反射的MVVM的实现。不过说实话，个人觉得比较鸡肋。渲染逻辑在实际编写中，有大量的图片加载逻辑、自定义View逻辑等等，仍然无法完全用Xml的表达式来替代，而必须使用各类Attribute Setters, Converters。这意味着渲染逻辑会存在于**两个以上位置**，这将会对代码的可维护性造成极大的破坏，因为不同的程序员对一个渲染步骤是放在Java还是放在Xml中用表达式有着不同的理解。对于比Reactjs这种js框架，其渲染逻辑和样式Html是放在一处的，避免了这个问题。与此同时，Data-Binding也不能解决Activity的代码量过大的问题，只减少了部分代码，同时让一切变的更混乱了。

Anyway，如果你确定想使用Data-Binding，Pan也仍然支持，稍加改造即可。

首先，我们添加一个DataBindingViewModel，支持设置Data和其Binding：

```Java
public abstract class DataBindingViewModel<D extends BaseObservable, B extends ViewDataBinding> extends GeneralViewModel {

    protected D mData;
    protected B mViewDataBinding;

    public DataBindingViewModel<D, B> setData(D data){
        mData = data;
        bindData(data);
        return this;
    }

    public D getData(){
        return mData;
    }

    protected abstract void bindData(D data);

    @Override
    protected void onInit() {
        super.onInit();
        mViewDataBinding = DataBindingUtil.bind(getRootView());
    }

}
```
在onInit的时候，自动找到Binding的对象，这样子类只需重写bindData这一步即可，同时，子类的render留空，例如：

```Java
public class MainDataBindingViewModel extends DataBindingViewModel<User, ActivityMainBinding> {

        @Override
        protected void bindData(User data) {
            mViewDataBinding.setUser(data);
        }

        @Override
        public MainDataBindingViewModel render() {
            return this;
        }
    }
```
User是一个实现了BaseObserverable的实体类，比较普通，下面是User类：

```Java
public class User extends BaseObservable {

    String name;

    @Bindable
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
        notifyPropertyChanged(BR.name);
    }

    public User(){}

    public User(String name){
        this.name = name;
    }
}
```
搭配的界面Xml:

```XML
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable name="user" type="cn.campusapp.sample_databinding.User"/>
    </data>

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        >
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.name}"/>
    </RelativeLayout>
</layout>

```

这样，搭配的Controller通过控制mUser对象的field，达到自动刷新界面的效果。如果想要实现DataBinding中的onclick，也比较简单，按照文档代理到Controller即可，可以直接看[源码中的sample-databinding模块](https://github.com/campusappcn/Pan/tree/master/sample-databinding)，就不再赘述。