---
layout: post
title: Android Permission处理
date: 2017-6-27
excerpt: "Android Permission处理"
categories: Android
tags: [Android 基础]
comments: true
---

# 简介

从Android M 6.0开始, app需要在运行时请求权限。当然不是指所有权限，仅google定义的部分危险权限。

6.0之前的系统，dangerous权限是安装时授权的。（targetSdkVersion 23及以上）

permission的保护等级有4种

- normal 低风险权限，只要申请了就可以使用（在AndroidManifest.xml中添加uses-permission标签），安装时不需要用户确认
- dangerous 高风险权限，安装时需要用户的确认才可使用
- signature 只有当申请权限的应用程序的数字签名与声明此权限的应用程序的数字签名相同时（如果是申请系统权限，则需要与系统签名相同），才能将权限授给它
- signatureOrSystem 签名相同，或者申请权限的应用为系统应用（在system image中）

signature 和 signatureOrSystem不是特别常用，通常用在系统app或是特殊的工具app等。

# Dangerous Permission

## Dangerous定义

先来看下goolge定义的Dangerous Permission范畴：

![](http://i.imgur.com/Hy6SbLq.jpg)

"adb shell pm list permissions -g -d" 命令也可以查看

app常用到的storage，phone等都属于这类。

[详细信息参看这里normal-dangerous](https://developer.android.com/guide/topics/security/permissions.html#normal-dangerous)

## 处理方案

方案逻辑

1. app OnResume时判断是否有选择永久拒绝？
2. 若永久拒绝，则弹出自定义dialog请user选择退出或是跳转到“应用信息”中开启权限（永久拒绝时，调用requestPermissions系统权限Dialog是不显示的）
3. 若没有永久拒绝 弹出系统Dialog请求权限（也可在导览页面说明为何需要使用这些权限）
   若用户拒绝，则退出app（逻辑可以视app需求调整）

如果可以的话，在app开启之前可以有一页导览页面跟user说明为何会使用到这些权限，以便于user了解，提高允许概率。

核心Code

		(1) onResume检查权限是否允许----------------------
		
		    @Override
		    protected void onResume() {
		        super.onResume();
		        if (!isUserChooseDenyLastTime) {
		            dealWithPermission();
		        }
		    }
		
		    /**
		     * deal with normal dangerous permission,such as Camera, Storage, etc
		     */
		    private void dealWithPermission() {
		        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
		            return;
		        }
		
		        if (userNotGranted()) {// permission not granted
		            String[] notGranted = userChooseNeverShow();
		            if (notGranted.length > 0 && notGranted[0] != null) {
		                showNeverShowHintDialogue(notGranted);
		            } else {
		                if (!mSystemPermissionShowing) {
		                    // show system permission dialogue
		                    mSystemPermissionShowing = true;
		                    showSystemRequestDialog();
		                }
		            }
		        } else {// permission granted
		            permissionGranted();
		        }
		    }

		 (2) 重写onRequestPermissionsResult中处理交互逻辑：系统Permission窗口
		
		    @Override
		    public void onRequestPermissionsResult(int requestCode, String[] permissions,
		                                           int[] grantResults) {
		        if (requestCode == PERMISSION_REQUEST_CODE_RECORD) {
		            mSystemPermissionShowing = false;
		            String[] notGranted = userNotGrantedMore();
		            if (notGranted.length > 0 && notGranted[0] != null) {
		                // user choose deny,
		                permissionDeny(notGranted);
		                isUserChooseDenyLastTime = true;
		            } else {
		                permissionGranted();
		            }
		        }
		        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
		    }
		
		    private boolean getPermissionFistDeny(String permission) {
		        SharedPreferences sp = getSharedPreferences(sPermissonFile, MODE_PRIVATE);
		        return sp.getBoolean(permission, true);
		    }
		
		    private void setPermissionDeny(String permission) {
		        SharedPreferences sp = getSharedPreferences(sPermissonFile, MODE_PRIVATE);
		        SharedPreferences.Editor editor = sp.edit();
		        editor.putBoolean(permission, false);
		        editor.apply();
		    }
		
		    /**
		     * if user choose never show for any one of permissions
		     */
		    private String[] userChooseNeverShow() {
		        String[] permissions = getPermissions();
		        String[] notGranted = new String[permissions.length];
		        int pos = 0;
		        for (int i = 0; i < permissions.length; i++) {
		            // if user choose never show before, request system permission will not work
		            boolean never_show = !ActivityCompat.shouldShowRequestPermissionRationale(BasePermissionActivity.this,
		                    permissions[i]);
		            boolean notGrant = ContextCompat.checkSelfPermission(BasePermissionActivity.this,
		                    permissions[i]) != PackageManager.PERMISSION_GRANTED;
		            boolean firstDeny = getPermissionFistDeny(permissions[i]);
		            if (notGrant && never_show && !firstDeny) {
		                notGranted[pos] = permissions[i];
		                pos++;
		            }
		        }
		        return notGranted;
		    }
		
		    /**
		     * if user not granted any one of permissions
		     */
		    private boolean userNotGranted() {
		        String[] permissions = getPermissions();
		        for (int i = 0; i < permissions.length; i++) {
		            int hasPermission = ContextCompat.checkSelfPermission(BasePermissionActivity.this,
		                    permissions[i]);
		            if (hasPermission != PackageManager.PERMISSION_GRANTED) {
		                return true;
		            }
		        }
		
		        return false;
		    }
		
		    private String[] userNotGrantedMore() {
		        String[] permissions = getPermissions();
		        String[] notGranted = new String[permissions.length];
		        int pos = 0;
		        for (int i = 0; i < permissions.length; i++) {
		            int hasWriteStoragePermission = ContextCompat.checkSelfPermission(BasePermissionActivity.this,
		                    permissions[i]);
		            if (hasWriteStoragePermission != PackageManager.PERMISSION_GRANTED) {
		                notGranted[pos] = permissions[i];
		                pos++;
		                setPermissionDeny(permissions[i]);
		            }
		        }
		
		        return notGranted;
		    }
		
		    /**
		     * show system permission request dlg
		     */
		    private void showSystemRequestDialog() {
		        // show system permission dialogue
		        ActivityCompat.requestPermissions(BasePermissionActivity.this, getPermissions(),
		                PERMISSION_REQUEST_CODE_RECORD);
		    }
		
		    /**
		     * show never show hint dlg
		     */
		    private void showNeverShowHintDialogue(String[] notGranted) {
		        mNeverShowHintDlg = new AlertDialog.Builder(BasePermissionActivity.this)
		                .setCancelable(false)
		                .setTitle(neverShowRes == null ? TITLE : neverShowRes[0])
		                .setMessage(neverShowRes == null ? MESSAGE : neverShowRes[1])
		                .setPositiveButton(neverShowRes == null ? POSITIVE : neverShowRes[2], new DialogInterface.OnClickListener() {
		                    @Override
		                    public void onClick(DialogInterface dialog, int which) {
		                        // user choose never show,you could change your resolution here for your project
		                        gotoSettingsAppDetail();
		                    }
		                })
		                .setNegativeButton(neverShowRes == null ? NEGATIVE : neverShowRes[3], new DialogInterface.OnClickListener() {
		                    @Override
		                    public void onClick(DialogInterface dialog, int which) {
		                        permissionDeny(notGranted);
		                    }
		                })
		                .show();
		    }
		
		    private void gotoSettingsAppDetail() {
		        Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
		        intent.addCategory(Intent.CATEGORY_DEFAULT);
		        intent.setData(Uri.parse("package:" + getPackageName()));
		        startActivity(intent);
		    }
		

		（3）资源释放---------------------------------------------------
		
		    @Override
		    protected void onDestroy() {
		        super.onDestroy();
		
		        // dismiss never show dlg hint
		        if (mNeverShowHintDlg != null && mNeverShowHintDlg.isShowing()) {
		            mNeverShowHintDlg.dismiss();
		        }
		    }
		}

     

这也是google推荐的设计，详情可以参看[这里](https://material.io/guidelines/patterns/permissions.html#permissions-denied-permissions)

[Google Best Practices](https://developer.android.com/training/permissions/best-practices.html)

- 用Intent启动其他应用来完成功能.
- 只用真的需要的权限.
- 不要一次请求多个权限来烦用户,有的权限可以等到要用的时候再请求.
- 向用户解释为什么需要这个权限.
- 从Android 6.0开始,每一条权限,都需要测试开关两种状态下是不是都能让应用正常运行,而不是崩溃。并且相关的权限可能会需要测试不同的组合.

以上只是针对单个权限，如果是多个权限，在请求和call back的地方稍做调整就可以了。

# 特殊权限

有一类权限比较特殊，虽然不属于特殊权限，但是也需要运行时授权才可以使用。

SYSTEM_ALERT_WINDOW 和 WRITE_SETTINGS 这两个权限特别敏感，因此大多数应用不建议使用它们。

## SYSTEM_ALERT_WINDOW

有些悬浮球等一定要用到SYSTEM_ALERT_WINDOW，需要单独授权。路径是：Settings->Apps->App Setting->Draw over other apps

核心Code

    public static int OVERLAY_PERMISSION_REQ_CODE = 1000;
    
    @TargetApi(Build.VERSION_CODES.M)
    public void requestDrawOverLays() {
        if (!Settings.canDrawOverlays(MainActivity.this)) {
            Log.d(TAG, "can not DrawOverlays need requset permission");
            Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:" + MainActivity.this.getPackageName()));
            startActivityForResult(intent, OVERLAY_PERMISSION_REQ_CODE);
        } else {
          // Already hold the SYSTEM_ALERT_WINDOW permission, do addview or something.
        }
    }
    
    @TargetApi(Build.VERSION_CODES.M)
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == OVERLAY_PERMISSION_REQ_CODE) {
            if (!Settings.canDrawOverlays(this)) {
                // SYSTEM_ALERT_WINDOW permission not granted...
                Log.d(TAG, "DrawOverlays Permission was denied")
            } else {
                // Already hold the SYSTEM_ALERT_WINDOW permission, do addview or something.
                Log.d(TAG, "DrawOverlays Permission allowed")
            }
        }
    }
    
## WRITE_SETTINGS
    
在android 6.0及以后，WRITE_SETTINGS权限的保护等级已经由原来的dangerous升级为signature

因此APP需要用系统签名或者成为系统预装软件才能够申请此权限。需要提示用户跳转到修改系统的设置界面去授予此权限

核心Code

        public static int REQUEST_CODE_ASK_WRITE_SETTINGS = 1000;

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            if (!Settings.System.canWrite(this)) {
                Intent intent = new Intent(Settings.ACTION_MANAGE_WRITE_SETTINGS,
                        Uri.parse("package:" + getPackageName()));
                intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                startActivityForResult(intent, REQUEST_CODE_ASK_WRITE_SETTINGS);
            } else {
                // Already hode ACTION_MANAGE_WRITE_SETTINGS permission

            }
        }


# 自己尝试封装BasePermissionActivity.java

设计逻辑

- onResume中请求权限，如果deny过就不在请求
- 若用户选择never show, 则弹出提示说明，请用户到app info界面开启权限

[BasePermissionActivity Github地址](https://github.com/vivianking6855/android-open/tree/master/Common/appbase/src/main/java/com/open/appbase/activity)

使用code如下：

	public class HomeActivity extends BasePermissionActivity {
	
		......
	
	    @Override
	    protected void loadData() {
	        setPermissionAlterDialog(permissionStrs);
	    }
		......
	
	    @Override
	    protected String[] getPermissions() {
	        return new String[]{Manifest.permission.READ_EXTERNAL_STORAGE, Manifest.permission.CAMERA};
	    }
	
	    @Override
	    protected void permissionGranted() {
	
	    }
	
	    @Override
	    protected void permissionDeny(String[] notGranted) {
	        Toast.makeText(HomeActivity.this, "Photo will not work", Toast.LENGTH_SHORT).show();
	    }
	}


# 第三方库

针对Permission也有很多第三方库，大家可以参看

hotchemi’s PermissionsDispatcher。: [https://github.com/hotchemi/PermissionsDispatcher](https://github.com/hotchemi/PermissionsDispatcher)

RxPermissions: [https://github.com/tbruyelle/RxPermissions](https://github.com/tbruyelle/RxPermissions)

	- 核心原理是动态添加一个Fragment来处理权限
	- 允许单个和多个权限请求
	- 但是在onResume中调用会无限循环，需要外部加入特殊逻辑处理

Grant: [https://github.com/anthonycr/Grant](https://github.com/anthonycr/Grant)

[easypermissions](https://github.com/googlesamples/easypermissions) 

三方库的比较可以参照[目前最流行的运行时权限请求框架PermissionsDispatcher、RxPermissions和easypermissions的使用和对比](https://blog.csdn.net/totond/article/details/73648103)

![](https://i.imgur.com/CZcnKyS.jpg)

# 小结

我们的项目中使用的是自己封装的BasePermissionActivity，里面包含了dangerous权限，SYSTEM_ALERT_WINDOW, WRITE_SETTINGS这些功能。

如果是三方app建议使用easypermissions

# Reference

API Guides:[ https://developer.android.com/guide/topics/security/permissions.html]( https://developer.android.com/guide/topics/security/permissions.html)

Training: [https://developer.android.com/training/permissions/index.html](https://developer.android.com/training/permissions/index.html)

Runtime permissions: [https://source.android.com/devices/tech/config/runtime_perms.html](https://source.android.com/devices/tech/config/runtime_perms.html)

permission element: [http://developer.android.com/guide/topics/manifest/permission-element.html](http://developer.android.com/guide/topics/manifest/permission-element.html)

设计Patterns -Permissions: [https://www.google.com/design/spec/patterns/permissions.html](https://www.google.com/design/spec/patterns/permissions.html)
 
博客文章:[Android M 新的运行时权限开发者需要知道的一切](http://jijiaxin89.com/2015/08/30/Android-s-Runtime-Permission/)

博客文章:Android M Permission 运行时权限： [http://www.cnblogs.com/mengdd/p/4892856.html](http://www.cnblogs.com/mengdd/p/4892856.html)

[在android M版本SYSTEM_ALERT_WINDOW权限无法获取问题](http://www.jianshu.com/p/2746a627c6d2)

 
