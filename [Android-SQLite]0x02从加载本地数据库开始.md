
## 0x00 引言

>好久以前就写了，但是一直拖拖拉拉的，在草稿箱里放了两个星期还没写完，想想这样托下去又要废掉了，还是分开来吧，写多少是多少。

Android的SQLite数据库简单使用一段时间了，现在想抽些时间总结下，不然总感觉很乱。


## 0x01 SQLiteExpert

先说一个工具，`SQLite Expert`，一款SQLite数据库管理工具，下载地址： [http://www.sqliteexpert.com/](http://www.sqliteexpert.com/)，Personal Edition是免费的，日常使用基本足够，需要专业版的可以自行网上找寻。当然，其他免费的工具还有很多，如：`SQLite Database Browser`、 `SQLiteManager` 等。
![](http://img.blog.csdn.net/20130916222611000?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


## 0x02 加载本地数据库

假设我们SDCard的根目录上已经存在一个数据库，名称为：ChinaCity.db。这时候我们可以这样操作：

``` java
public void readDataBaseFromSDCard() {
    String dbPath = Environment.getExternalStorageDirectory() + "/ChinaCity.db";

    File dbFile = new File(dbPath);
    if (!dbFile.exists()) {
        Toast.makeText(getApplicationContext(), "请先点击拷贝到SDCard", Toast.LENGTH_LONG).show();
        return;
    }

    openOrCreateDatabase(dbPath, SQLiteDatabase.CREATE_IF_NECESSARY, null);

    SQLiteDatabase db = SQLiteDatabase.openDatabase(dbPath, null, SQLiteDatabase.OPEN_READWRITE);
    // db = openOrCreateDatabase(dbPath, SQLiteDatabase.OPEN_READWRITE, null);

    StringBuffer sb = new StringBuffer();
    Cursor cursor = db.rawQuery("select * from china_provinces_code", null);
    while (cursor.moveToNext()) {
        int id = cursor.getInt(cursor.getColumnIndex("_id"));
        String name = cursor.getString(cursor.getColumnIndex("name"));
        sb.append(id + ":" + name + " \n");
    }
    System.out.println(sb.toString());
}
```

先用SQLiteDatabase的一个静态方法openDatabase打开一个数据库，第一个参数为数据库文件路径，第二个参数一般为null，第三个参数为打开数据库的方式，由于只需要读取数据，所以我们选择 `SQLiteDatabase.OPEN_READONLY`，此种方式的好处是数据库存在的话就不会出错；另外一种为 `SQLiteDatabase.OPEN_READWRITE`，以读写的方式打开。执行上面代码，我们会看与上图信息相同的：

![](http://img.blog.csdn.net/20130916225300890?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

开发的时候，我们可能会需要一些初始化数据，比如城市代码信息，这样我们就可以事先创建好一个数据库，写入初始数据，将其放入自己的apk中一起分发。这个时候我们就可以直接从资源文件中读取这个数据库文件写入到应用的数据库目录或者SDCard中，然后就可以对其操作，拷贝可以这样操作：

``` java
/**
 * 拷贝资源中数据库
 *
 * @param where 1SDCARD,2LOCAL
 */
public void copyDataBase(int where) {
    // 每个应用都有一个数据库目录，他位于 /data/data/packagename/databases/目录下
    String packageName = "com.ttdevs.citydata"; // xml中配置的
    String dbName = "ChinaCity.db";
    String dbPath = null;
    if (where == 1) { // sdcard
        dbPath = Environment.getExternalStorageDirectory() + File.separator + dbName;
    } else { // local, TODO Environment.getDataDirectory()
        dbPath = "/data/data/" + packageName + "/databases/" + dbName;
    }

    if (where == 2) {
        new File("/data/data/" + packageName + "/databases/").mkdirs();
    }

    if (where == 1 && !Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState())) {
        return; // 未挂载外部存储，拷贝到内部不用判断
    }

    File dbFile = new File(dbPath);
    if (dbFile.exists()) {
        dbFile.delete();
    }
    try {
        dbFile.createNewFile();
    } catch (IOException e1) {
        e1.printStackTrace();
        return;
    }

    try {
        InputStream is = getResources().getAssets().open(dbName);
        OutputStream os = new FileOutputStream(dbPath);

        byte[] buffer = new byte[1024];
        int length = 0;
        while ((length = is.read(buffer)) > 0) {
            os.write(buffer, 0, length);
        }

        os.flush();
        os.close();
        is.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
    Toast.makeText(getApplicationContext(), "拷贝成功", Toast.LENGTH_LONG).show();
}
```

代码不是很严谨，凑活着看哈。经过以上步骤，我们就可以开始使用数据库，数据库的常用操作，在接下来的介绍中继续。


## 0x03 下载

ChinaCity.db数据库： [下载](http://download.csdn.net/detail/ttdevs/6332431)

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


