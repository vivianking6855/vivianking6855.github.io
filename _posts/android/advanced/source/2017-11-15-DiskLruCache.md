---
layout: post
title: DiskLruCache源码学习
date: 2017-11-15
excerpt: "DiskLruCache源码学习"
categories: Android
tags: [开源源码学习]
comments: true
lefttrees: true
---

* content
{:toc}



# DiskLruCache源码学习

JakeWharton大神的DiskLruCache下载地址： [https://github.com/JakeWharton/DiskLruCache/](https://github.com/JakeWharton/DiskLruCache/)

核心的实现在DiskLruCache.java中

下面介绍几个关键的public方法

## 一、 open ： 创建DiskLruCache

DiskLruCache不能通过构造方法来new，只能通过open方法来创建

public static DiskLruCache open(File directory, int appVersion, int valueCount, long maxSize)  

参数说明：

1. File directory 指定数据的缓存地址 （比如：/sdcard/Android/data/<package name>/cache/ or /storage/emulated/0/Android/data/package name/cache/）
2. int appVersion 指定当前应用的version code
3. int valueCount 是Key所对应的文件数，通常设定 1. 对应后面的get方法的第一个参数（比如getString(int index)，index要设定为0，代表我们去第一个，话说我们也只有一个）
4. long maxSize 最大缓存大小 通常设定 5M（5*1024*1024） or 10M（10*1024*1024）

open的处理逻辑：

1. 首先检查存不存在journal.bkp（journal的备份文件）
2. 如果存在：然后检查journal文件是否存在，如果在，删除bkp文件；如果不存在，将bkp文件重命名为journal文件
3. 如果找不到journal文件，则创建，只写入文件头(5行)。 
4. 如果找到，则遍历该文件，将里面所有的CLEAN记录的key，存到lruEntries中。移除REMOVE记录

经过open以后，journal文件肯定存在了；lruEntries里面肯定有值了；size存储了当前所有的实体占据的容量；

## 二、 写入

DiskLruCache的写入操作是通过Editor来完成的，基本调用步骤：

edit(String key) 获取editor对象 -> set(int index, String value) or newOutputStream(int index) 写入数据 -> commit() or abort() 提交

public Editor edit(String key)

参数说明：

1. key 存储时制定的uid key, 必须是字母、数字、下划线、横线（-）组成，且长度在1-120之间

edit的逻辑:

1. 验证key，必须是字母、数字、下划线、横线（-）组成，且长度在1-120之间
2. 通过key获取实体。只要不是正在编辑这个实体，理论上都能返回一个合法的editor对象。
3. 如果不存在，则创建一个Entry加入到lruEntries中（如果存在，直接使用），然后为entry.currentEditor进行赋值为new Editor(entry);
4. 最后在journal文件中写入一条DIRTY记录，代表这个文件正在被操作。

注意，如果entry.currentEditor != null不为null的时候，意味着该实体正在被编辑，会retrun null ;

写入支援String和OutputStream

- public void set(int index, String value)
- public OutputStream newOutputStream(int index)

set写入逻辑

1. Writer（OutputStreamWriter）直接写入，UTF_8格式

newOutputStream写入逻辑：

1. 首先校验index是否在valueCount范围内，index 传入0（key 一对一）
2. 接下来通过entry.getDirtyFile(index)，拿到一个dirty File对象（中转文件，文件格式为key.index.tmp） 
3. 将这个文件的FileOutputStream通过FaultHidingOutputStream封装下传给我们

最后，别忘了我们通过os写入数据以后，需要调用commit方法。

提交或放弃

- public void commit() 
- public void abort()

commit逻辑

1. 首先通过hasErrors判断，是否有错误发生（FileOutputStream如果发生IOException,hasErrors赋值为true)
2. 如果有异常调用completeEdit(this, false)，调用remove(entry.key),删除dirtyFile,从lruEntries中移除，然后写入一条REMOVE记录。
3. 如果没有异常就调用completeEdit(this, true),将dirtyFile->cleanFile，将readable=true，写入CLEAN记录。

abort就是执行completeEdit(this, false)，即有异常的时候的处理。只是没有remove，因为还没commit加入。

## 三、 读取

InputStream 读取

public synchronized Snapshot get(String key) 

DiskLruCache.Snapshot调用getInputStream(0) or getString(0)

public InputStream getInputStream(int index) 

public String getString(int index)

get方法比较简单，如果取到的为null，或者readable=false，则返回null.

否则将cleanFile的FileInputStream进行封装返回Snapshot，且写入一条READ语句。 

## 其他方法

remove()

如果实体存在且不在被编辑，就可以直接进行删除，然后写入一条REMOVE记录。

close()

这个方法用于将DiskLruCache关闭掉，是和open()方法对应的一个方法。

关闭前，会判断所有正在编辑的实体，调用abort方法，最后关闭journalWriter。

关闭掉了之后就不能再调用DiskLruCache中任何操作缓存数据的方法，通常只应该在Activity的onDestroy()方法中去调用close()方法。

## 四、 缓存文件结构

DiskLurCache维护一个journal日志文件和N多cache文件（常用设定一个key对应一个文件）

首先看前五行：

- 第一行固定字符串libcore.io.DiskLruCache
- 第二行DiskLruCache的版本号，源码中为常量1
- 第三行为你的app的版本号，当然这个是你自己传入指定的
- 第四行指每个key对应几个文件，一般为1
- 第五行，空行

以上5行是文件头，DiskLruCache初始化的时候写入。如果该文件存在需要校验该文件头。

接下来的行，可以认为是操作记录。

- DIRTY： 表示一个entry正在被写入（其实就是把文件的OutputStream交给你了）。
- 写入分两种情况，如果成功会紧接着写入一行CLEAN的记录；如果失败，会增加一行REMOVE记录。
- REMOVE除了上述的情况呢，当你自己手动调用remove(key)方法的时候也会写入一条REMOVE记录。
- READ就是说明有一次读取的记录。
- 每个CLEAN的后面还记录了文件的长度，注意可能会一个key对应多个文件，那么就会有多个数字（参照文件头第四行）。

![](https://i.imgur.com/HEmsAmm.jpg)

![](https://i.imgur.com/Qq8wYax.jpg)


# 使用Code示例

Application中初始化，以及释放

    public class UserApplication extends Application {
    
        @Override
        public void onCreate() {
            super.onCreate();
    
            // init cache
            DiskCacheManager.INSTANCE.initDiskCacheManager(this);
            DiskLruCacheUtils.setDiskLruCache(DiskCacheManager.INSTANCE.getDiskLruCache(this));
        }
    
        @Override
        public void onTerminate() {
            super.onTerminate();
    
            DiskCacheManager.INSTANCE.releaseDiskCacheManager();
        }
        
        ......
    }
    
缓存数据

    public class DownloadPresenter {
    
        ......
    
        private void cacheToDisk(List<DataModel> list) {
            for (DataModel data : list) {
                DiskLruCacheUtils.set(data.id, data.description);
            }
        }
        
        ......
    }
    
    public String getFromDiskCache(String key) {
        return DiskLruCacheUtils.getString(key);
    }
    
DiskLruCacheUtils

    public final class DiskLruCacheUtils {
        private static final String TAG = "DiskLruCacheUtils";
        // DiskLruCache object, set from outside
        private static DiskLruCache mDiskLruCache;
    
        /**
         * Sets disk lru cache.
         * you need setDiskLruCache before you call any method here
         *
         * @param cache the cache
         */
        public static void setDiskLruCache(DiskLruCache cache) {
            mDiskLruCache = cache;
        }
    
        /**
         * Set String value
         *
         * @param key   the key
         * @param value the value
         */
        public static void set(String key, String value) {
            DiskLruCache.Editor editor = getEditor(key);
            try {
                if (editor != null) {
                    editor.set(0, value);
                    // write ,CLEAN
                    editor.commit();
                }
            } catch (IOException e) {
                Log.w(TAG, "set() ex ", e);
                abortEditor(editor);
            }
        }
    
        /**
         * Gets string value
         *
         * @param key the key
         * @return the string
         */
        public static String getString(String key) {
            DiskLruCache.Snapshot snapShot = getSnapshot(key);
            try {
                if (snapShot != null) {
                    return snapShot.getString(0);
                }
            } catch (IOException e) {
                Log.w(TAG, "getString() ex ", e);
            }
            return "";
        }
    }

[Cache Code Github下载地址](https://github.com/vivianking6855/android-advanced/tree/master/CacheDemo)

# 总结

1. 对文件读写完整性设计思路 DIRTY-CLEAN; DIRTY-ABORT; 可借鉴学习
2. Lru思想也是使用LinkedHashMap

    LinkedHashMap内部自己维护了一套元素访问顺序的列表, 最常用的元素放在链表的最后。LRU的关键是如果size大于了缓存最大容量，则删除链表的顶端元素
    
3. 接口只有对String和 OutputStream,InputStream. 重新封装了下，可以查看DiskLruCacheUtils


# Reference

[Android DiskLruCache 源码解析](http://blog.csdn.net/lmj623565791/article/details/47251585)

[base-diskcache](https://github.com/hongyangAndroid/base-diskcache)