# Mosby MVP使用文档

> 索引
>
> *   [入门](#1)
>     *   [MVP简介](#1.1)
>     *   [Mosby简介](#1.2)
>     *   [Hello MVP World](#1.3)
>     *   [MvpPresenter的基类](#1.4)
> *   [基础](#2)
>     *   [LCE视图](#2.1)
>     *   [MvpLceActivity和MvpLceFragment](#2.2)
>     *   [ViewState简介](#2.3)
> *   [拓展](#3)
>     *   [思考](#3.1)

## 入门

### MVP简介

MVP的出发点是关注点分离，将视图和业务逻辑解耦。Model-View-Presenter三个部分可以简单理解为：

*   **Model**是将在视图中显示的数据。
*   **View**是显示数据(model)的界面，同时将用户指令(事件)发送给Presenter来处理。View通常含有Presenter的引用。在Android中Activity，Fragment和ViewGroup都扮演视图的角色。
*   **Presenter**是中间人，同时有两者的引用。**请注意单词model非常有误导性**。它应该是**获取或处理model的业务逻辑**。例如：如果你的数据库表中存储着User，而你的视图想显示用户列表，那么Presenter将有一个数据库业务逻辑(例如DAO)类的引用，Presenter通过它来查询用户列表。

> **思考**：MVC，MVP和MVVM之间有什么区别和联系？

**消极视图**：在MVP中，View是**消极视图(Passive View)**，也就是说它尽量不去主动做事，而是让Presenter通过抽象方式控制View，例如Presenter调用`view.showLoading()`方法来显示加载效果，但Presenter不应该控制View的具体实现，例如动画，所以Presenter不应该调用`view.startAnimation()`这样的方法。

![xiaojishitu](http://7xsct4.com1.z0.glb.clouddn.com/16-5-4/67007311.jpg)

### Mosby简介

**设计目标**：让你能用清晰的Model-View-Presenter架构来构建Android app。

**注意**：Mosby是一个库(library)，不是一个框架(framework)。

> **思考**：什么是library？什么是framework？它们的区别是什么？

Mosby的内核是一个基于委托模式(delegation)的很精简的库。你可以使用委托(delegation)和组合(composition)将Mosby集成到你的开发技术栈中。这样你就能避免框架(framework)带来的限制和约束。

> **思考**：什么是委托模式？委托和继承的区别是什么？使用委托有什么好处？

**依赖**：

Mosby被分成模块，你可以选择你需要的功能：

```
dependencies {
    compile 'com.hannesdorfmann.mosby:mvp:2.0.1'
    compile 'com.hannesdorfmann.mosby:viewstate:2.0.1'
}

```

### Hello MVP World

先来用Mosby MVP库来实现一个最简单的功能，页面有两个Button和一个TextView，需求如下：

*   点击Hello按钮，显示红色文本 "Hello" + 随机数；
*   点击Goodbye按钮，显示蓝色文本 "Goodbye" + 随机数；

这里假设随机数的生成过程涉及到复杂的业务逻辑计算，是一个耗时操作，需要2s时间。

**第一步**我们用一个`AsyncTask`来实现这个模拟的业务逻辑，在自定义的`AsyncTask`中，要定义一个监听器，用来传递业务逻辑执行结果：

```
public class GreetingGeneratorTask extends AsyncTask<Void, Void, Integer>{

    // Callback - listener
    public interface GreetingTaskListener{
        void onGreetingGenerated(String greetingText);
    }

    ......

    // 模拟计算过程，返回一个随机值。
    @Override
    protected Integer doInBackground(Void... params) {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return (int)(Math.random() * 100);
    }

    @Override
    protected void onPostExecute(Integer randomInt) {
        listener.onGreetingGenerated(baseText + " " + randomInt);
    }
}

```

**第二步**定义视图接口，视图接口需要继承`MvpView`：

```
public interface HelloWorldView extends MvpView{

    void showHello(String greetingText);

    void showGoodbye(String greetingText);
}

```

注意这里的`MvpView`是所有视图的顶层接口，它是一个空接口，没有定义任何方法。

**第三步**实现Presenter，Presenter需要执行业务逻辑，并针对不同的执行结果调用视图的对应方法。

Presenter的顶层接口是`MvpPresenter`，它有两个方法：

```
public interface MvpPresenter<V extends MvpView> {

  /**
   * 将View附着到Presenter上
   */
  public void attachView(V view);

  /**
   * 在视图被摧毁时调用。典型场景是Activity.onDestroy()和Fragment.onDestroyView()方法
   */
  public void detachView(boolean retainInstance);
}

```

Mosby提供了`MvpPresenter`接口的基类实现，在这里我们继承`MvpBasePresenter`:

```
public class HelloWorldPresenter extends MvpBasePresenter<HelloWorldView>{

    private GreetingGeneratorTask greetingTask;

    private void cancelGreetingTaskIfRunning(){
        if (greetingTask != null){
            greetingTask.cancel(true);
        }
    }

    public void greetHello(){
        cancelGreetingTaskIfRunning();

        greetingTask = new GreetingGeneratorTask("Hello", new GreetingGeneratorTask.GreetingTaskListener() {
            @Override
            public void onGreetingGenerated(String greetingText) {
                if (isViewAttached()){
                    getView().showHello(greetingText);
                }
            }
        });
        greetingTask.execute();
    }

    ......

    @Override
    public void detachView(boolean retainInstance) {
        super.detachView(retainInstance);
        if (!retainInstance){
            cancelGreetingTaskIfRunning();
        }
    }
}

```

注意在`detachView`方法中取消后台任务的处理。

**第四步**实现Activity，让我们的Activity继承`MvpActivity`，并实现`HelloWorldView`接口。

`MvpActivity`有两个泛型，分别是Presenter和View的具体类型：

```
public class HelloWorldActivity extends MvpActivity<HelloWorldView, HelloWorldPresenter> implements HelloWorldView{
    ......
}

```

继承`MvpActivity`后，只有一个抽象方法`createPresenter()`需要实现：

```
public HelloWorldPresenter createPresenter() {
    return new HelloWorldPresenter();
}

```

`HelloWorldView`还有两个方法需要实现：

```
@Override
public void showHello(String greetingText) {
    greetingTextView.setTextColor(Color.RED);
    greetingTextView.setText(greetingText);

}

@Override
public void showGoodbye(String greetingText) {
    greetingTextView.setTextColor(Color.BLUE);
    greetingTextView.setText(greetingText);
}

```

点击按钮后，使用Presenter来完成相关操作：

```
@OnClick(R.id.helloButton)
public void onHelloButtonClicked(){
    presenter.greetHello();
}

@OnClick(R.id.goodbyeButton)
public void onGoodbyeButtonClicked(){
    presenter.greetGoodbye();
}

```

### MvpPresenter的基类

**Presenter默认实现一：**使用弱引用保存视图引用，在调用`getView()`之前必须判断`isViewAttached()`。

```
public class MvpBasePresenter<V extends MvpView> implements MvpPresenter<V> {

  private WeakReference<V> viewRef;

  @Override public void attachView(V view) {
    viewRef = new WeakReference<V>(view);
  }

  @Nullable public V getView() {
    return viewRef == null ? null : viewRef.get();
  }

  public boolean isViewAttached() {
    return viewRef != null && viewRef.get() != null;
  }

  @Override public void detachView(boolean retainInstance) {
    if (viewRef != null) {
      viewRef.clear();
      viewRef = null;
    }
  }
}

```

**Presenter默认实现二：**使用Null Object Pattern，在调用`getView()`时无需判断。

```
public class MvpNullObjectBasePresenter<V extends MvpView> implements MvpPresenter<V> {

  private V view;

  @Override public void attachView(V view) {
    this.view = view;
  }

  @NonNull public V getView() {
    if (view == null) {
      throw new NullPointerException("MvpView reference is null. Have you called attachView()?");
    }
    return view;
  }

  @Override public void detachView(boolean retainInstance) {
    if (view != null) {

      Type[] types =
          ((ParameterizedType) getClass().getGenericSuperclass()).getActualTypeArguments();

      Class<V> viewClass = (Class<V>) types[0];
      view = NoOp.of(viewClass);
    }
  }
}

```

> 思考：什么是空对象模式(Null Object Pattern)？

## 基础

### LCE视图

在开发Android应用过程中，我们会发现很多页面有相似的结构和UI逻辑，所以我们常常在写重复代码。如果能抽象出相似页面的View接口，然后封装页面的基类，就能让开发方便很多。Mosby就给我们提供了一个这样的视图模板，叫做**LCE View**。

LCE代表Loading-Content-Error(加载-内容-错误)，此视图有三种状态：显示加载中，显示数据内容，或者显示错误视图。例如在如下的场景中：

假设我们要在`ListView`中显示一个国家列表，国家列表的数据是从网络获取的，是一个耗时操作。在加载过程中，我们要显示一个`ProgressBar`，如果加载出错，我们要显示一条错误信息。另外，还要用`SwipeRefreshLayout`来让用户可以下拉刷新。

LCE View的接口定义如下：

```
public interface MvpLceView<M> extends MvpView {

  /**
   * 显示加载视图，加载视图的id必须为R.id.loadingView
   */
  public void showLoading(boolean pullToRefresh);

  /**
   * 显示内容视图，内容视图的id必须为R.id.contentView
   *
   * <b>The content view must have the id = R.id.contentView</b>
   */
  public void showContent();

  /**
   * 显示错误视图，错误视图必须是TextView，id必须是R.id.errorView
   */
  public void showError(Throwable e, boolean pullToRefresh);

  /**
   * 设置将在showContent()中显示的数据
   */
  public void setData(M data);

  /**
   * 加载数据，此方法中常需要调用Presenter的对应方法。因此此方法不可在Presenter
   * 中使用，避免循环调用。
   * 参数pullToRefresh代表此次加载是否由下拉刷新触发。
   */
  public void loadData(boolean pullToRefresh);
}

```

> **思考**：LCE视图中考虑了下拉刷新，但没有考虑上拉加载，如果服务器是分页接口，需要添加上拉加载，应该怎样定义视图接口？

### MvpLceActivity和MvpLceFragment

Mosby封装了LCE视图的基类，现在我们用MvpLceActivity或MvpLceFragment来实现上面所说的加载国家列表的场景。

**第一步**完成界面布局，注意id必须使用上面指定的名称，错误视图只能是一个TextView：

```
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    ......>
    <include layout="@layout/loading_view" />
    <include layout="@layout/error_view" />
    <android.support.v4.widget.SwipeRefreshLayout
        android:id="@+id/contentView"
        ......>
        <ListView ....../>
    </android.support.v4.widget.SwipeRefreshLayout>
</FrameLayout>

```

**第二步**继承`MvpLceView`实现自己的视图接口，此处需要指定泛型，作为数据类型：

```
public interface CountriesView extends MvpLceView<List<Country>>{
}

```

**第三部**实现Presenter，在这里我们做了一个接口和一个实现：

接口定义：

```
public interface CountriesPresenter extends MvpPresenter<CountriesView>{

    void loadCountries(final boolean pullToRefresh);
}

```

具体实现：

```
public class SimpleCountriesPresenter extends MvpNullObjectBasePresenter<CountriesView>
        implements CountriesPresenter{

    ......

    @Override
    public void loadCountries(final boolean pullToRefresh) {
        getView().showLoading(pullToRefresh);

        ......

        countriesLoader = new CountriesAsyncLoader(++failingCounter % 2 != 0, new CountriesAsyncLoader.CountriesLoaderListener() {
            @Override
            public void onSuccess(List<Country> countries) {
                getView().setData(countries);
                getView().showContent();
            }

            @Override
            public void onError(Exception e) {
                getView().showError(e, pullToRefresh);
            }
        });

        countriesLoader.execute();
    }

    ......
}

```

上面代码中就使用到了`MvpLceView`中除`loadData()`外的全部四个方法。

**第四步**实现Activity或者Fragment，先以Activity为例，需要继承`MvpLceActivity`:

```
public class CountriesActivity extends MvpLceActivity<SwipeRefreshLayout, List<Country>, CountriesView, CountriesPresenter>
        implements SwipeRefreshLayout.OnRefreshListener, CountriesView{

```

`MvpLceActivity`中定义了四个泛型，分别是ContentView的类型，Data的类型，视图接口的类型和Presenter的类型。此处ContentView使用的是SwipeRefreshLayout。

继承`MvpLceActivity`后有两个方法需要实现：

```
@Override
protected String getErrorMessage(Throwable e, boolean pullToRefresh) {
    if (pullToRefresh) {
        return "Error while loading countries";
    } else {
        return "Error while loading countries. Click here to retry";
    }
}

@NonNull
@Override
public CountriesPresenter createPresenter() {
    return new SimpleCountriesPresenter();
}

```

实现`CountriesView`接口后，重写如下几个方法：

```
@Override
public void setData(List<Country> data) {
    adapter.clear();
    adapter.addAll(data);
    adapter.notifyDataSetChanged();
}

@Override
public void showContent() {
    super.showContent();
    contentView.setRefreshing(false);
}

@Override
public void showError(Throwable e, boolean pullToRefresh) {
    super.showError(e, pullToRefresh);
    contentView.setRefreshing(false);
}

@Override
public void loadData(boolean pullToRefresh) {
    presenter.loadCountries(pullToRefresh);
}

```

实现`OnRefreshListener`接口后，需要实现一个方法：

```
@Override
public void onRefresh() {
    loadData(true);
}

```

如果要用Fragment，方法基本上一样，只需继承`MvpLceFragment`，唯一的区别是，Activity的初始化在`onCreate()`中完成，Fragment的初始化在`onViewCreated()`中完成：

在Activity中：

```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.countries_list);
    ButterKnife.bind(this);

    contentView.setOnRefreshListener(this);
    adapter = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1);
    listView.setAdapter(adapter);
    loadData(false);
}

```

在Fragment中：

```
@Override
public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
    super.onViewCreated(view, savedInstanceState);
    ButterKnife.bind(this, view);

    contentView.setOnRefreshListener(this);

    contentView.setOnRefreshListener(this);
    adapter = new ArrayAdapter<>(getContext(), android.R.layout.simple_list_item_1);
    listView.setAdapter(adapter);
    loadData(false);
}

```

### ViewState简介

在Android开发中有一个很麻烦的问题，就是在界面被销毁、被重建的过程中保存和恢复视图状态。界面被系统回收和重建常常发生在这两个场景中：

*   Configuration变化，例如屏幕在横竖屏之间切换，语言环境变化等。
*   界面切到后台(例如用户按Home键)，Android在内存过低时自动回收此Activity，在界面重新显示时重建Activity。

> **思考**：两种Activity被回收和重建的场景，有什么区别？

Mosby提供了一个**ViewState**特性来解决这一问题。`ViewState`是一个接口，只有一个`apply`方法：

```
public interface ViewState<V extends MvpView> {

  /**
   * Called to apply this viewstate on a given view.
   *
   * @param view The {@link MvpView}
   * @param retained true, if the components like the viewstate and the presenter have been
   * retained
   * because the {@link Fragment#setRetainInstance(boolean)} has been set to true
   */
  public void apply(V view, boolean retained);
}

```

例如，上面讲过的MvpLceFragment，如果想在横竖屏切换过程中保存和恢复视图状态，只需改成继承`MvpLceViewStateFragment`，实现如下一个方法即可：

```
@Override
public LceViewState<List<Country>, CountriesView> createViewState() {
    setRetainInstance(true);
    return new RetainingLceViewState<>();
}

```

这是针对Mosby提供的LceView的ViewState，如果是我们的自定义视图，也可以实现自己的ViewState。整个ViewState特性的实现原理和应用方法比较复杂，这里不做过多介绍。

## 扩展

### 思考

*   MVC，MVP和MVVM之间有什么区别和联系？

*   什么是委托模式？委托和继承的区别是什么？使用委托有什么好处？

*   什么是library？什么是framework？它们的区别是什么？

> 提示：library和framework的关键区别是“控制反转”(Inversion of Control)。当你调用library中的方法时，你掌握控制权。但使用framework时，控制是倒转的：由framework来调用你的代码。
>
> ![52280993](http://7xsct4.com1.z0.glb.clouddn.com/16-5-10/52280993.jpg)

*   什么是空对象模式(Null Object Pattern)？

*   LCE视图中考虑了下拉刷新，但没有考虑上拉加载，如果服务器是分页接口，需要添加上拉加载，应该怎样定义视图接口？

*   两种Activity被回收和重建的场景，有什么区别？

> 提示：参考下面两个方法：
>
> *   `Fragmemt.setRetainInstance(boolean retain)`
> *   `Activity.onRetainNonConfigurationInstance()`