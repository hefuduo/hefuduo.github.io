---
layout:     post
title:      "Android-Universal-ImageLoader源代码分析2"
subtitle:   "图片库源码分析"
date:       2019-04-18 11:19:00
author:     "LeoHe"
categories: 源码阅读
toc: true
tags:
    - Android
    - open source library
    - Java
    - 源码分析	
    - 图片库
---



[TOC]

## 包结构与架构

包的结构与说明
```
universalimageloader| 

	|-cache
		|-disc
		|-memory
	|-core
		|-assist
		|-decode
		|-display
		|-download
		|-imageaware
		|-listener
		|-process
	|-utils
```

<!-- more -->

## 整体设计架构与说明



### 结构图表示



![UIL](/img/uil/UIL.png)

### 各模块的说明



整个库一级划分为`cache`, `core` , `util` 三个大模块.

1. `core`: 核心模块分为`decode(解码)`, `display(展示)`,`download(下载)`, `imageAware(显示图片的包装)`,`process(处理图片)` 五个重要的模块.此外,尤其要注意

   `ImageLoaderEngine`, `ImageDecorder`, `BitmapDisplayer`, `BitmapProcessor`,  `ImageDownloader`, `ImageAware` 这几个重要的类.

2. `cache`: 缓存模块分为`disc`缓存和`memory`缓存两个模块

3. `utils`:工具类模块, 如缓存工具类|Log|图像处理等. 





## 加载图片流程

### 用一张图简单的标识图片加载流程

![](https://raw.githubusercontent.com/nostra13/Android-Universal-Image-Loader/master/wiki/UIL_Flow.png)



### 加载流程图



![UIL流程图1](/img/uil/UIL流程图1.png)

### 详细执行流程

#### 加载的时序图

`loadImage`本质也是调用`displayImage`因此直接看`displayImage`的执行流程即可.

`ImageLoader.getInstance().displayImage(uri, imageview, options, listener);`

![UIL时序图](/img/uil/UIL时序图.png)

有缓存的时候, 先去检查缓存.参考流程图

#### DisplayTask 的详细执行流程图

> 参考`LoadAndDisplayimageTask`中的`run`方法



![LoadAndDisplayImageTask](/img/uil/LoadAndDisplayImageTask.png)







### //TODO UML



## 缓存算法详解

### 包结构

缓存算法从缓存位置上划分为两种, `memory`和`disc`.

```
|-cache
	|-disc
		|-impl(具体实现)
		|-naming(文件命名工具)
		|-DiskCache:interface(对外接口)
	|-memory
		|-impl(具体实现)
		|-BaseMemoryCache
		|-LimitedMemoryCache
		|-MemoryCache:interface(对外接口)
```



### UML

#### 内存缓存



![memocache](/img/uil/memocache.png)



#### 磁盘缓存







## 解码器与下载器

```java
//BaseImageDownloader
//根据URI的类型, 获取不同的InputStream
@Override
	public InputStream getStream(String imageUri, Object extra) throws IOException {
		switch (Scheme.ofUri(imageUri)) {
			case HTTP:
			case HTTPS:
				return getStreamFromNetwork(imageUri, extra);
			case FILE:
				return getStreamFromFile(imageUri, extra);
			case CONTENT:
				return getStreamFromContent(imageUri, extra);
			case ASSETS:
				return getStreamFromAssets(imageUri, extra);
			case DRAWABLE:
				return getStreamFromDrawable(imageUri, extra);
			case UNKNOWN:
			default:
				return getStreamFromOtherSource(imageUri, extra);
		}
	}
```

```java
//BaseImageDecoders
//将inputStream解码为Bitmap类型

@Override
	public Bitmap decode(ImageDecodingInfo decodingInfo) throws IOException {
		Bitmap decodedBitmap;
		ImageFileInfo imageInfo;

		InputStream imageStream = getImageStream(decodingInfo);
		if (imageStream == null) {
			L.e(ERROR_NO_IMAGE_STREAM, decodingInfo.getImageKey());
			return null;
		}
		try {
			imageInfo = defineImageSizeAndRotation(imageStream, decodingInfo);
			imageStream = resetStream(imageStream, decodingInfo);
			Options decodingOptions = prepareDecodingOptions(imageInfo.imageSize, decodingInfo);
			decodedBitmap = BitmapFactory.decodeStream(imageStream, null, decodingOptions);
		} finally {
			IoUtils.closeSilently(imageStream);
		}

		if (decodedBitmap == null) {
			L.e(ERROR_CANT_DECODE_IMAGE, decodingInfo.getImageKey());
		} else {
			decodedBitmap = considerExactScaleAndOrientatiton(decodedBitmap, decodingInfo, imageInfo.exif.rotation,
					imageInfo.exif.flipHorizontal);
		}
		return decodedBitmap;
	}
```

