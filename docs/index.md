# 欢迎来到Pan!

Pan是一个轻量级的安卓MV*框架，帮助梳理安卓的前端代码。

使用Pan将大大减少Activity的代码，使得Activity的代码缩减到100行甚至50行以内。

## Demo


```Java
public class MainActivity extends PanFragmentActivity {

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

## MV*模型

