## 一、Fragment简介 ##
Fragment是一种可以嵌入在Activity中的UI片段，它能弥补手机屏幕小的缺点，让程序更加合理和充分地利用屏幕的控件。Fragment拥有自己的生命周期，可以接受处理用户的事件，并且可以在一个Activity中动态的添加，替换和删除不同的Fragment。

> Fragment一种是android.app.Fragment，另一种是support包中的android.support.v4.Fragment。建议使用support包中的Fragment，因为support是不断更新的，使用support包中的Fragment，Activity必须继承FragmentActivity。（AppCompatActivity是FragmentActivity的子类）

- Fragment依赖于Activity，不能独立存在
- 一个Activity里可以有多个Fragment
- 一个Fragment可以被多个Activity重用
- Fragment有自己的生命周期，并接受输入事件
- 我们能在Activity运行时动态地添加或删除Fragment

## 二、Fragment的生命周期 ##
![](https://i.imgur.com/9PvGHX3.png)

- onAttach：Fragment与Activity相关联的时候调用，一般可以从这个方法中获得对Activity的引用，还可以通过getArguments获取参数。
- onCreate：Fragment被创建时调用。
- onCreateView：创建并返回视图布局。
- onActivityCreated：当Activity完成onCreate时返回并调用。
- onViewStateRestored：Fragment保存其视图的状态。
- onStart：当Fragment可见时调用。
- onResume：当Fragment可见并可交互时调用。
- onPause：当Fragment可见不可交互时调用。
- onStop：当Fragment不可见也不可交互时调用。
- OnDestoryView：当Fragment的UI从视图中移除时调用。
- onDestory：当Fragment销毁时调用。
- onDetach：当Fragment和Activity解除关联时调用。

> 上面的方法中，只有onCreateView在重写时不用调用super，其他都要调用

## 三、Fragment基本使用 ##
### 1. 创建Fragment的UI ###
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center">
    <TextView
        android:id="@+id/name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="28dp"/>
</LinearLayout>
```
布局中只有一个TextView，用来显示传过来的数据


### 2. 创建Fragment ###
```
public class MyFragment extends Fragment {
    public Activity mActivity;
    public String name;

    private TextView tvName;
    public static MyFragment newInstance(String name) {

        Bundle args = new Bundle();
        args.putString("name",name);
        MyFragment fragment = new MyFragment();
        //通过setArguments放入参数
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        //将Context对象强制转换成Activity对象
        mActivity= (Activity) context;
        if(getArguments()!=null) {
            //取出数据
            name = getArguments().getString("name");
        }
    }

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        //第三个参数必须为false，因为在Fragment内部实现中，会把布局加到container中，如果为true，就会有重复添加
        return inflater.inflate(R.layout.fragment_demo,container,false);
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        tvName=view.findViewById(R.id.name);
        tvName.setText(name);
    }
}
```

### 三、把Fragment添加到Activity中 ###
```
public class MainActivity extends AppCompatActivity {
    private FrameLayout frameLayout;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        frameLayout = findViewById(R.id.frameLayout);
        getSupportFragmentManager()
                .beginTransaction()
                .add(R.id.frameLayout, MyFragment.newInstance("测试"))
                .commit();
    }
}
```

在Activity中添加Fragment有两种方式：
- 静态添加：通过xml的方式添加，缺点一旦添加就不能更改。
- 动态添加：运行时添加，比较灵活，建议使用这种方式。

注意点：
- 使用support包中的Fragment，需要使用getSupportFragmentManager获取FragmentManager。
- FragmentTransaction对象有以下几种方法：
	- add()：向Activity中添加一个Fragment
	- remove()：从Activity中移除一个Fragment，如果这个被移除的Fragment没有添加到回退栈中，就会被销毁
	- replace():使用另外一个Fragment替换当前的，实际上是先remove再add
	- hide()：隐藏当前的Fragment，使其不可见
	- show()：显示之前的Fragment
	- commit()：提交事务，所有对Fragment的操作就需要commit，异步操作


## 四、回退栈 ##
Fragment的回退栈是用来保存每一次Fragment事物发生的变化，如果将Fragment任务添加到回退栈中，当用户点击后退按钮时，将看到上一次保存的Fragment。一旦Fragment完全从后退栈中退出，用户再次点击后退栈，则退出当前Activity。

> 使用FragmentTransaction.addToBackStack(String) 将当前事物的变化情况添加到回退栈中

### 实例 ###
```
FragmentTransaction transaction=getFragmentManager().beginTransaction();
                
transaction.hide(MyFragment.this);
transaction.add(R.id.frameLayout,YourFragment.newInstance("小明"));

//transaction.replace(R.id.frameLayout,YourFragment.newInstance("小明"));
                
transaction.addToBackStack(null);
transaction.commit();
```
调用transaction.addToBackStack方法会将当前事物添加进回退栈中，所以当前的Fragment不会销毁，但是视图层次依然会被销毁，所以返回时，视图层仍然是重新绘制的。如果不希望视图重新绘制，可以先隐藏当前Fragment，再add另一个Fragment实例。

> - getSupportFragmentManagerManager：得到是的所在Fragment的父容器的管理器
> - getChildFragmentManager：得到的是所在Fragment里面子容器的管理器，主要是Fragment嵌套时使用

## 五、Fragment与Activity之间的通信 ##

### 1. Activity向Fragment传数据 ###
Activity向Fragment传数据比较简单，通过setArguments将参数传入到Bundle中，在通过getArguments取出即可。
### 2. Fragment向Activity传数据 ###
Fragment向Activity传数据可以用接口的方式，在Fragment中编写接口，在Activity中实现
### 3. Demo ###
Activity文件
```
public class MainActivity extends AppCompatActivity {
    private TextView input;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        input=findViewById(R.id.input);
        FragmentOne fragmentOne=FragmentOne.newInstance("Lina");
        //实现接口
        fragmentOne.setReceiverMagListener(new FragmentOne.ReceiverMsgListener() {
            @Override
            public void receive(String name) {
                input.setText(name);
            }
        });
        getSupportFragmentManager()
                .beginTransaction()
                .add(R.id.fragment,fragmentOne)
                .commit();
    }
}
```
Fragment文件
```
public class FragmentOne extends Fragment {
    private String name;
    private Button send;
    private EditText edit;
    private Activity mActivity;
    private ReceiverMsgListener listener;
    public static FragmentOne newInstance(String name) {

        Bundle args = new Bundle();
        args.putString("name",name);
        FragmentOne fragment = new FragmentOne();
        fragment.setArguments(args);
        return fragment;
    }
    //设置接口，有Activity实现接口
    public FragmentOne setReceiverMagListener(ReceiverMsgListener listener){
        this.listener=listener;
        return this;
    }

    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        mActivity= (Activity) context;
        name=getArguments().getString("name");
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_one,container,false);
    }

    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        send=view.findViewById(R.id.send);
        edit=view.findViewById(R.id.edit);
        edit.setText(name);
        send.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String str=edit.getText().toString();
                //调用接口
                listener.receive(str);
            }
        });
    }
    //定义接口
    public interface ReceiverMsgListener{
        void receive(String name);
    }
}
```


