# 欢迎来到Pan!

Pan是一个轻量级的安卓MV*框架，帮助梳理安卓的前端代码。

使用Pan将大大减少Activity的代码，使得Activity的代码缩减到100行甚至50行以内。
同时，大大提升界面代码的可复用性，提升代码的可维护性。

## Demo


```Java
public class MainActivity extends PanActivity {

    MainViewModel mMainViewModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mMainViewModel = Pan.with(this, MainViewModel.class)
                .controlledBy(MainController.class)
                .getViewModel()
                .setHelloString("hello Pan!")
                .render(); 
    }

    //Done! 
    //No need for more codes!
}
```

还等什么？[Getting Started!](./Getting_Started)

## Features

1. 核心：掏空Activity的代码
2. 分离渲染逻辑(ViewModel)、控制逻辑(Controller)
3. 复用页面部件，包括XML和ViewModel，且与Activity/Fragment解耦
4. 复用业务逻辑，即Controller
5. 兼容已有代码，无侵入性
6. 插件化，易于拓展，方便
7. 模型中立，可以Data-Binding结合拓展成MVVM框架，也可以按照特定的使用习惯当做MVP框架

实际上，由于ViewModel和Activity/Fragment解耦，可以让框架的灵活度直接上一个台阶，完成很多之前难以做到的需求，例如：

1. 类似[Conductor](https://github.com/bluelinelabs/Conductor)，全局使用一个Activity，通过路由切换View。这个只需要给Pan写一个路由插件即可，ViewModel对当前所处环境并不关心。

## MV*模型

Pan的模型比MVC、MVP、MVVM更务实，只追求实际使用中的易于上手，和上述设计目标的实现，我们可以称之为[MVW(Whatever)](http://stackoverflow.com/questions/13329485/mvw-what-does-it-stand-for)。

但你仍然可以把Pan当做MVP、MVVM框架使用，例如Pan可以直接[等价到Mosby(MVP)的代码](MVVM_MVP/#panmvpmosby)。

详细的设计思路和哲学，请参考[Pan的简介和设计思路](http://blog.campusapp.cn/2016/03/18/2016-03-18-Pan%E7%9A%84%E7%AE%80%E4%BB%8B%E5%92%8C%E8%AE%BE%E8%AE%A1%E6%80%9D%E8%B7%AF/)

![Pan的MV*模型](https://img.alicdn.com/imgextra/i4/56380417/TB2KrLBlVXXXXcWXXXXXXXXXXXX_!!56380417.png)


