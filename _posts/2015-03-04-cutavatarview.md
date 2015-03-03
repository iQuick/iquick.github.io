---
layout:		post
title:		仿QQ头像截取 CutAvatarView
category:	[Android]
tags:		[Android]
published:	true
---
# 仿QQ头像截取 CutAvatarView
---

前段时间仿QQ写的截取头像的自定View，仿QQ头像截取 CutAvatarView 继承至 ImageView 

![CutAvatarView]({{BASE_PATH}}/images/cut-avatar.jpg)

<!--break-->

## 使用

xml 布局文件

<pre class="prettyprint linenums">

&lt; ?xml version="1.0" encoding="utf-8"? >
&lt;RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    &lt;me.imli.qqcutavatar.view.CutAvatarView
        android:id="@+id/cut_avatar_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    &lt;Button
        android:id="@+id/btn_cut"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentRight="true"
        android:text="截取" />

&lt;/RelativeLayout>

</pre>



CutAvatarActivity 代码

<pre class="prettyprint linenums">
public class CutAvatarActivity extends Activity {
	
	public static Bitmap bitmap;
	private CutAvatarView mCutAvatarView;
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_cut_avatar);
		
		mCutAvatarView = (CutAvatarView) findViewById(R.id.cut_avatar_view);
		// 这里一定要设置一张图片进去，不然会报错
		mCutAvatarView.setImageResource(R.drawable.avatar);
		
		findViewById(R.id.btn_cut).setOnClickListener(doCut());
	}
	
	
	private View.OnClickListener doCut() {
		return new View.OnClickListener() {
			
			@Override
			public void onClick(View v) {
				if (bitmap != null && bitmap.isRecycled()) {
					bitmap.recycle();
				}
				bitmap = mCutAvatarView.clip(true);
				setResult(RESULT_OK);
				finish();
			}
		};
	}
	
}
</pre>



## 核心代码
<pre class="prettyprint linenums">

@Override
protected void onDraw(Canvas canvas) {
	super.onDraw(canvas);

	// 在截取头像中
	if (isTouCutBitmap) return;
	if (mRectF == null || mRectF.isEmpty()) mRectF = new RectF(0, 0, getWidth(), getHeight());
	int sc = canvas.saveLayer(mRectF, null, Canvas.MATRIX_SAVE_FLAG
			| Canvas.CLIP_SAVE_FLAG | Canvas.HAS_ALPHA_LAYER_SAVE_FLAG
			| Canvas.FULL_COLOR_LAYER_SAVE_FLAG
			| Canvas.CLIP_TO_LAYER_SAVE_FLAG | Canvas.ALL_SAVE_FLAG);
	// 绘制蒙版
	canvas.drawRect(mRectF, mRectPaint);
	canvas.drawCircle(getWidth() / 2, getHeight() / 2, mCircleRadius, mCirclePaint);
	canvas.restoreToCount(sc);
	// 绘制圆环
	canvas.drawCircle(getWidth() / 2, getHeight() / 2, mCircleRadius, mRoundPaint);
}

/** 存储float类型的x，y值，就是你点下的坐标的X和Y */
private PointF mPre = new PointF(), mMid = new PointF();
private float dist = 1f;
@SuppressLint("ClickableViewAccessibility")
@Override
public boolean onTouchEvent(MotionEvent event) {
	switch (event.getAction() & MotionEvent.ACTION_MASK) {
	case MotionEvent.ACTION_DOWN:
		mCurMode = DRAG;
		mSaveMatrix.set(mMatrix);
		mPre.set(event.getX(), event.getY());
		break;
	case MotionEvent.ACTION_POINTER_DOWN:
		mCurMode = ZOOM;
		mSaveMatrix.set(mMatrix);
		dist = spacing(event);
		midPoint(mMid, event);
		break;
	case MotionEvent.ACTION_MOVE:
		if (mCurMode == DRAG) {
			mMatrix.set(mSaveMatrix);
			mMatrix.postTranslate(event.getX() - mPre.x, event.getY() - mPre.y);
		} else if (mCurMode == ZOOM) {
			float scale = spacing(event) / dist;;
			mMatrix.set(mSaveMatrix);
			mMatrix.getValues(mMXValues);
			if (mMXValues[0] * scale > MAX_SCALE) {
				scale = MAX_SCALE / mMXValues[0];
			} else if (mMXValues[0] * scale < MIN_SCALE) {
				scale = MIN_SCALE / mMXValues[0];
			}
			mMatrix.postScale(scale, scale, mMid.x, mMid.y);
		} else if (mCurMode == NONE) {
		}
		checkBoundary(mMatrix);
		break;
	case MotionEvent.ACTION_POINTER_UP:
		mCurMode = NONE;
		break;
	case MotionEvent.ACTION_UP:
		mCurMode = NONE;
		break;
	default:
		break;
	}
	setImageMatrix(mMatrix);
	return true;
}

float[] mMXValues = new float[9];
/**
 * 检查是否越界
 * @param matrix
 */
protected void checkBoundary(Matrix matrix) {
	if (matrix == null) return;
	matrix.getValues(mMXValues);
	
	// 计算缩放是否小于要截取的大小
	float size = mImgBitmap.getWidth() < mImgBitmap.getHeight() ? mImgBitmap.getWidth() : mImgBitmap.getHeight();
    if (mMXValues[0] < mCircleRadius * 2 / size || mMXValues[4] < mCircleRadius * 2 / size) {
    	mMXValues[0] = mCircleRadius * 2 / size;
    	mMXValues[4] = mCircleRadius * 2 / size;
    }
    // 计算位移是否越界
    if (mMXValues[2] > getWidth() / 2 - mCircleRadius) {
    	mMXValues[2] = getWidth() / 2 - mCircleRadius;
    }
    if (mMXValues[5] > getHeight() / 2 - mCircleRadius) {
    	mMXValues[5] = getHeight() / 2 - mCircleRadius;
    }
    if (mMXValues[2] < - mImgBitmap.getWidth() * mMXValues[0] + getWidth() / 2 + mCircleRadius) {
    	mMXValues[2]= - mImgBitmap.getWidth() * mMXValues[0] + getWidth() / 2 + mCircleRadius;
    }
    if (mMXValues[5] < - mImgBitmap.getHeight() * mMXValues[4] + getHeight() / 2 + mCircleRadius) {
    	mMXValues[5]= - mImgBitmap.getHeight() * mMXValues[4] + getHeight() / 2 + mCircleRadius;
    }
    matrix.setValues(mMXValues);
}
</pre>



## 截取头像部分代码
<pre class="prettyprint linenums">
	
/**
 * 截取头像
 * @return
 */
public Bitmap clip() {
	return clip(false);
}

/**
 * 截取头像
 * @param isCircle 是滞是圆形
 * @return
 */
public Bitmap clip(boolean isCircle) {
	Paint paint = new Paint();
	paint.setAntiAlias(true);
	// 为了不带半透明的背景，从新刷新下imageview 好获干净的位图 然后截取
	invalidate();
	// 截图
	isTouCutBitmap = true;
	setDrawingCacheEnabled(true);
	Bitmap bitmap = getDrawingCache().copy(getDrawingCache().getConfig(), false);
	setDrawingCacheEnabled(false);
	Bitmap head = Bitmap.createBitmap((int)mCircleRadius * 2, (int)mCircleRadius * 2, Config.ARGB_8888);
	Canvas canvas = new Canvas(head);
	if (isCircle) {
		canvas.drawRoundRect(new RectF(0, 0, 2 * mCircleRadius, 2 * mCircleRadius), mCircleRadius, mCircleRadius, paint);
		paint.setXfermode(new PorterDuffXfermode(Mode.SRC_IN));
	}
	
	RectF dst = new RectF(-bitmap.getWidth() / 2 + mCircleRadius, -getHeight() / 2 + mCircleRadius, bitmap.getWidth() - bitmap.getWidth() / 2 + mCircleRadius, getHeight() - getHeight() / 2 + mCircleRadius);
	canvas.drawBitmap(bitmap, null, dst, paint);
	isTouCutBitmap = false;
	return head;
}
</pre>



## Github
[https://github.com/iQuick/QQCutAvatar](https://github.com/iQuick/QQCutAvatar)



## 类似参考

* [http://www.eoeandroid.com/thread-559381-1-1.html](http://www.eoeandroid.com/thread-559381-1-1.html) 教程写的不错，但有些BUG，代码写的也有些复杂