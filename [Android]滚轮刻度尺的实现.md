
---
title: 「Android」滚轮刻度尺的实现
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


## 0x00 滚轮刻度尺（卷尺）

遇到下面这样一个需求：

![滚轮刻度尺](http://img.blog.csdn.net/20140831150159887?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  


## 0x01 问题分析

卷尺，通过左右滑动来选择不同的刻度值。这方面的东西以前没弄过，以目前你的能力，想了几种思路都死在了半路上。比如上面的刻度线如何弄，滑动的时候又该如何弄；下面的数字又如何弄；看起来像圆圈的效果该如何弄。时间紧迫，就俩晚上的时间。没有好的思路就参考别人的先吧，说来也巧，两天前刚看过一个日期选择控件，还有以前看的一个[仿iPhone滚动控件](http://blog.csdn.net/lilu_leo/article/details/7397697) ，效果类似：

![](http://img.blog.csdn.net/20140831151514968?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

![](http://img.blog.csdn.net/20140831151748056?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  

本想找作者[傲慢的上校](http://blog.csdn.net/lilu_leo)交流下，但是时间比较紧，源码都给了也不是很好意思。大致的浏览了下，可能涉及下面几个东西：

- 背景：这个用shape实现。之前有研究过，也用过，但是还没实现过要求的效果
- 刻度和数字：这个就不要乱想了，直接draw。相对来说还是比较简单的，就是画直线和字
- 滚动：滚动的时候不停的重绘实现一个滚动的效果。弄过，但是不确定实现的是啥样的效果
- 快速滚动：Scroller和VelocityTracker可能是需要用到的东西。几乎完全没弄过，骚年，学习吧（需求的要求中，这个优先级可以最低）
- 需求：刻度的单位是可以变的，比如十格一个单位，或者两格一个单位，在或者可以是任意的（这个前期思路没想好，实现起来就困难了，最后只弄了两种）

其实，到了这一步基本上就已经可以实现了，看个最终效果先：

![](http://img.blog.csdn.net/20140831153554458?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdHRkZXZz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)  


## 0x02 实现

下面一步一步来。

在这之前还有个地方要说的，就是控件的接口：对外提供一个方法实现控件初始化和接收控件选择的值：显示的单位，最大值，最小值，当前值，回调接口。有了这些，先从最难的入手。

- 实现刻度和数字，并可以滑动

这个地方很关键，每个人有每个人的思路，而且思路的好坏直接影响到后面对不同单位的实现。目前的思路是根据当前显示的数值mValue，从控件中间向两边画刻度线，滑动的时候同时改变显示的值mValue，不足最小刻度的四舍五入：

``` java
@Override
protected void onDraw(Canvas canvas) {
   super.onDraw(canvas);

   drawScaleLine(canvas);
   // drawWheel(canvas);
   drawMiddleLine(canvas);
}

private void drawWheel(Canvas canvas) {
   Drawable wheel = getResources().getDrawable(R.drawable.bg_wheel);
   wheel.setBounds(0, 0, getWidth(), getHeight());
   wheel.draw(canvas);
}

/**
* 从中间往两边开始画刻度线
*
* @param canvas
*/
private void drawScaleLine(Canvas canvas) {
   canvas.save();

   Paint linePaint = new Paint();
   linePaint.setStrokeWidth(2);
   linePaint.setColor(Color.BLACK);

   TextPaint textPaint = new TextPaint(Paint.ANTI_ALIAS_FLAG);
   textPaint.setTextSize(TEXT_SIZE * mDensity);

   int width = mWidth, drawCount = 0;
   float xPosition = 0, textWidth = Layout.getDesiredWidth("0", textPaint);

   for (int i = 0; drawCount <= 4 * width; i++) {
       int numSize = String.valueOf(mValue + i).length();

       xPosition = (width / 2 - mMove) + i * mLineDivider * mDensity;
       if (xPosition + getPaddingRight() < mWidth) {
           if ((mValue + i) % mModType == 0) {
               canvas.drawLine(xPosition, getPaddingTop(), xPosition, mDensity * ITEM_MAX_HEIGHT, linePaint);

               if (mValue + i <= mMaxValue) {
                   switch (mModType) {
                       case MOD_TYPE_HALF:
                           canvas.drawText(String.valueOf((mValue + i) / 2), countLeftStart(mValue + i, xPosition, textWidth), getHeight() - textWidth, textPaint);
                           break;
                       case MOD_TYPE_ONE:
                           canvas.drawText(String.valueOf(mValue + i), xPosition - (textWidth * numSize / 2), getHeight() - textWidth, textPaint);
                           break;

                       default:
                           break;
                   }
               }
           } else {
               canvas.drawLine(xPosition, getPaddingTop(), xPosition, mDensity * ITEM_MIN_HEIGHT, linePaint);
           }
       }

       xPosition = (width / 2 - mMove) - i * mLineDivider * mDensity;
       if (xPosition > getPaddingLeft()) {
           if ((mValue - i) % mModType == 0) {
               canvas.drawLine(xPosition, getPaddingTop(), xPosition, mDensity * ITEM_MAX_HEIGHT, linePaint);

               if (mValue - i >= 0) {
                   switch (mModType) {
                       case MOD_TYPE_HALF:
                           canvas.drawText(String.valueOf((mValue - i) / 2), countLeftStart(mValue - i, xPosition, textWidth), getHeight() - textWidth, textPaint);
                           break;
                       case MOD_TYPE_ONE:
                           canvas.drawText(String.valueOf(mValue - i), xPosition - (textWidth * numSize / 2), getHeight() - textWidth, textPaint);
                           break;

                       default:
                           break;
                   }
               }
           } else {
               canvas.drawLine(xPosition, getPaddingTop(), xPosition, mDensity * ITEM_MIN_HEIGHT, linePaint);
           }
       }

       drawCount += 2 * mLineDivider * mDensity;
   }

   canvas.restore();
}
```

- 滑动的加速问题

这里用到两个类Scroller和VelocityTracker，关于这两个类之后有机会会详细介绍，这里简单提下：VelocityTracker的作用是在用户加速滑动时计算该滑动多远，拿到这个之后通过Scroller来执行滑动过程的计算，最后是真实的“移动”——根据mValue的值进行重绘：

``` java
@Override
public boolean onTouchEvent(MotionEvent event) {
   int action = event.getAction();
   int xPosition = (int) event.getX();

   if (mVelocityTracker == null) {
       mVelocityTracker = VelocityTracker.obtain();
   }
   mVelocityTracker.addMovement(event);

   switch (action) {
       case MotionEvent.ACTION_DOWN:

           mScroller.forceFinished(true);

           mLastX = xPosition;
           mMove = 0;
           break;
       case MotionEvent.ACTION_MOVE:
           mMove += (mLastX - xPosition);
           changeMoveAndValue();
           break;
       case MotionEvent.ACTION_UP:
       case MotionEvent.ACTION_CANCEL:
           countMoveEnd();
           countVelocityTracker(event);
           return false;
       // break;
       default:
           break;
   }

   mLastX = xPosition;
   return true;
}

private void countVelocityTracker(MotionEvent event) {
   mVelocityTracker.computeCurrentVelocity(1000);
   float xVelocity = mVelocityTracker.getXVelocity();
   if (Math.abs(xVelocity) > mMinVelocity) {
       mScroller.fling(0, 0, (int) xVelocity, 0, Integer.MIN_VALUE, Integer.MAX_VALUE, 0, 0);
   }
}

private void changeMoveAndValue() {
   int tValue = (int) (mMove / (mLineDivider * mDensity));
   if (Math.abs(tValue) > 0) {
       mValue += tValue;
       mMove -= tValue * mLineDivider * mDensity;
       if (mValue <= 0 || mValue > mMaxValue) {
           mValue = mValue <= 0 ? 0 : mMaxValue;
           mMove = 0;
           mScroller.forceFinished(true);
       }
       notifyValueChange();
   }
   postInvalidate();
}

private void countMoveEnd() {
   int roundMove = Math.round(mMove / (mLineDivider * mDensity));
   mValue = mValue + roundMove;
   mValue = mValue <= 0 ? 0 : mValue;
   mValue = mValue > mMaxValue ? mMaxValue : mValue;

   mLastX = 0;
   mMove = 0;

   notifyValueChange();
   postInvalidate();
}

private void notifyValueChange() {
   if (null != mListener) {
       if (mModType == MOD_TYPE_ONE) {
           mListener.onValueChange(mValue);
       }
       if (mModType == MOD_TYPE_HALF) {
           mListener.onValueChange(mValue / 2f);
       }
   }
}

@Override
public void computeScroll() {
   super.computeScroll();
   if (mScroller.computeScrollOffset()) {
       if (mScroller.getCurrX() == mScroller.getFinalX()) { // over
           countMoveEnd();
       } else {
           int xPosition = mScroller.getCurrX();
           mMove += (mLastX - xPosition);
           changeMoveAndValue();
           mLastX = xPosition;
       }
   }
}
```

- 圆圈背景的实现

这个用shape来做，可以使用setBackgroundDrawable()来做，也可以在draw中进行直接绘制，效果相同。还有一些细节问题，比如滑动时刻度线超过边界，滑动距离大时候显示不完整等问题，这个只有做了才会发现。下面是shape背景的代码：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
   android:shape="rectangle" >
    
   <!-- two set color way -->
   <gradient
       android:angle="0"
       android:centerColor="#66FFFFFF"
       android:endColor="#66AAAAAA"
       android:startColor="#66AAAAAA" />
    
   <corners android:radius="6dp" />
    
   <stroke
       android:width="6dp"
       android:color="#FF666666" />
    
</shape>
```

用代码可以这样写：

``` java
private GradientDrawable createBackground() {
   float strokeWidth = 4 * mDensity; // 边框宽度
   float roundRadius = 6 * mDensity; // 圆角半径
   int strokeColor = Color.parseColor("#FF666666");// 边框颜色
   // int fillColor = Color.parseColor("#DFDFE0");// 内部填充颜色

   setPadding((int) strokeWidth, (int) strokeWidth, (int) strokeWidth, 0);

   int colors[] = {0xFF999999, 0xFFFFFFFF, 0xFF999999};// 分别为开始颜色，中间夜色，结束颜色
   GradientDrawable bgDrawable = new GradientDrawable(GradientDrawable.Orientation.LEFT_RIGHT, colors);// 创建drawable
   // bgDrawable.setColor(fillColor);
   bgDrawable.setCornerRadius(roundRadius);
   bgDrawable.setStroke((int) strokeWidth, strokeColor);
   // setBackgroundDrawable(gd);
   return bgDrawable;
}
```


## 0x03 完整的代码
    
``` java
package com.ttdevs.wheel.widget;

/**
 * 卷尺控件类。由于时间比较紧，只有下班后有时间，因此只实现了基本功能。<br>
 * 细节问题包括滑动过程中widget边缘的刻度显示问题等<br>
 * <p>
 * 周末有时间会继续更新<br>
 *
 * @author ttdevs
 * @version create：2014年8月26日
 */
@SuppressLint("ClickableViewAccessibility")
public class TuneWheel extends View {

    public interface OnValueChangeListener {
        public void onValueChange(float value);
    }

    public static final int MOD_TYPE_HALF = 2;
    public static final int MOD_TYPE_ONE = 10;

    private static final int ITEM_HALF_DIVIDER = 40;
    private static final int ITEM_ONE_DIVIDER = 10;

    private static final int ITEM_MAX_HEIGHT = 50;
    private static final int ITEM_MIN_HEIGHT = 20;

    private static final int TEXT_SIZE = 18;

    private float mDensity;
    private int mValue = 50, mMaxValue = 100, mModType = MOD_TYPE_HALF, mLineDivider = ITEM_HALF_DIVIDER;
    // private int mValue = 50, mMaxValue = 500, mModType = MOD_TYPE_ONE,
    // mLineDivider = ITEM_ONE_DIVIDER;

    private int mLastX, mMove;
    private int mWidth, mHeight;

    private int mMinVelocity;
    private Scroller mScroller;
    private VelocityTracker mVelocityTracker;

    private OnValueChangeListener mListener;

    @SuppressWarnings("deprecation")
    public TuneWheel(Context context, AttributeSet attrs) {
        super(context, attrs);

        mScroller = new Scroller(getContext());
        mDensity = getContext().getResources().getDisplayMetrics().density;

        mMinVelocity = ViewConfiguration.get(getContext()).getScaledMinimumFlingVelocity();

        // setBackgroundResource(R.drawable.bg_wheel);
        setBackgroundDrawable(createBackground());
    }

    private GradientDrawable createBackground() {
        float strokeWidth = 4 * mDensity; // 边框宽度
        float roundRadius = 6 * mDensity; // 圆角半径
        int strokeColor = Color.parseColor("#FF666666");// 边框颜色
        // int fillColor = Color.parseColor("#DFDFE0");// 内部填充颜色

        setPadding((int) strokeWidth, (int) strokeWidth, (int) strokeWidth, 0);

        int colors[] = {0xFF999999, 0xFFFFFFFF, 0xFF999999};// 分别为开始颜色，中间夜色，结束颜色
        GradientDrawable bgDrawable = new GradientDrawable(GradientDrawable.Orientation.LEFT_RIGHT, colors);// 创建drawable
        // bgDrawable.setColor(fillColor);
        bgDrawable.setCornerRadius(roundRadius);
        bgDrawable.setStroke((int) strokeWidth, strokeColor);
        // setBackgroundDrawable(gd);
        return bgDrawable;
    }

    /**
     * 考虑可扩展，但是时间紧迫，只可以支持两种类型效果图中两种类型
     *
     * @param value    初始值
     * @param maxValue 最大值
     * @param model    刻度盘精度：<br>
     *                 {@link MOD_TYPE_HALF}<br>
     *                 {@link MOD_TYPE_ONE}<br>
     */
    public void initViewParam(int defaultValue, int maxValue, int model) {
        switch (model) {
            case MOD_TYPE_HALF:
                mModType = MOD_TYPE_HALF;
                mLineDivider = ITEM_HALF_DIVIDER;
                mValue = defaultValue * 2;
                mMaxValue = maxValue * 2;
                break;
            case MOD_TYPE_ONE:
                mModType = MOD_TYPE_ONE;
                mLineDivider = ITEM_ONE_DIVIDER;
                mValue = defaultValue;
                mMaxValue = maxValue;
                break;

            default:
                break;
        }
        invalidate();

        mLastX = 0;
        mMove = 0;
        notifyValueChange();
    }

    /**
     * 设置用于接收结果的监听器
     *
     * @param listener
     */
    public void setValueChangeListener(OnValueChangeListener listener) {
        mListener = listener;
    }

    /**
     * 获取当前刻度值
     *
     * @return
     */
    public float getValue() {
        return mValue;
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        mWidth = getWidth();
        mHeight = getHeight();
        super.onLayout(changed, left, top, right, bottom);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        drawScaleLine(canvas);
        // drawWheel(canvas);
        drawMiddleLine(canvas);
    }

    private void drawWheel(Canvas canvas) {
        Drawable wheel = getResources().getDrawable(R.drawable.bg_wheel);
        wheel.setBounds(0, 0, getWidth(), getHeight());
        wheel.draw(canvas);
    }

    /**
     * 从中间往两边开始画刻度线
     *
     * @param canvas
     */
    private void drawScaleLine(Canvas canvas) {
        canvas.save();

        Paint linePaint = new Paint();
        linePaint.setStrokeWidth(2);
        linePaint.setColor(Color.BLACK);

        TextPaint textPaint = new TextPaint(Paint.ANTI_ALIAS_FLAG);
        textPaint.setTextSize(TEXT_SIZE * mDensity);

        int width = mWidth, drawCount = 0;
        float xPosition = 0, textWidth = Layout.getDesiredWidth("0", textPaint);

        for (int i = 0; drawCount <= 4 * width; i++) {
            int numSize = String.valueOf(mValue + i).length();

            xPosition = (width / 2 - mMove) + i * mLineDivider * mDensity;
            if (xPosition + getPaddingRight() < mWidth) {
                if ((mValue + i) % mModType == 0) {
                    canvas.drawLine(xPosition, getPaddingTop(), xPosition, mDensity * ITEM_MAX_HEIGHT, linePaint);

                    if (mValue + i <= mMaxValue) {
                        switch (mModType) {
                            case MOD_TYPE_HALF:
                                canvas.drawText(String.valueOf((mValue + i) / 2), countLeftStart(mValue + i, xPosition, textWidth), getHeight() - textWidth, textPaint);
                                break;
                            case MOD_TYPE_ONE:
                                canvas.drawText(String.valueOf(mValue + i), xPosition - (textWidth * numSize / 2), getHeight() - textWidth, textPaint);
                                break;

                            default:
                                break;
                        }
                    }
                } else {
                    canvas.drawLine(xPosition, getPaddingTop(), xPosition, mDensity * ITEM_MIN_HEIGHT, linePaint);
                }
            }

            xPosition = (width / 2 - mMove) - i * mLineDivider * mDensity;
            if (xPosition > getPaddingLeft()) {
                if ((mValue - i) % mModType == 0) {
                    canvas.drawLine(xPosition, getPaddingTop(), xPosition, mDensity * ITEM_MAX_HEIGHT, linePaint);

                    if (mValue - i >= 0) {
                        switch (mModType) {
                            case MOD_TYPE_HALF:
                                canvas.drawText(String.valueOf((mValue - i) / 2), countLeftStart(mValue - i, xPosition, textWidth), getHeight() - textWidth, textPaint);
                                break;
                            case MOD_TYPE_ONE:
                                canvas.drawText(String.valueOf(mValue - i), xPosition - (textWidth * numSize / 2), getHeight() - textWidth, textPaint);
                                break;

                            default:
                                break;
                        }
                    }
                } else {
                    canvas.drawLine(xPosition, getPaddingTop(), xPosition, mDensity * ITEM_MIN_HEIGHT, linePaint);
                }
            }

            drawCount += 2 * mLineDivider * mDensity;
        }

        canvas.restore();
    }

    /**
     * 计算没有数字显示位置的辅助方法
     *
     * @param value
     * @param xPosition
     * @param textWidth
     * @return
     */
    private float countLeftStart(int value, float xPosition, float textWidth) {
        float xp = 0f;
        if (value < 20) {
            xp = xPosition - (textWidth * 1 / 2);
        } else {
            xp = xPosition - (textWidth * 2 / 2);
        }
        return xp;
    }

    /**
     * 画中间的红色指示线、阴影等。指示线两端简单的用了两个矩形代替
     *
     * @param canvas
     */
    private void drawMiddleLine(Canvas canvas) {
        // TOOD 常量太多，暂时放这，最终会放在类的开始，放远了怕很快忘记
        int gap = 12, indexWidth = 8, indexTitleWidth = 24, indexTitleHight = 10, shadow = 6;
        String color = "#66999999";

        canvas.save();

        Paint redPaint = new Paint();
        redPaint.setStrokeWidth(indexWidth);
        redPaint.setColor(Color.RED);
        canvas.drawLine(mWidth / 2, 0, mWidth / 2, mHeight, redPaint);

        Paint ovalPaint = new Paint();
        ovalPaint.setColor(Color.RED);
        ovalPaint.setStrokeWidth(indexTitleWidth);
        canvas.drawLine(mWidth / 2, 0, mWidth / 2, indexTitleHight, ovalPaint);
        canvas.drawLine(mWidth / 2, mHeight - indexTitleHight, mWidth / 2, mHeight, ovalPaint);

        // RectF ovalRectF = new RectF(mWidth / 2 - 10, 0, mWidth / 2 + 10, 4 *
        // mDensity); //TODO 椭圆
        // canvas.drawOval(ovalRectF, ovalPaint);
        // ovalRectF.set(mWidth / 2 - 10, mHeight - 8 * mDensity, mWidth / 2 +
        // 10, mHeight); //TODO

        Paint shadowPaint = new Paint();
        shadowPaint.setStrokeWidth(shadow);
        shadowPaint.setColor(Color.parseColor(color));
        canvas.drawLine(mWidth / 2 + gap, 0, mWidth / 2 + gap, mHeight, shadowPaint);

        canvas.restore();
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int action = event.getAction();
        int xPosition = (int) event.getX();

        if (mVelocityTracker == null) {
            mVelocityTracker = VelocityTracker.obtain();
        }
        mVelocityTracker.addMovement(event);

        switch (action) {
            case MotionEvent.ACTION_DOWN:

                mScroller.forceFinished(true);

                mLastX = xPosition;
                mMove = 0;
                break;
            case MotionEvent.ACTION_MOVE:
                mMove += (mLastX - xPosition);
                changeMoveAndValue();
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                countMoveEnd();
                countVelocityTracker(event);
                return false;
            // break;
            default:
                break;
        }

        mLastX = xPosition;
        return true;
    }

    private void countVelocityTracker(MotionEvent event) {
        mVelocityTracker.computeCurrentVelocity(1000);
        float xVelocity = mVelocityTracker.getXVelocity();
        if (Math.abs(xVelocity) > mMinVelocity) {
            mScroller.fling(0, 0, (int) xVelocity, 0, Integer.MIN_VALUE, Integer.MAX_VALUE, 0, 0);
        }
    }

    private void changeMoveAndValue() {
        int tValue = (int) (mMove / (mLineDivider * mDensity));
        if (Math.abs(tValue) > 0) {
            mValue += tValue;
            mMove -= tValue * mLineDivider * mDensity;
            if (mValue <= 0 || mValue > mMaxValue) {
                mValue = mValue <= 0 ? 0 : mMaxValue;
                mMove = 0;
                mScroller.forceFinished(true);
            }
            notifyValueChange();
        }
        postInvalidate();
    }

    private void countMoveEnd() {
        int roundMove = Math.round(mMove / (mLineDivider * mDensity));
        mValue = mValue + roundMove;
        mValue = mValue <= 0 ? 0 : mValue;
        mValue = mValue > mMaxValue ? mMaxValue : mValue;

        mLastX = 0;
        mMove = 0;

        notifyValueChange();
        postInvalidate();
    }

    private void notifyValueChange() {
        if (null != mListener) {
            if (mModType == MOD_TYPE_ONE) {
                mListener.onValueChange(mValue);
            }
            if (mModType == MOD_TYPE_HALF) {
                mListener.onValueChange(mValue / 2f);
            }
        }
    }

    @Override
    public void computeScroll() {
        super.computeScroll();
        if (mScroller.computeScrollOffset()) {
            if (mScroller.getCurrX() == mScroller.getFinalX()) { // over
                countMoveEnd();
            } else {
                int xPosition = mScroller.getCurrX();
                mMove += (mLastX - xPosition);
                changeMoveAndValue();
                mLastX = xPosition;
            }
        }
    }
}
```
  

