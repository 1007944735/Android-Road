## Tansition动画和共享元素 ##

Android为开发者提供很多动画的支持，从Draw Animation和View Animation，再到后面加入的Property Animation。而Transition Animation是在Api19中加入的。因为随着Android动画场面的越来越大，一个动画中涉及的view越来越多，如果使用是之前的动画，需要为每个view设置一个开始状态和结束状态，随着view个数的增加，情况会越来越复杂。

### 一、Transition 动画 ###

![](https://i.imgur.com/hfMrT7f.gif)

一、定义起始帧和结束帧
**起始帧**
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">

    <TextView
        android:layout_width="0dp"
        android:layout_height="100dp"
        android:layout_weight="1"
        android:background="#ff0000"
        android:transitionName="image1" />

    <TextView
        android:layout_width="0dp"
        android:layout_height="100dp"
        android:layout_weight="1"
        android:background="#00ff00"
        android:transitionName="image2" />

    <TextView
        android:layout_width="0dp"
        android:layout_height="100dp"
        android:layout_weight="1"
        android:background="#0000ff"
        android:transitionName="image3" />
</LinearLayout>
```
**结束帧**
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal">

    <TextView
        android:layout_width="0dp"
        android:layout_height="100dp"
        android:layout_marginTop="300dp"
        android:layout_weight="1"
        android:background="#ff0000"
        android:transitionName="image1" />

    <TextView
        android:layout_width="0dp"
        android:layout_height="100dp"
        android:layout_marginTop="200dp"
        android:layout_weight="1"
        android:background="#00ff00"
        android:transitionName="image2" />

    <TextView
        android:layout_width="0dp"
        android:layout_height="100dp"
        android:layout_marginTop="100dp"
        android:layout_weight="1"
        android:background="#0000ff"
        android:transitionName="image3" />

</LinearLayout>
```

> 对应的view下要设置相同的transitionName(xml中：android：transitionName；java中：view.setTranstionName)或者相同的id。其实Transition动画有对应的匹配规则：
> 1. MATH_NAME:如果指定了android:transtionName,那么会直接匹配两个相同的transitionName，优先级最高
> 2. MATH_INSTANCE:开始场景和结束场景中有相同的引用，优先级较高
> 3. MATH_ID:匹配相同的id，优先级较低
> 4. MATH_ITEM_ID:如果是ListItem的item，匹配相同的id

二、定义一个Activity，用来展示

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="move" />

    <LinearLayout
        android:id="@+id/rootView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <include layout="@layout/layout_transition_part_0" />
    </LinearLayout>
</LinearLayout>
```

> 这里需要一个父容器来包含开始帧，如这里的LinearLayout

三、开始动画

```
public class TransitionActivity extends AppCompatActivity {
    LinearLayout rootView;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_transition);
        rootView=findViewById(R.id.rootView);

    }

    public void move(View view) {
		//创建结束Scene
        Scene scene= Scene.getSceneForLayout(rootView,R.layout.layout_transition_part_1,this);
		//使用内置动画ChangeBounds
        ChangeBounds bounds=new ChangeBounds();
        bounds.setDuration(5000);
        bounds.setInterpolator(new LinearInterpolator());
		//开始动画
        TransitionManager.go(scene,bounds);
    }
}
```

> 使用Scene.getSceneForLayout()来创建一个结束的场景(Scene)，第一个参数是父容器，第二个参数是结束帧的布局文件，第三个参数是上下文对象，然后使用TransitionManager.go()来开启动画。这里使用了内置的动画效果ChangeBounds(能够处理view本身大小，位置的变化)，类似的内置动画效果还有Slide(边界),fade(透明度)等


### 二、延迟动画 ###

如果每次像上面的例子一样，需要定义个结束场景的layout，那么就会非常的麻烦，所以Transition框架提供了一个更加简单的机制，延迟动画

```
ChangeBounds bounds=new ChangeBounds();
bounds.setDuration(5000);
bounds.setInterpolator(new LinearInterpolator());
//开始延迟动画
TransitionManager.beginDelayedTransition(rootView,bounds);
改变view的属性
LinearLayout.LayoutParams params= (LinearLayout.LayoutParams) image1.getLayoutParams();
params.height=200;
image1.setLayoutParams(params);
LinearLayout.LayoutParams param2= (LinearLayout.LayoutParams) image2.getLayoutParams();
params.height=200;
image2.setLayoutParams(params);
LinearLayout.LayoutParams param3= (LinearLayout.LayoutParams) image3.getLayoutParams();
params.height=200;
image3.setLayoutParams(params);
```

> 在执行TransitionManager.beginDelayedTransition后，系统会保存一个当前视图树的状态场景，然后直接修改视图树，在下一次绘制时，系统会自动对比之前保存的视图树，然后执行一步动画

### 三、自定义Transition ###

上面的例子中，是使用内置的动画效果ChangeBounds来实现动画的，如果我们要改变view的背景色动画，可以自定义一个Transition，继承Transition

```
public class ChangeColor extends Transition {
    private static final String SHARE = "background";
	//记录开始状态的颜色
    @Override
    public void captureStartValues(TransitionValues transitionValues) {
        if (transitionValues.view.getBackground() instanceof ColorDrawable) {
            transitionValues.values.put(SHARE, ((ColorDrawable) transitionValues.view.getBackground()).getColor());
        }
    }
	//记录结束状态的颜色
    @Override
    public void captureEndValues(TransitionValues transitionValues) {
        if (transitionValues.view.getBackground() instanceof ColorDrawable) {
            transitionValues.values.put(SHARE, ((ColorDrawable) transitionValues.view.getBackground()).getColor());
        }
    }
	//创建动画，将颜色的值设置给view
    @Override
    public Animator createAnimator(ViewGroup sceneRoot, TransitionValues startValues, TransitionValues endValues) {
        if (null == startValues || null == endValues) {
            return null;
        }
        final View view=endValues.view;
        Object startColor=startValues.values.get(SHARE);
        Object endColor=endValues.values.get(SHARE);
        if(startColor!=endColor){
            ValueAnimator animator=ValueAnimator.ofObject(new ArgbEvaluator(),startColor,endColor);
            animator.setDuration(5000);
            animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
                @Override
                public void onAnimationUpdate(ValueAnimator animation) {
                    int values= (int) animation.getAnimatedValue();
                    view.setBackgroundColor(values);
                }
            });
            return animator;
        }
        return null;
    }
	//返回定义的储存key
    @Override
    public String[] getTransitionProperties() {
        return new String[]{SHARE};
    }
}
```

### 四、源码简要分析 ###

```
Scene scene= Scene.getSceneForLayout(rootView,R.layout.layout_transition_part_1,this);
TransitionManager.go(scene,bounds);
```
Scene.getSceneForLayout方法的作用是创建一个Scene，创建一个SpareArray防止重复创建

```
public static Scene getSceneForLayout(ViewGroup sceneRoot, int layoutId, Context context) {
        SparseArray<Scene> scenes = (SparseArray<Scene>) sceneRoot.getTag(
                com.android.internal.R.id.scene_layoutid_cache);
        if (scenes == null) {
            scenes = new SparseArray<Scene>();
            sceneRoot.setTagInternal(com.android.internal.R.id.scene_layoutid_cache, scenes);
        }
        Scene scene = scenes.get(layoutId);
        if (scene != null) {
            return scene;
        } else {
            scene = new Scene(sceneRoot, layoutId, context);
            scenes.put(layoutId, scene);
            return scene;
        }
    }
```

TransitionManager.go()方法直接调用changeScene(),changeScene()依次调用sceneChangeSetup，scene.enter，sceneChangeRunTransition

```
private static void changeScene(Scene scene, Transition transition) {

	final ViewGroup sceneRoot = scene.getSceneRoot();
	if (!sPendingTransitions.contains(sceneRoot)) {
	if (transition == null) {
		scene.enter();
	} else {
		sPendingTransitions.add(sceneRoot);

		Transition transitionClone = transition.clone();
		transitionClone.setSceneRoot(sceneRoot);

		Scene oldScene = Scene.getCurrentScene(sceneRoot);
		if (oldScene != null && oldScene.isCreatedFromLayoutResource()) {
			transitionClone.setCanRemoveViews(true);
		}
		//准备工作，记录场景的状态并停止进行中的动画
		sceneChangeSetup(sceneRoot, transitionClone);
		//先移除rootView中的所有view，再将layout加入到rootView中
		scene.enter();
		//开启动画设置
		sceneChangeRunTransition(sceneRoot, transitionClone);
		}
	}
}
```

### 五、页面过渡动画 ###

Android5.0引入了页面内容过渡动画和也弥漫共享动画，这两个动画都是基于Transition动画。在Android5.0之前，我们从一个Activity跳转到另一个Activity时，需要设置一个动画，一般通过
overridePendingTransition,如下，但这种方式只能针对页面的所有元素
```
Intent intent=new Intent(this,MainActivity.class);
startActivity(intent);
overridePendingTransition(R.anim.enter,R.anim.exit);
```

**页面内容过渡元素**

页面内容过渡元素指的是页面整体的过渡，即从一个Activity过渡到另一个Activity。页面内容过渡分为四个动画(Activity A 跳转到Activity B，再从Activity B回到Activity A),exit(从A离开)，enter(B进入),return(从B离开),reenter(A再次进入)

![](https://i.imgur.com/87PBA3Y.gif)

```
public class InterimActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.layout_interim);
        getWindow().setExitTransition(new Explode());
        findViewById(R.id.text).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent=new Intent(InterimActivity.this,TransitionActivity.class);
                startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(InterimActivity.this).toBundle());
            }
        });
    }
}
```

> 启动页面是需要调用startActivity(Intent intent, @Nullable Bundle options) ，返回时需要将finish改为finishAfterTransition。


**共享元素**

共享元素是从A页面中的一个view过渡到B页面中的一个view可以做一个平滑的动画

![](https://i.imgur.com/tV4U8aM.gif)

1. 在两个Activity的布局中为需要进行过渡变化的view设置相同的transitionName

```
android:transitionName //xml
view.setTransitionName() //java
```
> 需要为列表中的每个item设置不同的transitionName

2. 在startActivity中，通过ActivityOptionsCompat.makeSceneTransitionAnimation()，关联两个Activity中间的view

```
startActivity(intent, ActivityOptionsCompat.makeSceneTransitionAnimation(this, view, "" + urls.get(position)).toBundle());
```

**更新共享元素的对应关系**

```
setExitSharedElementCallback(new SharedElementCallback() {
	@Override
	public void onMapSharedElements(List<String> names, Map<String, View> sharedElements) {
		if (bundle != null) {
		sharedElements.clear();
		names.clear();
		int i = bundle.getInt("index", 0);
		ImageView imageView = manager.findViewByPosition(i).findViewById(R.id.picture);
//		sharedElements.put("" + datas.get(i), imageView);
		sharedElements.put("" + urls.get(i), imageView);
		bundle = null;
		}
	}
});
```
在SharedElementCallback的onMapSharedElements中更新共享元素的对应

**延迟共享元素**

有时候希望跳转之后的共享元素完全layout完毕之后再进行执行，可以使用postponeEnterTransition()和startPostponedEnterTransition()

### 六、demo ###

CommonShareActivity的xml文件
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <android.support.v7.widget.RecyclerView
        android:id="@+id/recycle"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    </android.support.v7.widget.RecyclerView>
</LinearLayout>
```

CommonShareActivity
```
public class CommonShareActivity extends AppCompatActivity implements OnItemClickListener {
    private RecyclerView recycle;
//    private List<Integer> datas;
    private List<String> urls;
    private CommonAdapter adapter;
    private Bundle bundle;
    private GridLayoutManager manager;


    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_common_share);
        recycle = findViewById(R.id.recycle);
//        datas = initData();
        urls = initData();
        manager = new GridLayoutManager(this, 3);
        recycle.setLayoutManager(manager);
//        adapter = new CommonAdapter(this, datas);
        adapter = new CommonAdapter(this, urls);
        adapter.setOnItemClickListener(this);
        recycle.setAdapter(adapter);

        setExitSharedElementCallback(new SharedElementCallback() {
            @Override
            public void onMapSharedElements(List<String> names, Map<String, View> sharedElements) {
                if (bundle != null) {
                    sharedElements.clear();
                    names.clear();
                    int i = bundle.getInt("index", 0);
                    ImageView imageView = manager.findViewByPosition(i).findViewById(R.id.picture);
//                    sharedElements.put("" + datas.get(i), imageView);
                    sharedElements.put("" + urls.get(i), imageView);
                    bundle = null;
                }
            }
        });
    }

//    private List<Integer> initData() {
//        List<Integer> datas = new ArrayList();
//        datas.add(R.drawable.head_1);
//        datas.add(R.drawable.head_2);
//        datas.add(R.drawable.head_3);
//        datas.add(R.drawable.head_4);
//        return datas;
//    }

    private List<String> initData() {
        List<String> datas = new ArrayList();
        datas.add("http://tupian.qqjay.com/tou2/2017/1216/643d1e4ab592af2a66cfc8ef1a012e79.jpg");
        datas.add("http://img4.duitang.com/uploads/item/201507/11/20150711193532_E2zF8.thumb.700_0.jpeg");
        datas.add("http://b-ssl.duitang.com/uploads/item/201508/23/20150823182247_xJzS8.jpeg");
        datas.add("http://b-ssl.duitang.com/uploads/item/201511/21/20151121150533_uyRf5.jpeg");
        return datas;
    }

    @Override
    public void onItemClick(View view, int position) {
        Intent intent = new Intent(this, ImageActivity.class);
        intent.putExtra("position", position);
//        intent.putIntegerArrayListExtra("datas", (ArrayList<Integer>) datas);
//        startActivity(intent, ActivityOptionsCompat.makeSceneTransitionAnimation(this, view, "" + datas.get(position)).toBundle());


        intent.putStringArrayListExtra("datas", (ArrayList<String>) urls);
        startActivity(intent, ActivityOptionsCompat.makeSceneTransitionAnimation(this, view, "" + urls.get(position)).toBundle());
    }

    class CommonAdapter extends RecyclerView.Adapter<VH> {
//        private List<Integer> datas;
        private List<String> urls;
        private Context context;
        private OnItemClickListener listener;
        private VH vh;

//        public CommonAdapter(Context context, List<Integer> datas) {
//            this.datas = datas;
//            this.context = context;
//        }

        public CommonAdapter(Context context, List<String> urls) {
            this.urls = urls;
            this.context = context;
        }

        @Override
        public VH onCreateViewHolder(ViewGroup parent, int viewType) {
            View view = LayoutInflater.from(context).inflate(R.layout.item_image, parent, false);
            vh = new VH(view);
            return vh;
        }

        @Override
        public void onBindViewHolder(VH holder, final int position) {
//            int width = getResources().getDisplayMetrics().widthPixels / 3;
//            ViewGroup.LayoutParams params = holder.picture.getLayoutParams();
//            params.width = width;
//            params.height = width;
//            holder.picture.setTransitionName("" + datas.get(position));
            holder.picture.setTransitionName("" + urls.get(position));
//            holder.picture.setLayoutParams(params);
//            holder.picture.setBackgroundResource(datas.get(position));
            Glide.with(context).load(urls.get(position)).into(holder.picture);
            holder.picture.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    listener.onItemClick(v, position);
                }
            });
        }

        @Override
        public int getItemCount() {
            return urls.size();
        }

        public void setOnItemClickListener(OnItemClickListener listener) {
            this.listener = listener;
        }
    }


    class VH extends RecyclerView.ViewHolder {
        public ImageView picture;

        public VH(View itemView) {
            super(itemView);
            picture = itemView.findViewById(R.id.picture);
        }
    }

    @Override
    public void onActivityReenter(int resultCode, Intent data) {
        super.onActivityReenter(resultCode, data);
        if (resultCode == -1) {
            bundle = new Bundle(data.getExtras());
        }
    }
}
```

ImageActivity的xml文件
```
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <android.support.v4.view.ViewPager
        android:id="@+id/viewPager"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    </android.support.v4.view.ViewPager>
</FrameLayout
```

ImageActivity
```
public class ImageActivity extends AppCompatActivity {
    private int position;
    private ViewPager viewPager;
    //    private List<Integer> datas;
    private List<String> urls;
    private ViewPagerAdapter adapter;
    private int width;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_image);
        viewPager = findViewById(R.id.viewPager);
        position = getIntent().getIntExtra("position", 0);
//        datas = getIntent().getIntegerArrayListExtra("datas");
        urls = getIntent().getStringArrayListExtra("datas");
        width=getResources().getDisplayMetrics().widthPixels;
        postponeEnterTransition();
//        adapter = new ViewPagerAdapter(this, datas);
        adapter = new ViewPagerAdapter(this, urls);
        viewPager.setAdapter(adapter);
        viewPager.setCurrentItem(position, false);
        setEnterSharedElementCallback(new SharedElementCallback() {
            @Override
            public void onMapSharedElements(List<String> names, Map<String, View> sharedElements) {
                sharedElements.clear();
//                sharedElements.put("" + datas.get(viewPager.getCurrentItem()), adapter.getImageView());
                sharedElements.put("" + urls.get(viewPager.getCurrentItem()), adapter.getImageView());
            }
        });
    }

    class ViewPagerAdapter extends PagerAdapter {
        //        public List<Integer> datas;
        public List<String> urls;
        public Context context;
        public View parentView;


//        public ViewPagerAdapter(Context context, List<Integer> datas) {
//            this.datas = datas;
//            this.context = context;
//        }

        public ViewPagerAdapter(Context context, List<String> urls) {
            this.urls = urls;
            this.context = context;
        }

        @Override
        public int getCount() {
            return urls.size();
        }

        @Override
        public boolean isViewFromObject(View view, Object object) {
            return view == object;
        }

        @Override
        public Object instantiateItem(ViewGroup container, final int position) {

            View view = LayoutInflater.from(context).inflate(R.layout.item_image_pager, container, false);
            final ImageView pi = view.findViewById(R.id.pic);
//            pi.setTransitionName("" + datas.get(position));
//            pi.setImageDrawable(getDrawable(datas.get(position)));
            pi.setTransitionName("" + urls.get(position));
            pi.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    finishAfterTransition();
                }
            });
            Glide.with(context)
                    .load(urls.get(position))
                    .asBitmap()
                    .into(new BitmapImageViewTarget(pi){
                        @Override
                        public void onResourceReady(Bitmap resource, GlideAnimation<? super Bitmap> glideAnimation) {
//                            pi.setImageBitmap(resource);
                            Log.d("TAG",position+"|BeforeWidth:"+resource.getWidth());
                            Log.d("TAG",position+"|BeforeHeight:"+resource.getHeight());
                            float a=(float) width/(float) resource.getWidth();
                            Matrix matrix=new Matrix();
                            matrix.setScale(a,a);
                            Bitmap bitmap=Bitmap.createBitmap(resource,0,0, resource.getWidth(),resource.getHeight(),matrix,false);
                            Log.d("TAG",position+"|AfterWidth:"+bitmap.getWidth());
                            Log.d("TAG",position+"|AfterHeight:"+bitmap.getHeight());
                            pi.setImageBitmap(bitmap);

                        }
                    });
//            Glide.with(context)
//                    .load(urls.get(position))
//                    .asBitmap()
//                    .into(pi);
//            startPostponedEnterTransition();
            container.addView(view);
            return view;
        }

        @Override
        public void destroyItem(ViewGroup container, int position, Object object) {
            container.removeView((View) object);
        }

        @Override
        public void setPrimaryItem(ViewGroup container, int position, Object object) {
            parentView = (View) object;
        }

        public ImageView getImageView() {
            return parentView.findViewById(R.id.pic);
        }
    }

    @Override
    public void finishAfterTransition() {
        Intent intent = new Intent();
        intent.putExtra("index", viewPager.getCurrentItem());
        setResult(-1, intent);
        super.finishAfterTransition();
    }
}
```



