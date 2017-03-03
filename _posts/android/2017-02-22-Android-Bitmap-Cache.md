---
layout: post
title: 2017-2-22 Android Bitmap的加载和Cache
date: 2017-2-22
excerpt: "Android Bitmap的加载和Cache"
tags: [Android基础]
comments: true
---


## 简介

内容包括三部分：

- 如何有效加载Bitmap
    - 由于Bitmap的特殊性以及Android对单个应用所施加的内存限制，比如16MB，这导致Bitmap加载时很容易出现OOM
    - java.lang.OutOfMemoryError: bitmap size exceeds VM budget
    - 因此高效的加载Bitmap时非常重要的
- Android中常用的缓存策略
    - 缓存策略是一个通用的思想，可以用在很多场景中。实际开发中经常需要用到Bitmap做缓存。
    - 通过缓存不需每次从网上请求图片或者从设备中加载图片，可以极大地提高图片的加载效率以及产品的用户体验。
    - 较常用的缓存策略是LruCache和DiskLruCache
        - LruCache和DiskLruCache是采用了LRU(Least Recently Used)近期最少使用算法的两种缓存。
        - LruCache内存缓存，DiskLruCache存储设备缓存
- 如何优化列表卡顿现象
    - ListView和GridView由于要加载大量的自视图，用户快速滑动时就容易出现卡顿现

## 一、 Bitmap的高效加载

- 如何加载一个图片

    BitmapFactory提供了很多方法支持从文件系统，资源，输入流以及字节数组中加载Bitmap多些。
    
- 如何高效的加载Bitmap
- 核心思想：用BitmapFactory.Options来加载所需尺寸的图片
    - inSampleSize参数设定为整数N，图片大小是原图大小的1/N
    - inSampleSize的取值应该总是2的指数

核心Code:

        public Bitmap decodeSampledBitmapFromResource(Resources res,
                                                      int resId, int reqWidth, int reqHeight) {
            // First decode with inJustDecodeBounds=true to check dimensions
            final BitmapFactory.Options options = new BitmapFactory.Options();
            options.inJustDecodeBounds = true;
            BitmapFactory.decodeResource(res, resId, options);
    
            // Calculate inSampleSize
            options.inSampleSize = calculateInSampleSize(options, reqWidth,
                    reqHeight);
    
            // Decode bitmap with inSampleSize set
            options.inJustDecodeBounds = false;
            return BitmapFactory.decodeResource(res, resId, options);
        }
    
        public int calculateInSampleSize(BitmapFactory.Options options,
                                     int reqWidth, int reqHeight) {
        if (reqWidth == 0 || reqHeight == 0) {
            return 1;
        }

        // Raw height and width of image
        final int height = options.outHeight;
        final int width = options.outWidth;
        Log.d(TAG, "origin, w= " + width + " h=" + height);
        int inSampleSize = 1;

        if (height > reqHeight || width > reqWidth) {
            final int halfHeight = height / 2;
            final int halfWidth = width / 2;

            // Calculate the largest inSampleSize value that is a power of 2 and
            // keeps both height and width larger than the requested height and width.
            while ((halfHeight / inSampleSize) >= reqHeight
                    && (halfWidth / inSampleSize) >= reqWidth) {
                inSampleSize *= 2;
            }
        }

        Log.d(TAG, "sampleSize:" + inSampleSize);
        return inSampleSize;
    }
    
## 二、 Android中的缓存策略

缓存策略在Android中有着广泛的使用场景，尤其是在图片加载这个场景下，缓存策略就变得更为重要。

经典的使用

- 解决网络下载时，流量消耗问题。缓存可以避免过多的流量消耗。
- 很多时候为了提高应用的用户体验，往往还会把图片在内存中在缓存一份。
- 当应用打算从网络请求一张图片时，先从内存中取，没有就从存储设备中获取。如果没有就从网络下载。

缓存策略

- 主要包含：缓存的添加、获取和删除三类操作。
- 目前常用的一种算法是LRU（Least Recently Used),是近期最少使用算法。
    - 核心思想是：当缓存满时，会优先淘汰那些近期最少使用的缓存对象
    - 采用LRU算法的缓存有：LruCache 和 DiskLruCache
    - LruCache用于实现内存缓存
    - DiskLruCache则用于存储设备缓存

### LruCache

Android 3.1提供的一个缓存类。是一个泛型类，内部采用一个LinkedHashMap以强引用的方式存储外界的缓存对象。

    public class LruCache<K, V> {

- 强引用：直接的对象引用
- 软引用：当对象只有软引用时，系统内存不足时此对象会被gc回收
- 弱引用：当对象只有弱引用时，对象会随时被gc回收

核心Code:

    int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);
    int cacheSize = maxMemory / 8;
    mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
        @Override
        protected int sizeOf(String key, Bitmap bitmap) {
            return bitmap.getRowBytes() * bitmap.getHeight() / 1024;
        }
    };
        
    private void addBitmapToMemoryCache(String key, Bitmap bitmap) {
        if (getBitmapFromMemCache(key) == null) {
            mMemoryCache.put(key, bitmap);
        }
    }


    private Bitmap getBitmapFromMemCache(String key) {
        return mMemoryCache.get(key);
    }
    
### DiskLruCache

- 用于实现存储设备缓存，即磁盘缓存。
- 通过将缓存对象写入文件系统从而实现缓存的效果。
- Android官方文档的推荐，但它不属于Android的一部分，[下载地址](https://android.googlesource.com/platform/libcore/+/android-4.1.1_r1/luni/src/main/java/libcore/io/DiskLruCache.java)

#### 1. 创建核心Code

        private static final long DISK_CACHE_SIZE = 1024 * 1024 * 50;
        
        File diskCacheDir = getDiskCacheDir(mContext, "bitmap");
        if (!diskCacheDir.exists()) {
            diskCacheDir.mkdirs();
        }
        
        if (getUsableSpace(diskCacheDir) > DISK_CACHE_SIZE) {
            try {
                mDiskLruCache = DiskLruCache.open(diskCacheDir, 1, 1,
                        DISK_CACHE_SIZE);
                mIsDiskLruCacheCreated = true;
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        
- open的第一参数代表存储路径。
    - 如果希望应用卸载后目录一起删除，可以放在SD缓存目录（sdcard/Android/data/package_name/cache）
    - 如果希望卸载后保留缓存数据，选择SD卡上的其他特定目录
- 第二个参数是应用的版号，一般设为1即可。
- 第三个参数表示单个节点所对应的数据的个数，一般设为1
- 第四个参数表述缓存的总大小，例如50M

#### 2. DiskLruCache的缓存添加

- 缓存添加时通过Editor完成的，Editor表示一个缓存对象的编辑对象。
- 以图片缓存举例
    - key一般采用url的md5值，因为图片的url中通常可能有特殊字符

核心Code:

    private String hashKeyFormUrl(String url) {
        String cacheKey;
        try {
            final MessageDigest mDigest = MessageDigest.getInstance("MD5");
            mDigest.update(url.getBytes());
            cacheKey = bytesToHexString(mDigest.digest());
        } catch (NoSuchAlgorithmException e) {
            cacheKey = String.valueOf(url.hashCode());
        }
        return cacheKey;
    }

    private String bytesToHexString(byte[] bytes) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < bytes.length; i++) {
            String hex = Integer.toHexString(0xFF & bytes[i]);
            if (hex.length() == 1) {
                sb.append('0');
            }
            sb.append(hex);
        }
        return sb.toString();
    }

写入磁盘核心Code:

        String key = hashKeyFormUrl(url);
        DiskLruCache.Editor editor = mDiskLruCache.edit(key);
        if (editor != null) {
            OutputStream outputStream = editor.newOutputStream(DISK_CACHE_INDEX);
            if (downloadUrlToStream(url, outputStream)) {
                editor.commit();
            } else {
                editor.abort();
            }
            mDiskLruCache.flush();
        }

#### 3. DiskLruCache的缓存查找

- 先从url获取key
- 通过get方法得到Snapshot对象，接着可以得到缓存的文件输入流
- 最后获取Bitmap对象

查找核心Code:

        Bitmap bitmap = null;
        String key = hashKeyFormUrl(url);
        DiskLruCache.Snapshot snapShot = mDiskLruCache.get(key);
        if (snapShot != null) {
            FileInputStream fileInputStream = (FileInputStream)snapShot.getInputStream(DISK_CACHE_INDEX);
            FileDescriptor fileDescriptor = fileInputStream.getFD();
            bitmap = mImageResizer.decodeSampledBitmapFromFileDescriptor(fileDescriptor,
                    reqWidth, reqHeight);
            ......
        }
        
## ImageLoader实践

一个优秀的ImageLoader应该具备：

- 图片的同步加载
- 图片的异步加载
- 图片的压缩
- 内存缓存
- 磁盘缓存
- 网络拉取

内存缓存和磁盘缓存是ImageLoader的核心，也是ImageLoader的意义所在。

除此之外ImageLoader还需要处理一些特殊情况，比如ListView或GridView中View复用时图片显示问题。

比如itemA的图片还在下载，user滑动后itemB复用了itemA。待itemA的图片下载完毕后，itemB中显示了itemA的图片。

ImageLoader需要正确的处理这些特殊情况。

(1) 图片压缩ImageResizer：ImageResizer.java

(2) 内存缓存和磁盘缓存实现

核心Code:
    
    private LruCache<String, Bitmap> mMemoryCache;
    private DiskLruCache mDiskLruCache;

    private ImageLoader(Context context) {
        mContext = context.getApplicationContext();
        int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);
        int cacheSize = maxMemory / 8;
        mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
            @Override
            protected int sizeOf(String key, Bitmap bitmap) {
                return bitmap.getRowBytes() * bitmap.getHeight() / 1024;
            }
        };
        File diskCacheDir = getDiskCacheDir(mContext, "bitmap");
        if (!diskCacheDir.exists()) {
            diskCacheDir.mkdirs();
        }
        if (getUsableSpace(diskCacheDir) > DISK_CACHE_SIZE) {
            try {
                mDiskLruCache = DiskLruCache.open(diskCacheDir, 1, 1,
                        DISK_CACHE_SIZE);
                mIsDiskLruCacheCreated = true;
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

(3) 同步加载和异步加载

- 同步接口： public Bitmap loadBitmap(String uri, int reqWidth, int reqHeight) {
- 异步接口： public void bindBitmap(final String uri, final ImageView imageView, final int reqWidth, final int reqHeight) {
    - 异步加载时用线程池 + Handler更新ImageView
    - 设定ImageView的Tag来区分，避免发生列表错位问题
    
    避免发生列表核错位心Code
    
        private Handler mMainHandler = new Handler(Looper.getMainLooper()) {
        @Override
        public void handleMessage(Message msg) {
            LoaderResult result = (LoaderResult) msg.obj;
            ImageView imageView = result.imageView;
            String uri = (String) imageView.getTag(TAG_KEY_URI);
            if (uri.equals(result.uri)) {
                imageView.setImageBitmap(result.bitmap);
            } else {
                Log.w(TAG, "set image bitmap,but url has changed, ignored!");
            }
        }
    };

## ImageLoader的使用

照片墙

- 从网络加载图片并在GridView中显示.覆盖了：图片加载，缓存策略，列表的滑动流畅性
- 非WIFI环境下，流量提示

    提示核心Code
    
    
        if (!mIsWifi) {
            AlertDialog.Builder builder = new AlertDialog.Builder(this);
            builder.setMessage("初次使用会从网络下载大概5MB的图片，确认要下载吗？");
            builder.setTitle("注意");
            builder.setPositiveButton("是", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    Utils.sCanGetBitmapFromNetWork = true;
                    mImageAdapter.notifyDataSetChanged();
                }
            });
            builder.setNegativeButton("否", null);
            builder.show();
        }
        
- 优化列表的卡顿现象
    - 主线程中不做太耗时的操作：getView中通过ImageLoader的bindBitmap来异步加载图片
    - 用户频繁滑动，瞬间可能产生上百个异步任务，会造成线程池拥堵和大量的UI更新。
        - 列表滑动时停止加载图片
        - getView仅列表静止时才能加载图片
        - [开启硬件加速](https://developer.android.com/guide/topics/graphics/hardware-accel.html)。绝大多数情况下，硬件加速都可以解决莫名的卡顿问题。

解决列表卡顿核心Code

    // MainActivity
    mImageGridView.setOnScrollListener(this);

    @Override
    public void onScrollStateChanged(AbsListView view, int scrollState) {
        if (scrollState == AbsListView.OnScrollListener.SCROLL_STATE_IDLE) {
            Utils.sIsGridViewIdle = true;
            mImageAdapter.notifyDataSetChanged();
        } else {
            Utils.sIsGridViewIdle = false;
        }
    }
    
    //ImageAdapter
    if (Utils.sIsGridViewIdle && Utils.sCanGetBitmapFromNetWork) {
        imageView.setTag(uri);
        mImageLoader.bindBitmap(uri, imageView, Utils.sImageWidth, Utils.sImageWidth);
    }


[Github Code](https://github.com/vivianking6855/android-advanced/tree/master/ImageCache/app/src/main/java/com/vv/imagecache)


## 开源库

图片缓存已经有很多的开源库：([详情介绍点击这里，搜索ImageLoader](https://github.com/Trinea/android-open-project)）

1. Android-Universal-Image-Loader
2. picasso
3. Cube ImageLoader
4. fresco
5. Glide

<br>



> [Android 开源项目分类汇总 ](https://github.com/Trinea/android-open-project)

> [Android 学习资料收集](https://github.com/Freelander/Android_Data)


