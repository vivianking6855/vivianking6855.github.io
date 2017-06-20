---
layout: post
title: Android Canvas
date: 2017-6-19
excerpt: "Android Canvas"
categories: Android
tags: [Android 进阶]
comments: true
lefttrees: true
---

* content
{:toc}



# 简介

Canvas画布，可以在上面画很多东西。它提供了一套画图的API。通常自定义控件的时候用到。

## 自定义控件注意事项

自定义控件需要注意：

- 能够用Android基础控件解决的问题就尽量用基础控件，其次是用基础控件的组合。如果是确实有必要自定义才考虑自定义。

- 自定义的控件，既需要耗费较长的开发时间，又不一定能保证有基础控件那么高的效率（基础控件都是谷歌优化过了的）。

自定义控件可以参考这篇[Android-CustomizedView](http://vivianking6855.github.io/2016/11/09/Android-CustomizedView/)

## View和SurfaceView

- 如果是简单的处理，帧率比较小的动画，比如说象棋游戏之类的。可以直接用View的Canvas画
- 如果是大型的游戏或高品质动画建议使用SurfaceView的画布来做。因为SurfaceView中定义一个专门的线程来完成画图工作，不需要等待View的刷图，提高性能。

# Canvas类常用方法

- drawRect(RectF rect, Paint paint) 绘制区域，参数一为RectF一个区域 
- drawPath(Path path, Paint paint) 绘制路径
- drawBitmap(Bitmap bitmap, Rect src, Rect dst, Paint paint)绘图，当原始Rect不等于目标Rect时性能将会有大幅损失
    - bitmap: Bitmap对象
    - src： Bitmap源区域
    - dst: 目标区域,在canvas的位置和大小
    - paint: 画笔
- drawLine(float startX, float startY, float stopX, float stopY, Paintpaint) 画线
    - startX, startY, stopX, stopY 其实点坐标
- drawPoint(float x, float y, Paint paint) 画点
- drawText(String text, float x, floaty, Paint paint) 
- drawOval(RectF oval, Paint paint) 画椭圆
- drawCircle(float cx, float cy, float radius,Paint paint) 画圆
    - cx, cy 圆心
    - radius 半径
- drawArc(RectF oval, float startAngle, float sweepAngle, boolean useCenter, Paint paint) 画弧
    - oval：圆弧所在的椭圆对象。
    - startAngle：圆弧的起始角度。
    - sweepAngle：圆弧的角度。
    - useCenter：是否显示半径连线，true表示显示圆弧与圆心的半径连线，false表示不显示。

每个方法中都有一个Paint参数，用来定义Canvas上的画笔、画刷、颜料等等。Paint类常用方法:

- setARGB(int a, int r, int g, int b) Paint对象颜色，argb，a是alpha透明值
- setAlpha(int a) 设置alpha不透明度，范围为0~255
- setAntiAlias(boolean aa) 是否抗锯齿
- setColor(int color)  设置颜色，可以用Color类中的一些常见颜色
- setTextScaleX(float scaleX)  设置文本缩放倍数，1.0f为原始
- setTextSize(float textSize)  设置字体大小
- setUnderlineText(booleanunderlineText)  设置下划线
- setStrokeWidth (float width); 设置线宽
- setStyle (Paint.Style style) 设置Style，例如setStyle(Paint.Style.STROKE)是设置空心

# 普通View的canvas画图

基础的Canvas用法效果图

![](http://i.imgur.com/j7pqlhl.png)

核心Code

    public class CanvasBasicUsage extends View {
        private final static String TAG = CircleProgress.class.getSimpleName();
    
        private Context mContext;
    
        // view paints
        private Paint mTextPaint;
        private Paint mCirPaint;
        private Paint mLinePaint;
        private Paint mArcPaint;
        private RectF mArcRect = new RectF(150, 300, 300, 450);
        private Paint mShapePaint;
        private Path mShapePath;
        private RectF mRoundRect = new RectF(350, 300, 450, 400);
        private Paint mPointPaint;
        private Bitmap drawBitmap;
        private Paint mBezierPaint;
        private Path mBezierPath;
    
        public CanvasBasicUsage(Context context, AttributeSet attrs) {
            super(context, attrs);
            mContext = context;
            init();
        }
    
        private void init() {
            initPaints();
        }
    
        private void initPaints() {
            if (mTextPaint == null) {
                mTextPaint = new Paint();
                mTextPaint.setColor(ContextCompat.getColor(mContext, R.color.light_green));
                mTextPaint.setTextSize(50);
            }
            if (mCirPaint == null) {
                mCirPaint = new Paint();
                final int[] arcColor = new int[]{ContextCompat.getColor(mContext, R.color.firstColor),
                        ContextCompat.getColor(mContext, R.color.secondColor)};
                LinearGradient gradient = new LinearGradient(150, 300, 300, 450, arcColor, null, LinearGradient.TileMode.CLAMP);
                mCirPaint.setAntiAlias(true); // 消除锯齿
                mCirPaint.setShader(gradient);
            }
            if (mLinePaint == null) {
                mLinePaint = new Paint();
                mLinePaint.setColor(ContextCompat.getColor(mContext, R.color.deep_green));
                mLinePaint.setStyle(Paint.Style.STROKE); // 设置空心
                mLinePaint.setStrokeWidth(5);
            }
            if (mArcPaint == null) {
                mArcPaint = new Paint();
                mArcPaint.setStyle(Paint.Style.STROKE); // 设置空心
                mArcPaint.setStrokeWidth(5);
                mArcPaint.setAntiAlias(true); // 消除锯齿
                final int[] arcColor = new int[]{ContextCompat.getColor(mContext, R.color.light_green),
                        ContextCompat.getColor(mContext, R.color.deep_green)};
                LinearGradient gradient = new LinearGradient(150, 300, 300, 450, arcColor, null, LinearGradient.TileMode.CLAMP);
                mArcPaint.setShader(gradient);
            }
            if (mShapePaint == null) {
                mShapePaint = new Paint();
                mShapePaint.setStyle(Paint.Style.STROKE); // 设置空心
                final int[] shapeColor = new int[]{ContextCompat.getColor(mContext, R.color.firstColor),
                        ContextCompat.getColor(mContext, R.color.deep_green)};
                LinearGradient gradient = new LinearGradient(150, 300, 300, 450, shapeColor, null, LinearGradient.TileMode.CLAMP);
                mShapePaint.setShader(gradient);
                mShapePaint.setStrokeWidth(5);
                // set path
                mShapePath = new Path();
                mShapePath.moveTo(500, 200);
                mShapePath.lineTo(600, 200);
                mShapePath.lineTo(650, 250);
                mShapePath.lineTo(650, 230);
                mShapePath.lineTo(500, 200);
                mShapePath.close();
            }
            if (mPointPaint == null) {
                mPointPaint = new Paint();
                mPointPaint.setStyle(Paint.Style.FILL);
                mPointPaint.setStrokeWidth(10);
                mPointPaint.setColor(ContextCompat.getColor(mContext, R.color.deep_green));
            }
            if (drawBitmap == null) {
                drawBitmap = BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher_round);
            }
            if(mBezierPath == null){
                mBezierPaint = new Paint();
                mBezierPaint.setStyle(Paint.Style.STROKE);
                mBezierPaint.setStrokeWidth(5);
                mBezierPaint.setColor(ContextCompat.getColor(mContext, R.color.deep_green));
                mBezierPath=new Path();
                mBezierPath.moveTo(200, 580);// path start point
                //mBezierPath.quadTo(250, 420, 400, 500); // 二阶贝塞尔曲线
                mBezierPath.cubicTo(250, 420, 400, 650, 500, 500); // 三阶贝塞尔曲线，设置控制点和结束点（最后一个是结束点）
            }
        }
    
        @Override
        protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
            super.onLayout(changed, left, top, right, bottom);
        }
    
        @Override
        protected void onDraw(Canvas canvas) {
            super.onDraw(canvas);
            // draw circle
            canvas.drawText("Circle", 10, 80, mTextPaint);
            canvas.drawText("Circle", 10, 80, mTextPaint);
            canvas.drawCircle(220, 70, 40, mTextPaint);
            canvas.drawCircle(400, 80, 80, mCirPaint);
            // draw points
            canvas.drawPoints(new float[]{500, 80, 550, 80, 600, 80}, mPointPaint);
    
            // draw line
            canvas.drawText("Line", 10, 220, mTextPaint);
            canvas.drawLine(150, 200, 250, 200, mLinePaint);
            canvas.drawLine(350, 200, 450, 200, mLinePaint);
            // multi-line shape
            canvas.drawPath(mShapePath, mShapePaint);
    
            // draw arc
            canvas.drawText("Arc", 10, 350, mTextPaint);
            canvas.drawArc(mArcRect, 180, 180, false, mArcPaint);
            // draw round rect
            canvas.drawRoundRect(mRoundRect, 20, 15, mCirPaint);
            // draw bitmap
            canvas.drawBitmap(drawBitmap, 500, 300, mPointPaint);
    
            // bezier path
            canvas.drawText("Bezier", 10, 500, mTextPaint);
            canvas.drawPath(mBezierPath, mBezierPaint);
        }
    
        @Override
        protected void onDetachedFromWindow() {
            super.onDetachedFromWindow();
            if (drawBitmap != null && !drawBitmap.isRecycled()) {
                drawBitmap.recycle();
                drawBitmap = null;
            }
        }
    }

自定义progress效果图

![](http://i.imgur.com/u6iAIDX.jpg)

核心Code

MainActivity --------------------------------------

    private void setCircleProgress() {
            Thread td = new Thread(new Runnable() {
                @Override
                public void run() {
                    dealCircleProgress();
                }
            });
            td.start();
        }
    
        /**
         * position 0~360
         * speed 1~100
         */
        private void dealCircleProgress() {
            int pos = 0;
            while (!isStop) {
                // count position 0~360
                pos++;
                pos = pos % 360;
                // send msg to update ui
                Message msg = mHandler.obtainMessage(MSG_CIRCLE, pos);
                mHandler.sendMessage(msg);
                // sleep speed time (1~100)
                SystemClock.sleep(100 / speed);
            }
        }

CircleProgress-----------------------------------------------

    /**
     * Created by vivian on 2017/6/16.
     */
    
    public class CircleProgress extends View {
        private final static String TAG = CircleProgress.class.getSimpleName();
    
        private Context mContext;
    
        // view paints
        private Paint firstPaint; // first circle paint
        private Paint secondPaint; // second circle paint
        private int firstColor; // first circle color
        private int secondColor;// second circle color
        private int circleWidth; // 设置圆环的宽度
        int centre; // 圆心
        int radius; // 半径
        RectF ovalRect;
    
        // progress 0-360
        private int progress = 0;
    
        public CircleProgress(Context context, AttributeSet attrs) {
            super(context, attrs);
    
            mContext = context;
            init(attrs);
        }
    
        private void init(AttributeSet attrs) {
            setAttrs(attrs);
            initPaints();
        }
    
        private void setAttrs(AttributeSet attrs) {
            try {
                TypedArray typedArray = mContext.obtainStyledAttributes(attrs, R.styleable.CircleProgress);
                firstColor = typedArray.getColor(R.styleable.CircleProgress_firstColor, Color.GREEN);
                secondColor = typedArray.getColor(R.styleable.CircleProgress_secondColor, Color.GREEN);
                circleWidth = typedArray.getDimensionPixelSize(R.styleable.CircleProgress_circleWidth, SizeUtils.dp2px(mContext, 30));
                typedArray.recycle();
            } catch (Exception ex) {
                Log.w(TAG, "setAttrs ex", ex);
            }
        }
    
        private void initPaints() {
            if (firstPaint == null) {
                firstPaint = new Paint();
                firstPaint.setColor(firstColor);
                firstPaint.setStrokeWidth(circleWidth);
                firstPaint.setAntiAlias(true); // 消除锯齿
                firstPaint.setStyle(Paint.Style.STROKE); // 设置空心
            }
    
            if (secondPaint == null) {
                secondPaint = new Paint();
                secondPaint.setColor(secondColor);
                secondPaint.setStrokeWidth(circleWidth); // 设置圆环的宽度
                secondPaint.setAntiAlias(true); // 消除锯齿
                secondPaint.setStyle(Paint.Style.STROKE); // 设置空心
            }
        }
    
        @Override
        protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
            super.onLayout(changed, left, top, right, bottom);
    
            setPositionData();
        }
    
        private void setPositionData() {
            centre = getWidth() / 2; // 获取圆心的x坐标
            radius = centre - circleWidth / 2;// 半径
            // 用于定义的圆弧的形状和大小的界限
            ovalRect = new RectF(centre - radius,
                    centre - radius,
                    centre + radius,
                    centre + radius);
        }
    
        @Override
        protected void onDraw(Canvas canvas) {
            canvas.drawCircle(centre, centre, radius, firstPaint);
            canvas.drawArc(ovalRect, -90, progress, false, secondPaint);
            //Log.d(TAG,"onDraw: centre = " + centre + "; radius = " + radius);
        }
    
        /**
         * it must run in ui thread
         *
         * @param value
         */
        public void setProgress(int value) {
            try {
                if (progress == value) {
                    Log.w(TAG, "setProgress not changed, since same value");
                    return;
                }
    
                progress = value;
                if (progress >= 360) {
                    progress = 0;
                }
    
                invalidate();
            } catch (Exception ex) {
                Log.w(TAG, "setProgress ex", ex);
            }
        }
    
    }


# SurfaceView的canvas

SurfaceView中定义一个专门的线程来完成画图工作，应用程序不需要等待View的刷图，提高性能。主要用在游戏，高品质动画方面的画图。

效果图

圆形的波形图动画

![](http://i.imgur.com/2Mnuzq1.png)

![](http://i.imgur.com/dIzjU1k.png)


核心Code

    
    /**
     *
     */
    public class VisualizerViewSurface extends SurfaceView implements
            SurfaceHolder.Callback,
            Visualizer.OnDataCaptureListener {
        private final static String TAG = VisualizerViewSurface.class.getSimpleName();
    
        // if no data item count >= 35 means no data. not play now
        private final static int COUNT_NO_DATA = 35;
    
        // Bezier params
        private final static float a = 0.25f;
        private final static float b = 0.25f;
        private PointF[] wavePoints; // big wave Bezier points
        private int[] controlPoints = new int[4]; // Bezier four control points
    
        // draw paints
        private Paint shortWavePaint;
        private Paint bigWavePaint;
        private Paint mFadePaint;
        private byte[] points; // short wave points
        private byte[] bigwavePoints; // big wave points
        private int divider = 4;  // big wave points percent, only use 1/4 of all points
        private int shortRadius;  // short wave radius
        private int bigWaveRadius; // big wave radius
        private int maxWaveHeight; // short wave max wave height
        private int bigWaveHeight; // big wave max wave height
        // PorterDuff.Mode used to clear last draw content
        private PorterDuffXfermode clear;
        private PorterDuffXfermode src;
        // half size of view, used for canvas.translate(halfWidth, halfHeight);
        private int halfWidth;
        private int halfHeight;
        private float[] mFFTPoints; // fft point x,y [x1,y1,x2,y2......]
        private Path path; // short wave path
        private Path bigPath; // big wave path
        private float[] cartPoint; // temp points
    
        // handler msg
        private HandlerThread handlerThread;
        private final static int MSG_DRAW = 1;
    
        private SurfaceHolder holder;
        private Visualizer mVisualizer;
        private IAudioPlayListener audioPlayListener;
    
        // music play or pause listener, user to update ui status
        public interface IAudioPlayListener {
            void onAudioPlay();
    
            void onAudioStop();
        }
    
        public void setAudioPlayListener(IAudioPlayListener listener) {
            this.audioPlayListener = listener;
        }
    
        private class DrawHandler extends Handler {
    
            DrawHandler(Looper looper) {
                super(looper);
            }
    
            @Override
            public void handleMessage(Message msg) {
                if (msg.what == MSG_DRAW) {
                    drawVisualizer((byte[]) msg.obj);
                }
            }
        }
    
        private DrawHandler handler;
    
        public VisualizerViewSurface(Context context) {
            this(context, null);
        }
    
        public VisualizerViewSurface(Context context, AttributeSet attrs) {
            super(context, attrs);
            holder = getHolder();
            setZOrderOnTop(true);
            holder.setFormat(PixelFormat.TRANSLUCENT);
            holder.addCallback(this);
            if (shortWavePaint == null) {
                shortWavePaint = new Paint();
                shortWavePaint.setAntiAlias(true);
                shortWavePaint.setStrokeWidth(SizeUtils.dp2px(context, 1));
                shortWavePaint.setStyle(Paint.Style.STROKE);
                shortWavePaint.setStrokeCap(Paint.Cap.ROUND);
                shortWavePaint.setPathEffect(new CornerPathEffect(30));
            }
            if (bigWavePaint == null) {
                bigWavePaint = new Paint(shortWavePaint);
                bigWavePaint.setStyle(Paint.Style.STROKE);
                bigWavePaint.setStrokeWidth(SizeUtils.dp2px(context, 1));
            }
            path = new Path();
            bigPath = new Path(path);
            clear = new PorterDuffXfermode(PorterDuff.Mode.CLEAR);
            src = new PorterDuffXfermode(PorterDuff.Mode.SRC);
            mFadePaint = new Paint();
            cartPoint = new float[2];
        }
    
        private int noDataTimes = 0;
    
        public void initShader(Context context) {
            int angle = 180;
            int startX = (int) (Math.cos(Math.toRadians(angle)) * (bigWaveRadius + bigWaveHeight));
            int startY = -(int) (Math.sin(Math.toRadians(angle)) * (bigWaveRadius + bigWaveHeight));
            int endX = -startX;
            int endY = -startY;
    
            final int[] bigColor = new int[]{ContextCompat.getColor(context, R.color.bigWaveStart),
                    ContextCompat.getColor(context, R.color.bigWaveEnd)};
            final int[] shotColor = new int[]{ContextCompat.getColor(context, R.color.shortWaveStart),
                    ContextCompat.getColor(context, R.color.shortWaveEnd)};
            LinearGradient bigWaveGradient = new LinearGradient(startX, startY, endX, endY, bigColor, null, LinearGradient.TileMode.CLAMP);
            LinearGradient shortWaveGradient = new LinearGradient(startX, startY, endX, endY, shotColor, null, LinearGradient.TileMode.CLAMP);
            bigWavePaint.setShader(bigWaveGradient);
            shortWavePaint.setShader(shortWaveGradient);
        }
    
        private void drawVisualizer(byte[] bytes) {
            if (bytes == null) {
                return;
            }
            int temp = 0;
            //shortwave
            for (int i = 0, j = 0; j < points.length && i < bytes.length; ) {
                points[j] = (byte) Math.hypot(bytes[i], bytes[i + 1]);
                temp = temp + points[j];
                i += 2;
                j++;
            }
            if (temp <= 0) {
                // no data count
                if (noDataTimes > COUNT_NO_DATA) {
                    noDataTimes = COUNT_NO_DATA + 1;
                    if (audioPlayListener != null) {
                        audioPlayListener.onAudioStop();
                    }
                }
                noDataTimes++;
                // clear canvas
                Canvas canvas = holder.lockCanvas();
                if (canvas == null) {
                    return;
                }
                mFadePaint.setXfermode(clear);
                canvas.drawPaint(mFadePaint);
                mFadePaint.setXfermode(src);
                holder.unlockCanvasAndPost(canvas);
                return;
            } else {
                noDataTimes = 0;
                if (audioPlayListener != null) {
                    audioPlayListener.onAudioPlay();
                }
            }
    
            // shot wave path
            setShotWavePath();
            // big wave path
            setBigWavePoint(bytes);
            setBezierCurvePath();
    
            Canvas canvas = holder.lockCanvas();
            if (canvas == null) {
                return;
            }
            // clear canvas
            mFadePaint.setXfermode(clear);
            canvas.drawPaint(mFadePaint);
            mFadePaint.setXfermode(src);
            // draw two waves
            canvas.save();
            // 0,0 move to (halfWidth, halfHeight)
            canvas.translate(halfWidth, halfHeight);
            canvas.drawPath(bigPath, bigWavePaint);
            canvas.rotate(90);
            canvas.drawPath(path, shortWavePaint);
            canvas.restore();
            holder.unlockCanvasAndPost(canvas);
        }
    
        private void setShotWavePath() {
            int scale = 1;
            int halfLength = points.length / 2;
    
            // cartPoint is scaled height based on maxWaveHeight / 128
            //move to first line start point
            cartPoint[0] = 0;
            cartPoint[1] = scale * (points[1] < 0 ? 0 : points[1]) * maxWaveHeight / 128;
            toPolar(cartPoint, shortRadius);
            mFFTPoints[0] = cartPoint[0];
            mFFTPoints[1] = cartPoint[1];
            // shot wave -- path
            path.reset();
            path.moveTo(cartPoint[0], cartPoint[1]);
            for (int i = 1; i < points.length; i++) {
                if (i > halfLength) {
                    scale = 3;
                }
                cartPoint[0] = (float) i / (points.length);
                cartPoint[1] = scale * (points[i] < 0 ? 0 : points[i]) * maxWaveHeight / 128;
    
                toPolar(cartPoint, shortRadius);
                mFFTPoints[i * 2] = cartPoint[0];
                mFFTPoints[i * 2 + 1] = cartPoint[1];
                path.lineTo(cartPoint[0], cartPoint[1]);
            }
            //connect circle header and tail
            path.lineTo(mFFTPoints[0], mFFTPoints[1]);
        }
    
        private void setBigWavePoint(byte[] bytes) {
            //big wave
            bigPath.reset();
            for (int i = 0, j = 0; j < bigwavePoints.length && i < bytes.length; ) {
                bigwavePoints[j] = (byte) Math.hypot(bytes[i], bytes[i + 1]);
                i += 2;
                j++;
            }
    
            for (int i = 0; i < bigwavePoints.length; i++) {
                cartPoint[0] = (float) i / (bigwavePoints.length);
                cartPoint[1] = (bigwavePoints[i] < 0 ? 0 : bigwavePoints[i]) * (bigWaveHeight) / 128;
                if (cartPoint[0] >= 0.5) {// if index > half, scale set to 1.2f, height scale will be larger
                    cartPoint[1] = cartPoint[1] * 1.2f;
                }
                toPolar(cartPoint, bigWaveRadius);
                wavePoints[i].x = cartPoint[0];
                wavePoints[i].y = cartPoint[1];
            }
        }
    
        private void setBezierCurvePath() {
            int i = 0;
            controlPoints[0] = (int) (wavePoints[i].x + (wavePoints[i + 1].x - wavePoints[wavePoints.length - 1].x) * a);
            controlPoints[1] = (int) (wavePoints[i].y + (wavePoints[i + 1].y - wavePoints[wavePoints.length - 1].y) * a);
            controlPoints[2] = (int) (wavePoints[i + 1].x - (wavePoints[i + 2].x - wavePoints[i].x) * b);
            controlPoints[3] = (int) (wavePoints[i + 1].y - (wavePoints[i + 2].y - wavePoints[i].y) * b);
    
            //first wave
            drawBigWaveLine((int) wavePoints[i].x, (int) wavePoints[i].y, controlPoints[0],
                    controlPoints[1], controlPoints[2], controlPoints[3], (int) wavePoints[i + 1].x, (int) wavePoints[i + 1].y);
    
            //middle wavies
            for (i = 1; i < wavePoints.length - 2; i++) {
                controlPoints = calculateControlPoint(wavePoints, i);
                drawBigWaveLine((int) wavePoints[i].x, (int) wavePoints[i].y, controlPoints[0],
                        controlPoints[1], controlPoints[2], controlPoints[3], (int) wavePoints[i + 1].x, (int) wavePoints[i + 1].y);
            }
    
            //we draw the last two line out of the loop because the loop can not draw the
            //last two point because of the logic in calculateControlPoint and to avoid add 'if'in
            //calculateControlPoint
            i = wavePoints.length - 2;
            controlPoints[0] = (int) (wavePoints[i].x + (wavePoints[i + 1].x - wavePoints[i - 1].x) * a);
            controlPoints[1] = (int) (wavePoints[i].y + (wavePoints[i + 1].y - wavePoints[i - 1].y) * a);
            controlPoints[2] = (int) (wavePoints[i + 1].x - (wavePoints[0].x - wavePoints[i].x) * b);
            controlPoints[3] = (int) (wavePoints[i + 1].y - (wavePoints[0].y - wavePoints[i].y) * b);
    
            drawBigWaveLine((int) wavePoints[i].x, (int) wavePoints[i].y, controlPoints[0],
                    controlPoints[1], controlPoints[2], controlPoints[3], (int) wavePoints[i + 1].x, (int) wavePoints[i + 1].y);
    
            //connect the tail and header
            i = wavePoints.length - 1;
            controlPoints[0] = (int) (wavePoints[i].x + (wavePoints[0].x - wavePoints[i - 1].x) * a);
            controlPoints[1] = (int) (wavePoints[i].y + (wavePoints[0].y - wavePoints[i - 1].y) * a);
            controlPoints[2] = (int) (wavePoints[0].x - (wavePoints[1].x - wavePoints[i].x) * b);
            controlPoints[3] = (int) (wavePoints[0].y - (wavePoints[1].y - wavePoints[i].y) * b);
            drawBigWaveLine((int) wavePoints[i].x, (int) wavePoints[i].y, controlPoints[0], controlPoints[1],
                    controlPoints[2], controlPoints[3], (int) wavePoints[0].x, (int) wavePoints[0].y);
        }
    
        private static final float PER_BIGWAVE_LINE_COUNT = 15;
    
        /**
         * calculator big path base on bezier
         *
         * @param x0
         * @param y0
         * @param x1
         * @param y1
         * @param x2
         * @param y2
         * @param x3
         * @param y3
         */
        private void drawBigWaveLine(int x0, int y0, int x1, int y1, int x2, int y2, int x3, int y3) {
            for (int p = 0; p < PER_BIGWAVE_LINE_COUNT - 1; p++) {
                Point point = CalculateBezierPointForCubic(p / PER_BIGWAVE_LINE_COUNT, x0, y0, x1, y1, x2, y2, x3, y3);
                int br = (int) Math.hypot(point.x, point.y);
                bigPath.moveTo(point.x * bigWaveRadius / br, point.y * bigWaveRadius / br);
                bigPath.lineTo(point.x, point.y);
                point = CalculateBezierPointForCubic((p + 1) / PER_BIGWAVE_LINE_COUNT, x0, y0, x1, y1, x2, y2, x3, y3);
                br = (int) Math.hypot(point.x, point.y);
                bigPath.moveTo(point.x * bigWaveRadius / br, point.y * bigWaveRadius / br);
                bigPath.lineTo(point.x, point.y);
            }
        }
    
        @Override
        protected void onSizeChanged(int w, int h, int oldw, int oldh) {
            super.onSizeChanged(w, h, oldw, oldh);
            halfWidth = w / 2;
            halfHeight = h / 2;
            ViewGroup parent = (ViewGroup) getParent();
            bigWaveHeight = Math.min(parent.getHeight(), parent.getWidth()) / 2 - bigWaveRadius;
        }
    
        public void setBigWaveRadius(int radius) {
            this.bigWaveRadius = radius;
        }
    
        public void setShortWaveRadius(int radius) {
            shortRadius = radius;
        }
    
        public void setMaxWaveHeight(int maxWaveHeight) {
            this.maxWaveHeight = maxWaveHeight;
        }
    
        @Deprecated
        public void setBigWaveHeight(int height) {
            bigWaveHeight = height;
        }
    
        /**
         * calculator absolute x,y pixel points (cartesian 笛卡尔坐标) and set to cartPoint (same input)
         *
         * @param cartesian
         * @param radius
         */
        private void toPolar(float[] cartesian, int radius) {
            double angle = (cartesian[0]) * 2 * Math.PI;
            int x = (int) ((radius + cartesian[1]) * Math.sin(angle));
            int y = (int) ((radius + cartesian[1]) * Math.cos(angle));
            cartesian[0] = x;
            cartesian[1] = y;
        }
    
        private int[] calculateControlPoint(PointF[] points, int i) {
            float a = 0.25f;
            float b = 0.25f;
            int[] cps = new int[4];
    
            cps[0] = (int) (points[i].x + (points[i + 1].x - points[i - 1].x) * a);
            cps[1] = (int) (points[i].y + (points[i + 1].y - points[i - 1].y) * a);
            cps[2] = (int) (points[i + 1].x - (points[i + 2].x - points[i].x) * b);
            cps[3] = (int) (points[i + 1].y - (points[i + 2].y - points[i].y) * b);
    
            return cps;
        }
    
        @Override
        public void surfaceCreated(SurfaceHolder holder) {
            handlerThread = new HandlerThread("visualizer");
            handlerThread.start();
            handler = new DrawHandler(handlerThread.getLooper());
        }
    
        @Override
        public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
        }
    
        @Override
        public void surfaceDestroyed(SurfaceHolder holder) {
            handler.removeMessages(MSG_DRAW);
            handlerThread.quit();
        }
    
        private void initVisualizer() {
            if (mVisualizer == null) {
                try {
                    mVisualizer = VisualizeManager.getInstance();
                    mVisualizer.setEnabled(false);
                    mVisualizer.setCaptureSize(Visualizer.getCaptureSizeRange()[1]);
                    mVisualizer.setDataCaptureListener(this, Visualizer.getMaxCaptureRate() / 2, false, true);
                    // Visualizer.getCaptureSizeRange()[1] = 1024,
                    // shot wave count is 1/divider = 1/4; /2 means 傅里叶变化的取值规则
                    points = new byte[(Visualizer.getCaptureSizeRange()[1] / divider) / 2];
                    mFFTPoints = new float[points.length * 2];
                    // big wave count is 1/divider*8;
                    bigwavePoints = new byte[(Visualizer.getCaptureSizeRange()[1] / (divider * 8)) / 2];
                    wavePoints = new PointF[bigwavePoints.length];
                    for (int i = 0; i < wavePoints.length; i++) {
                        wavePoints[i] = new PointF(0, 0);
                    }
                } catch (RuntimeException e) {
                    Log.w(TAG, "Visualizer Init fail", e);
                }
            }
        }
    
        public void setVisualizer(boolean control) {
            if (control) {
                initVisualizer();
            }
            try {
                if (mVisualizer != null && mVisualizer.getEnabled() != control) {
                    mVisualizer.setEnabled(control);
                    if (!control) {
                        Thread.sleep(100);
                        VisualizeManager.destoryVisualizer();
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
    
            if (!control) {
                mVisualizer = null;
            }
        }
    
        @Override
        public void onWaveFormDataCapture(Visualizer visualizer, byte[] waveform, int samplingRate) {
        }
    
        @Override
        public void onFftDataCapture(Visualizer visualizer, byte[] fft, int samplingRate) {
            if (handlerThread != null && !handlerThread.isAlive() && handler != null) {
                return;
            }
            Message msg = handler.obtainMessage(MSG_DRAW);
            msg.obj = fft;
            handler.sendMessage(msg);
        }
    
        /**
         * B(t) = P0 * (1-t)^3 + 3 * P1 * t * (1-t)^2 + 3 * P2 * t^2 * (1-t) + P3 * t^3, t ∈ [0,1]
         *
         * @param t  曲线长度比例
         * @param x0 y0 起始点
         * @param x1 y1 控制点1
         * @param x2 y2 控制点2
         * @param x3 y3 终止点
         * @return t对应的点
         */
        private Point CalculateBezierPointForCubic(float t, int x0, int y0, int x1, int y1, int x2, int y2, int x3, int y3) {
            Point point = new Point();
            float temp = 1 - t;
            point.x = (int) (x0 * temp * temp * temp + 3 * x1 * t * temp * temp + 3 * x2 * t * t * temp + x3 * t * t * t);
            point.y = (int) (y0 * temp * temp * temp + 3 * y1 * t * temp * temp + 3 * y2 * t * t * temp + y3 * t * t * t);
            return point;
        }
    
        private static class VisualizeManager {
    
            private static Visualizer visualizer = null;
    
            synchronized private static Visualizer getInstance() {
                if (visualizer == null) {
                    visualizer = new Visualizer(0);
                }
                return visualizer;
            }
    
            private static void destoryVisualizer() {
                if (visualizer != null) {
                    visualizer.release();
                    visualizer = null;
                }
            }
        }
    }


# 渲染效率

在画的使用也要注意渲染效率

- 移除不必要的background
- 用clipRect优化
- onDraw方法要避免执行大量的操作
    - onDraw中不要创建新的局部对象。频繁调用时，如果一瞬间产生大量的临时对象会占用过多的内存而且会导致系统更加频发的gc，降低程序的执行效率。
    - onDraw中不要做耗时任务，也不能执行成千上万的循环操作，尽管每次循环都很轻量级，但是大量的循环仍然十分强占CPU的时间片，会造成View的绘制过程不流畅。

[更多性能优化](http://vivianking6855.github.io/2017/02/27/Android-optimization-1-method/)


# 开源库

[Android 开源项目分类汇总](https://github.com/Trinea/android-open-project) ： 里面自定义控件非常之多，可以学习

[List of Android UI/UX Libraries](https://github.com/wasabeef/awesome-android-ui)：一些惊艳的自定义控件

<br>



> [Android 开源项目分类汇总 ](https://github.com/Trinea/android-open-project)

> [Android 学习资料收集](https://github.com/Freelander/Android_Data)


