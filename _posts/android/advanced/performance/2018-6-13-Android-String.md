---
layout: post
title: 性能优化 - Android String性能小结
date: 2018-6-13
excerpt: "性能优化 - Android String性能小结"
categories: Android 性能优化
tags: [Android 性能优化]
comments: true
---


* content
{:toc}



# 简介

对比String一些操作的性能


导图

![](https://i.imgur.com/FjFBDza.png)


# 1. 常量拼接

字符串常量/确定量拼接方法对比：

![](https://i.imgur.com/P3jVojv.png)


注意：常量字符串相加，编译器会直接优化。因此看到的时间和memory都是0

测试结果可能会受GC的影响，仅作为参考。


测试代码：


	public void constString() {
        // "+"
        final String TAG1 = "[const +]: ";
        long startMem = startRecordMemory(TAG1);
        long lastTimeNanos = System.nanoTime();
        for (int i = 0; i < LOOP; i++) {
            String str = S1 + S2 + S3 + S4;
        }
        System.out.println(diff(TAG1, lastTimeNanos));
        stopRecordMemory(TAG1, startMem);

        // "StringBuilder"
        final String TAG2 = "[const StringBuilder]: ";
        startMem = startRecordMemory(TAG2);
        lastTimeNanos = System.nanoTime();
        for (int i = 0; i < LOOP; i++) {
            StringBuilder buffer = new StringBuilder();
            buffer.append(S1);
            buffer.append(S2);
            buffer.append(S3);
            buffer.append(S4);
            String str = buffer.toString();
        }
        System.out.println(diff(TAG2, lastTimeNanos));
        stopRecordMemory(TAG2, startMem);

        // "concat"
        final String TAG3 = "[const concat]: ";
        startMem = startRecordMemory(TAG3);
        lastTimeNanos = System.nanoTime();
        for (int i = 0; i < LOOP; i++) {
            String str = S1.concat(S2).concat(S3).concat(S4);
        }
        System.out.println(diff(TAG3, lastTimeNanos));
        stopRecordMemory(TAG3, startMem);

        System.out.println("const string end =================");
    }


# 2. 动态拼接

字符串动态/不确定量拼接

![](https://i.imgur.com/NgNLLbS.png)


注意：

"+" 动态字符串时，编译器将“+”编译为StringBuilder实现

- new StringBuilder().append("one").append("two").toString()
- 如果表达式里有多个+号的话，后面相应也会多多几个StringBuilder.append的调用，最后才是toString方法。
- 每次拼接都会创建一个StringBuilder对象，并且还要调用toString()方法将其转换为字符串，这样性能就大大降低了。

测试源码：

    public void dynamicString() {
        dynamicPlus();
        dynamicStringBuilder();
        dynamicStringBuilderOptimized();
        dynamicConcat();
    }

    // good way for dynamic combine with StringBuilder and evaluated length
    private void dynamicStringBuilderOptimized() {
        final String TAG = "[StringBuilder & evaluated len]: ";
        long startMem = startRecordMemory(TAG);
        long lastTimeNanos = System.nanoTime();
        String[] strList = new String[LOOP];
        // 1. evaluate
        int evaluateLen = 0;
        for (int i = 0; i < LOOP; i++) {
            strList[i] = "test" + i + S1;
            evaluateLen += strList[i].length();
        }

        // 2. create StringBuilder with evaluated size
        StringBuilder stringBuilder = new StringBuilder(evaluateLen);
        for (int i = 0; i < LOOP; i++) {
            stringBuilder.append(strList[i]);
        }
        strList = null;

        // 3. get string
        System.out.println(stringBuilder.toString());

        // collection memory information
        System.out.println(diff(TAG, lastTimeNanos));
        stopRecordMemory(TAG, startMem);
    }

    // the worst way for dynamic combine: memory churn appeared with "+"
    private void dynamicPlus() {
        final String TAG = "[bad String +]: ";
        long startMem = startRecordMemory(TAG);
        long lastTimeNanos = System.nanoTime();
        String dynamic_temp = "";
        for (int i = 0; i < LOOP; i++) {
            dynamic_temp += "test_" + i + S1;
        }
        System.out.println(dynamic_temp);
        System.out.println(diff(TAG, lastTimeNanos));
        stopRecordMemory(TAG, startMem);
    }

    // normal way for dynamic combine with StringBuilder
    private void dynamicStringBuilder() {
        final String TAG = "[StringBuilder]: ";
        long startMem = startRecordMemory(TAG);

        long lastTimeNanos = System.nanoTime();
        StringBuilder stringBuilder = new StringBuilder(LOOP);
        for (int i = 0; i < LOOP; i++) {
            stringBuilder.append("test" + i + S1);
        }

        System.out.println(stringBuilder.toString());
        System.out.println(diff(TAG, lastTimeNanos));
        stopRecordMemory(TAG, startMem);
    }

    // normal way for dynamic combine with StringBuilder
    private void dynamicConcat() {
        final String TAG = "[Concat]: ";
        long startMem = startRecordMemory(TAG);

        long lastTimeNanos = System.nanoTime();
        String start = "test" + 0 + S1;
        for (int i = 1; i < LOOP; i++) {
            start = start.concat("test" + i + S1);
        }

        System.out.println(start);
        System.out.println(diff(TAG, lastTimeNanos));
        stopRecordMemory(TAG, startMem);
    }


注意下面的是动态拼接

String A = "aaa";
String B = "bbb";
String C = A + B;

# 3. String的创建

先来看下这几种方式的行为：

- String s = newString("1")，生成了常量池中的“1” 和堆空间中的字符串对象。

- s.intern()，这一行的作用是s对象去常量池中寻找后发现"1"已经存在于常量池中了。

- String s2 = "1"，这行代码是生成一个s2的引用指向常量池中的“1”对象。


注意：

问题一：下面创建了几个String对象?

String s1 = new String("s1") ; 

String s2 = new String("s1") ;

答案：3个。编译期Constant Pool中创建1个,运行期heap中创建2个.

问题二：intern()方法

- intern()方法在JDK1.6和JDK1.7中的作用

比如String s = new String("SEU_Calvin")，再调用s.intern()，此时返回值还是字符串"SEU_Calvin"，表面上看起来好像这个方法没什么用处。

但实际上，在JDK1.6中它做了个小动作：检查字符串池里是否存在"SEU_Calvin"这么一个字符串，如果存在，就返回池里的字符串；

如果不存在，该方法会把"SEU_Calvin"添加到字符串池中，然后再返回它的引用。然而在JDK1.7中却不是这样的，后面会讨论。

可以从下面的测试代码来体会下三者的效果 （测试环境1.8， JDK1.6以后常量池被放置在了堆空间，常量池位置的不同会影响intern()方法）

但是在JDK1.7中，常量池中不需要再存储一份对象了，可以直接存储堆中的引用。这份引用直接指向 s3 引用的对象，也就是说s3.intern() ==s3会返回true。


测试Code：

    private static void noteOne() {
        String s1 = new String("1");
        s1.intern(); // 常量池中已经有"1", 什么都不做
        String s2 = "1";
        System.out.println("s1 == s2 " + (s1 == s2)); // false

        String s3 = new String("1") + new String("1");
        s3.intern(); // 常量池中新增"11",并且它的ref = s3
        String s4 = "11";
        System.out.println("s3 == s4 " + (s3 == s4)); // true; JDK1.6及以下是false
    }

    private static void noteTwo(){
        String s1 = new String("1");
        String s2 = "1";
        s1.intern(); // 常量池中已经有"1", 什么都不做
        System.out.println("s1 == s2"  + s1 == s2); // false

        String s3 = new String("1") + new String("1");
        String s4 = "11";
        s3.intern(); // 常量池中已经有"11", 什么都不做
        System.out.println("s3 == s4 " + s3 == s4); // false
    }

    private static void noteThree() {
        String s1 = "HelloWorld";
        String s2 = new String("HelloWorld");
        String s3 = "Hello";
        String s4 = "World";
        String s5 = "Hello" + "World";
        String s6 = s3 + s4;

        System.out.println("s1 == s2 " + (s1 == s2)); // false
        System.out.println("s1 == s5 " + (s1 == s5)); // true
        System.out.println("s1 == s6 " + (s1 == s6)); // false
        System.out.println("s1 == s6 " + (s1 == s6.intern())); // true
        System.out.println("s1 == s6 " + (s2 == s2.intern())); // false
    }

# 4. String的截取

![](https://i.imgur.com/yTu2dnY.png)


测试源码：


	public static void start() {
        String originStr = getOriginStr();
        // split
        splitStr(originStr);
        // StringTokenizer
        stringTokenizerStr(originStr);
        // substring()
        substringStr(originStr);
    }

    private static void splitStr(String orgin) {
        String TAG = "[split]: ";
        long startMem = startRecordMemory(TAG);
        long lastTimeNanos = System.nanoTime();
        String[] result = orgin.split("\\.");
        System.out.println(TAG + "result " + result.length);
        System.out.println(diff(TAG, lastTimeNanos));
        stopRecordMemory(TAG, startMem);
        System.out.println();
    }

    private static void stringTokenizerStr(String origin) {
        String TAG = TAG = "[StringTokenizer]: ";
        long startMem = startRecordMemory(TAG);
        long lastTimeNanos = System.nanoTime();
        StringTokenizer token = new StringTokenizer(origin, ".");
        System.out.println(TAG + "result " + token.countTokens());
        System.out.println(diff(TAG, lastTimeNanos));
        stopRecordMemory(TAG, startMem);
        System.out.println();
    }

    private static void substringStr(String origin) {
        String TAG = "[substring]: ";
        long startMem = startRecordMemory(TAG);
        long lastTimeNanos = System.nanoTime();
        int len = origin.lastIndexOf(".");
        int k = 0, count = 0;
        //String sub;
        for (int i = 0; i <= len; i++) {
            if (origin.substring(i, i + 1).equals(".")) {
                if (count == 0) {
                    origin.substring(0, i);
                } else {
                    origin.substring(k + 1, i);
                    if (i == len) {
                        origin.substring(len + 1, origin.length());
                    }
                }
                k = i;
                count++;
            }
        }

        System.out.println(TAG + "result " + count);
        System.out.println(diff(TAG, lastTimeNanos));
        stopRecordMemory(TAG, startMem);
        System.out.println();
    }

    private static String getOriginStr() {
        final int len = 10000;
        StringBuilder result = new StringBuilder();
        Random random = new Random();
        for (int i = 0; i < len; i++) {
            result.append(random.nextInt(9)).append(random.nextInt(9)).append(random.nextInt(9)).append(".");
        }

        return result.toString();
    }


# 参考源码

concat源码

    /**
     * Concatenates the specified string to the end of this string.
     * <p>
     * If the length of the argument string is {@code 0}, then this
     * {@code String} object is returned. Otherwise, a
     * {@code String} object is returned that represents a character
     * sequence that is the concatenation of the character sequence
     * represented by this {@code String} object and the character
     * sequence represented by the argument string.<p>
     * Examples:
     * <blockquote><pre>
     * "cares".concat("s") returns "caress"
     * "to".concat("get").concat("her") returns "together"
     * </pre></blockquote>
     *
     * @param   str   the {@code String} that is concatenated to the end
     *                of this {@code String}.
     * @return  a string that represents the concatenation of this object's
     *          characters followed by the string argument's characters.
     */
    public String concat(String str) {
        int olen = str.length();
        if (olen == 0) {
            return this;
        }
        if (coder() == str.coder()) {
            byte[] val = this.value;
            byte[] oval = str.value;
            int len = val.length + oval.length;
            byte[] buf = Arrays.copyOf(val, len);
            System.arraycopy(oval, 0, buf, val.length, oval.length);
            return new String(buf, coder);
        }
        int len = length();
        byte[] buf = StringUTF16.newBytesFor(len + olen);
        getBytes(buf, 0, UTF16);
        str.getBytes(buf, len, UTF16);
        return new String(buf, UTF16);
    }

append源码

	public AbstractStringBuilder append(String str){
        //如果为null值，则把null作为字符串处理
        if(str==null) str = "null";
        int len = str.length();
        //字符长度为0，则返回本身
        if(len == 0) return this;
        int newCount = count +len;
        //追加后的字符数组长度是否超过当前值
        if(newCount > value.length()){
            //加长，并做数组拷贝
            expanCapacity(newCount);
        }
        //字符串复制到目标数组
        str.getChars(0, len, value, count);
        count = newCount;
        return this;
    }


# 总结

1. 常量拼接请使用方法1：“+”
2. 动态拼接建议使用方法3, 或2
3. 创建String，请使用方法1：直接赋值的方式。因为new String会额外在堆空间中的创建字符串对象
4. 合理使用intern()重用方法，过多得使用会导致 PermGen过度增长，出现OOM。JDK1.6及之前，常量池中会再存储一份对象。之后是可以直接存储堆中的引用。

备注测试结果会受GC的影响，每次都有些许波动

# Reference

- [1] [Android 性能优化之String篇](http://blog.csdn.net/vfush/article/details/53038437) 
- [2] [Java 性能优化之 String 篇](https://www.ibm.com/developerworks/cn/java/j-lo-optmizestring/)
- [3] [性能优化 - 目录](http://vivianking6855.github.io/2018/01/24/Android-optimization-index/)
- [4] [Java技术——你真的了解String类的intern()方法吗](https://blog.csdn.net/seu_calvin/article/details/52291082)