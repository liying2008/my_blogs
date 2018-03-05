---
title: 在Android中全屏显示GIF图片
date: 2017-08-20
tags: [Android, GIF]
categories: Android基础
---
# 1、自定义一个GifView

首先自定义一个GifView，用于显示Gif图片。GifView的代码参考自[https://github.com/Cutta/GifView](https://github.com/Cutta/GifView)。

```java
package cc.duduhuo.gifviewdemo.view;

import android.annotation.SuppressLint;
import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Movie;
import android.os.Build;
import android.os.SystemClock;
import android.support.annotation.NonNull;
import android.util.AttributeSet;
import android.view.View;

/**
 * =======================================================
 * 作者：liying - liruoer2008@yeah.net
 * 日期：2017/8/19 20:09
 * 版本：1.0
 * 描述：自定义显示Gif图片的View
 * 备注：
 * =======================================================
 */
public class GifView extends View {
    private static final int DEFAULT_MOVIE_VIEW_DURATION = 1000;

    private int mMovieResourceId;
    private Movie mMovie;

    private long mMovieStart;
    private int mCurrentAnimationTime;

    /**
     * Position for drawing animation frames in the center of the view.
     */
    private float mLeft;
    private float mTop;

    /**
     * Scaling factor to fit the animation within view bounds.
     */
    private float mScale;

    /**
     * Scaled movie frames width and height.
     */
    private int mMeasuredMovieWidth;
    private int mMeasuredMovieHeight;

    private volatile boolean mPaused;
    private boolean mVisible = true;

    public GifView(Context context) {
        this(context, null);
    }

    public GifView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public GifView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        init();
    }

    private void init() {
        /*
         * Starting from HONEYCOMB(Api Level:11) have to turn off HW acceleration to draw
         * Movie on Canvas.
         */
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
            setLayerType(View.LAYER_TYPE_SOFTWARE, null);
        }
    }

    public void setGifResource(int movieResourceId) {
        this.mMovieResourceId = movieResourceId;
        mMovie = Movie.decodeStream(getResources().openRawResource(mMovieResourceId));
        requestLayout();
    }

    public int getGifResource() {
        return this.mMovieResourceId;
    }


    public void play() {
        if (this.mPaused) {
            this.mPaused = false;
            /*
             * Calculate new movie start time, so that it resumes from the same
             * frame.
             */
            mMovieStart = SystemClock.uptimeMillis() - mCurrentAnimationTime;
            invalidate();
        }
    }

    public void pause() {
        if (!this.mPaused) {
            this.mPaused = true;

            invalidate();
        }

    }


    public boolean isPaused() {
        return this.mPaused;
    }

    public boolean isPlaying() {
        return !this.mPaused;
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        if (mMovie != null) {
            int movieWidth = mMovie.width();
            int movieHeight = mMovie.height();

			/*
             * Calculate horizontal scaling
			 */
            float scaleH = 1f;
            int measureModeWidth = MeasureSpec.getMode(widthMeasureSpec);

            if (measureModeWidth != MeasureSpec.UNSPECIFIED) {
                int maximumWidth = MeasureSpec.getSize(widthMeasureSpec);
                if (movieWidth > maximumWidth) {
                    scaleH = (float) movieWidth / (float) maximumWidth;
                }
            }

			/*
             * calculate vertical scaling
			 */
            float scaleW = 1f;
            int measureModeHeight = MeasureSpec.getMode(heightMeasureSpec);

            if (measureModeHeight != MeasureSpec.UNSPECIFIED) {
                int maximumHeight = MeasureSpec.getSize(heightMeasureSpec);
                if (movieHeight > maximumHeight) {
                    scaleW = (float) movieHeight / (float) maximumHeight;
                }
            }

			/*
             * calculate overall scale
			 */
            mScale = 1f / Math.max(scaleH, scaleW);

            mMeasuredMovieWidth = (int) (movieWidth * mScale);
            mMeasuredMovieHeight = (int) (movieHeight * mScale);

            setMeasuredDimension(mMeasuredMovieWidth, mMeasuredMovieHeight);

        } else {
            /*
             * No movie set, just set minimum available size.
			 */
            setMeasuredDimension(getSuggestedMinimumWidth(), getSuggestedMinimumHeight());
        }
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        super.onLayout(changed, l, t, r, b);
        /* Calculate mLeft / mTop for drawing in center */
        mLeft = (getWidth() - mMeasuredMovieWidth) / 2f;
        mTop = (getHeight() - mMeasuredMovieHeight) / 2f;

        mVisible = getVisibility() == View.VISIBLE;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        if (mMovie != null) {
            if (!mPaused) {
                updateAnimationTime();
                drawMovieFrame(canvas);
                invalidateView();
            } else {
                drawMovieFrame(canvas);
            }
        }
    }

    /**
     * Invalidates view only if it is mVisible.
     * <br>
     * {@link #postInvalidateOnAnimation()} is used for Jelly Bean and higher.
     */
    @SuppressLint("NewApi")
    private void invalidateView() {
        if (mVisible) {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
                postInvalidateOnAnimation();
            } else {
                invalidate();
            }
        }
    }

    /**
     * Calculate current animation time
     */
    private void updateAnimationTime() {
        long now = android.os.SystemClock.uptimeMillis();

        if (mMovieStart == 0) {
            mMovieStart = now;
        }

        int dur = mMovie.duration();

        if (dur == 0) {
            dur = DEFAULT_MOVIE_VIEW_DURATION;
        }

        mCurrentAnimationTime = (int) ((now - mMovieStart) % dur);
    }

    /**
     * Draw current GIF frame
     */
    private void drawMovieFrame(Canvas canvas) {
        mMovie.setTime(mCurrentAnimationTime);

        canvas.save();
        canvas.scale(mScale, mScale);
        mMovie.draw(canvas, mLeft / mScale, mTop / mScale);
        canvas.restore();
    }

    @SuppressLint("NewApi")
    @Override
    public void onScreenStateChanged(int screenState) {
        super.onScreenStateChanged(screenState);
        mVisible = screenState == SCREEN_STATE_ON;
        invalidateView();
    }

    /**
     * get gif width
     *
     * @return width
     */
    public int getGifWidth() {
        return mMovie.width();
    }

    /**
     * get gif height
     *
     * @return height
     */
    public int getGifHeight() {
        return mMovie.height();
    }

    @SuppressLint("NewApi")
    @Override
    protected void onVisibilityChanged(@NonNull View changedView, int visibility) {
        super.onVisibilityChanged(changedView, visibility);
        mVisible = visibility == View.VISIBLE;
        invalidateView();
    }

    @Override
    protected void onWindowVisibilityChanged(int visibility) {
        super.onWindowVisibilityChanged(visibility);
        mVisible = visibility == View.VISIBLE;
        invalidateView();
    }
}
```

重要的是要添加<code>getGifWidth()</code>和<code>getGifHeight()</code>两个方法，用于获取Gif图片的宽高。
（注：获取Gif图片的宽和高可以简单等价于获取对应Movie对象的宽和高。）

# 2、在布局文件中使用GifView

自定义的View可以在布局文件中直接使用。

```xml
<cc.duduhuo.gifviewdemo.view.GifView
    android:id="@+id/gifView"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center"/>
```

# 3、缩放Gif图片，使之全屏显示

首先需要调用<code>GifView</code>的<code>setGifResource(int)</code>方法设置要显示的Gif图片。

```java
GifView gifView = (GifView) findViewById(R.id.gifView);
gifView.setGifResource(R.drawable.gif1);    // 设置显示的Gif图片
```

其次，调用<code>GifView</code>的<code>getGifWidth()</code>方法和<code>getGifHeight()</code>方法得到Gif图片的宽和高。再获取到当前屏幕的宽和高，分别计算屏幕和Gif图片的宽度和高度比例。为了使图片的宽高比例保持不变，选择缩放比例较小的一边进行缩放。

```java
int width = gifView.getGifWidth();
int height = gifView.getGifHeight();
int screenWidth = getScreenSize().width();
int screenHeight = getScreenSize().height();

if (width > 0 && height > 0) {
	float wScale = (float) screenWidth / width;
	float hScale = (float) screenHeight / height;
	if (wScale < 1 || hScale < 1) {
		// 如果图片的宽或高大于屏幕的宽或高，则图片会自动缩小至全屏
	} else if (wScale <= hScale) {
		// 宽度全屏
		gifView.setScaleX(wScale);
		gifView.setScaleY(wScale);
	} else {
		// 高度全屏
		gifView.setScaleX(hScale);
		gifView.setScaleY(hScale);
	}
}
```

<code>getScreenSize()</code>方法用于获取屏幕的宽度和高度。

```java
/**
 * 得到屏幕的尺寸
 *
 * @return 包含屏幕尺寸的Rect对象
 */
private Rect getScreenSize() {
	Rect localRect = new Rect();
	getWindow().getDecorView().getWindowVisibleDisplayFrame(localRect);
	return localRect;
}
```

# 4、效果演示

|宽度全屏|高度全屏|
|:---:|:---:|
|![宽度全屏](http://img.blog.csdn.net/20170820001655683)|![高度全屏](http://img.blog.csdn.net/20170820001722379)|

# 5、代码下载

Demo代码下载：[http://download.csdn.net/download/u012939909/9941754](http://download.csdn.net/download/u012939909/9941754)