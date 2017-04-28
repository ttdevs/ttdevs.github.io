
---
title: 「Android」仿联系人列表上的字母索引
date: 1970-01-01 00:00:00
updated: 2017-04-27 19:07:57
comments: true
tags:
    - Android
categories:
    - 技术
toc: true
cover: cover.jpg 
---



这个小功能github有很多。不同的应用需求会有少许差别，比如listview滑动时字母是不是跟随滑动；手动点击字母是不是在屏幕中间显示一个提示；点击时索引的背景显示出来，离开后背景消失等等，当然这些都是细节问题。实现思路上也可以有多种，比如自己去draw每个字母，然后处理滑动、借助TextView来展示字母列表等。看了几个demo，感觉和自己的需求有些差别，而且这些demo为了实现”大而全”有些多余的东西，因此决定自己写个。


## 0x01 分析

我们的需求是这样的：中文参与索引，字母或者其他开头的item不参与索引，直接放到第一个“#”里；点击字母索引其被点击字母列表颜色跟随变化；手动滑动listview字母索引的字母颜色也跟随变化；自己顺手加了个点击某个字母弹出提示的接口。最后效果如下图：

![Letter Index](http://img.blog.csdn.net/20140831140659685?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  


## 0x02 实现

决定直接使用垂直方向的LinearLayout嵌套TextView来展示字母列表。思路如下：根据索引高度，大概计算每个字母的高度，然后添加到Linearlayout中；然后处理Linearlayout的touch事件，拿到坐标计算当前应该索引哪个。其中遇到的主要问题是对TextView的把握，比如它的高度如何计算？（真正实现的时候你会发现它有个paddingTop和paddingBottom）其他的比如滑动逻辑这些调试下即可，直接上代码：

``` java
/**
 * 右侧字母索引。可作为一个独立的view插入自己的布局中<br>
 * <br>
 * 描述：<br>
 * 线型布局中嵌套一个线型布局（目的为了增大可滑动区域的面积，若不考虑此可不使用嵌套）。<br>
 * 每个字母插入内嵌的线性布局中，根据控件的高度计算每个字母的尺寸。<br>
 * <br>
 * 使用：<br>
 * 将此view插入你的布局文件中，初始化完成之后设置你的ListView和索引用的数据源(需要自己组织排序数据源)即可。<br>
 * <br>
 * 数据源的组织：若某项不参与排序则数据源中设置为'0'或其他小于'A'的ASCII字符，内部会将所有字符转换成大写，所以务必在外部做好排序。<br>
 * <br>
 * 建议：若你的数据中包括ASCII码为a0、20的字符，建议剔除，如：<br>
 * str.replace(' ', ' '); // a0->20 str.replaceAll(" ", "");<br>
 * <br>
 * 如果你处理的是中文，可使用libs中的pinyin4j处理
 *
 * @author ttdevs 2014-07-31
 */
public class ListViewLetterIndicator extends LinearLayout implements OnScrollListener {

    private static final String INDICATOR = "#ABCDEFGHIJKLMNOPQRSTUVWXYZ";

    private LinearLayout llMain;

    private ListView mListView; // ListView
    private List<String> mData; // 数据源
    private TextView tvAlert; // 显示当前的字母

    private int mIndex; // 当前所处的indicator位置
    private boolean scrollable = true; //

    public ListViewLetterIndicator(Context context) {
        this(context, null);
    }

    public ListViewLetterIndicator(Context context, AttributeSet attrs) {
        super(context, attrs);

        setOrientation(LinearLayout.VERTICAL);
        setBackgroundColor(getResources().getColor(R.color.letter_indicator_background));
        // setPadding(8, 8, 8, 8);

        llMain = new LinearLayout(getContext());
        llMain.setOrientation(LinearLayout.VERTICAL);
        llMain.setGravity(Gravity.CENTER);
        // llMain.setPadding(2, 2, 2, 2);

        int width = (int) getResources().getDimension(R.dimen.letter_indicator_width);
        addView(llMain, width, LinearLayout.LayoutParams.MATCH_PARENT);
    }

    /**
     * 设置数据源
     *
     * @param lv   绑定的ListView
     * @param data 排序用的数据源
     */
    public void setData(ListView lv, List<String> data) {
        setData(lv, data, null);
    }

    /**
     * 设置数据源
     *
     * @param lv   绑定的ListView
     * @param data 排序用的数据源
     * @param tv   显示当前所处字母的TextView
     */
    @SuppressLint("DefaultLocale")
    public void setData(ListView lv, List<String> data, TextView tv) {
        mListView = lv;
        mData = data;
        tvAlert = tv;

        for (int i = 0; i < mData.size(); i++) {
            String str = mData.get(i);
            mData.set(i, str.toUpperCase());
        }

        mListView.setOnScrollListener(this);
        mIndex = 0;
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
        initView();
    }

    private void initView() {
        int childCount = llMain.getChildCount();
        if (childCount == INDICATOR.length()) {
            // llMain.invalidate();
            return;
        }

        int height = llMain.getHeight();
        int textHeight = (int) Math.floor(height / (INDICATOR.length() + 6));

        LinearLayout.LayoutParams llpText = new LinearLayout.LayoutParams(LinearLayout.LayoutParams.WRAP_CONTENT,
                LinearLayout.LayoutParams.WRAP_CONTENT);
        for (int i = 0; i < INDICATOR.length(); i++) {
            String str = String.valueOf(INDICATOR.charAt(i));
            TextView tvIndicator = new TextView(getContext());
            tvIndicator.setText(str);
            tvIndicator.setIncludeFontPadding(false);
            tvIndicator.setTextSize(TypedValue.COMPLEX_UNIT_PX, textHeight);
            tvIndicator.setTextColor(getResources().getColor(R.color.letter_indicator_text_normal));
            // tvIndicator.setPadding(0, -4, 0, -4);
            llMain.addView(tvIndicator, llpText);
        }
    }

    @SuppressLint("DefaultLocale")
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        int action = ev.getAction();
        switch (action) {
            case MotionEvent.ACTION_DOWN:
            case MotionEvent.ACTION_MOVE:
                scrollable = false;
                float y = ev.getY();
                float childY = 0;
                int index = 0;
                for (int i = 0; i < INDICATOR.length(); i++) {
                    TextView view = (TextView) llMain.getChildAt(i);
                    childY = view.getTop();
                    int height = view.getHeight();
                    if (childY < y && childY + height > y) {
                        index = i;
                        break;
                    }
                    view = null; // not neccessary
                }

                TextView view = (TextView) llMain.getChildAt(INDICATOR.length() - 1);
                if (y > view.getTop()) {
                    index = INDICATOR.length() - 1;
                }
                view = null;

                changeIndicatorColor(index);

                char indexIndicator = INDICATOR.charAt(index);// A:65, #:23
                if (indexIndicator < 'A') {
                    mListView.setSelection(0);
                } else {
                    for (int i = 0; i < mData.size(); i++) {
                        if (mData.get(i).charAt(0) >= indexIndicator) {
                            mListView.setSelection(i);
                            return true;
                        }
                    }
                }

                break;
            case MotionEvent.ACTION_UP:
                postDelayed(new Runnable() {

                    @Override
                    public void run() {
                        scrollable = true;
                    }
                }, 100);
                showText("", false);
                break;

            default:
                break;
        }
        return true;
    }

    @Override
    public void onScrollStateChanged(AbsListView view, int scrollState) {
        // invalidate();
    }

    @SuppressLint("DefaultLocale")
    @Override
    public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
        try {
            if (!scrollable || null == mData || mData.size() == 0) {
                return;
            }
            if (null != mData) {
                String str = mData.get(firstVisibleItem);
                char indicator = str.charAt(0);
                if (indicator < 'A') {
                    changeIndicatorColor(0);
                    return;
                }
                for (int i = 1; i < INDICATOR.length(); i++) {
                    if (INDICATOR.charAt(i) == indicator) {
                        changeIndicatorColor(i);
                        return;
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private void changeIndicatorColor(int index) {
        if (mIndex != 0 && mIndex == index) {
            return;
        }

        TextView tv = (TextView) llMain.getChildAt(mIndex);
        tv.setTextColor(getResources().getColor(R.color.letter_indicator_text_normal));

        tv = (TextView) llMain.getChildAt(index);
        tv.setTextColor(getResources().getColor(R.color.letter_indicator_text_select));

        mIndex = index;

        showText(String.valueOf(INDICATOR.charAt(mIndex)), true);
    }

    private void showText(String text, boolean isShow) {
        if (null != tvAlert) {
            tvAlert.setText(text);
            tvAlert.setVisibility(isShow ? View.VISIBLE : View.INVISIBLE);
        }
    }

    // private Drawable initBackground() {
    // float[] roundRect = new float[] { 12, 12, 12, 12, 12, 12, 12, 12 };
    // RoundRectShape reoundRechShape = new RoundRectShape(roundRect, null,
    // null);
    // ShapeDrawable drawable = new ShapeDrawable(reoundRechShape);
    // drawable.getPaint().setColor(Color.parseColor(GRAY_AA));
    // // drawable.getPaint().setStyle(Paint.Style.FILL);
    // return drawable;
    // }
}
```
  
测试过程中发现，有些item明明是汉字开头的（这个时候你可能还需要个汉字转拼音的工具包），却出现在“#”里，仔细检查，你会发现，那个并不是空格，细节请看代码注释。 由于很简单，不做详细介绍，仅仅做个记录，有问题可以留言一起讨论。


## 0x04 Demo

最后附个测试demo： [下载](http://download.csdn.net/detail/ttdevs/7846559)

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)

