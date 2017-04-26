现有的APP尝试向Material Design靠齐，开始使用Toolbar代替之前的ActionBar。

Toolbar和ActionBar的直观区别就是需要我们自己将ToolBar加到自己的布局文件中。目前的情况是：在我们的现有项目上改动，多数的Activity都是继承一个BaseActivity。为了用最小的代价达到目的，简单的思考之后，做了如下的改动：

``` java
/**
 * 带ToolBar的基类
 */
public class BaseActivity extends ActionBarActivity {
    private static final int BASE_VIEW_ID = R.layout.activity_base;
    private static final LayoutParams LAYOUT_PARAMS = new LayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT);

    private LinearLayout mParentView;
    private Toolbar mToolBar;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ViewUtils.startTranslucent(this);
        super.onCreate(savedInstanceState);
        setContentView(BASE_VIEW_ID);

    }

    @Override
    public void setContentView(int layoutResID) {
        if (BASE_VIEW_ID == layoutResID) {
            super.setContentView(layoutResID);

            mParentView = (LinearLayout) findViewById(R.id.base_parent_view);
            mToolBar = (Toolbar) findViewById(R.id.toolbar);
            initToolbar(mToolBar);
            return;
        }

        mParentView.addView(getLayoutInflater().inflate(layoutResID, null), LAYOUT_PARAMS);
    }

    @Override
    public void setContentView(View view) {
        mParentView.addView(view, LAYOUT_PARAMS);
    }

    private void initToolbar(Toolbar toolbar) {
        setSupportActionBar(toolbar);
        getSupportActionBar().setDisplayHomeAsUpEnabled(true);
        toolbar.setNavigationOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                finish();
            }
        });
    }

    public Toolbar getToolBar() {
        return mToolBar;
    }

    public void setBackground(int colorId) {
        if (null != mParentView) {
            mParentView.setBackgroundColor(getResources().getColor(colorId));
        }
    }
}
```

布局文件activity_base.xml：

``` java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
   xmlns:android="http://schemas.android.com/apk/res/android"
   android:id="@+id/base_parent_view"
   android:layout_width="match_parent"
   android:layout_height="match_parent"
   android:background="@color/global_background_status_bar"
   android:fitsSystemWindows="true"
   android:orientation="vertical">
    
   <include layout="@layout/subview_toolbar"/>
</LinearLayout>
```

在基类中添加如上代码，基本可以用最小的改动达到使用Toolbar的目的。但是，这样也存在一个问题，就是会使我们的每个Activity的布局层次多了一层。

如果有更好的思路，欢迎分享～～

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


