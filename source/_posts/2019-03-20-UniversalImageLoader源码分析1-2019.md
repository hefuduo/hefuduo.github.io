---
layout:     post
title:      "Android-Universal-ImageLoader源代码分析1"
subtitle:   "图片库源码分析"
date:       2019-03-20 11:19:00
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

# 写在前面

## AUIL 的特点

- 多线程下载图片(同步异步)
- 提供多种ImageLoader的用户配置(thread executors, downloaders,decoder, memory and disk cache, display image options, etc.)
- 为每一个显示图片的调用提供了非常丰富的配置(stub image, caching switch, decoding options, Bitmap processing and displaying, etc)
- 提供图片的内存和文件缓存
- 图片加载的监听和下载进度的监听

<!-- more -->

## 使用

### QuickSetup

1. include library

   ```groovy
   implementation 'com.nostra13.universalimageloader:universal-image-loader:1.9.5'
   ```

2. config manifest permission

   ```xml
   manifest>
   	<!-- Include following permission if you load images from Internet -->
   	<uses-permission android:name="android.permission.INTERNET" />
   	<!-- Include following permission if you want to cache images on SD card -->
   	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
   	...
   </manifest>
   ```

3. config imageloader and initial 

   ```java
   public class MyActivity extends Activity {
   	@Override
   	public void onCreate() {
   		super.onCreate();
   
   		// Create global configuration and initialize ImageLoader with this config
   		ImageLoaderConfiguration config = new ImageLoaderConfiguration.Builder(this)
   			...
   			.build();
   		ImageLoader.getInstance().init(config);
   		...
   	}
   }
   ```

   

4. Display Options

   ```java
   // DON'T COPY THIS CODE TO YOUR PROJECT! This is just example of ALL options using.
   // See the sample project how to use ImageLoader correctly.
   DisplayImageOptions options = new DisplayImageOptions.Builder()
   		.showImageOnLoading(R.drawable.ic_stub) // resource or drawable
   		.showImageForEmptyUri(R.drawable.ic_empty) // resource or drawable
   		.showImageOnFail(R.drawable.ic_error) // resource or drawable
   		.resetViewBeforeLoading(false)  // default
   		.delayBeforeLoading(1000)
   		.cacheInMemory(false) // default
   		.cacheOnDisk(false) // default
   		.preProcessor(...)
   		.postProcessor(...)
   		.extraForDownloader(...)
   		.considerExifParams(false) // default
   		.imageScaleType(ImageScaleType.IN_SAMPLE_POWER_OF_2) // default
   		.bitmapConfig(Bitmap.Config.ARGB_8888) // default
   		.decodingOptions(...)
   		.displayer(new SimpleBitmapDisplayer()) // default
   		.handler(new Handler()) // default
   		.build();
   ```

   

### Usage

#### 可接受的的URI资源示例

```
"http://site.com/image.png" // from Web
"file:///mnt/sdcard/image.png" // from SD card
"file:///mnt/sdcard/video.mp4" // from SD card (video thumbnail)
"content://media/external/images/media/13" // from content provider
"content://media/external/video/media/13" // from content provider (video thumbnail)
"assets://image.png" // from assets
"drawable://" + R.drawable.img // from drawables (non-9patch images)
```

**NOTE:** Use `drawable://` only if you really need it! Always **consider the native way** to load drawables - `ImageView.setImageResource(...)` instead of using of `ImageLoader`.



#### 简单使用

```java
// Load image, decode it to Bitmap and display Bitmap in ImageView (or any other view 
//	which implements ImageAware interface)
ImageLoader imageLoader = ImageLoader.getInstance();
imageLoader.loadImage(imageUri, imageview);
```



```java
// Load image, decode it to Bitmap and return Bitmap to callback
imageLoader.loadImage(imageUri, new SimpleImageLoadingListener() {
	@Override
	public void onLoadingComplete(String imageUri, View view, Bitmap loadedImage) {
		// Do whatever you want with Bitmap
	}
});
```



```java
// Load image, decode it to Bitmap and return Bitmap synchronously
Bitmap bmp = imageLoader.loadImageSync(imageUri);
```

#### 完全使用

```java
// Load image, decode it to Bitmap and display Bitmap in ImageView (or any other view 
//	which implements ImageAware interface)
imageLoader.displayImage(imageUri, imageView, options, new ImageLoadingListener() {
	@Override
	public void onLoadingStarted(String imageUri, View view) {
		...
	}
	@Override
	public void onLoadingFailed(String imageUri, View view, FailReason failReason) {
		...
	}
	@Override
	public void onLoadingComplete(String imageUri, View view, Bitmap loadedImage) {
		...
	}
	@Override
	public void onLoadingCancelled(String imageUri, View view) {
		...
	}
}, new ImageLoadingProgressListener() {
	@Override
	public void onProgressUpdate(String imageUri, View view, int current, int total) {
		...
	}
});
// Load image, decode it to Bitmap and return Bitmap to callback
ImageSize targetSize = new ImageSize(80, 50); // result Bitmap will be fit to this size
imageLoader.loadImage(imageUri, targetSize, options, new SimpleImageLoadingListener() {
	@Override
	public void onLoadingComplete(String imageUri, View view, Bitmap loadedImage) {
		// Do whatever you want with Bitmap
	}
});
```

```java
// Load image, decode it to Bitmap and return Bitmap synchronously
ImageSize targetSize = new ImageSize(80, 50); // result Bitmap will be fit to this size
Bitmap bmp = imageLoader.loadImageSync(imageUri, targetSize, options);
```

#### Load & Display Task Flow

重要, 这个是AUIL的加载图片流程

![AUIL](https://github.com/nostra13/Android-Universal-Image-Loader/raw/master/wiki/UIL_Flow.png)



## 用户须知



1. **Caching is NOT enabled by default**如果需要开启内存或者是磁盘缓存,需要配置`DisplayImageOptions`

   ```java
   // Create default options which will be used for every 
   //  displayImage(...) call if no options will be passed to this method
   DisplayImageOptions defaultOptions = new DisplayImageOptions.Builder()
      		...
              .cacheInMemory(true)
              .cacheOnDisk(true)
              ...
              .build();
   ImageLoaderConfiguration config = new ImageLoaderConfiguration.Builder(getApplicationContext())
              ...
              .defaultDisplayImageOptions(defaultOptions)
              ...
              .build();
   ImageLoader.getInstance().init(config); // Do it on Application start
   ```

   ```
   // Then later, when you want to display image
   ImageLoader.getInstance().displayImage(imageUrl, imageView); // Default options will be used
   ```

   或者这样:

   ```
   DisplayImageOptions options = new DisplayImageOptions.Builder()
      		...
              .cacheInMemory(true)
              .cacheOnDisk(true)
              ...
              .build();
   ImageLoader.getInstance().displayImage(imageUrl, imageView, options); // Incoming options will be used
   ```

2. 注意`WRITE_EXTERNAL_STORAGE`这个权限的申请.
3. 如果发生OOM可以尝试关掉内存缓存, 或减少线程池的大小, 或使用RGB-565的图像配置替换ARGB-8888, 或者裁剪图片显示小图.
4. MemoryChache的配置
   1. 强引用 `LruMemoryCache`
   2. 强弱引用混用 1,3除外的算法.
   3. 只是用弱引用 `WeakMemoryCache`

5. `DiscChache`也是可配置的`UnlimitedDiscCache` 是最快的,但是占用存储空间

6. 使用`RoundedBigmaDisplayer`或`FadeInbitmapDisplayer` 装饰图片加载

7. 为了是`ListView` or `GrideView`的性能提升, 使用`PauseOnScrollListener`

   ```java
   boolean pauseOnScroll = false; // or true
   boolean pauseOnFling = true; // or false
   PauseOnScrollListener listener = new PauseOnScrollListener(imageLoader, pauseOnScroll, pauseOnFling);
   listView.setOnScrollListener(listener);
   ```

8. If you see in logs some strange supplement at the end of image URL (e.g. http://anysite.com/images/image.png_230x460) then it doesn't mean this URL is used in requests. This is just "URL + target size", also this is key for Bitmap in memory cache. This postfix (_230x460) is NOT used in requests.

   

