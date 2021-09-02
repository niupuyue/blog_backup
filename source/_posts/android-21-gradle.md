---
title: 重拾android路(二十) gradle
date: 2018-2-27 11:27:11
tags:
  - gradle
  - android 
---

概念
<!--more-->
Gradle是帮助我们管理Android项目的工具，可以帮我们实现快速引入依赖库，编译方式，多渠道打包等工作。一开始在开发Android项目中使用的是eclipse，其中引入依赖库时，需要从网上下载下来，然后再导入到项目中，过程非常繁琐。不过在使用了Gradle之后，我就感觉我可能离不开这个玩意儿了。
首先来看一些Gradle的官方概念，算了，直接在网上找了一个我觉得是比较好的解释
> Gradle就是工程的管理，帮我们做了依赖,打包,部署,发布,各种渠道的差异管理等工作。举个例子形容，如果我是一个做大事的少爷平时管不了这么多小事情，那Gradle就是一个贴心的秘书或者管家，把一些杂七杂八的小事情都帮我们做好了，让我们可以安心的打代码，其他事情可以交给管家管

# 分析
下面我们对一个功能进行逐步的分析
![在这里插入图片描述](/assets/gradle/gradle01.png)
## gradle-wrapper.properties
wrapper是对gradle的一层包装，方便与开发团队在整体中使用同一的gradle版本号，使用相同的gradle版本进行构建，就可以避免很多不必要的麻烦和问题。上面这个图是在Android项目中的文件形式，下面这个是在文件目录中的展示形式。
![在这里插入图片描述](/assets/gradle/gradle02.png)
那么既然是统一gradle版本等信息，我们就要看一下里面都有什么内容，这个是我的项目中有的内容
```
#Fri May 17 18:24:11 CST 2019
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-4.4-all.zip
```
![在这里插入图片描述](/assets/gradle/gradle03.png)
这里有一些字段名需要我们注意的
|字段名  | 说明 |
|--|--|
| distributionBase | 下载的gradle压缩包解压之后存储的主目录 |
| distributionPath  | 相对于distributionBase的解压后的gradle压缩包路径  |
| zipStoreBase  | 同distributionBase 类似，只不过是存放zip压缩包的 |
| zipStorePath  | 同distributionPath类似，只不过是存放zip压缩包 |
|  distributionUrl  | gradle发行版压缩包的下载网址 |

OK，通过这个我们就能知道这些资源下载的路径，比如我的路径就是这样子的
![在这里插入图片描述](/assets/gradle/gradle04.png)
那么有时候我们在下载这些gradle版本的时候会发现非常慢，这是一些特殊情况造成的，解决办法是：
1. 科学上网
2. 使用本地gradle版本进行编译，不要使用默认的gradle版本

## settings.gradle
settings.gradle是用来初始化和配置工程树的，放在根工程目录下。
gradle的众多工程是通过工程树的形式展示的，相当于在Android studio中的Project和Module的关系相类似，根工程相当于Android studio中的Project，一个根工程可以包含很多个子工程，也就是很多Module
![在这里插入图片描述](/assets/gradle/gradle05.png)
我这里的gradle中没有包含太多东西，因为这是一个demo工程，那么在实际开发中，我们可能使用到的Module就会比较多
![在这里插入图片描述](/assets/gradle/gradle06.png)
## Groovy
Groovy是基于JVM的动态脚本语言，与java语法相类似，完全兼容java，并且在java的基础上添加了很多动态类型和灵活特性，比如支持密保，支持DSL。其实刚开始学习这些知识的时候，总是将gradle和groovy弄混，感觉两者之间的相差比较少。其实我们可以这样理解，gradle相当于是水，而groovy相当于是氢原子和氧原子，就是说groovy是用来组成和编写gradle。其实我们后面说道的gradle都是使用groovy脚本编写的

## build.gradle（project）
![在这里插入图片描述](/assets/gradle/gradle07.png)
### bulidscript
构建脚本，在这里声明gradle自身所需要使用的资源，这里包括了依赖项，第三方插件，maven或者jcenter仓库路径
### ext
这里在我的gradle文件中并没有体现出来，不过我们可以通过另外一个项目来展示
![在这里插入图片描述](/assets/gradle/gradle08.png)
ext表示的是自定义属性，这里我们可以将一些配置信息，特别是配置的版本信息放在这里面声明，等到后面我们改变版本的时候，只需要在这里改变即可，不需要逐一查找。具体的使用方法是
1. 在项目的根目录中创建一个名为config.gradle的文件
![在这里插入图片描述](/assets/gradle/gradle09.png)
这个config.gradle就是我们的一些配置信息，里面的内容如下如是，内容较多
```
ext{
    android = [
            compileSdkVersion         : 28,
            buildToolsVersion         : "28.0.0",
            minSdkVersion             : 17,
            targetSdkVersion          : 26,
            versionCode               : 100,
            versionName               : "1.0.0",
            androidSupportSdkVersion  : "28.0.0",
            retrofitSdkVersion        : "2.0.0",
            dagger2SdkVersion         : "2.6",
            rxlifecycleSdkVersion     : "0.6.1",
            espressoSdkVersion        : "2.2.2"
    ]
    dependencies = [
            "javax.annotation"       : 'javax.annotation:jsr250-api:1.0',
            "appcompat-v7"           : "com.android.support:appcompat-v7:${android["androidSupportSdkVersion"]}",
            "design"                 : "com.android.support:design:${android["androidSupportSdkVersion"]}",
            "support-v4"             : "com.android.support:support-v4:${android["androidSupportSdkVersion"]}",
            "cardview-v7"            : "com.android.support:cardview-v7:${android["androidSupportSdkVersion"]}",
            "annotations"            : "com.android.support:support-annotations:${android["androidSupportSdkVersion"]}",
            "recyclerview-v7"        : "com.android.support:recyclerview-v7:${android["androidSupportSdkVersion"]}",
            //网络框架
            "retrofit"               : "com.squareup.retrofit2:retrofit:2.0.0",
            "retrofit-converter-gson": "com.squareup.retrofit2:converter-gson:2.0.0",
            "retrofit-adapter-rxjava": "com.squareup.retrofit2:adapter-rxjava:2.0.0",
            "converter-scalars"      : "com.squareup.retrofit2:converter-scalars:2.0.0",
            "okhttp3"                : "com.squareup.okhttp3:okhttp:3.2.0",
            "gson"                   : "com.google.code.gson:gson:2.6.2",

            //dagger2
            "dagger2"                : "com.google.dagger:dagger:${android["dagger2SdkVersion"]}",
            "dagger2-apt-compiler"   : "com.google.dagger:dagger-compiler:${android["dagger2SdkVersion"]}",
            "rxandroid"              : "io.reactivex:rxandroid:1.1.0",
            "rxjava"                 : "io.reactivex:rxjava:1.1.0",
            "autolayout"             : "com.zhy:autolayout:1.4.5",
            "butterknife"            : "com.jakewharton:butterknife:8.4.0",
            "butterknife-apt"        : "com.jakewharton:butterknife-compiler:8.4.0",

            "rxpermissions"          : "com.tbruyelle.rxpermissions:rxpermissions:0.7.0@aar",
            //图片处理框架
            "glide"                  : "com.github.bumptech.glide:glide:3.7.0",

            //test
            "junit"                  : "junit:junit:4.12",
            "androidJUnitRunner"     : "android.support.test.runner.AndroidJUnitRunner",
            "timber"                 : "com.jakewharton.timber:timber:4.1.2",
            "canary-debug"           : "com.squareup.leakcanary:leakcanary-android:1.4-beta2",
            "canary-release"         : "com.squareup.leakcanary:leakcanary-android-no-op:1.4-beta2",
            "Android-AdvancedWebView": "com.github.delight-im:Android-AdvancedWebView:v3.0.0",
            "jpush"                  : "cn.jiguang.sdk:jpush:3.0.9",
            "jpushcore"              : "cn.jiguang.sdk:jcore:1.1.7",
            "scale-image-view"       : "com.davemorrissey.labs:subsampling-scale-image-view:3.6.0",//加载大图，可缩放的ImageView
            "banner"                 : "com.youth.banner:banner:1.4.10",
            "MagicIndicator"         : "com.github.hackware1993:MagicIndicator:1.5.0",//一个tab导航栏
            "logging"                : "com.squareup.okhttp3:logging-interceptor:3.1.2",
            "PersistentCookieJar"    : 'com.github.franmontiel:PersistentCookieJar:v1.0.1',//用于持久化cookie工具
            "carrousellayout"        : 'com.dalong:carrousellayout:1.0.0',//类似旋转木马的效果图
            "recyclercoverflow"      : 'com.chenlittleping:recyclercoverflow:1.0.5',//类似旋转木马的效果图的RV
            "materialtabstrip"       : 'com.jpardogo.materialtabstrip:library:1.1.0',
            "kenburnsview"           : 'com.flaviofaria:kenburnsview:1.0.7',
            "materialviewpager"      : 'com.github.florent37:materialviewpager:1.2.3',
            "avi"                    : 'com.wang.avi:library:2.1.3',//一个酷炫的加载动画
            "vcharts"                : 'com.vinctor:vcharts:1.0.0',//一个环形统计图
            "material-calendarview"  : 'com.prolificinteractive:material-calendarview:1.4.3',//一个material design风格的日历控件
            "commonrefreshlayout"    : 'com.mylhyl:commonrefreshlayout:2.4',//一个支持上拉加载更多的RecyclerView
            "openDefaultRelease"     : 'com.sina.weibo.sdk:core:2.0.3:openDefaultRelease@aar',//微博分享
            "umengcommon"            : 'com.umeng.sdk:common:latest.integration',//友盟统计sdk
            "umenganalytics"         : 'com.umeng.sdk:analytics:latest.integration',//友盟统计sdk
            "multidex"               : 'com.android.support:multidex:1.0.1',//解决兼容性问题
            "arouter-api"            : 'com.alibaba:arouter-api:1.3.1',//ARouter api
            "arouter-compiler"       : 'com.alibaba:arouter-compiler:1.1.4',//ARouter
            "constraint-layout"      : 'com.android.support.constraint:constraint-layout:1.1.0',//约束性布局
            "test:runner"            : 'com.android.support.test:runner:1.0.2',//测试依赖，新建项目时会默认添加，一般不建议添加
            "espresso-core"          : 'com.android.support.test.espresso:espresso-core:3.0.2',//测试依赖，新建项目时会默认添加，一般不建议添加
    ]
}
```
![在这里插入图片描述](/assets/gradle/gradle10.png)
2. 需要在build.gradle(project)中引入该配置文件，如下所示
```
// Top-level build file where you can add configuration options common to all sub-projects/modules.
apply from: "config.gradle"
buildscript {
    
    repositories {
        google()
        jcenter()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.0'
        classpath 'org.greenrobot:greendao-gradle-plugin:3.0.0'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        google()
        jcenter()
        mavenCentral()
        maven { url "https://jitpack.io" }
        maven { url 'https://dl.bintray.com/umsdk/release' }
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```
![在这里插入图片描述](/assets/gradle/gradle11.png)
3. 使用的时候直接可以在Module的builde.gradle中使用，如下所示
![在这里插入图片描述](/assets/gradle/gradle12.png)
### repositories
仓库，jcenter()、maven()和google()就是托管第三方插件的平台

### dependencies
简单的配置完仓库是不够的，我们还需要在dependencies{}里面的配置里，把需要配置的依赖用classpath配置上，因为这个dependencies在buildscript{}里面，所以代表的是Gradle需要的插件。
```
allprojects {
    repositories {
        google()
        jcenter()
        mavenCentral()
        maven { url "https://jitpack.io" }
        maven { url 'https://dl.bintray.com/umsdk/release' }
    }
}
```
### allprojects
allprojects块的repositories用于多项目构建，为所有项目提供共同所需依赖包。而子项目可以配置自己的repositories以获取自己独需的依赖包。
这里我们需要说明一下buildscript和allprojects的区别
> 简单的来说，buildscript中主要用来申明gradle脚本自身所需要使用的资源，也就是说他是管家所需要的资源，跟我们的关系并不大，而allprojects是当前项目中所有Module所需要使用的资源，也就是说如果我们的Module库中都是用到的资源，可以放在这里声明。

### test clean
![在这里插入图片描述](/assets/gradle/gradle13.png)
运行gradle clean时，执行此处定义的task。该任务继承自Delete，删除根目录中的build目录。相当于执行Delete.delete(rootProject.buildDir)。其实这个任务的执行就是可以删除生成的Build文件的，跟Android Studio的clean是一个道理。

## build.gradle（Module）
各个Module中的gradle文件，如图所示
```
apply plugin: 'com.android.application'
apply plugin: 'org.greenrobot.greendao'

android {
    compileSdkVersion rootProject.ext.android.compileSdkVersion
    defaultConfig {
        applicationId "com.paulniu.iying"
        minSdkVersion 17
        targetSdkVersion 26
        versionCode 1
        versionName "1.0.0"
        multiDexEnabled true
    }
    buildTypes {
        release {
            buildConfigField "boolean", "IS_RELEASE", "true"
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        debug {
            buildConfigField "boolean", "IS_RELEASE", "false"
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    greendao {
        // 指定数据库版本
        schemaVersion 1
        // DaoSession DaoMaster 以及所有实体类的dao生成的目录，默认是entity所在的包名
        // daoPackage 包名
        daoPackage 'com.paulniu.iying.greendao.dao'
        // 自定义生成的数据库文件目录，可以将生成的文件放在java文件夹下
        targetGenDir 'src/main/java'
    }
    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
}

repositories {
    flatDir {
        dirs 'libs'
    }
}

dependencies {
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    compile project(path: ':baselibs')
    // 系统依赖
    implementation 'com.android.support:appcompat-v7:28.0.0'
    implementation 'com.android.support:design:28.0.0'
    implementation 'com.android.support:cardview-v7:28.0.0'
    implementation 'com.android.support:support-compat:28.0.0'
    implementation 'com.android.support.constraint:constraint-layout:1.1.3'
    implementation 'com.android.support:recyclerview-v7:28.0.0'
    testImplementation 'junit:junit:4.12'
    //GreenDao
    api 'org.greenrobot:greendao:3.0.1'
    api 'org.greenrobot:greendao-generator:3.0.0'
    // view simple animations
    api 'com.daimajia.easing:library:2.0@aar'
    api 'com.daimajia.androidanimations:library:2.3@aar'
    // Glide
    implementation 'com.github.bumptech.glide:glide:4.9.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.9.0'
    // RxJava RxAndroid
    implementation 'io.reactivex.rxjava2:rxjava:2.2.7'
    implementation 'io.reactivex.rxjava2:rxandroid:2.0.1'
    // Multidex
    compile 'com.android.support:multidex:1.0.3'
    // refresh
    api 'com.scwang.smartrefresh:SmartRefreshLayout:1.1.0-alpha-18'
    api 'com.scwang.smartrefresh:SmartRefreshHeader:1.1.0-alpha-18'
    // refrofit
    api 'com.squareup.retrofit2:retrofit:2.5.0'
    api 'com.squareup.okhttp3:okhttp:3.2.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.5.0'
    implementation 'com.squareup.retrofit2:adapter-rxjava2:2.5.0'
    // cache Cookie
    implementation 'com.github.franmontiel:PersistentCookieJar:v1.0.1'
    // JSON
    implementation 'com.google.code.gson:gson:2.8.2'
    // eventbus
    implementation 'org.greenrobot:eventbus:3.0.0'
    // startbar
    implementation 'me.zhanghai.android.materialratingbar:library:1.3.1'
    // Logger
    implementation 'com.orhanobut:logger:2.2.0'
    // multisnaprecyclerview
    implementation 'com.github.takusemba:multisnaprecyclerview:1.3.4'
    // BaseRecyclerViewAdapterHelper
    implementation 'com.github.CymChad:BaseRecyclerViewAdapterHelper:2.9.30'
    // RxPermission
    implementation 'com.github.tbruyelle:rxpermissions:0.10.2'

}
```
![在这里插入图片描述](/assets/gradle/gradle14.png)
#### apply plugin
![在这里插入图片描述](/assets/gradle/gradle15.png)
这里是引入gradle插件，引入的方式分为两种
1. apply plugin: 'xxxxxx'  二进制插件，二进制插件一般都是被打包在一个jar里独立发布的，比如我们自定义的插件，再发布的时候我们也可以为其指定plugin id，这个plugin id最好是一个全限定名称，就像你的包名一样
2. apply from: 'xxxxx'   应用脚本插件，其实这不能算一个插件，它只是一个脚本。应用脚本插件，其实就是把这个脚本加载进来，和二进制插件不同的是它使用的是from关键字.后面紧跟一个脚本文件，可以是本地的，也可以是网络存在的，如果是网络上的话要使用HTTP URL.虽然它不是一个真正的插件，但是不能忽视它的作用.它是脚本文件模块化的基础，我们可以把庞大的脚本文件.进行分块、分段整理.拆分成一个个共用、职责分明的文件，然后使用apply from来引用它们，比如我们可以把常用的函数放在一个Utils.gradle脚本里，供其他脚本文件引用。示例中我们把 App的版本名称和版本号单独放在一个脚本文件里，清晰、简单、方便、快捷.我们也可以使用自动化对该文件自动处理，生成版本
> Gradle插件的作用
> 把插件应用到你的项目中，插件会扩展项目的功能，帮助你在项目的构建过程中做很多事情。1.可以添加任务到你的项目中，帮你完成一些亊情，比如测试、编译、打包。2.可以添加依赖配置到你的项目中，我们可以通过它们配置我们项目在构建过程中需要的依赖.比 如我们编译的时候依赖的第三方库等。3.可以向项目中现有的对象类型添加新的扩展属性、 方法等，让你可以使用它们帮助我们配置、优化构建，比如android{}这个配置块就是Android Gradle插件为Project对象添加的一个扩展。4. 可以对项目进行一些约定，比如应用Java插 件之后，约定src/main/java目录下是我们的源代码存放位置，在编译的时候也是编译这个目录下的Java源代码文件

#### android{}
Android插件提供的一个扩展类型，可以让我们自定义Android Gradle工程，是Android Gradle工程配置的唯一入口
#### compileSdkVersion
编译所依赖的Android SDK的版本，这里是API Level。
#### buildToolsVersion
构建该Android工程所用构建工具的版本
#### defaultConfig{}
defaultConfig是默认的配置，它是一个ProductFlavor。ProductFlavor允许我们根据不同的情况同时生成多个不同的apk包。
#### applicationId
配置我们的包名，包名是app的唯一标识，其实他跟AndroidManifest里面的package是可以不同的，他们之间并没有直接的关系。package指的是代码目录下路径；applicationId指的是app对外发布的唯一标识，会在签名、申请第三方库、发布时候用到。
#### minSdkVersion
支持的Android系统的api level，这里是15，也就是说低于Android 15版本的机型不能使用这个app
#### targetSdkVersion
表明我们是基于哪个Android版本开发的，这里是22
#### versionCode
表明我们的app应用内部版本号，一般用于控制app升级，当然我在使用的bugly自动升级能不能接受到升级推送就是基于这个
#### versionName
表明我们的app应用的版本名称，一般是发布的时候写在app上告诉用户的，这样当你修复了一个bug并更新了版本，别人却发现说怎么你这个bug还在，你这时候就可以自信的告诉他自己看下app的版本号
#### multiDexEnabled
用于配置该BuildType是否启用自动拆分多个Dex的功能。一般用程序中代码太多，超过了65535个方法的时候
#### ndk{}
多平台编译，生成有so包的时候使用，包括四个平台'armeabi', 'x86', 'armeabi-v7a', 'mips'。一般使用第三方提供的SDK的时候，可能会附带so库。
#### sourceSets
源代码集合，是Java插件用来描述和管理源代码及资源的一个抽象概念，是一个Java源代码文件和资源文件的集合，我们可以通过sourceSets更改源集的Java目录或者资源目录等。
例如：通过sourceSets告诉了Gradle我的关于jni so包的存放路径就在app/libs上了，叫他编译的时候自己去找
```
 sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
```
#### buildTypes
构建类型，在Android Gradle工程中，它已经帮我们内置了debug和release两个构建类型，两种模式主要车别在于，能否在设备上调试以及签名不一样，其他代码和文件资源都是一样的。一般用在代码混淆，而指定的混淆文件在下图的目录上，minifyEnabled=true就会开启混淆：
![在这里插入图片描述](/assets/gradle/gradle16.png)
```
name：build type的名字
 
applicationIdSuffix：应用id后缀
 
versionNameSuffix：版本名称后缀
 
debuggable：是否生成一个debug的apk
 
minifyEnabled：是否混淆
 
proguardFiles：混淆文件
 
signingConfig：签名配置
 
manifestPlaceholders：清单占位符
 
shrinkResources：是否去除未利用的资源，默认false，表示不去除。
 
zipAlignEnable：是否使用zipalign工具压缩。
 
multiDexEnabled：是否拆成多个Dex
 
multiDexKeepFile：指定文本文件编译进主Dex文件中
 
multiDexKeepProguard：指定混淆文件编译进主Dex文件中
```
#### signingConfigs
签名配置，一个app只有在签名之后才能被发布、安装、使用，签名是保护app的方式，标记该app的唯一性。如果app被恶意删改，签名就不一样了，无法升级安装，一定程度保护了我们的app。而signingConfigs就很方便为我们提供这个签名的配置。storeFile签名文件，storePassword签名证书文件的密码，storeType签名证书类型，keyAlias签名证书中秘钥别名，keyPassword签名证书中改密钥的密码。
默认情况下，debug模式的签名已经被配置好了，使用的就是Android SDK自动生成的debug证书，它一般位于$HOME/.android/debug.keystore,其key和密码是已经知道的，一般情况下我们不需要单独配置debug模式的签名信息。
```
signingConfigs {
        debug {
            storeFile file(getProp("RELEASE_STORE_FILE"))
            storePassword getProp("RELEASE_STORE_PASSWORD")
            keyAlias getProp("RELEASE_KEY_ALIAS")
            keyPassword getProp("RELEASE_KEY_PASSWORD")
        }

        release {
            storeFile file(getProp("RELEASE_STORE_FILE"))
            storePassword getProp("RELEASE_STORE_PASSWORD")
            keyAlias getProp("RELEASE_KEY_ALIAS")
            keyPassword getProp("RELEASE_KEY_PASSWORD")
        }
    }
```
#### productFlavors
配置多渠道的打包的工具，使用productFlavors区分不同的产品，定义不同的逻辑，使构建部分有差异的Android项目更加方便
一般分为两个步骤去执行：
1. 配置渠道
2. 配置渠道相关信息

##### 配置渠道
```
android {
	productFlavors {
        changel1{
            manifestPlaceholders = [
                    UMENG_CHANNEL_VALUE : "changel1",
                    UMENG_APPKEY        : "xxxxxxxxxxxxxxxxxxxxxxxxxxx",
                    UMENG_MESSAGE_SECRET: "xxxxxxxxxxxxxxxxxxxxxxxxxxx"
            ]
        }
        changel2{
            manifestPlaceholders = [
                    UMENG_CHANNEL_VALUE : "changel2",
                    UMENG_APPKEY        : "xxxxxxxxxxxxxxxxxxxxxxxxxxx",
                    UMENG_MESSAGE_SECRET: "xxxxxxxxxxxxxxxxxxxxxxxxxxx"
            ]
        }
    }
}
```
这里我只添加了两个渠道，一个是渠道1，另外一个是渠道2.
##### 配置渠道相关信息
给渠道定义的渠道号（说白了就是对app分发渠道的一个标识，不懂的孩纸就需要自行上网搜索了）。渠道统计里面最经常使用的估计是非友盟莫属了。这里我们也以友盟作为例子
```
<meta-data
	android:name="UMENG_CHANNEL"
	android:value="origin" />
```
用过友盟的朋友都知道，我们需要在app的androidManifest.xml文件里面去配置这样一个渠道信息。以便友盟进行区别分析和统计。在平常开发过程中我们通常会写死在xml配置里面。而此时如果涉及到多渠道打包，我们就不能这么操作了。这里我们做一点小小的变化，把上面的值origin抽取到如下名为CHANNEL_VALUE的变量中
```
<meta-data
	android:name="UMENG_CHANNEL"
	android:value="${CHANNEL_VALUE}" />
```
如果直接这样写，可能连编译这一关都通过不了，因为他没有找到CHANNEL_VALUE这个数据，至少我是这样子的，所以我又给这个变量添加了一个声明
![在这里插入图片描述](/assets/gradle/gradle17.png)
如果我们需要一次性打包多个渠道，那么需要添加下面这段代码
```
android {
   ...
    productFlavors {
        ...
    }
	productFlavors.all { flavor ->
        flavor.manifestPlaceholders = [CHANNEL_VALUE: name]
    }
}
```
#### manifestPlaceholders
占位符，我们可以通过它动态配置AndroidManifest文件一些内容，譬如app的名字，或者在多渠道打包中替换不同的APP_KEY
#### buildConfigField
BuildConfig文件的一个函数，而BuildConfig这个类是Android Gradle构建脚本在编译后生成的。而buildConfigField就是其中的自定义函数变量，我们可以自定义一些变量帮我们实现我们想要的效果，如下所示：
![在这里插入图片描述](/assets/gradle/gradle18.png)
这里我们添加了两个对象，一个是构建时间，也就是我们打包的时间，另外一个是设置当前App默认的代理，方便我们抓包测试，其中在debug中我们设置了默认的代理地址。这里面getProp()和buildTime()是我们自己定义的方法，具体的操作如下
```
// 获取打包时间
static def buildTime() {
    return "\"" + new Date().format("yyyy-MM-dd HH:mm", TimeZone.getTimeZone("UTC")) + "\"";
}

/**
 * 获取配置文件信息
 */
def String getProp(String keyName) {
    def props = new Properties()
    file("../local.properties").withInputStream { stream ->
        props.load(stream)
    }
    props.get(keyName)
}
```
![配置文件信息](/assets/gradle/gradle19.png)
#### dexOptions
Android中的Java源代码被编译成class字节码后，在打包成apk的时候被dx命令优化成Android虚拟机可执行的DEX文件。DEX文件比较紧凑，Android费尽心思做了这个DEX格式，就是为了能使我们的程序在Android中平台上运行快一些。对于这些生成DEX文件的过程和处理，Android Gradle插件都帮我们处理好了，Android Gradle插件会调用SDK中的dx命令进行处理。但是有的时候可能会遇到提示内存不足的错误，大致提示异常是java,lang.OutOfMemoryError: GC overhead limit exceeded,为什么会提示内存不足呢？ 其实这个dx命令只是一个脚本，它调用的还是Java编写的dx.jar库，是Java程序处理的，所以当内存不足的时候，我们会看到这个Java异常信息.默认情况下给dx分配的内存是一个G8,也就是 1024MB
```
dexOptions {
        jumboMode = true  // 忽略方法数检查
        preDexLibraries = true
        threadCount = 8
        maxProcessCount 8
    }
```
### dependencies
我们最常使用的就是这个了
```
dependencies {
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    compile project(path: ':baselibs')
    // 系统依赖
    implementation 'com.android.support:appcompat-v7:28.0.0'
    implementation 'com.android.support:design:28.0.0'
    implementation 'com.android.support:cardview-v7:28.0.0'
    implementation 'com.android.support:support-compat:28.0.0'
    implementation 'com.android.support.constraint:constraint-layout:1.1.3'
    implementation 'com.android.support:recyclerview-v7:28.0.0'
    testImplementation 'junit:junit:4.12'
    //GreenDao
    api 'org.greenrobot:greendao:3.0.1'
    api 'org.greenrobot:greendao-generator:3.0.0'
    // view simple animations
    api 'com.daimajia.easing:library:2.0@aar'
    api 'com.daimajia.androidanimations:library:2.3@aar'
    // Glide
    implementation 'com.github.bumptech.glide:glide:4.9.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.9.0'
    // RxJava RxAndroid
    implementation 'io.reactivex.rxjava2:rxjava:2.2.7'
    implementation 'io.reactivex.rxjava2:rxandroid:2.0.1'
    // Multidex
    compile 'com.android.support:multidex:1.0.3'
    // refresh
    api 'com.scwang.smartrefresh:SmartRefreshLayout:1.1.0-alpha-18'
    api 'com.scwang.smartrefresh:SmartRefreshHeader:1.1.0-alpha-18'
    // refrofit
    api 'com.squareup.retrofit2:retrofit:2.5.0'
    api 'com.squareup.okhttp3:okhttp:3.2.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.5.0'
    implementation 'com.squareup.retrofit2:adapter-rxjava2:2.5.0'
    // cache Cookie
    implementation 'com.github.franmontiel:PersistentCookieJar:v1.0.1'
    // JSON
    implementation 'com.google.code.gson:gson:2.8.2'
    // eventbus
    implementation 'org.greenrobot:eventbus:3.0.0'
    // startbar
    implementation 'me.zhanghai.android.materialratingbar:library:1.3.1'
    // Logger
    implementation 'com.orhanobut:logger:2.2.0'
    // multisnaprecyclerview
    implementation 'com.github.takusemba:multisnaprecyclerview:1.3.4'
    // BaseRecyclerViewAdapterHelper
    implementation 'com.github.CymChad:BaseRecyclerViewAdapterHelper:2.9.30'
    // RxPermission
    implementation 'com.github.tbruyelle:rxpermissions:0.10.2'
}
```
1. compile fileTree(include: ['*.jar'], dir: 'libs')，这样配置之后本地libs文件夹下的扩展名为jar的都会被依赖，非常方便。
2. 如果你要引入某个本地module的话，那么需要用compile project('×××')。
3. 如果要引入网上仓库里面的依赖，我们需要这样写compile group：'com.squareup.okhttp3',name:'okhttp',version:'3.0.1',当然这样是最完整的版本，缩写就把group、name、version去掉，然后以":"分割即可。compile 'com.squareup.okhttp3:okhttp:3.0.1'
4. gradle提供的依赖配置
![在这里插入图片描述](/assets/gradle/gradle20.png)
5. gradle3.0以后build.gradle中的依赖默认为implementation，而不是之前的compile

最后说一下别的东西，当我们的项目逐渐变大时，引入的各种依赖库也会增加，这样就会有依赖库的相互冲突，那么解决冲突问题也变得很重要。在网上也有各种各样的解决办法，我后面会慢慢补充


# 参考资料
[彻底弄明白Android中Gradle的配置](https://blog.csdn.net/heng615975867/article/details/80346723)
[Gradle应用](https://github.com/D-clock/Doc/tree/master/Android/Gradle)

