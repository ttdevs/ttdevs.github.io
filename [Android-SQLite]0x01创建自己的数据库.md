
## 0x01 创建自己的数据库

大多数情况，我们还是需要自己去维护一个数据库，常见的包括数据库的创建，升级，销毁等操作。 android提供了SQLiteOpenHelper抽象类，我们创建SQLiteOpenHelper的实现类，重写他的onCreate(), onUpgrade() 或者 onOpen()方法，对数据库进行管理。如下：

``` java
public class DataBaseOpenHelper extends SQLiteOpenHelper {
    private static final String DATABASE_NAME = "ChinaCity.db";
    private static final int DATABASE_VERSION = 1; // Version must be >= 1

    public DataBaseOpenHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
        System.out.println("DataBaseOpenHelper");

    }

    public DataBaseOpenHelper(Context context, int version) {
        super(context, DATABASE_NAME, null, version);
        System.out.println("DataBaseOpenHelper version");
    }

    @Override
    public void onOpen(SQLiteDatabase db) {
        if (!db.isReadOnly()) {
            db.execSQL("PRAGMA foreign_keys = ON;"); // Enable foreign key constraints 
        }
        // create table test(id integer references students(id),score integer check (score<=100 and score<=0),primary key(id,score))
        super.onOpen(db);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        CityData.createTable(db);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        CityData.dropTable(db);
        CityData.createTable(db);
    }

    public static class CityData implements BaseColumns {
        public static final String TYPE_TEXT = " text ";
        public static final String TYPE_INTEGER = " integer ";
        public static final String COMMA_SEP = ",";

        // *******************************china_city_code***********************************
        public static final String TABLE_NAME_CITY = "china_city_code";
        public static final String COLUMN_NAME_PROVINCE = "province";
        public static final String COLUMN_NAME_CITY = "city";
        public static final String COLUMN_NAME_COUNTY = "county";
        public static final String COLUMN_NAME_CODE = "code";
        public static final String SQL_CREATE_CITY = "CREATE TABLE IF NOT EXISTS " + TABLE_NAME_CITY + " (" +
                _ID + TYPE_INTEGER + " PRIMARY KEY AUTOINCREMENT, " +
                COLUMN_NAME_PROVINCE + TYPE_TEXT + COMMA_SEP +
                COLUMN_NAME_CITY + TYPE_TEXT + COMMA_SEP +
                COLUMN_NAME_COUNTY + TYPE_TEXT + COMMA_SEP +
                COLUMN_NAME_CODE + TYPE_INTEGER
                + " )";
        public static final String SQL_CREATE_CITY_INDEX = "CREATE UNIQUE INDEX IF NOT EXISTS " + COLUMN_NAME_CODE + " ON " + TABLE_NAME_CITY + "(" + COLUMN_NAME_CODE + ")";
        public static final String SQL_DELETE_CITY = "DROP TABLE IF EXISTS " + TABLE_NAME_CITY;

        // *******************************china_provinces_code*******************************
        public static final String TABLE_NAME_PROVINCE = "china_provinces_code";
        public static final String COLUMN_NAME_ID = "id";
        public static final String COLUMN_NAME_NAME = "name";
        public static final String SQL_CREATE_PROVINCE = "CREATE TABLE IF NOT EXISTS " + TABLE_NAME_PROVINCE + " (" +
                _ID + TYPE_INTEGER + " PRIMARY KEY AUTOINCREMENT, " +
                COLUMN_NAME_ID + TYPE_INTEGER + COMMA_SEP +
                COLUMN_NAME_NAME + TYPE_TEXT
                + " )";
        public static final String SQL_DELETE_PROVINCE = "DROP TABLE IF EXISTS " + TABLE_NAME_PROVINCE;

        public static void createTable(SQLiteDatabase db) {
            db.execSQL(SQL_CREATE_PROVINCE);
            db.execSQL(SQL_CREATE_CITY);
            db.execSQL(SQL_CREATE_CITY_INDEX);
        }

        public static void dropTable(SQLiteDatabase db) {
            db.execSQL(SQL_DELETE_PROVINCE);
            db.execSQL(SQL_DELETE_CITY);
        }
    }
}
```

构造函数中我们需要传入四个参数，第一个参数为上下文，第二个为数据库名称，第三个参数一般为null，第四个为数据库的版本。onCreate()方法中主要执行数据库的创建操作。onUpgrade()方法主要在数据库升级时调用，源码 `android.database.sqlite.SQLiteOpenHelper.getDatabaseLocked(boolean writable)` 中这样描述：

``` java
private SQLiteDatabase getDatabaseLocked(boolean writable) {
   if (mDatabase != null) {
       if (!mDatabase.isOpen()) {
           // Darn!  The user closed the database by calling mDatabase.close().
           mDatabase = null;
       } else if (!writable || !mDatabase.isReadOnly()) {
           // The database is already open for business.
           return mDatabase;
       }
   }

   if (mIsInitializing) {
       throw new IllegalStateException("getDatabase called recursively");
   }

   SQLiteDatabase db = mDatabase;
   try {
       mIsInitializing = true;

       if (db != null) {
           if (writable && db.isReadOnly()) {
               db.reopenReadWrite();
           }
       } else if (mName == null) {
           db = SQLiteDatabase.create(null);
       } else {
           try {
               if (DEBUG_STRICT_READONLY && !writable) {
                   final String path = mContext.getDatabasePath(mName).getPath();
                   db = SQLiteDatabase.openDatabase(path, mFactory,
                           SQLiteDatabase.OPEN_READONLY, mErrorHandler);
               } else {
                   db = mContext.openOrCreateDatabase(mName, mEnableWriteAheadLogging ?
                           Context.MODE_ENABLE_WRITE_AHEAD_LOGGING : 0,
                           mFactory, mErrorHandler);
               }
           } catch (SQLiteException ex) {
               if (writable) {
                   throw ex;
               }
               Log.e(TAG, "Couldn't open " + mName
                       + " for writing (will try read-only):", ex);
               final String path = mContext.getDatabasePath(mName).getPath();
               db = SQLiteDatabase.openDatabase(path, mFactory,
                       SQLiteDatabase.OPEN_READONLY, mErrorHandler);
           }
       }

       onConfigure(db);

       final int version = db.getVersion();
       if (version != mNewVersion) {
           if (db.isReadOnly()) {
               throw new SQLiteException("Can't upgrade read-only database from version " +
                       db.getVersion() + " to " + mNewVersion + ": " + mName);
           }

           db.beginTransaction();
           try {
               if (version == 0) {
                   onCreate(db);
               } else {
                   if (version > mNewVersion) {
                       onDowngrade(db, version, mNewVersion);
                   } else {
                       onUpgrade(db, version, mNewVersion);
                   }
               }
               db.setVersion(mNewVersion);
               db.setTransactionSuccessful();
           } finally {
               db.endTransaction();
           }
       }

       onOpen(db);

       if (db.isReadOnly()) {
           Log.w(TAG, "Opened " + mName + " in read-only mode");
       }

       mDatabase = db;
       return db;
   } finally {
       mIsInitializing = false;
       if (db != null && db != mDatabase) {
           db.close();
       }
   }
}
```

当当前版本小于新版本时，会调用onUpgrade()方法。所以上面SQLiteOpenHelper的实现思路为：构造方法中初始化一下数据库的必要参数，如数据库版本，数据库名等；在onCreate()方法中创建数据库的表；当我们需要对数据库进行升级的时候，修改数据库的版本号，这样就可以触发onUpgrade()方法，这里我们做最简单的处理：drop掉所有表然后重新创建。当然我们还可以在onOpen()方法中做一下数据库的设置操作，如设置外键生效。这样我们的数据库管理类就实现了。


## 0x02 拿到数据库对象

有了数据库管理类，对数据库进行操作我们最好封装一个操作类，在这个类中对数据库中的表进行操作。下面的代码也是网上比较常见的（对于数据库对象，也有处理成单例）。先看代码：

``` java
public class DataBaseManager {

    private DataBaseOpenHelper dbHelper;
    private SQLiteDatabase db;

    public DataBaseManager(Context context) throws SQLException {
        this.dbHelper = new DataBaseOpenHelper(context);
        this.db = dbHelper.getWritableDatabase();
    }

    public CopyOfDataBaseManager(Context context, int version) throws SQLException {
        this.dbHelper = new DataBaseOpenHelper(context, version);
        this.db = dbHelper.getWritableDatabase();
    }

    public void closeDataBase() {
        if (db != null && db.isOpen()) {
            db.close();
        }
    }

    public boolean isInitData(String tableName) {
        int count = 0;
        String sql = "select count(*) from " + tableName;
        Cursor cursor = db.rawQuery(sql, null);
        if (cursor.moveToFirst()) {
            count = cursor.getInt(0);
        }
        return count > 0;
    }

    public void clearTable(String tableName) {
        String sql = "delete from " + tableName + ";";
        sql += "update sqlite_sequence set seq = 0 where name = " + tableName + ";";
        db.execSQL(sql);
    }

    public void insertProvinceData(int id, String name) {
        String sql = "insert into " + CityData.TABLE_NAME_PROVINCE + " (" + CityData.COLUMN_NAME_ID + ","
                + CityData.COLUMN_NAME_NAME + ") VALUES(" + id + ",'" + name + "');";
        db.execSQL(sql);
        System.out.println(sql);
    }

    // TODO其他一下对表进行的操作
}
```

首先我们创建一个DataBaseOpenHelper的实例，然后通过它拿到SQLiteDatabase对象，这样我们就可以对数据库进行操作了。DataBaseOpenHelper有两种get方法：`getWritableDatabase()` 和 `getReadableDatabase()`，大家都应该知道着两种方法的含义，`getReadableDatabase()` 拿到的数据库对象不可以进行插入修改等写操作。在这两种方法的说明在我们可能需要注意几点：

- 当我们没有申请数据库操作的权限或者磁盘已满，会报错
- 数据库的更新可能需要很长时间，因此我们不能在主线程中调用
- 当我们不再对数据库进行操作时，别忘记关闭数据库

接下来就是数据库的常用操作了


![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

