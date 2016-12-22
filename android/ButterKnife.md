# ButterKnife使用手册

> [官方网站](http://jakewharton.github.io/butterknife/) | [GitHub](https://github.com/JakeWharton/butterknife)

*   索引
    *   [简介](#1)
    *   [注解](#2)
    *   [导入项目](#3)
    *   [使用说明](#4)

## 简介

ButterKnife意为“黄油刀”，意思是此开源库可以让你的应用开发过程像用刀切黄油一样干净利落。此库的核心功能是通过注解来实现视图的注入，从而在代码中避免冗余的方法调用和丑陋的内部类监听器。

简单的理解“绑定”的概念，就是讲View和任意一个对象相互关联，从而在该对象中，不用`findViewById`就能使用View上的所有子视图，不用`setOnClickListener`就能给控件设置监听。

## 注解

注解(Annotation)是JDK 1.5之后引入的特性，它是一种元数据(meta-data)，即描述数据的数据。元数据这个概念比较难理解，简单的理解，就是对代码的标注，告诉编译器某一段代码需要做怎样的处理。

其实我们早就见过注解，在方法重写时，@Override就是一个注解，去掉@Override，代码一样可以运行，增加这个注解的好处，一是增加代码可读性，二是编译器能帮你做一些自动检查，增加代码健壮性。

Android引入了两个非常有用的注解，@Nullable和@NonNull，用来标识方法参数或者方法返回值等是否可以为null。以前编程时，为了避免NullPointerException，最终代码中往往到处都是`if(Obj == null){...}`这样的代码，这是很糟糕的编码方式，有了这两个注解，代码就清晰多了。

另外Android中注解的一个重要应用是代替枚举类型，这方面可以参考在Toast中的使用。

## 导入项目

在Android Studio中使用，只需添加依赖：

```
compile 'com.jakewharton:butterknife:7.0.1'

```

另外最好在build.gradle中关闭一项lint警告：

```
lintOptions {
  disable 'InvalidPackage'
}

```

## 使用说明

ButterKnife通过@Bind注解和视图Id帮你做两件事：找到视图和将视图转型为对应的子类型。例如以下的代码：

```
class ExampleActivity extends Activity {
  @Bind(R.id.title) TextView title;
  @Bind(R.id.subtitle) TextView subtitle;
  @Bind(R.id.footer) TextView footer;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    ButterKnife.bind(this);
    // TODO Use fields...
  }
}

```

ButterKnife不是通过反射，而是通过自动生成代码来执行View的查找，上面的例子中，ButterKnife最终自动生成的代码大概是这样的：

```
public void bind(ExampleActivity activity) {
  activity.subtitle = (android.widget.TextView) activity.findViewById(2130968578);
  activity.footer = (android.widget.TextView) activity.findViewById(2130968579);
  activity.title = (android.widget.TextView) activity.findViewById(2130968577);
}

```

### 资源绑定

ButterKnife预定义了注解@BindBool, @BindColor, @BindDimen, @BindDrawable, @BindInt, @BindString，用来执行资源的绑定。如下所示：

```
class ExampleActivity extends Activity {
  @BindString(R.string.title) String title;
  @BindDrawable(R.drawable.graphic) Drawable graphic;
  @BindColor(R.color.red) int red; // int or ColorStateList field
  @BindDimen(R.dimen.spacer) Float spacer; // int (for pixel size) or float (for exact value) field
  // ...
}

```

### 非Activity绑定

除了Activity，你也可以将任意的对象与View绑定。

例如在Fragment中：

```
public class FancyFragment extends Fragment {
  @Bind(R.id.button1) Button button1;
  @Bind(R.id.button2) Button button2;

  @Override public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
    ButterKnife.bind(this, view);
    // TODO Use fields...
    return view;
  }
}

```

或在适配器中：

```
public class MyAdapter extends BaseAdapter {
  @Override public View getView(int position, View view, ViewGroup parent) {
    ViewHolder holder;
    if (view != null) {
      holder = (ViewHolder) view.getTag();
    } else {
      view = inflater.inflate(R.layout.whatever, parent, false);
      holder = new ViewHolder(view);
      view.setTag(holder);
    }

    holder.name.setText("John Doe");
    // etc...

    return view;
  }

  static class ViewHolder {
    @Bind(R.id.title) TextView name;
    @Bind(R.id.job_title) TextView jobTitle;

    public ViewHolder(View view) {
      ButterKnife.bind(this, view);
    }
  }
}

```

任何原来需要使用`findViewById`的地方都可以用`ButterKnife.bind`替代。

用于绑定的API：

*   如果使用了MVC模式，将控制器的Activity和视图绑定，可以用`ButterKnife.bind(this, activity)`。
*   将自定义视图绑定到自身，可以用`ButterKnife.bind(this)`。

### 绑定视图列表

可以将多个视图放在List或数组中。

```
@Bind({ R.id.first_name, R.id.middle_name, R.id.last_name })
List<EditText> nameViews;

```

`apply`方法让你可以同时操作列表中的所有视图：

```
ButterKnife.apply(nameViews, DISABLE);
ButterKnife.apply(nameViews, ENABLED, false);

```

`Action`和`Setter`接口可以用来指定简单的操作：

```
static final ButterKnife.Action<View> DISABLE = new ButterKnife.Action<View>() {
  @Override public void apply(View view, int index) {
    view.setEnabled(false);
  }
};
static final ButterKnife.Setter<View, Boolean> ENABLED = new ButterKnife.Setter<View, Boolean>() {
  @Override public void set(View view, Boolean value, int index) {
    view.setEnabled(value);
  }
};

```

`apply`方法还能用来设置View的属性：

```
ButterKnife.apply(nameViews, View.ALPHA, 0.0f);

```

### 监听接口的绑定

监听器可以自动在方法上配置，如下：

```
@OnClick(R.id.submit)
public void submit(View view) {
  // TODO submit data to server...
}

```

监听器方法的参数是可选的：

```
@OnClick(R.id.submit)
public void submit() {
  // TODO submit data to server...
}

```

监听器方法的参数可以是具体的类型，它会被自动转型：

```
@OnClick(R.id.submit)
public void sayHi(Button button) {
  button.setText("Hello!");
}

```

也可以在单个绑定中指定多个ID来进行常见的事件处理：

```
@OnClick({ R.id.door1, R.id.door2, R.id.door3 })
public void pickDoor(DoorView door) {
  if (door.hasPrizeBehind()) {
    Toast.makeText(this, "You win!", LENGTH_SHORT).show();
  } else {
    Toast.makeText(this, "Try again", LENGTH_SHORT).show();
  }
}

```

自定义视图自己的监听器可以省去ID：

```
public class FancyButton extends Button {
  @OnClick
  public void onClick() {
    // TODO do something!
  }
}

```

### 绑定的重置

Fragment生命周期和Activity不同，当在onCreateView中绑定视图后，要在onDestroyView中将这些视图置为null，ButterKnife提供了Unbinder接口来自动做这件事，如下：

```
public class FancyFragment extends Fragment {
  @Bind(R.id.button1) Button button1;
  @Bind(R.id.button2) Button button2;
  @Unbinder ButterKnife.Unbinder unbinder;

  @Override public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
    ButterKnife.bind(this, view);
    // TODO Use fields...
    return view;
  }

  @Override public void onDestroyView() {
    super.onDestroyView();
    unbinder.unbind();
  }
}

```

### 可选绑定

在默认情况下，如果ID对应的视图没有找到，会抛异常。要使用可选绑定，添加@Nullable或@Optional注解即可：

```
@Nullable @Bind(R.id.might_not_be_there) TextView mightNotBeThere;

@Optional @OnClick(R.id.maybe_missing) void onMaybeMissingClicked() {
  // TODO ...
}

```

### 多方法监听器

如果注解对应的监听器有多个回掉方法，可以绑定任意一个方法。每种注解都有默认的回掉方法，可以通过callback参数指定为其它的回掉方法：

```
@OnItemSelected(R.id.list_view)
void onItemSelected(int position) {
  // TODO ...
}

@OnItemSelected(value = R.id.maybe_missing, callback = NOTHING_SELECTED)
void onNothingSelected() {
  // TODO ...
}

```

### 其它

对于无法直接绑定视图的常见，ButterKnife提供了`findById`方法简化操作，它通过泛型操作将返回值自动转型。

```
View view = LayoutInflater.from(context).inflate(R.layout.thing, null);
TextView firstName = ButterKnife.findById(view, R.id.first_name);
TextView lastName = ButterKnife.findById(view, R.id.last_name);
ImageView photo = ButterKnife.findById(view, R.id.photo);

```