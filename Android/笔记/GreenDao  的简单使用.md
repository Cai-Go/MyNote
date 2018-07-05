# GreenDao 的简单使用

## GreenDao 的使用方法
1. app 目录下的 build.gradle 配置如下：
````groovy
	apply plugin: 'org.greenrobot.greendao' 
	
	...
    dependencies {
	   implementation 'org.greenrobot:greendao:3.2.2' 
	}
````

2. 在bulid.gradle下进行配置
````groovy
	buildscript {
    repositories {
        jcenter()
        mavenCentral()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.1'
        classpath 'org.greenrobot:greendao-gradle-plugin:3.2.2'
    	}
	}
````

3. 对greendao的generator生成文件进行配置
````groovy
	greendao {
    	schemaVersion 1                // 版本
    	daoPackage '生成文件包名'      // 一般为app包名+生成文件的文件夹名
    	targetGenDir 'src/main/java' // 生成文件路径
	}
````

4. 使用 GreenDao
新建一个 MyApplication 类，继承自 Application，
````java
	
public class MyApplication extends Application {

	private static volatile MyApplication instance = null;

	private DaoMaster.DevOpenHelper mHelper;

	private SQLiteDatabase db;

	private DaoMaster mDaoMaster;

	private DaoSession daoSession;

	private  MyApplication(){}

	public static synchronized MyApplication getInstance(){
		if (instance == null) {
			instance = new MyApplication();
		}
		return instance;
	}

	@Override
	public void onCreate(){
		super.onCreate();
		setDataBase();
	}


	/**
	 *配置数据库
	 *
	 */
	private void setDataBase(){
		// 创建数据库 shop.db
		mHelper = new DaoMaster.DevOpenHelper(this,"shop.db",null);
		// 获取可写数据库
		db = mHelper.getWritableDatabase();
		// 回去数据库对象
		DaoMaster = new DaoMaster(db);
		// 获取Dao 对象管理者
		daoSession = mDaoMaster.newSession();

	}

	public DaoSession gerDaoSession(){
		return daoSession;
	}

	public SQLiteDatabase getDb(){
		return db;
	}

}

````

## 操作
### 增

insert(Entity entity):插入一条记录

````java
	 /**
     * 增
     */
    public void insert(){
        String date = new Date().toString();
        //第一个是id值，因为是自增长所以不用传入
        mDayStep = new dayStep(null,date,0);
        dao.insert(mDayStep);
    }

````

### 删

deleteByKey(Long key):根据主键删除一条记录。
delete(Entity entity):根据实体类删除一条记录，一般结合查询方法没，查询出一条记录之后删除。
deleteAll():删除所有记录。

````java
	  /**
     * 删
     * @param i 删除数据的id
     */
    public void delete(long i){
        //当然Greendao还提供了其他的删除方法，只是传值不同而已
        dao.deleteByKey(i);
    }

````
### 改

update(Entitiy entity)：更新一条记录；
````java
	 /**
     *改
     * @param i
     * @param date
     */
    public void correct(long i,String date){
        mDayStep =  new dayStep((long) i,date,0);
        dao.update(mDayStep);
    }

````

### 查

- loadAll() : 查询所有记录；
- load(Long key) : 根据主键查询一条记录；
- queryBuilder().list() : 返回:List；
- queryBuilder.where(UserDao.Properties.Name.eq("")).list() : 返回:List；
- queryRaw(String where,String selectionArg) : 返回:List。

````java
	
	 /**
     * 查
     */
    public void Search(){
        //方法一
        List<dayStep> mDayStep = dao.loadAll();
        //方法二
        //List<dayStep> mDayStep = dao.queryBuilder().list();
        //方法三 惰性加载
        //List<dayStep> mDayStep = dao.queryBuilder().listLazy();
        for (int i = 0; i < mDayStep.size(); i++) {
            String date = "";
            date = mDayStep.get(i).getDate();
            Log.d("cc", "id:  "+i+"date:  "+date);
        }
 
    }


 	/**
     *查找符合某一字段的所有元素
     */
   public void searchEveryWhere(String str){
	List<dayStep> mList = dao.queryBuilder()
                .where(dao.date
                	.eq(str))
                .build()
                .listLazy();
	}


````
### 修改或者替换
````java
	/**
     *修改或者替换（有的话就修改，没有则替换）
     */ 
    public void insertOrReplace(long i,String date){
      	mDayStep = new dayStep((long) i,date,0);
      	dao.insertOrReplace(mDayStep);
    }

````
## Greendao注解含义

@Entity 实体标识：

- schema：告知GreenDao当前实体属于哪个schema；
- active：标记一个实体处于活跃状态，活动实体有更新、删除和刷新方法；
- nameInDb：在数据库中使用的别名，默认使用的是实体的类名；
- indexs：定义索引，可以跨越多个列；
- createInDb：标记创建数据库表。

基础属性注解：

- @Id：主键 Long 型，可以通过@Id(autoincrement = true)设置自增长；
- @Property：设置一个非默认关系映射所对应的列名，默认是使用字段名。例如： @Property(nameInDb = "name")；
- @NotNull：设置数据库表当前列不能为空；
- @Transient：添加此标记后不会生成数据库表的列。

索引注解：

- @Index：使用@Index作为一个属性来创建一个索引，通过name设置索引别名，也可以通过unique给索引添加约束；
- @Unique：向数据库添加了一个唯一的约束。

关系注解：

- @ToOne：定义与另一个实体（一个实体对象）的关系；
- @ToMany：定义与多个实体对象的关系。


