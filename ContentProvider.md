## 一、ContentProvider简介 ##
ContentProvider,即内容提供程序，是Android四大组件之一。内容提供程序管理对结构化数据集的访问。它们封装数据，并提供用于定于数据安全性的机制。

> ContentProvider 内容提供者
> ContentResolver 内容访问者

## 二、内容URI ##
- 定义：统一资源标识符
- 作用：ContentProvider使用内容URI的路径部分来选择要访问的表
- 使用方式：
	> content://com.carson.provider/User/1
	> content://　主题，ContentProvider的URI前缀
	> com.carson.provider 授权信息，ContentProvider的唯一标识
	> User 表名，ContentProvider指向数据库中的某个表名
	> 1 记录，表中的某个记录（若无指定，则返回全部记录）
	
	> \* :匹配任意长度的任何有效字符的字符串
	> \# :匹配任意长的数字字符的字符串

## 三、MIME数据类型 ##
- 组成：由两部分组成，类型+子类型

	```
	text/html
	text/css
	text/xml
	application/pdf
	```
- 作用：指定摸个扩展名的文件用某种应用程序打开
- ContentProvider根据URI返回MIME类型
	> ContentProvider.getType(uri)
- MIME类型形式
	```
	//形式1：单条记录
	vnd.android.cursor.item/自定义
	//形式2：多条记录
	vnd.android.cursor.dir/自定义
	```

## 四、ContentProvider类 ##
- 主要方法

	```
  	public Uri insert(Uri uri, ContentValues values) 
  	// 外部进程向 ContentProvider 中添加数据

  	public int delete(Uri uri, String selection, String[] selectionArgs) 
  	// 外部进程 删除 ContentProvider 中的数据

  	public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)
  	// 外部进程更新 ContentProvider 中的数据

  	public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs,  String sortOrder)　 
  	// 外部应用 获取 ContentProvider 中的数据

	// 注：
  	// 1. 上述4个方法由外部进程回调，并运行在ContentProvider进程的Binder线程池中（不是主线程）
 	// 2. 存在多线程并发访问，需要实现线程同步
   	// a. 若ContentProvider的数据存储方式是使用SQLite & 一个，则不需要，因为SQLite内部实现好了线程同步，若是多个SQLite则需要，因为SQL对象之间无法进行线程同步
  		// b. 若ContentProvider的数据存储方式是内存，则需要自己实现线程同步

	public boolean onCreate() 
	// ContentProvider创建后 或 打开系统后其它进程第一次访问该ContentProvider时 由系统进行调用
	// 注：运行在ContentProvider进程的主线程，故不能做耗时操作

	public String getType(Uri uri)
	// 得到数据类型，即返回当前 Url 所代表数据的MIME类型
	```

> 自定义ContentProvider，上述6个方法必须实现
## 五、ContentResolver类 ##
- 作用：统一管理不同的ContentProvider间的操作

	> 通过不同的URI即可操作不同的ContentProvider中的数据
	> 外部进程通过ContentResolver类和ContentProvider进行交互

- 使用方式
	```
	// 外部进程向 ContentProvider 中添加数据
	public Uri insert(Uri uri, ContentValues values)　 
	
	// 外部进程 删除 ContentProvider 中的数据
	public int delete(Uri uri, String selection, String[] selectionArgs)

	// 外部进程更新 ContentProvider 中的数据
	public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)　 

	// 外部应用 获取 ContentProvider 中的数据
	public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)
	```
- 实例说明
	```
	// 使用ContentResolver前，需要先获取ContentResolver
	// 可通过在所有继承Context的类中 通过调用getContentResolver()来获得ContentResolver
	ContentResolver resolver =  getContentResolver(); 

	// 设置ContentProvider的URI
	Uri uri = Uri.parse("content://cn.scu.myprovider/user"); 

	// 根据URI 操作 ContentProvider中的数据
	// 此处是获取ContentProvider中 user表的所有记录 
	Cursor cursor = resolver.query(uri, null, null, null, "userid desc");
	```
Android 提供了3个用于辅助ContentProvider的工具类
- ContentUris：
	a. 作用：操作Uri
	b. 核心方法：withAppendedId()和pauseId()
	> withAppendedId()：向Uri追加一个id
	> pauseId()：从Uri中获取id
- UriMatcher：
	a. 作用：在ContentProvider中注册URI；根据URI匹配ContentProvider中对应的数据库
	b. 具体使用：
		步骤1. 初始化UriMatcher
		步骤2.在ContentProvider中注册URI（addUri）
		步骤3.根据URI匹配code（matcher()）
- ContentObserver：
	- 作用：观察Uri引起ContentProvider中的数据变化&通知外界
	> 当ContentProvider中的数据发生变化时，就会触发该类
	- 具体使用：
	```
	// 步骤1：注册内容观察者ContentObserver
	getContentResolver().registerContentObserver（uri）；
	// 通过ContentResolver类进行注册，并指定需要观察的URI

	// 步骤2：当该URI的ContentProvider数据发生变化时，通知外界（即访问该ContentProvider数据的访问者）
	public class UserContentProvider extends ContentProvider {
    	public Uri insert(Uri uri, ContentValues values) {
        	db.insert("user", "userid", values);
        	getContext().getContentResolver().notifyChange(uri, null);
        	// 通知访问者
    	}
	}

	// 步骤3：解除观察者
	getContentResolver().unregisterContentObserver（uri）；
	// 同样需要通过ContentResolver类进行解除
	```
## 六、Demo ##
1. 创建数据库类DBHelper
```
public class DBHelper extends SQLiteOpenHelper {

    // 数据库名
    private static final String DATABASE_NAME = "finch.db";

    // 表名
    public static final String USER_TABLE_NAME = "user";
    public static final String JOB_TABLE_NAME = "job";

    private static final int DATABASE_VERSION = 1;
    //数据库版本号

    public DBHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {

        // 创建两个表格:用户表 和职业表
        db.execSQL("CREATE TABLE IF NOT EXISTS " + USER_TABLE_NAME + "(_id INTEGER PRIMARY KEY AUTOINCREMENT," + " name TEXT)");
        db.execSQL("CREATE TABLE IF NOT EXISTS " + JOB_TABLE_NAME + "(_id INTEGER PRIMARY KEY AUTOINCREMENT," + " job TEXT)");
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion)   {

    }
}
```

2. 自定义ContentProvider类
```
public class MyContentProvider extends ContentProvider {
    //ContentProvider 唯一标识符
    public static final String AUTHORITIES="com.zust.provider";

    public static final int User_Code=1;
    public static final int Job_Code =2;

    private static final UriMatcher matcher;

    private SQLiteDatabase db;
    //注册URI
    static {
        //UriMatcher.NO_MATCH 创建时不进行匹配
        //初始化
        matcher=new UriMatcher(UriMatcher.NO_MATCH);

        matcher.addURI(AUTHORITIES,"user",User_Code);
        matcher.addURI(AUTHORITIES,"job",Job_Code);
    }
    @Override
    public boolean onCreate() {
        DBHelper helper=new DBHelper(getContext());
        db=helper.getWritableDatabase();
        db.execSQL("delete from user");
        db.execSQL("insert into user values(1,'Carson');");
        db.execSQL("insert into user values(2,'Kobe');");

        db.execSQL("delete from job");
        db.execSQL("insert into job values(1,'Android');");
        db.execSQL("insert into job values(2,'iOS');");
        return true;
    }

    @Nullable
    @Override
    public Cursor query(@NonNull Uri uri, @Nullable String[] projection, @Nullable String selection, @Nullable String[] selectionArgs, @Nullable String sortOrder) {
        String table=getTableName(uri);
        return db.query(table,projection,selection,selectionArgs,null,null,sortOrder,null);
    }



    @Nullable
    @Override
    public String getType(@NonNull Uri uri) {
        return null;
    }

    @Nullable
    @Override
    public Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
        String table=getTableName(uri);
        db.insert(table,null,values);
        //当该Url的ContentProvider发生变化时，通知外界
        getContext().getContentResolver().notifyChange(uri,null);
        return uri;
    }

    @Override
    public int delete(@NonNull Uri uri, @Nullable String selection, @Nullable String[] selectionArgs) {
        return 0;
    }

    @Override
    public int update(@NonNull Uri uri, @Nullable ContentValues values, @Nullable String selection, @Nullable String[] selectionArgs) {
        return 0;
    }

    private String getTableName(Uri uri) {
        String table=null;
        switch (matcher.match(uri)){
            case 1:
                table=DBHelper.USER_TABLE_NAME;
                break;
            case 2:
                table=DBHelper.JOB_TABLE_NAME;
                break;
        }
        return table;
    }
}
```
3. 注册ContentProvider类
```
<provider
            android:authorities="com.zust.provider"
            android:name=".MyContentProvider"/>
```

4. 获取ContentResolver查询
```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity);
        /*
        对User表进行操作
         */
        //设置Uri
        Uri uri_user=Uri.parse("content://com.zust.provider/user");

        //插入数据
        ContentValues values=new ContentValues();
        values.put("_id",3);
        values.put("name","Marry");
        //获取ContentResolver
        ContentResolver resolver=getContentResolver();
        resolver.insert(uri_user,values);


        //查询数据
        Cursor cursor=resolver.query(uri_user,new String[]{"_id","name"},null,null,null);
        while (cursor.moveToNext()){
            System.out.println("query book:" + cursor.getInt(0) +" "+ cursor.getString(1));
        }
        //关闭游标
        cursor.close();
    }

}
```

5. 结果
![](https://i.imgur.com/YUo9w2N.png)