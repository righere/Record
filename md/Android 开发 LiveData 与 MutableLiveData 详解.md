> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/guanxinjing/p/11544273.html)

前言
==

　　LiveData 与 ViewMode 是经常搭配在一起使用的, 但是为了不太混乱, 我还是拆分开来说明, 此篇博客只讲解 LiveData 与 MutableLiveData 的概念与使用方式 (但是会涉及到 ViewMode 的部分代码).

　　转载请注明来源：[https://www.cnblogs.com/guanxinjing/p/11544273.html](https://www.cnblogs.com/guanxinjing/p/11544273.html "https://www.cnblogs.com/guanxinjing/p/11544273.html")

LiveData 是干什么的?
===============

　　由于 LiveData 和 MutableLiveData 都是一个概念的东西 (只是作用范围不同) 所以就不重复解释了, 直接理解 LiveData 就可以明白 MutableLiveData

　　直接理解 LiveData 的字面意思是前台数据, 其实这其实是很准确的表达. 下面我们来说说 LiveData 的几个特征:

1. 首先 LiveData 其实与数据实体类 (POJO 类) 是一样的东西, 它负责暂存数据.

2. 其次 LiveData 其实也是一个观察者模式的数据实体类, 它可以跟它注册的观察者回调数据是否已经更新.

3.LiveData 还能知晓它绑定的 Activity 或者 Fragment 的生命周期, 它只会给前台活动的 activity 回调 (这个很厉害). 这样你可以放心的在它的回调方法里直接将数据添加到 View, 而不用担心会不会报错.(你也可以不用费心费力判断 Fragment 是否还存活)

LiveData 与 MutableLiveData 区别
=============================

LiveData 与 MutableLiveData 的其实在概念上是一模一样的. 唯一几个的区别如下:

1.MutableLiveData 的父类是 LiveData

2.LiveData 在实体类里可以通知指定某个字段的数据更新.

3.MutableLiveData 则是完全是整个实体类或者数据类型变化后才通知. 不会细节到某个字段

4.LiveData 不可变, MutableLiveData 是可变的 -- 这个可能会让人有点误解, 在博客最下面会解释.

LiveData 简单使用 Demo
==================

创建 LiveData
-----------

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
public class DemoData extends LiveData<DemoData> {
    private int tag1;
    private int tag2;
    
    public int getTag1() {
        return tag1;

    }
    public void setTag1(int tag1) {
        this.tag1 = tag1;
        postValue(this);
    }

    public int getTag2() {
        return tag2;
    }

    public void setTag2(int tag2) {
        this.tag2 = tag2;
        postValue(this);
    }
}

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

很简单, 只要继承 LiveData 并且在泛型里写下你的实体类, **唯一需要注意的,postValue(this);** 这个方法是用于回调数据更新的方法. 你可以在你需要被观察的数据里添加.

创建 ViewModel
------------

我们需要在 ViewModel 实例化 DemoData 这个类. ViewModel(这个会在另一篇博客介绍) 这个是用于管理多个 Activity 或者 Fragment 数据的类。ViewModel 是 MVVM 的概念。你可以百度一下，google 提供这套东西就是为了 MVVM。

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
public class DemoViewModel extends ViewModel {
    // TODO: Implement the ViewModel
    private DemoData mDemoData = new DemoData();

    public DemoData getDemoData() {
        return mDemoData;
    }
}

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

在 Activity 或者 Fragment 绑定
-------------------------

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
public class Demo2Activity extends AppCompatActivity {
    private static final String TAG = "Demo2Activity";
    private Button mBtnAddData;
    private DemoViewModel mDemoViewModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_demo2);
        mBtnAddData = findViewById(R.id.btn_add_data);
        mDemoViewModel = ViewModelProviders.of(this).get(DemoViewModel.class);//获取ViewModel,让ViewModel与此activity绑定
        mDemoViewModel.getDemoData().observe(this, new Observer<DemoData>() { //注册观察者,观察数据的变化
            @Override
            public void onChanged(DemoData demoData) {
                Log.e(TAG, "onChanged: 数据有更新");
            }
        });
        
        mBtnAddData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.e(TAG, "onClick: 已经点击");
                mDemoViewModel.getDemoData().setTag1(123); //这里手动用按键点击更新数据

            }
        });
    }
}

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

当我们点击按键后就会有数据更新后的回调触发:

```
2019-09-18 19:45:53.821 6649-6649/demo.yt.com.demo E/Demo2Activity: onClick: 已经点击
2019-09-18 19:45:53.824 6649-6649/demo.yt.com.demo E/Demo2Activity: onChanged: 数据有更新

```

**前面提过了, 但是这里还是需要重新提一下! 注意! 这个数据只给前台的活动回调.**

MutableLiveData 简单使用 Demo
=========================

　　 前面已经解释了, 所以我们这边直接看代码

创建 MutableLiveData
------------------

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
public class DemoViewModel extends ViewModel {
    // TODO: Implement the ViewModel
    private MutableLiveData<String> myString = new MutableLiveData<>();

    public MutableLiveData<String> getMyString(){
        return myString;
    }

    public void setMyString(String string) {
        this.myString.setValue(string);
    }
}

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

因为 MutableLiveData 只是作用于变量所以我们直接就可以在 ViewModel 里实例化它, 并且在泛型里标注变量的类型.

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
public class MutableLiveData<T> extends LiveData<T> {
    @Override
    public void postValue(T value) {
        super.postValue(value);
    }

    @Override
    public void setValue(T value) {
        super.setValue(value);
    }
}

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

在 Activity 或者 Fragment 绑定
-------------------------

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
public class Demo1Activity extends AppCompatActivity {
    private static final String TAG = "Demo1Activity";
    private DemoViewModel mDemoViewModel;
    private Button mBtn1;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_demo);
        mBtn1 = findViewById(R.id.btn_1);
        mDemoViewModel = ViewModelProviders.of(this).get(DemoViewModel.class);//获取ViewModel,让ViewModel与此activity绑定
        mDemoViewModel.getMyString().observe(this, new Observer<String>() { //注册观察者
            @Override
            public void onChanged(String s) {
                Log.e(TAG, "onChanged: 值有变化="+s);
            }
        });

        mBtn1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mDemoViewModel.setMyString("测试"); //用手动按键点击改变值


            }
        });
    }
}

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

 结果:

```
2019-09-18 19:59:38.294 6961-6961/demo.yt.com.demo E/Demo1Activity: onChanged: 值有变化=测试

```

API 全解
======

postValue()
-----------

　　可能你已经在上面看到几次调用此方法了。postValue 的特性如下：

　　1. 此方法可以在**其他线程**中调用

　　2. 如果在主线程执行发布的任务之前多次调用此方法，则仅将分配最后一个值。

　　3. 如果同时调用 .postValue(“a”) 和. setValue(“b”)，一定是值 b 被值 a 覆盖。

setValue()
----------

　　setValue() 的特性如下：  

　　1. 此方法**只能在主线程**里调用

getValue()
----------

　　返回当前值。 注意，在后台线程上调用此方法并不能保证将接收到最新的值。  

removeObserver(@NonNull final Observer<? super T> observer) 
------------------------------------------------------------

　　移除指定的观察者

例子：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
        Observer<String> observer = new Observer<String>() {
            @Override
            public void onChanged(String s) {
                mText.setText("内容改变=" + s);
            }
        };
        mMainViewModel.getContent().observe(this, observer);//绑定
        mMainViewModel.getContent().removeObserver(observer);//解除

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

removeObservers(@NonNull final LifecycleOwner owner)
----------------------------------------------------

　　移除当前 Activity 或者 Fragment 的全部观察者

```
mMainViewModel.getContent().removeObservers(this);

```

 hasActiveObservers()
---------------------

　　如果此 LiveData 具有活动（Activity 或者 Fragment 在前台, 当前屏幕显示）的观察者，则返回 true。其实如果这个数据的观察者在最前台就返回 true，否则 false。

hasObservers()
--------------

　　如果此 LiveData 具有观察者，则返回 true。

observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer)
-----------------------------------------------------------------------------

　　设置此 LiveData 数据当前 activity 或者 Fragment 的观察者，会给此 activity 或者 Fragment 在前台时回调数据。

observeForever(@NonNull Observer<? super T> observer)
-----------------------------------------------------

　　1. 设置永远观察者，永远不会被自动删除。您需要手动调用 removeObserver（Observer）以停止观察此 LiveData，

　　2. 设置后此 LiveData，一直处于活动状态，不管是否在前台哪里都会获得回调。

MutableLiveData 与 LiveData 区别：可变与不可变
====================================

初步学习后, 很多人容易误解所谓的 LiveData 可变与 MutableLiveData 不可变, 认为 LiveData 不是也可以 set 与 post 数据嘛? 怎么就不可变了

其实关键是 LiveData 是抽象类它的 setValue 与 postValue 方法都是 protected 关键字, 而 MutableLiveData 是继承 LiveData 后将 setValue 与 postValue 都 public 了. 这里的可变与不可变是对外部调用时的情况, LiveData 对外隐藏了 setValue 与 postValue.

这里又引申了一个 android 官方实现 MutableLiveData 模版代码, 让 MutableLiveData 的 setValue 与 postValue 不暴露出去, 代码如下:

kotlin 写法:

```
    val name: LiveData<NameBean> get() = _name
    private val _name = MutableLiveData<NameBean>()

```

java 写法:

```
    private MutableLiveData<NameBean> name = new MutableLiveData<>();

    public LiveData<NameBean> getName(){
        return name;
    }

```

在外部调用时可以看到的情况, 只有 getValue() 方法暴露

![](https://img2020.cnblogs.com/blog/1497956/202111/1497956-20211126150437938-1725193363.png)

END