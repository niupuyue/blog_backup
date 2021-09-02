---
title: 重拾android路(十三) 权限
date: 2016-08-05 23:29:59
tags:
  - android
---

众所周知，在之前的应用中，Android系统的用户体验确实没有像IOS那么令人满意。这个主要是因为多方面原因造成的。其中一个就是因为Android系统本身就是属于开源系统，他的所有代码都是公开的，程序员开发者可以根据自己的需要进行修改。那么这样大大开放了开发者的思维，但是同样也有各种各样的问题。其中一个就是关于安全性问题的。那么在Android6.0之后，引入了动态权限管理，这可以很大程度的解决很多关于Android用户体验的问题。那么这篇博客就是好好的记录一下关于权限的问题。
<!--more-->
# Android系统权限的概念
Android是一个权限分割的操作系统，每个应用都有独特的系统标识，一般情况下，如果某一个应用没有权限的时候是无法进行一系列的操作的。每个应用的运行都相当于是在应用沙盒中执行的。所谓的应用沙盒，我们可以理解为一个封闭的盒子，如果没有给他一些窗口，外部是无法访问到的。当我们需要执行一些操作的时候，就需要申请权限。例如读取SD卡，获取网络状态等等。
Android系统的权限声明一般是在清单配置文件中的。在Android6.0之前，我们需要吧所有的权限声明一次性的写在清单文件中，那么一般情况下，用户可能根本就没有看，就直接安装了，这样并不是很安全。举个例子🌰，可能一个应用需要读取手机通讯录，而通讯录属于用户隐私。那么如果用户直接赋予了这个权限，就相当于是用户的个人信息被泄露了。那这个时候APP就可以随心所欲了。因此，Android6.0把权限分为正常权限和危险权限。正常权限写在清单配置文件中，在应用被安装的时候自动赋予。而危险权限则必须由用户明确授予。

# 危险权限和对应的分组
我们这里只关注危险权限，因为正常权限的内容和之前是一样的。
危险权限都属于权限组，应用向用户申请危险权限的时候，系统会弹出对话框，描述应用需要访问的权限，如果用户同意了，则权限组中的所有权限都可以被使用。

| 权限组 | 权限|
|------|------|
|CALENDAR| Read_calendar  Write_calendar|
|CAMERA |CAMERA | 
|CONTACTS | READ_CONTACTS  WRITE_CONTACTS  GET_CONTACTS |
|LOCATION | ACCESS_FINE_LOCATION   ACCESS_COARSE_LOCATION |
|MICROPHONE |RECORD_AUDIO|
|PHONE | READ_PHONE_STATE CALL_PHONE READ_CALL_LOG WRITE_CALL_LOG ADD_VOICEMAIL   USE_SIP     PROCESS_OUTGOING_CALLS |
| SENSORS | BODY_SENSORS |
| SMS | SEND_SMS RECEIVE_SMS  READ_SMS RECEIVE_WAP_PUSH RECEIVE_MMS |
| STORAGE | READ_EXTERNAL_STORAGE  WRITE_EXTERNAL_STORAGE|

# 声明权限的正确姿势
- 需要申请的所有权限在AndroidManifest文件中声明
```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```
- 使用时检查权限，没有权限则申请
```
        //使用兼容库就无需判断系统版本
        int hasWriteStoragePermission = ContextCompat.checkSelfPermission(getApplication(), Manifest.permission.WRITE_EXTERNAL_STORAGE);
        if (hasWriteStoragePermission == PackageManager.PERMISSION_GRANTED) {
                        //拥有权限，执行操作
            initScan();
        }else{
                        //没有权限，向用户请求权限
            ActivityCompat.requestPermissions(thisActivity, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, MyApplication.CODE_FOR_WRITE_PERMISSION);
        }
```
- 覆写onRequestPermissionsResult方法
```
@Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
                //通过requestCode来识别是否同一个请求
        if (requestCode == CODE_FOR_WRITE_PERMISSION){
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED){
                //用户同意，执行操作
                initScan();
            }else{
                //用户不同意，向用户展示该权限作用
                if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.WRITE_EXTERNAL_STORAGE)) {
                    new AlertDialog.Builder(thisActivity)
                            .setMessage(R.string.storage_permissions_remind)
                            .setPositiveButton("OK", (dialog1, which) ->
                                    ActivityCompat.requestPermissions(this,
                                            new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},
                                            EventConstConfig.CODE_FOR_CAMERA_PERMISSION))
                            .setNegativeButton("Cancel", null)
                            .create()
                            .show();
                }
            }
        }
    }
```
> shouldShowRequestPermissionRationale方法返回值分几种情况，怎么使用看应用的具体交互需求。
> 1. 第一次请求该权限，返回false。
> 2. 请求过该权限并被用户拒绝，返回true。
> 3. 请求过该权限，但用户拒绝的时候勾选不再提醒，返回false。

# 几个需要注意的地方
1. 使用兼容库
checkSelfPermission、requestPermissions等几个权限相关的方法用v4包里的可以兼容6.0以下版本，否则需要包一层版本判断。
2. targetSDKVersion的问题
我遇到的问题就是这个，有个细节没注意到。Android系统触发动态申请权限的条件其实有两个，设备系统版本在Android 6.0以上并且targetSDKVersion>=23。因此其实在targetSDKVersion版本小于23的情况下，即使在6.0以上的设备运行也不会挂，但会在安装时列出所有权限，同6.0以下的设备。官方建议保持targetSDKVersion在最新的版本
3. 使用第三方库AndPermission
<a href="https://github.com/yanzhenjie/AndPermission/blob/master/README-CN.md">项目地址</a>

