---
title: 重拾android路(三十三) 指纹
date: 2019-09-26 11:35:51
tags:
  - android
  - 指纹
---
Android指纹识别，提升App用户体验

<!--more-->

指纹识别大家应该都不陌生，一些比较隐私的App都会添加指纹识别技术，以保障用户的信息安全。不仅如此，一些比较新的手机已经添加了人脸识别技术，这样更是方便了用户。但是指纹识别依然是主流，对于一般的千元机来说，指纹识别也是标配。那么我们是可以通过Google API实现指纹识别的基础功能的。

## 指纹识别的兼容性和安全性

除了实现指纹识别的基础功能，我们还应该关注两个问题：兼容性和安全性

首先兼容性，指纹识别的基础功能是在Android6.0以后才开放出来的，那么在Android6.0以下，如果厂商也对手机的指纹识别进行了定制化操作，那么就会出现兼容性的问题。
其次安全性，由于已添加的指纹是存储在手机上的，GoogleAPI验证指纹之后的返回值仅仅是true或false，我们很难无条件的相信这个识别结果的。比如用户的手机如果root了，那么指纹识别是有可能会被劫持进而返回错误的识别结果。

> 当然上面所说的情况发生概率比较低，如果指纹识别的应用场景非交易非支付，仅仅类似于启动App这样的简单操作，那么基础的指纹识别功能已经能够满足了。

## 指纹识别的实践

先看代码和实现效果，再看原理和源码

我们想做一个这样的App，当应用启动的时候要求指纹识别，如果识别通过，展示主页面，如果识别不通过，则提示识别失败，请重新识别。
在使用指纹识别时，指纹识别框是可以让用户自定义或者使用原生的识别框的。



## 指纹识别API的版本演进
在Android6.0开放了指纹识别API，存在于<p style="color:red">android.hardware.fingerprint</p>包下，核心类是FingerprintManager，提供了基础的指纹识别功能。要注意的是，FingerprintManager在android9.0做了@Deprecated标记，将被弃用

后来，在<p style="color:red">android.support.v4.hardware.fingerprint</p>包和<p style="color:red">androidx.core.hardware.fingerprint</p>包中，<p style="color:red">FngerprintManger</p>升级为<p style="color:red">FingerprintMangerCompat</p>,对功能进行了增强，也做了一些兼容性处理，比如增加了系统版本号的判断，对指纹支持加密处理等，实际上阅读源码我们会发现，其实还是调用了FingerprintManger实现的

再之后，android9.0Google对生物识别进行了进一步增强，开放了以<p style="color:red">BiometricPrompt</p>为核心的新Api，存在于<p style="color:red">androidx.biometric</p>包和<p style="color:red">android.hardware.biometrics</p>包下。这里提供的是支持设备提供的生物识别，包括指纹，虹膜，面部等

### 指纹识别的关键方法 authenticate

这个是指纹识别中最核心的方法，用于拉起指纹识别扫描器进行指纹识别
以<p style="color:red">FingerprintManagerCompat</p>中的authenticate方法为例，定义如下
![authenticate定义](/assets/fingerprint/fingerprint_01.png)
![authenticate定义](/assets/fingerprint/fingerprint_02.png)

参数说明
1. FingerprintManagerCompat.CrytoObject crypto
    密码对象包装类，目前支持Signature形式和Cipher形式的密码对象加密
    作用是，指纹扫描器会使用这个对象判断指纹认证结果的合法性，Android6.0是@Nullable，但不建议传Null，其在Android9.0之后就变成了@NonNull
2. int flags
    可选标志，暂无使用的地方，传0即可
3. CancellationSignal cancel
    这个对象的作用是用来取消指纹扫描操作的。比如用户点击了识别框上的取消按钮或者密码验证按钮，就要及时取消扫描器的扫描操作.如果不执行的话，会造成好电，而且在超时时间内无法再次唤起指纹识别        

# 参考资料

[指纹识别](https://blog.csdn.net/hailong0529/article/details/95406183)