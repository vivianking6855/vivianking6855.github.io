---
layout: post
title: 2017-1-24 RemoteViews
date: 2017-1-24
excerpt: "RemoteViews"
tags: [Android]
comments: true
---

## 一、RemoteViews含义

RemoteViews:远程View

- 和远程Service一样，RemoteViews表示一个View的结构，可以在其他进程中显示。
- 由于它在其他进程中显示，为了能够更新它的界面，RemoteViews提供了一组基础的操作作用于跨进程更新它的界面。
- RemoteViews在Android中的使用场景有两种
    - Notification通知栏
    - 桌面小部件


## 二、RemoteViews的基础概念

- PendingIntent
    - 待定，即将发生的Intent
    - 在将来的某个不确定的时刻发生，不同于Intent的立刻发生
    - 典型使用场景是给RemoteViews添加点击事件，因为RemoteViews是在远程使用，没办法像其他View一样添加onClickLisenter事件
    - 三种待定意图
        - 启动Activity
        - 启动Service
        - 发送广播

## 四、RemoteViews内部机制

- 应用中每调一次set方法，RemoteViews中就会添加一个对应的Action对象
- 用NotificationManager和AppWidgetManager来提交我们的更新时，Action对象就会传输到远程进程并在远程进程中依次执行
- 远程进程通过RemoteViews的apply方法来进行View的更新操作，RemoteViews的apply方法内部则会去遍历所有的Actions对象并调用

![](http://i.imgur.com/oGJR6JL.jpg)


## 五、RemoteViews实践

模拟的通知栏效果并实现跨进程的UI更新,逻辑如下：

- 有A,B 2个Activity分别运行在不同的进程中，一个A(MainActivity),一个B(ClientActivity)
- A接受通知，模拟通知栏的效果
- B可以不停的地发送通知栏消息（模拟消息）。通知方案选用Broadcast（系统是用Binder）

核心Code:

    === B send msg ===

    public void onButtonClick(View v) {
        RemoteViews remoteViews = new RemoteViews(getPackageName(), R.layout.layout_simulated_notification);
        DateFormat df = new SimpleDateFormat("HH:mm:ss");
        remoteViews.setTextViewText(R.id.msg, "msg from process : " + Process.myPid() + "\r\n"
                + df.format(new Date())
                + "\r\n点击开启B");

        // set pending intent, when click textview
        PendingIntent openActivityPendingIntent = PendingIntent.getActivity(
                this, 0, new Intent(this, ClientActivity.class), PendingIntent.FLAG_UPDATE_CURRENT);
        remoteViews.setOnClickPendingIntent(R.id.notify_layout, openActivityPendingIntent);
        Intent intent = new Intent(Const.REMOTE_ACTION);
        intent.putExtra(Const.EXTRA_REMOTE_VIEWS, remoteViews);
        sendBroadcast(intent);
        finish();
    }
    
    
    === A receive ===
    
     // receive remote view broadcast and update ui
    private LinearLayout mRemoteViewsContent;
    private BroadcastReceiver mRemoteViewsReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            RemoteViews remoteViews = intent
                    .getParcelableExtra(Const.EXTRA_REMOTE_VIEWS);
            if (remoteViews != null) {
                updateUI(remoteViews);
            }
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initView();
    }

    private void initView() {
        mRemoteViewsContent = (LinearLayout) findViewById(R.id.remote_views_content);
        IntentFilter filter = new IntentFilter(Const.REMOTE_ACTION);
        registerReceiver(mRemoteViewsReceiver, filter);

        TextView hint = (TextView) findViewById(R.id.hint);
        hint.setText("logic:\r\n"
                + "1. 有A，B 2个Activity运行在不同的进程\r\nA(MainActivity),B(ClientActivity)\r\n"
                + "2. A接受通知，模拟通知栏的效果。\r\n"
                + "3. B不停的地发送通知栏消息（模拟消息）\r\n\r\n"
                + "MainActivity（A) process : " + Process.myPid());
    }

    private void updateUI(RemoteViews remoteViews) {
        // if A,B belongs to the same application use this way
        // View view = remoteViews.apply(this, mRemoteViewsContent);
        // if A,B belongs to different application,you may use this way
        int layoutId = getResources().getIdentifier("layout_simulated_notification", "layout", getPackageName());
        View view = getLayoutInflater().inflate(layoutId, mRemoteViewsContent, false);
        remoteViews.reapply(this, view);

        mRemoteViewsContent.addView(view);
    }



[github Code](https://github.com/vivianking6855/android-advanced/tree/remoteviews)



<br/>
<br/>


> [《Android开发艺术探索》](http://download.csdn.net/download/jsntghf/9602444)

> [《Android开发艺术探索》 Github Code](https://github.com/singwhatiwanna/android-art-res)
