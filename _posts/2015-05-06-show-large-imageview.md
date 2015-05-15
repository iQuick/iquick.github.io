---
layout:		post
title:		Android 显示长图的问题
category:	[Android]
tags:		[Android]
published:	true
---
Android 显示长图
---

最近在做一个项目，由于许多地方需要显示图片。大部分都是那种长图片，但是有些机器只支持显示最大图片 2048 * 2048 ，所以在部分机器测试的就出现问题了。由于图片的宽度不大，又不能缩放，就只能截取 Bitmap 中的部分并绘制的方法。

<!--break-->

以下是源码

<pre class="prettyprint linenums">
package cn.uue.yixiaoba.widget;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;

import cn.uue.yixiaoba.util.LogUtil;
import android.annotation.TargetApi;
import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.BitmapRegionDecoder;
import android.graphics.Canvas;
import android.graphics.Matrix;
import android.graphics.PixelFormat;
import android.graphics.Rect;
import android.graphics.drawable.BitmapDrawable;
import android.graphics.drawable.Drawable;
import android.os.Build;
import android.util.AttributeSet;
import android.widget.ImageView;

public class LargeImageView extends ImageView {
	
	private static final int MAX_HEIGHT = 1024;
	private static final int MAX_WIDTH = 1024;
	
	private int mWidth = 0;
	private int mHeight = 0;
	
	private float mScale;
	
	/**
	 * 大图切割
	 */
	private BitmapRegionDecoder mBitmapRegionDecoder;
	
	public LargeImageView(Context context) {
		this(context, null);
	}

	public LargeImageView(Context context, AttributeSet attrs) {
		this(context, attrs, 0);
	}

	public LargeImageView(Context context, AttributeSet attrs, int defStyle) {
		super(context, attrs, defStyle);
		this.init();
	}
	
	private void init() {
	}
	
	@Override
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
		super.onMeasure(widthMeasureSpec, heightMeasureSpec);
		
		int width = getMeasuredWidth();
		int height = getMeasuredHeight();
		if (width > mWidth) {
			mScale = (float)(width) / mWidth;
			mHeight = (int)(mHeight * mScale);
			mWidth = width;
		}
		
		setMeasuredDimension(Math.max(width, mWidth), Math.max(height, mHeight));
	}
	
	@TargetApi(Build.VERSION_CODES.GINGERBREAD_MR1)
	private void canvasLarge(Canvas canvas, BitmapRegionDecoder decoder) {
		if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.GINGERBREAD_MR1) {
			int widthCount = decoder.getWidth() / MAX_WIDTH + (decoder.getWidth() % MAX_WIDTH == 0 ? 0 : 1);
			int heightCount = decoder.getHeight() / MAX_HEIGHT + (decoder.getWidth() % MAX_HEIGHT == 0 ? 0 : 1);
			
			for (int i = 0; i < heightCount; i++) {
				for (int j = 0; j < widthCount; j++) {
					Rect rect = new Rect(j * MAX_WIDTH, i * MAX_HEIGHT, ((j + 1) * MAX_WIDTH < decoder.getWidth()) ? (j + 1) * MAX_WIDTH : decoder.getWidth(), ((i + 1) * MAX_HEIGHT < decoder.getHeight()) ? (i + 1) * MAX_HEIGHT : decoder.getHeight()); 
					Bitmap bitmap = decoder.decodeRegion(rect, null);
					Matrix matrix = new Matrix();
					matrix.postScale(mScale, mScale); // 长和宽放大缩小的比例
					canvas.drawBitmap(Bitmap.createBitmap(bitmap, 0, 0, bitmap.getWidth(), bitmap.getHeight(), matrix, true), rect.left  * mScale, rect.top * mScale, null);
				}
			}
		}
	}

	/**
	 * set image from InputStream
	 * @param stream
	 */
	@TargetApi(Build.VERSION_CODES.GINGERBREAD_MR1)
	public void setImageStream(InputStream stream) {
		if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.GINGERBREAD_MR1) {
			try {
				mBitmapRegionDecoder = BitmapRegionDecoder.newInstance(stream, true);
				mWidth = mBitmapRegionDecoder.getWidth();
				mHeight = mBitmapRegionDecoder.getHeight();
				requestLayout();
			} catch (IOException e) {
				e.printStackTrace();
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
		invalidate();
	}
	
	/**
	 * set image from bitmap
	 * @param bm
	 */
	@Override
	public void setImageBitmap(Bitmap bm) {
		this.setImageStream(bitmapToStrem(bm));
	}
	
	/**
	 * set image from resource
	 * @param resId
	 */
	@Override
	public void setImageResource(int resId) {
		this.setImageBitmap(BitmapFactory.decodeResource(getResources(), resId));
	}
	
	/**
	 * The Method Null
	 */
	@Override
	public void setImageDrawable(Drawable drawable) {
		try {
			this.setImageBitmap(drawableToBitamp(drawable));
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	@Override
	protected void onDraw(Canvas canvas) {
		canvas.drawColor(0xffffffff);
		if (mBitmapRegionDecoder != null) {
			canvasLarge(canvas, mBitmapRegionDecoder);
		}
	}
	
	@Override
	public Drawable getDrawable() {
		// TODO Auto-generated method stub
		return super.getDrawable();
	}
	
	/**
	 * bitmap 转 strem
	 * @param bm
	 * @return
	 */
	protected InputStream bitmapToStrem(Bitmap bm) {
        return new ByteArrayInputStream(Bitmap2Bytes(bm));
	}
	
	/**
	 * Bitmap转Bytes
	 * @param bm
	 * @return
	 */
	public byte[] Bitmap2Bytes(Bitmap bm){    
		ByteArrayOutputStream baos = new ByteArrayOutputStream();      
		bm.compress(Bitmap.CompressFormat.JPEG, 100, baos);      
		return baos.toByteArray();    
	}
	
	
	/**
	 * Drawable 转 Bitmap
	 * @param drawable
	 * @return
	 */
	private Bitmap drawableToBitamp(Drawable drawable) {
        int w = drawable.getIntrinsicWidth();
        int h = drawable.getIntrinsicHeight();
        Bitmap.Config config = 
                drawable.getOpacity() != PixelFormat.OPAQUE ? Bitmap.Config.ARGB_8888
                        : Bitmap.Config.RGB_565;
        Bitmap bitmap = Bitmap.createBitmap(w,h,config);
        //注意，下面三行代码要用到，否在在View或者surfaceview里的canvas.drawBitmap会看不到图
        Canvas canvas = new Canvas(bitmap);   
        drawable.setBounds(0, 0, w, h);   
        drawable.draw(canvas);
        return bitmap;
	}
}


</pre>

由于对源码不是很熟悉，所以还是有些 BUG 的，但是显示个长图片还是没有什么问题的。