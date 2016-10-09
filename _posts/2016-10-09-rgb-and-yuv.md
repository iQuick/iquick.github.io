---
layout:		post
date: 		2016-10-09 15:47
category:	[Android]
tags:		[Android]
published:	true
catalog:    true
title: 		'音视频 - RGB & YUV 初探'
---

## 简述
以前对颜色空间的什么的没有什么认识, RGB 认识还是图片的三原色，由于最近在学习音视频方面的知识，所以也在网上看了不少的关于 YUV & RGB 的文章。
俗话说，好记性不如烂笔头。这里结合众多“大大”们的文章，在此记一笔...

## 名词讲解

#### RGB
RGB色彩模式是工业界的一种颜色标准，是通过对红(R)、绿(G)、蓝(B)三个颜色通道的变化以及它们相互之间的叠加来得到各式各样的颜色的，RGB即是代表红、绿、蓝三个通道的颜色，这个标准几乎包括了人类视力所能感知的所有颜色，是目前运用最广的颜色系统之一。它是最通用的面向硬件的彩色模型,该模型用于彩色监视器和一大类彩色视频摄像。

#### YUV
RGB（红绿蓝）是依据人眼识别的颜色定义出的空间，可表示大部分颜色。但在科学研究一般不采用RGB颜色空间，因为它的细节难以进行数字化的调整。它将色调，亮度，饱和度三个量放在一起表示，很难分开，所以继而“大大们”推出了 YUV 彩色模型。在 YUV 空间中，每一个颜色有一个亮度信号 Y，和两个色度信号 U 和 V。亮度信号是强度的感觉，它和色度信号断开，这样的话强度就可以在不影响颜色的情况下改变。

<!-- break -->

## RGB 与 YUV 互转

#### 公式
YUV 使用RGB的信息，但它从全彩色图像中产生一个黑白图像，然后提取出三个主要的颜色变成两个额外的信号来描述颜色。把这三个信号组合回来就可以产生一个全彩色图像。如下图：

![RGB&YUV 互转]({{ BASE_PATH }}/img/post/rgb-yuv/rgb2yuv.png)

公式：
```java
Y = 0.299 R + 0.587 G + 0.114 B
U = -0.1687 R - 0.3313 G + 0.5 B + 128
V = 0.5 R - 0.4187 G - 0.0813 B + 128

R = Y + 1.402 (V-128)
G = Y - 0.34414 (U-128) - 0.71414 (V-128)
B = Y + 1.772 (U-128)
```

#### 代码实现

YUV 转 RGB
```java
public static boolean YUV2RGB(RGB rgb, byte y, byte u, byte v) {

	// 转成 int 
	int iY = y & 0xff;
	int iU = u & 0xff;
	int iV = v & 0xff;
	
	return YUV2RGB(rgb, iY, iU, iV);
}

public static boolean YUV2RGB(RGB rgb, int y, int u, int v) {
	
	rgb.r = (int) (y + 1.402 * (v - 128));
	rgb.g = (int) (y - 0.34414 * (u - 128) - 0.71414 * (v - 128));
	rgb.r = (int) (y + 1.772 * (u - 128));
	
	// 调整出界
	if (!rgbAdjust(rgb)) {
		return false;
	}
	
	return true;
}
```

RGB 转 YUV
```java
public static boolean RGB2YUV(YUV yuv, byte r, byte g, byte b) {
	int iR = r & 0xff;
	int iG = g & 0xff;
	int iB = b & 0xff;
	return RGB2YUV(yuv, iR, iG, iB);
}

public static boolean RGB2YUV(YUV yuv, int r, int g, int b) {
	yuv.y = (int) (0.299 * r + 0.587 * g + 0.114 * b);
	yuv.u = (int) (-0.1687 * r - 0.3313 * g + 0.5 * b) + 128;
	yuv.v = (int) (0.5 * r -0.4187 * g - 0.813 * b) + 128;
	
	if (!yuvAdjust(yuv)) {
		return false;
	}
	
	return true;
}
```

## YUV（YCbCr）采样格式

### YUV 采样格式说明
主要的采样格式有 YCbCr 4:2:0、YCbCr 4:2:2、YCbCr 4:1:1和 YCbCr 4:4:4。其中YCbCr 4:1:1 比较常用，其含义为：每个点保存一个 8bit 的亮度值(也就是Y值), 每 2x2 个点保存一个 Cr 和Cb 值, 图像在肉眼中的感觉不会起太大的变化。所以, 原来用 RGB(R,G,B 都是 8bit unsigned) 模型, 4 个点需要 8x3=24 bites（如下图第一个图）. 而现在仅需要 8+(8/4)+(8/4)=12bites, 平均每个点占12bites(如下图第二个图)。这样就把图像的数据压缩了一半。

![图1]({{ BASE_PATH }}/img/post/rgb-yuv/img1.png)

![图2]({{ BASE_PATH }}/img/post/rgb-yuv/img2.png)

### YUV 4:4:4
YUV三个信道的抽样率相同，因此在生成的图像里，每个象素的三个分量信息完整（每个分量通常8比特），经过8比特量化之后，未经压缩的每个像素占用3个字节。

下面的四个像素为: [Y0 U0 V0] [Y1 U1 V1] [Y2 U2 V2] [Y3 U3 V3]

存放的码流为: Y0 U0 V0 Y1 U1 V1 Y2 U2 V2 Y3 U3 V3

### YUV 4:2:2
每个色差信道的抽样率是亮度信道的一半，所以水平方向的色度抽样率只是4:4:4的一半。对非压缩的8比特量化的图像来说，每个由两个水平方向相邻的像素组成的宏像素需要占用4字节内存（亮度2个字节,两个色度各1个字节）。。

下面的四个像素为: [Y0 U0 V0] [Y1 U1 V1] [Y2 U2 V2] [Y3 U3 V3]

存放的码流为: Y0 U0 Y1 V1 Y2 U2 Y3 V3

映射出像素点为：[Y0 U0 V1] [Y1 U0 V1] [Y2 U2 V3] [Y3 U2 V3]

### YUV 4:1:1
4:1:1的色度抽样，是在水平方向上对色度进行4:1抽样。对于低端用户和消费类产品这仍然是可以接受的。对非压缩的8比特量化的视频来说，每个由4个水平方向相邻的像素组成的宏像素需要占用6字节内存（亮度4个字节,两个色度各1个字节）。

下面的四个像素为: [Y0 U0 V0] [Y1 U1 V1] [Y2 U2 V2] [Y3 U3 V3]

存放的码流为: Y0 U0 Y1 Y2 V2 Y3

映射出像素点为：[Y0 U0 V2] [Y1 U0 V2] [Y2 U0 V2] [Y3 U0 V2]

### YUV 4:2:0
4:2:0并不意味着只有Y,Cb而没有Cr分量。它指得是对每行扫描线来说，只有一种色度分量以2:1的抽样率存储。相邻的扫描行存储不同的色度分量， 也就是说，如果一行是4:2:0的话，下一行就是4:0:2，再下一行是4:2:0...以此类推。对每个色度分量来说，水平方向和竖直方向的抽样率都是 2:1，所以可以说色度的抽样率是4:1。对非压缩的8比特量化的视频来说，每个由2x2个2行2列相邻的像素组成的宏像素需要占用6字节内存（亮度4个字节,两个色度各1个字节）。

下面八个像素为：[Y0 U0 V0] [Y1 U1 V1] [Y2 U2 V2] [Y3 U3 V3]

[Y5 U5 V5] [Y6 U6 V6] [Y7U7 V7] [Y8 U8 V8]

存放的码流为：Y0 U0 Y1 Y2 U2 Y3 Y5 V5 Y6 Y7 V7 Y8

映射出的像素点为：[Y0 U0 V5] [Y1 U0 V5] [Y2 U2 V7] [Y3 U2 V7] [Y5 U0 V5] [Y6 U0 V5] [Y7U2 V7] [Y8 U2 V7]


## YUV 格式

#### YUV存储格式（述像素的Y、U、V分量排列方式）

* 紧缩格式(packed formats)：将Y、U、V值储存成Macro Pixels阵列，和RGB的存放方式类似。
* 平面格式(planar formats)：将Y、U、V的三个分量分别存放在不同的矩阵中。

#### 常见格式 

常见 YUV 格式YV16、YV12、IYUV、I420、NV12、NV21

#### YV16
YUV422 有打包格式(Packed)，一如前文所述。同时还有平面格式(Planar)，即Y、U、V是分开存储的，每个分量占一块地方，其中Y为 width * height，而U、V合占 width * height ，该种格式每个像素占16比特。根据U、V的顺序，分出2种格式，U前V后即YUV422P，也叫I422，V前U后，叫YV16(YV表示Y后面跟着V，16表示16bit)

#### YV16 转 RGB
```java
/**
 * YV16 是 YUV:422 格式，是三个 plane,(Y)(U)(V)
 * 
 * @param src 数据源
 * @param width 
 * @param height
 * @return
 */
public static int[] YV16ToRGB(byte[] src, int width, int height) {
		int len = width * height;
		int yStart = 0;
		int uStart = len;
		int vStart = len * 3 / 2;
		
		// RGB数组
		int[] rgbs = new int[len * 3];
		
		// RGB
		RGB rgb = new RGB();
		// 计算转换
		for (int i = 0; i < width; i++) {
			for (int j = 0; j < height; j++) {
				
				int yIdx = yStart + i * width + j;
				int uIdx = uStart + i * width / 2 + j /2;
				int vIdx = vStart + i * width / 2 + j /2;
				
				boolean to = YUV2RGB(rgb, src[yIdx], src[uIdx], src[vIdx]);
				
				// 填充
				rgbs[yIdx * 3 + 0] = rgb.r;
				rgbs[yIdx * 3 + 1] = rgb.g;
				rgbs[yIdx * 3 + 2] = rgb.b;
			}
		}
		
		return rgbs;
}
```

#### YV12
数组后面紧接着所有 V (Cr) 样例。V 平面的跨距为 Y 平面跨距的一半，V 平面包含的行为 Y 平面包含行的一半。V 平面后面紧接着所有 U (Cb) 样例，它的跨距和行数与 V 平面相同。

#### YV12 转 RGB
```java
/**
 * 
 * YV12 是 yuv420 格式，是3个plane，排列方式为 (Y)(V)(U)  
 * 
 * @param src
 * @param width
 * @param height
 * @return
 */
public static int[] YV12ToRGB(byte[] src, int width, int height) {
	int len = width * height;
	int yStart = 0;
	int vStart = len;
	int uStart = len * 5 / 4;
	
	// RGB 数组
	int rgbs[] = new int[len * 3];

	// RGB
	RGB rgb = new RGB();
	// 计算转换
	for (int i = 0; i < height; i++) {
		for (int j = 0; j < width; j++) {

			int yIdx = yStart + i * width + j;
			int uIdx = uStart + (i/2)*(width/2) + j / 2;
			int vIdx = vStart + (i/2)*(width/2) + j / 2;
			
			boolean to = YUV2RGB(rgb, src[yIdx], src[uIdx], src[vIdx]);
			
			// 填充
			rgbs[yIdx * 3 + 0] = rgb.r;
			rgbs[yIdx * 3 + 1] = rgb.g;
			rgbs[yIdx * 3 + 2] = rgb.b;
		}
	}
	
    return rgbs;  
}  
```

## 源码
* Github：[https://github.com/iQuick/GeekStudy/tree/master/YUV&RGB](https://github.com/iQuick/GeekStudy/tree/master/YUV&RGB)

## 参考
* YUV : [https://en.wikipedia.org/wiki/YUV](https://en.wikipedia.org/wiki/YUV) 
* YUV 采样格式：[http://blog.csdn.net/firstlai/article/details/52169783](http://blog.csdn.net/firstlai/article/details/52169783)
* YUV & RGB　互转：[http://www.cnblogs.com/dwdxdy/p/3713990.html](http://www.cnblogs.com/dwdxdy/p/3713990.html)
* YUV & RGB　互转：[http://blog.csdn.net/huiguixian/article/details/17334195](http://blog.csdn.net/huiguixian/article/details/17334195)
* YUV 常见格式说明：[http://www.cnblogs.com/dwdxdy/p/3713968.html](http://www.cnblogs.com/dwdxdy/p/3713968.html)
* YUV 422P、YV16、NV16、NV61格式转换成RGB24：[http://blog.csdn.net/subfate/article/details/47304945](http://blog.csdn.net/subfate/article/details/47304945)