# 百度聚屏广告AdSDK接入文档V1.5.1

   * [百度聚屏广告AdSDK接入文档V1.5.1](#百度聚屏广告AdSDK接入文档V1.5.1)
      * [产品介绍](#产品介绍)
      * [版本更新说明](#版本更新说明)
      * [SDK接入须知](#sdk接入须知)
         * [接入设备性能要求](#接入设备性能要求)
         * [接入前信息准备](#接入前信息准备)
         * [身份校验（必须）](#身份校验必须)
            * [需要提供应用程序包名](#需要提供应用程序包名)
            * [需要APP签名的md5](#需要app签名的md5)
      * [SDK集成](#sdk集成)
         * [在Manifest.xml中加入：](#在manifestxml中加入)
         * [在根gradle文件下加入：](#在根gradle文件下加入)
         * [在app的gradle下加入：](#在app的gradle下加入)
      * [SDK初始化](#sdk初始化)
         * [在Application的onCreate中初始化。](#在application的oncreate中初始化)
         * [初始化时的参数设置（可选，推荐填写）](#初始化时的参数设置可选推荐填写)
      * [相关权限（必须）](#相关权限必须)
      * [使用广告组件](#使用广告组件)
         * [xml 布局接入，简单，<strong>推荐</strong>](#xml-布局接入简单推荐)
            * [视频组件](#视频组件)
            * [图片轮播组件](#图片轮播组件)
         * [代码示例，媒体方自己通过代码控制广告请求与播放](#代码示例媒体方自己通过代码控制广告请求与播放)
	 * [BDTwoInOneView代码示例](#BDTwoInOneView代码示例)
      * [相关接口介绍](#相关接口介绍)
         * [广告播放回调接口(IAdListener, package com.baidu.adsdk.interfaces)](#广告播放回调接口iadlistener-package-combaiduadsdkinterfaces)
         * [IController相关接口介绍](#IController相关接口介绍)
            * [请求广告loadAdAsync](#请求广告loadadasync)
            * [设置广告状态回调addAdListener](#设置广告状态回调addadlistener)
            * [移除广告状态回调removeAdListener](#移除广告状态回调removeadlistener)
            * [展示广告showAd](#展示广告showad)
            * [释放资源releaseAd](#释放资源releasead)
            * [设置广告位setSlotid](#设置广告位setslotid)
            * [音量控制setVolume](#音量控制setvolume)
            * [保持屏幕常亮keepScreenOn](#保持屏幕常亮keepscreenon)
            * [获取已准备好的广告数getPreparedAdCounts](#获取已准备好的广告数getpreparedadcounts)
            * [设置第三方播放器setIPlayer](#设置第三方播放器setiplayer)
            * [停止播放接口stopAd](#停止播放接口stopad)
            * [预先请求多个广告preLoadMoreAds](#预先请求多个广告preloadmoreads)
            * [返回当前的播放组件getAdView](#返回当前的播放组件getadview)
            * [截图captureAsync](#截图captureAsync)
	    * [让播放器提前去准备广告prepareAd](#让播放器提前去准备广告prepareAd)
	    * [设置最后一帧为黑屏setLastFrameBlack](#设置最后一帧为黑屏setLastFrameBlack)
	    * [设置组件的可见性setViewVisibility](#设置组件的可见性setViewVisibility)
	    * [设置组件的Z-order](#设置组件的Z-order)
            * [播放本地素材playLocalMedia](#播放本地素材playlocalmedia)
            * [监播回调setMonitorCallback](#监播回调setmonitorcallback)
            * [自定义播放器接口](#自定义播放器接口)
               * [使用方法：](#使用方法)
            * [请求第三方广告](#请求第三方广告)
               * [请求广告](#请求广告)
               * [计费](#计费)
      * [SDK使用环境限制](#sdk使用环境限制)
      * [附录](#附录)
         * [时序图](#时序图)
         * [错误列表（持续更新）](#错误列表持续更新)
         * [聚屏返回错误对照表](#聚屏返回错误对照表)
         * [Q&amp;A（持续更新）](#qa持续更新)

## 产品介绍
百度聚屏广告SDK，合入了百度安全监播端-云一体方案，端上通过自有安全播放器、广告安全校验等技术，保障端上播放行为安全可控；云上通过自研安全算法、区块链等技术，从播放可信、设备可信、数据可信3个维度，保障广告播放行为真实可信；同时，SDK内置广告播放器具备高兼容性，支持高清流畅播放mp4、jpg等主流图片/视频广告格式。通过以上核心能力，可从端到云，全面保障媒体广告播放的真实性、安全性和流畅性。

## 版本更新说明
### 1.5.1版本：
   1.使用BDGLGalleryView替换1.2版本的BDGalleryView；
   
   2.缓存路径设置：之前是默认在/data/包名下/cache/目录，1.5版本后需要用户在初始化SDK时手动调用：
   ```
   // 设置缓存路径，第二个参数为缓存路径，如果不设置，默认为/data/包名/下cache目录，
   // 第三个参数为缓存最长保留时间，单位为天。7天代表缓存目录下7天以前的广告会被清除，如果不设置，默认为3.
    AdWrapper.INSTANCE.setCacheDir(this, "/sdcard/test_media/", 7);
   ```
   3.新加了两个接口：
   ```
   // 非必须，播放完后最后一帧是否需要设置黑屏，通常在onAdFinish回调处调用，调用了会让最后一帧显示为黑屏
   fun setLastFrameBlack()
   // 非必须，这个接口只针对视频组件有效，onAdPrepared回调后代表广告已经下载好，
   // 此时调用这个接口可以让播放器提前去prepare这个广告。减少showAd需要准备广告的事件。
   fun prepareAd()
   ```
   
   4.新加了二合一的View:BDTwoInOneView
   主要用在二合一的广告位ID上。
   
   5.新加了控制组件可见性的接口：setViewVisibility，用法和view.setVisibility一样
   
   6.新加了控制组件Z-order的接口：setZOrderOnTop和setZOrderMediaOverlay，用法和SurfaceView一样
  
  
   
## SDK接入须知
### 接入设备性能要求
为保证SDK稳定运行，媒体接入设备，性能需达到以下要求： 

* CPU：性能不低于ARM Cortex-A7；
* RAM：预留200-300MB以上；
* ROM：
   * SDK 1.0版本接入需预留300MB内部存储空间；
   * SDK 1.1及以上版本接入，含SDK卡总共需预留800MB以上；
* 系统版本：Android 4.2.2以上；
* 网络下载速度：2MB/s以上；
### 接入前信息准备
SDK接入前，需提前准备以下参数：

* appSid：资源id，由聚屏平台分配；
* Slotid：广告位id，一般包含图片和视频至少各1个广告位id，由聚屏平台分配；
* mediaId：
   * 若接入设备的广告类型为户外，则需准备该字段，该字段需保证唯一性且已在聚屏平台注册；
   * 若接入设备的广告类型为OTT或开屏广告，则可以填写为null。
* channel：渠道名，一般在交付邮件中将会列出；
* lbsapi.API_KEY：百度定位SDK的KEY，需要在地图SDK后台自行创建.*请务必保证*：在程序中引入百度地图定位SDK v4.0以上版本（http://lbsyun.baidu.com/index.php?title=android-locsdk
）。
* BDAI_FACE_LICENSE:百度人脸SDK的授权License，需要在人脸SDK后台自行创建
 (https://cloud.baidu.com/product/imagerecognition 目前支持批量授权License)

以上信息若有疑问，可联系我们获取。
### 身份校验（必须）
以下信息需要给到百度，百度进行入库，否则将请求不到广告。出于安全性的考虑，后台需要对客户端鉴权，防止第三方拿到SDK之后作弊，目的是为了保障客户权益。
#### 需要提供应用程序包名
例如：`com.baidu.xxxx`
也就是app得gradle文件中的`applicationId`
#### 需要APP签名的md5

```
keytool -printcert -file CERT.RSA
```

签名后从APK解压出来则可以看到此文件，文件放于在META-INF文件夹。
PS：keytool工具为JDK里自带。可在jdk\bin或者jdk\jre\bin目录下找到。
**如果有debug运行需求，可以把debug版的签名一并给到我们**（一般一台机器一个debug签名，可以按需求加）



## SDK集成
### 在Manifest.xml中加入：
```
<meta-data
    android:name="BDAI_FACE_LICENSE"
    android:value="人脸批量授权License">
</meta-data>
<meta-data
    android:name="com.baidu.lbsapi.API_KEY"
    android:value="您的定位SDK EKY值">
</meta-data>
```
### 在根gradle文件下加入：
```javascript { .theme-peacock }

allprojects {
    repositories {
        google()
        jcenter()
        maven { url 'https://dl.bintray.com/bdaiotsecurity/maven' }
    }
}
```


### 在app的gradle下加入：
```c
repositories {
    flatDir {
        dirs 'libs'
    }
}
dependencies {
    // 广告sdk
    implementation (name: 'adsdk_release_vxxx', ext: 'aar')
    // 请求SDK
    implementation (name: 'libtianjiadsdk-release-vxxx', ext: 'aar')
    // 内部库
    implementation "com.baidu.adsdk:saferequest:1.5.x.x"
    // glide
    implementation("com.github.bumptech.glide:glide:4.8.0") {
        exclude group: 'com.android.support'
    }
    // kotlin基础库
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.3.31"
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.2.1'
    // 视频播放器
    implementation 'com.google.android.exoplayer:exoplayer-core:2.10.3'
    // protobuf
    implementation 'com.google.protobuf:protobuf-lite:3.0.0'
    // okhttp
    implementation 'com.squareup.okio:okio:1.13.0'
    implementation 'com.squareup.okhttp3:okhttp:3.9.0'
}
//加入java 8支持
compileOptions {
  targetCompatibility JavaVersion.VERSION_1_8
}
```


## SDK初始化
### 在Application的onCreate中初始化。
```javascript { .theme-peacock }
@Override
public void onCreate() {
    super.onCreate();
    /**
     * @param appSid : 填入聚屏分配的appSid
     * @param channel : 填入聚屏分配的channel
     * @param mediaId :
     * 1. 如果广告位类型是户外，必须填写mediaid（64位以内，仅英文或数字或下划线_或冒号:）, 需要自己保证设备唯一性。
     * 另外，这个mediaid需要在聚屏方注册。
     * 2. 如果你的广告位类型是ott或者开屏广告，则填写为null,sdk将会自动获取mac进行请求。
     */
    AdWrapper.INSTANCE.init(this, 聚屏分配的appSid, 渠道， mediaId);
    // 设置缓存路径，第二个参数为缓存路径，如果不设置，默认为/data/包名/下cache目录，第三个参数为缓存最长保留时间，单位为天。7天代表缓存目录下7天以前的广告会被清除，如果不设置，默认为3.
    AdWrapper.INSTANCE.setCacheDir(this, "/sdcard/test_media/", 7);
    AdWrapper.INSTANCE.enableTest(); // 如果需要使用测试环境才需要enable，否则不填
}
```

### 初始化时的参数设置（可选，推荐填写）
下面在AdWrapper中，可通过`AdWrapper.Instance.XXX`访问，在init之后调用
```ruby
/**
 * @param screenSize : 填入设备的尺寸，不是像素，是宽高，单位用英寸
 */
fun setScreenSize(screenSize: PointF)

/**
 * @param isAIO : 是否是一体机。比如手机就是个一体机，智能电视也是一体机，电视盒子就不是一体机。
 */
fun setIsAIO(isAIO: Boolean)

/**
 * 传递请求时刻5s内探测到的强度TOP5的mac
 * 仅传递安卓设备的mac，过滤掉iOS及其他设备
 * 如选择填写, 如果填写就会利用此类信息触发广告,能获取到尽量传，可以传了可以触发更多的广告
 */
fun setProbInfo(infos: Map<String, Int>)

/**
 * 设置缓存路径。
 */
fun setCacheDir(context: Application, cacheDirPath: String)
```


## 相关权限（必须）
如果APP的apilevel>=23 也就是 Android版本>=6.0。
需要在第一个activity里申请权限。

```javascript { .theme-peacock }
if (Build.VERSION.SDK_INT > 23) {
    requestPermissions(new String[]{"android.permission.READ_PHONE_STATE",
                "android.permission.ACCESS_FINE_LOCATION"}, 101);
}
```

如果你的应用程序满足以下两个任意条件，可以忽略此步骤。

* targetSdkVersion < 23 选择将target 23设置小于23来规避此问题。
* 设备系统小于Android 6.0。

## 使用广告组件
目前提供以下两种接入方式。（注意，对同一个广告组建，要么使用自动方式，要么使用手动方式，在自动模式下如果标记为auto=true，内部将会自动根据设置来进行轮播，此时不需要外部进行控制。)
另外，必须在使用到组件的activity或fragment里的类静态初始化块中调用onActivityInit接口。
```
{
    // 必须在使用到组件的activity或fragment里的类静态初始化块中调用
    AdWrapper.INSTANCE.onActivityInit(this);
}
```

### xml 布局接入，简单，**推荐**
在Activity/Fragment的layout中像普通view一样布局。
其中公共属性如下：
`slotid`此参数需要向聚屏申请（重要，必填）
`auto` 是否是自动播放广告（选填）
#### 视频组件
```javascript { .theme-peacock }
// 外部这个laoyou自己决定
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".activitys.VideoViewAutoActivity">

    // 如果之前用的是BDVideoView，可以直接换成BDGLVideoView
    <com.baidu.adsdk.view.BDGLVideoView
        android:id="@+id/bd_video_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
	// 设置是否自动模式，如果这里填false，则必须使用手动代码方式来控制广告请求与播放。
        app:auto="true"
	// 设置视频广告位id
        app:slotid="XXXX" />
</android.support.constraint.ConstraintLayout>
```
接入后，视频组件将会一直持续播放，并自动请求广告。


#### 图片轮播组件

```javascript { .theme-peacock }
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".activitys.GalleryViewActivity">

    // 如果之前使用的是BDGalleryView，可以直接替换成BDGLGalleryView
    <com.baidu.adsdk.view.BDGLGalleryView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
	// 设置是否自动模式，如果这里填false，则必须使用手动代码方式来控制广告请求与播放。
        app:auto="true"
	// 设置图片广告位id
        app:slotid="XXXXX" />
</LinearLayout>
```


### 代码示例，媒体方自己通过代码控制广告请求与播放
下面这里例子使用了一个视频组件与一个图片轮播组件，模拟了一种场景：
播一个视频，播放成功后再播一个图片，再播放一个视频，由此往复循环。
当然你也可以使用我们提供的mix(二合一组件），里面已经把这个逻辑做好了。
这个例子主要是演示了几个接口如何使用。
```javascript { .theme-peacock }
/**
 * 自己控制广告流,布局就是一个RelativeLayout，加上宽高都是MATCH_PARENT的视频和图片组件
 */
public class ManualActivity extends AppCompatActivity {

    Handler handler = new Handler();

    {
        AdWrapper.INSTANCE.onActivityInit(this);
    }

    /**
     * 视频广告组件
     */
    private BDGLVideoView videoView;
    /**
     * 视频广告控制器
     */
    private IController videoController;

    /**
     * 图片广告组件
     */
    private BDGLGalleryView galleryView;
    /**
     * 图片广告控制器
     */
    private IController galController;

    /**
     * 图片广告组件监听器
     */
    private GalListener galListener = new GalListener(this);

    /**
     * 视频广告组件监听器
     */
    private VideoListener videoListener = new VideoListener(this);

    static class VideoListener extends DefaultAdListener {
        WeakReference<ManualActivity> manualActivityWeakReference;

        VideoListener(ManualActivity activity) {
            manualActivityWeakReference = new WeakReference<>(activity);
        }

        @Override
        public void onAdPrepared(@NotNull RequestInfo info) {
            // 收到视频广告准备OK的时候，可以自己做些业务逻辑，
            // 这个例子是将图片组件隐藏，显示视频组件，然后调用showAd接口
            ManualActivity manualActivity = manualActivityWeakReference.get();
            if (manualActivity != null) {

                manualActivity.galController.setViewVisibility(View.INVISIBLE);
                manualActivity.videoController.setViewVisibility(View.VISIBLE);

                manualActivity.videoController.showAd();
            }

        }

        @Override
        public void onAdFailed(@NotNull RequestInfo info, int ec, @NotNull String msg) {
            // 当广告播放失败回调，这个例子在播放视频广告失败的时候继续请求视频广告
            Log.d("qwe123", "receive AdFailed");
            ManualActivity manualActivity = manualActivityWeakReference.get();
            if (manualActivity != null) {
                manualActivity.handler.postDelayed(new VideoLoadRunnable(manualActivity), 1000);
            }

        }

        @Override
        public void onAdFinish(@NotNull RequestInfo requestInfo) {
            // 当广告完成时回调，这个例子在播放视频广告播放完成的时候请求图片广告
            ManualActivity manualActivity = manualActivityWeakReference.get();
            if (manualActivity != null) {
                manualActivity.galController.loadAdAsync();
                // 最后一帧显示黑屏的功能，可要可不要
                manualActivity.videoView.getController().setLastFrameBlack();
            }
        }
    }

    static class VideoLoadRunnable implements Runnable {
        WeakReference<ManualActivity> manualActivityWeakReference;

        VideoLoadRunnable(ManualActivity activity) {
            manualActivityWeakReference = new WeakReference<>(activity);
        }

        @Override
        public void run() {
            ManualActivity activity = manualActivityWeakReference.get();
            if (activity != null) {
                activity.videoController.loadAdAsync();
            }
        }
    }

    static class GalLoadRunnable implements Runnable {
        WeakReference<ManualActivity> manualActivityWeakReference;

        GalLoadRunnable(ManualActivity activity) {
            manualActivityWeakReference = new WeakReference<>(activity);
        }

        @Override
        public void run() {
            ManualActivity activity = manualActivityWeakReference.get();
            if (activity != null) {
                activity.galController.loadAdAsync();
            }
        }
    }

    static class GalListener extends DefaultAdListener {
        WeakReference<ManualActivity> manualActivityWeakReference;

        GalListener(ManualActivity activity) {
            manualActivityWeakReference = new WeakReference<>(activity);
        }

        @Override
        public void onAdPrepared(@NotNull RequestInfo info) {
            // 收到图片广告准备OK的时候，可以自己做些业务逻辑，
            // 这个例子是将视频组件隐藏，显示图片组件，然后调用showAd接口
            ManualActivity manualActivity = manualActivityWeakReference.get();
            if (manualActivity != null) {

		manualActivity.videoController.setViewVisibility(View.INVISIBLE);
                manualActivity.galController.setViewVisibility(View.VISIBLE);
		
                manualActivity.galController.showAd();
            }
        }


        @Override
        public void onAdFailed(@NotNull RequestInfo info, int ec, @NotNull String msg) {
            Log.d("qwe123", "receive AdFailed");
            // 当图片广告播放失败回调，这个例子在图片广告失败再请求图片广告
            ManualActivity manualActivity = manualActivityWeakReference.get();
            if (manualActivity != null) {
                manualActivity.handler.postDelayed(new GalLoadRunnable(manualActivity), 1000);
            }
        }

        @Override
        public void onAdFinish(@NotNull RequestInfo requestInfo) {
            // 当图片广告播放完成回调，这个例子在图片广告播放完成请求视频广告
            ManualActivity manualActivity = manualActivityWeakReference.get();
            if (manualActivity != null) {
                manualActivity.videoController.loadAdAsync();
                manualActivity.galleryView.getController().setLastFrameBlack();
            }
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        videoController.removeAdListener(videoListener);
        galController.removeAdListener(galListener);

        handler.removeCallbacksAndMessages(null);
        videoController.releaseAd();
        galController.releaseAd();
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_video_manual);

        // 我们这个例子是再xml中布局视频与图片组件，
        // 你同样也可以在代码中使用LayoutParams布局
        videoView = findViewById(R.id.bdvv_test);
        galleryView = findViewById(R.id.bdvv_gal_test);

        // 初始化图片组件，设置监听器
        galController = galleryView.getController();
        galController.addAdListener(galListener);

        // 初始化视频组件，设置监听器
        videoController = videoView.getController();
        // 你可以通过setSlotid设置广告位id，也可以通过xml中设置
        videoController.setSlotid("XXXX");
        videoController.addAdListener(videoListener);

        // 发起视频组件的广告请求
        videoController.loadAdAsync();

    }

    /**
     * 提供一个骨架方法，业务方不需要重写每个回调时使用。仅为降低代码冗余度
     */
    public static class DefaultAdListener implements IAdListener {
        @Override
        public void onAdPrepared(@NotNull RequestInfo info) {

        }

        @Override
        public void onAdStart() {

        }

        @Override
        public void onAdFailed(@NotNull RequestInfo info, int ec, @NotNull String msg) {

        }

        @Override
        public void onAdClick() {

        }

        @Override
        public void onAdFinish(@NotNull RequestInfo info) {

        }

        @Override
        public void onAdDismissed() {

        }
    }
}
```

### BDTwoInOneView代码示例
```
/**
 * 二合一组件代码示例
 */
public class ManualTwoInOneActivity extends AppCompatActivity {

    Handler handler = new Handler();

    {
        AdWrapper.INSTANCE.onActivityInit(this);
    }

    /**
     * 广告组件
     */
    private BDTwoInOneView twoInOneView;
    /**
     * 广告控制器
     */
    private IController twoInOneController;

    private TwoInOneListener twoInOneListener = new TwoInOneListener(this);

    static class TwoInOneListener extends DefaultAdListener {
        WeakReference<ManualTwoInOneActivity> manualActivityWeakReference;

        TwoInOneListener(ManualTwoInOneActivity activity) {
            manualActivityWeakReference = new WeakReference<>(activity);
        }

        @Override
        public void onAdPrepared(@NotNull RequestInfo info) {
            ManualTwoInOneActivity manualActivity = manualActivityWeakReference.get();
            if (manualActivity != null) {
                manualActivity.twoInOneController.showAd();
            }
        }

        @Override
        public void onAdFailed(@NotNull RequestInfo info, int ec, @NotNull String msg) {
            // 当广告播放失败回调，这个例子在播放视频广告失败的时候继续请求视频广告
            Log.d("qwe123", "receive AdFailed");
            ManualTwoInOneActivity manualActivity = manualActivityWeakReference.get();
            if (manualActivity != null) {
                manualActivity.handler.postDelayed(new VideoLoadRunnable(manualActivity), 1000);
            }

        }

        @Override
        public void onAdFinish(@NotNull RequestInfo requestInfo) {
            // 当广告完成时回调，这个例子在播放视频广告播放完成的时候请求图片广告
            ManualTwoInOneActivity manualActivity = manualActivityWeakReference.get();
            if (manualActivity != null) {
//                manualActivity.videoView.setTopLayout(false);
                manualActivity.twoInOneController.loadAdAsync();
                // 最后一帧显示黑屏的功能，可要可不要
                manualActivity.twoInOneView.getController().setLastFrameBlack();
            }
        }
    }

    static class VideoLoadRunnable implements Runnable {
        WeakReference<ManualTwoInOneActivity> manualActivityWeakReference;

        VideoLoadRunnable(ManualTwoInOneActivity activity) {
            manualActivityWeakReference = new WeakReference<>(activity);
        }

        @Override
        public void run() {
            ManualTwoInOneActivity activity = manualActivityWeakReference.get();
            if (activity != null) {
                activity.twoInOneController.loadAdAsync();
            }
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        twoInOneController.removeAdListener(twoInOneListener);
        handler.removeCallbacksAndMessages(null);
        twoInOneController.releaseAd();
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_twoinone_manual);

        // 我们这个例子是再xml中布局视频与图片组件，
        // 你同样也可以在代码中使用LayoutParams布局
        twoInOneView = findViewById(R.id.bdtwo_in_one_view);

        // 初始化二合一组件，设置监听器
        twoInOneController = twoInOneView.getController();
        twoInOneController.addAdListener(twoInOneListener);

        // 发起广告请求
        twoInOneController.loadAdAsync();
    }

    /**
     * 提供一个骨架方法，业务方不需要重写每个回调时使用。仅为降低代码冗余度
     */
    public static class DefaultAdListener implements IAdListener {
        @Override
        public void onAdPrepared(@NotNull RequestInfo info) {

        }

        @Override
        public void onAdStart() {

        }

        @Override
        public void onAdFailed(@NotNull RequestInfo info, int ec, @NotNull String msg) {

        }

        @Override
        public void onAdClick() {

        }

        @Override
        public void onAdFinish(@NotNull RequestInfo info) {

        }

        @Override
        public void onAdDismissed() {

        }
    }
}

```

## 相关接口介绍

### 广告播放回调接口(IAdListener, package com.baidu.adsdk.interfaces)

此接口通过IController.addAdListener方法设置。以下接口回调都在主线程，各个回调接口的具体含义如下：

* onAdPrepared  已经下载好了，但是还没有开始播放。preparedinfo里面有广告请求的唯一标识符与广告位id，此时可以通过controller调用showAd。
* onAdStart  已经开始播放了。（是一个时间点）
* onAdFailed和onAdFinish平行，要么成功完成，要么失败
* onAdClick点击广告时回调
* onAdDismissed比如按了home键或者其他activtiy在这个View上面，这个方法就会被回调


### IController相关接口介绍
* IController是SDK的核心类，所有相关的业务逻辑都由IController来完成，通过播放组件View的getController方法就可以拿到这个类。以下接口按照常用性排序：

#### 请求广告loadAdAsync
* fun loadAdAsync()，**必须调用**，向服务端请求广告。

#### 设置广告状态回调addAdListener
* fun addAdListener(adListener: IAdListener)， **必须调用**，通过添加这个接口，在请求失败，下载失败时都能及时回调给APP，这样能及时作出相应的处理。

#### 移除广告状态回调removeAdListener
* fun removeAdListener(adListener: IAdListener)，**必须调用**，和addAdListener成对出现，通常在组件销毁时调用。
####  展示广告showAd
* fun showAd()，**必须调用**，调用时必须是在onAdPrepared回调完成以后。

#### 释放资源releaseAd
* fun releaseAd()，**必须调用**，广告组件需要销毁的时候调用，比如在Activity的onDestroy调用，一旦销毁，生命周期结束，不能再使用任何业务。
#### 设置广告位setSlotid
* fun setSlotid(slotid: String)，设置广告位ID，由聚屏服务端分配，**必须调用**，但如果在组件View的xml里面填了slotid属性，则可以不需要调用。

#### 音量控制setVolume
* fun setVolume(value: Float)，非必须调用，volume取值范围在0~1，0代表无声，1代表最大音量。需要在loadAdAsync之前调用，只针对视频有效，不设置则默认是系统媒体的音量。

#### 保持屏幕常亮keepScreenOn
* fun keepScreenOn(b: Boolean)，**非必须调用**，b代表是否需要保持屏幕常亮，默认是常亮的。

#### 获取已下载好的广告数getPreparedAdCounts
* fun getPreparedAdCounts() : Int，**非必须调用**，帮助媒体更好地控制请求和播放，避免频繁请求。如果这个数大于0，意味着之前下载好的广告还没有播放。
#### 设置第三方播放器setIPlayer
* fun setIPlayer(player: IPlayer)，**非必须调用**，可以支持第三方的播放器，播放器实例需要实现IPlayer接口，通过此方法传递给SDK。
#### 停止播放接口stopAd
* fun stopAd(skip: Boolean)，**非必须调用**，skip代表下次播放时是否需要跳过当前被停止的广告，一般传false即可。
* 此函数调用完以后下次会重新开始播放素材。

#### 设置组件的可见性setViewVisibility
* 非必须调用，设置我们组件的可见性。

#### 设置组件的Z-order
* 非必须调用，设置组件的Z-order，详情可见SurfaceView的setZOrderOnTop

#### 设置组件的Z-order:setZOrderMediaOverlay
* 非必须调用，设置组件的Z-order，详情可见SurfaceView的setZOrderMediaOverlay

#### 设置最后一帧为黑屏setLastFrameBlack
* 非必须调用，设置最后一帧为黑屏setLastFrameBlack(),播放完后最后一帧是否需要设置黑屏，通常在onAdFinish回调处调用，调用了会让最后一帧显示为黑屏.

#### 让播放器提前去准备广告prepareAd
* 非必须调用，这个接口只针对视频组件有效，onAdPrepared回调后代表广告已经下载好，此时调用这个接口可以让播放器提前去prepare这个广告。减少showAd需要准备广告的事件。

#### 预先请求多个广告preLoadMoreAds
* fun preLoadMoreAds(maxReqNums: Int, needAds: Int, callback: IPreLoadAdProgress)方法。非必须调用。参数1 maxReqNums: 这次任务的最大请求数；参数2 needAds: 这次任务需要下载的广告数，为避免计费串失效，最大为4；参数3 callback: 这次任务的请求过程的回调,如下,参数意义为第几个请求，是否成功，错误码，请求信息

```
interface IPreLoadAdProgress {
	
    fun progress(index: Int, suc: Boolean, errCode: Int, info: RequestInfo)

}
```
**注意点**：
① 因为每次请求不一定有广告，所以会有前面两个参数，避免频繁，无意义，不合理的请求；
② 参数1和参数2任一条件达到，此次任务结束；

**举例**：
* 如果调用了 preLoadMoreAds(10, 3, callback), 那么最多请求10次广告，最多能提前准备好三个要播放的广告。如果请求了10次，还是没有能准备到3个广告，此次任务就会结束。

**和loadAdAsync方法的区别：**
* preLoadMoreAds是提前准备，媒体可以根据回调判断提前请求的任务是否结束，结束后，调用loadAdAsync，会从已经准备好的广告列表里拿数据，有的话直接返回，没有的话再向聚屏服务器请求广告，其余流程不变。

#### 返回当前的播放组件getAdView
* fun getAdView(): View，非必须调用。
#### 截图captureAsync
* fun captureAsync(c: (bitmap: Bitmap?) -> Unit) ，此方法已弃用，请使用下面这个方法去截图：
  fun captureAsync(filePath: String, quality: Int, callback: ICaptureResult)
  参数介绍：filePath，截图文件存储的路径，文件名最好以.jpg结尾，quality是压缩质量，0~100，100代表不压缩，callback是截屏结果的回调。
* ICaptureResult介绍：
```
interface ICaptureResult {

    fun result(code: Int, errMsg: String, filePath: String)

}
``` 
 参数1 code表示回调结果:除了SUCCESS其他都代表失败。
    const val SUCCESS = 0
    const val CAPTURE_BLACK_ERROR = -1
    const val JPEG_ERROR = -2
    const val PARAM_FILENAME_INVALID = -3
    const val PARAM_QUALITY_INVALID = -4
    const val PLAY_BLACK_ERROR = -5
 参数2 errMsg代表错误信息，成功时为空。
 参数3 filePath就是一开始调用captureAsync传进来的参数而已。

#### 播放本地素材playLocalMedia
* fun playLocalMedia(file: File, url: String, md5: String = "",
                       rotation: Float = 0f, duration: Int = 0)，非必须调用。参数1是本地素材的File，参数2代表这个素材的url，可填空字符串；参数3代表这个素材的md5，如果不填或填空字符串，sdk内部会自行计算素材的md5；参数4代表旋转角度，取值范围是-180°~180°，比如90°，代表顺时针旋转90度；参数5代表广告播放时长，单位是秒。
* 一般媒体不需要调用这个函数。

#### 监播回调setMonitorCallback
* fun setMonitorCallback(callback: IMonitorCallback)，播放过程中会回调监播是否成功。
* IMonitorCallback里面的方法：fun result(info: RequestInfo?, suc: Boolean, code: Int, errMsg: String) ，参数1代表此次播放素材的info，参数2代表监播是否成功.

code:参数有以下几个值：
```
    const val SUCCESS = 0
    const val MONITOR_NOT_ADTA = -300
    const val MONITOR_ALREADY = -301
    const val MONITOR_CPAPTURE_NULL = -302
```

* 其中MONITOR_ALREADY代表这个广告已经监播过，已经监播过的广告，suc也为true.

### 自定义播放器接口
功能非必须，扩展类接口，SDK默认使用系统播放器，也支持第三方的播放器，但需要实现如下两个接口：

```javascript { .theme-peacock }
com.baidu.adsdk.interfaces

interface IPlayer {

    /**
     * @param path while the file is downloaded.
     * */
    fun preparing(path: String)

    /**
     * start to play prepare video or image.
     * Before invoke start(), you must invokes {@link #prepare()} first.
     * */
    fun start()

    /**
     * stop to play current video or image.
     * */
    fun stop()

    /**
     * When view is being destroy, this function must be invoked
     * */
    fun release()

    /**
     * set Display surfaceholder
     * */
    fun setDisplay(sh: SurfaceHolder)

    /**
     * set surface
     * */
    fun setSurface(surface: Surface)

    /**
     * get duration
     * @return
     * */
    fun getDuration(): Long

    /**
     * set volume
     * */
    fun setVolume(leftVolume: Float, rightVolume: Float)

    fun setIPlayerCallback(callback: IPlayerCallback)

}

interface IPlayerCallback {

    fun notifyPlayerPrepared()

    fun notifyPlayerFail(errorCode: Int)

    fun notifyPlayerCompetition()

}
```
#### 使用方法：
* ①自己的播放器实现IPlayer接口。
* ②SDK内部自己会调用setIPlayerCallback传给播放器，外部需要保存IPlayerCallback这个接口实例，在适当时候回调给SDK内部。
* ③通过Controller，调用setIPlayer即可。

### 请求第三方广告
功能非必须，扩展类接口。如果你还有第三方广告参与，需要做如下工作。
可以自定义的流程包含如下几个阶段：请求广告、监播结果、计费、自定义播放器
这部分整体流程如下图：
![image](https://github.com/biduaiotsecurity/adsdk_dev_help/blob/master/%E9%A2%9D%E5%A4%96%E7%89%B9%E6%80%A7.jpg)
#### 请求广告
这部分关心如下几个接口：
```javascript { .theme-peacock }
package com.baidu.adsdk.model.interfaces

interface IADModel {
    // 将在一开始被回调一次
    fun init(context: Context, sdkContext: SdkContext)

    // 请求广告
    fun requestAd(requestId: String, slotId: String, request: IRequestResult)

    // 计费接口，没有就返回null
    fun createBill(adData: AdData): IBill?
}

interface IRequestResult {
    // 使用者内部请求到广告后，将数据回传给SDK，这个接口在上面requestAd时已传入内部
    fun onRequestSuccess(adData: AdData)
    // 使用者内部请求到广告后，将数据回传给SDK，这个接口在上面requestAd时已传入内部
    fun onRequestFailed(code: Int, requestInfo: RequestInfo)
}


interface IModelManager {
    // 初始化，SDK将在初始化时调用此接口一次。
    fun init(sdkContext: SdkContext)
    // 返回一个图片类别的model给SDK进行播控，当有多个model时候，可以自己定制策略返回适当的model。
    fun getImageModel(): IADModel?
    // 返回一个视频类别的model给SDK进行播控，当有多个model时候，可以自己定制策略返回适当的model。
    fun getVideoModel(): IADModel?
}
```

* 实例化一个或多个IADModel，可以负责请求图片类广告、请求视频类广告。
* 实例化一个IModelManager。
* AdWrapper.INSTANCE.addModelManager(传入自己的实例)

#### 计费
这部分关心如下两个接口：

```javascript { .theme-peacock }
package com.baidu.adsdk.model.interfaces

interface IBill {
    /**
     * 给机会计费，只在这里传递通用数据，具体其它需要的数据可以在createBill里做自己设置进去。
     */
    fun calculateCost(adData: AdData, billResult: IBillResult)
}

interface IBillResult {
    fun onBillSuccess(adData: AdData)
    fun onBillFailed(adData: AdData, ec: Int)
}
```

* 创建一个实例实现IBill接口
* 在model的createBill接口实现部分返回你的IBill实例
* 在calculateCost实现计费逻辑，并将计费结果通过IBillResult接口返回给SDK


## SDK使用环境限制

* 不允许在播放过程中遮盖、隐藏、调整大小广告组件，否则将会影响收益。
* 广告组件大小不能小于300*300px， 否则将会影响收益。

## 附录

### 时序图
![image](https://github.com/biduaiotsecurity/adsdk_dev_help/blob/master/%E8%81%9A%E5%B1%8FSDK%E6%97%B6%E5%BA%8F%E5%9B%BE.jpg)


### 错误列表（持续更新）
```javascript { .theme-peacock }
-1 unknow 大多是网络问题，网络不好的情况
-2 illegal argument
-3 time_out 连接超时
-4 permission_deny
-5 url is empty
-6 md5 doesn't match 下载后的文件与云端md5匹配与否
-7 not_support_breakpoint
-8 unknown_host_exception
-9 socket_exception
-10 runtimeexception
-11 identity_verification_failed sdk身份校验错误，这个一般是因为app包名与app签名的md5没有入库导致的。
-15 sslhandshake_exception
-16 ssl_exception
-17 timeout_cancellation_exception 这个就是下载物料超时，这一块可以检查下网络
-101 get_media_connection_error
-102 media_play_unknown_error
-103 media_decoder_init_error
-104 media_play_pause
-200 surface_no_create

-201 prepare_fail_file_not_exist
-202 prepare_fail_inputstream_valid
-203 prepare_fail_unknown
-204 start_fail
-205 stop_fail

-300 monitor_no_adta
-302 monitor_capture_null

SDK身份校验错误码：
retCode 为 0x01 包名或者md5不匹配
	   0x02 请求格式不正确或者参数错误
	   0x03 解密失败
	   0x04 鉴权失败
```
### 聚屏返回错误对照表  
百度聚屏 API 错误码对照表 （适用于API v6.0及以上版本）  
   
1 前言  
错误码对照表文件作为对API 接口的补充说明，在API 接口升级后，会不定期更新维护。每个错误码对应的 处理方法均为引导性建议，对于具体的错误需要相关技术人员根据具体情况来处理。 对文档中未说明的内容，请咨询百度聚屏。  
注：联调前，请媒体技术人员先本地验证proto序列化、反序列化功能正确。proto版本为2.4  
2 无错误响应 

|错误码| 定义| 解释|
|----|----|----|
|0| NO_ERROR| 请求处理正确|  
   
3 请求信息错误  
  
3.1 请求基础信息错误  

|错误码| 定义| 解释|  
|----|----|----|  
|100001| MISSING_REQUEST| 请求参数缺失 |  

注：请求数据无法解析，或解析后请求ID缺失/格式异常时。返回http状态码400，无错误码  
3.2 API 版本信息错误  

|错误码| 定义| 解释 |  
|----|----|----|  
|102000|  MISSING_API_VERSION |使用API 版本信息缺失 |  
|102010| MISSING_API_VERSION_MAJOR| API API主版本信息缺失 |  
|102011 |ERROR_API_VERSION |API API版本信息错误 |  

3.3 APP_ID信息错误  

|错误码| 定义| 解释|  
|----|----|----|  
|103010  |MISSING_APP_ID| APP_ID信息缺失 |  
|103011| ERROR_APP_ID| APP_ID信息错误，MSSP未收录 |  
   
3.4 设备信息错误   

|错误码| 定义| 解释|  
|----|----|----|  
|100003|----|加解密失败| 
|104000| MISSING_DEVICE_INFO| 设备信息缺失|  
|104010| MISSING_DEVICE_TYPE| 设备类型信息缺失|  
|104011| ERROR_DEVICE_TYPE| 设备类型信息错误|  
|104020| MISSING_OS_TYPE| 操作系统信息缺失|  
|104021| ERROR_OS_TYPE| 操作系统信息错误|  
|104030| MISSING_OS_VERSION| 操作系统版本信息缺失|  
|104040| MISSING_OS_VERSION|_MAJOR 操作系统主版本信息缺失|  
|104050| MISSING_VENDOR| 厂商信息缺失|  
|104060| MISSING_MODEL| 设备型号信息缺失|  
|104070| MISSING_UDID| 设备唯一标识符缺失|  
|104071| ERROR_UDID| 设备唯一标识符未备案|  
|104090| MISSING_SCREEN_SIZE| 设备屏幕尺寸信息缺失|  
|104100| MISSING_SCREEN_SIZE_WIDTH| 设备屏幕尺寸宽度缺失|  
|104110| MISSING_SCREEN_SIZE_HEIGHT| 设备屏幕尺寸高度缺失|  
|104120| MISSING_UDID_ID_TYPE| 设备唯一标识符类型缺失|  
|104121| ERROR_UDID_ID_TYPE| 设备唯一标识符类型错误|  
|104130| MISSING_UDID_ID| 设备唯一标识符ID值缺失|  
|104140| ERROR_FORMAT_MAC| 设备mac不符合约定格式|   

3.5 网络环境信息错误  

|错误码| 定义| 解释|  
|----|----|----|  
|105000| MISSING_NETWORK_INFO| 网络环境信息缺失|  
|105010| MISSING_IPV4| 网络地址信息缺失|  
|105011| ERROR_FORMAT_IPV4| 网络地址信息格式错误|  
|105020| MISSING_CONNECTION_TYPE| 网络连接类型缺失|  
|105021| ERROR_CONNECTION_TYPE| 网络连接类型错误|  
|105030| MISSING_OPERATOR_TYPE| 网络运营商类型缺失|  
|105031| ERROR_OPERATOR_TYPE| 网络运营商类型错误|  
|105040| MISSING_AP_MAC| Wi-Fi 热点地址信息缺失|  
|105041| ERROR_FORMAT_AP_MAC| Wi-Fi 热点地址信息格式错误|  
|105050| MISSING_RSSI| Wi-Fi 热点信号强度信息缺失|  
|105060| MISSING_AP_NAME| Wi-Fi 热点名称缺失|  
|105070| MISSING_AP_CONNECTION| Wi-Fi 连接状态信息缺失|  

3.6 GPS 坐标信息错误  

|错误码| 定义| 解释|  
|----|----|----|  
|106000| MISSING_COORDINATE_TYPE| 坐标类型信息缺失|  
|106001| ERROR_COORDINATE_TYPE| 坐标类型信息错误|  
|106010| MISSING_LONGITUDE| 经度信息缺失|  
|106020| MISSING_LATITUDE| 纬度信息缺失|  
|106030|  MISSING_GPS_TIMESTAMP| 定位时间戳信息缺失|   

3.7 广告位信息错误    

|错误码| 定义| 解释|  
|----|----|----|  
|107000| MISSING_ADSLOT_ID| 广告位ID缺失|  
|107001| ERROR_ADSLOT_ID| 广告位ID未收录|  
|107003| NOT_MATCH_ADSLOT_ID| 广告位ID与APP_ID不匹配|  
|107010| MISSING_ADSLOT_SIZE| 广告位尺寸信息缺失|  
|107040| MISSING_ADSLOT| 广告位信息缺失|  

3.8 其他错误  

|错误码| 定义| 解释|  
|----|----|----|  
|400000| FLOW_DROP_BY_TIME_CONTORL| 时段流量丢弃(仅户外、出行)|  

4 响应信息错误  
4.1 广告请求  

|错误码| 定义| 解释|  
|----|----|----|  
|200000| NO_AD| 请求处理正确，无广告返回|  
|200003| AD_ASP_RETURN_ERR| 请求处理正确，无广告返回(无预算导致)|  
|201010| AD_NO_SIGN |广告无签名|  
|201020| MISSING_CRETIVE_TYPE| 广告创意类型信息丢失|  
|201021| ERROR_CRETIVE_TYPE| 广告创意类型信息无法识别|    
|201030| MISSING_INTERATION_TYPE| 广告动作类型信息丢失|   
|201031| ERROR_INTERATION_TYPE| 广告动作类型信息无法识别|  
|201040| MISSING_WIN_NOTICE_URL| 曝光汇报地址丢失|  
|201041| ERROR_WIN_NOTICE_URL_SIZE| 曝光汇报地址异常|  
|201050| MISSING_CLICK_URL| 点击响应地址丢失|   
|201060| MISSING_TITLE| 推广标题丢失|  
|201070| MISSING_DESCRIPTION| 推广描述丢失|  
|201080| MISSING_APP_PACKAGE| 推广应用包名丢失|  
|201090| MISSING_APP_SIZE| 推广应用包大小丢失|  
|201100| MISSING_ICON_SRC| 推广图标丢失|  
|201110| MISSING_IMAGE_SRC| 推广图片丢失|  
|201111| AD_BAD_JSON| 广告json串错误|  
  
5 典型case  
1) 接口返回http状态码400 请求数据无法解析，或解析后请求ID 缺失/格式异常时。返回http状态码400，无错误码。请本地验证请求数据 可解析，并检查请求ID格式满足要求：仅英文字母和数字，32 位，大小写不敏感  
2) 设备ID未备案（104071 ERROR_UDID） 请求参数中udid未在媒体配置表中提供，认为是非法流量，返回 104071错误。  
3) API版本信息错误 API主版本号必填，初始版本为6.0.0  
4) APP_ID未收录（103011 ERROR_APP_ID） app_id填写错误，请确认与mssp申请的app_id一致  
5) 广告位ID 未收录（107001 ERROR_ADSLOT_ID） 广告位id填写错误，请确认与mssp申请的广告位id一致  
6) 广告位ID 与APP_ID不匹配（107003 NOT_MATCH_ADSLOT_ID） 请确认请求参数中，广告位ID、APP_ID在媒体配置表是对应关系  
7) 无广告返回处理 200000 NO_AD：指百度聚屏内部错误，导致无广告返回。 201000 AD_NO_DATA：指请求处理正常，但无广告主投放，因此无广告返回；返回201000 可以认为接口测 试通过。  
8) proto版本错误 请按proto2版本开发。使用proto3可能出现解析异常等问题。  
9) 必填参数缺失 proto注释中标明为“必填”参数，不可缺失，否则接口返回错误。特别注意以下两点 - 参数是否“必填”，以proto注释为准。请勿参照字段限定修饰符中的optional/required - wifi ap参数为选填，如选择填写，则ap_mac/rssi/ap_name不可为空。类似的还有探针参数、GPS参数  
10) 参数格式错误 部分参数有格式校验，如request_id、ipv4、udid、ap_mac、client_mac等。校验不通过，接口返回错误  
11) 枚举值参数超出全集 os_type、connection_type、operator_type、coordinate_type取值不可超出proto 给出的枚举值，否则接 口返回错误  
12) 返回信息http头部字段 返回信息中，http头部字段Content-Length与返回内容实际大小不一致。解析时请勿使用。  

### Q&A（持续更新）
* 1）Q：Unable to resolve dependency for XXX Could not download safehttp.aar (com.baidu.safehttp:safehttp:1.0.29)
* A：到https://bintray.com/bdaiotsecurity 或者咨询相应的开发人员获取aidl-1.0.0、safehttp-1.0.13以及saferequest-1.2.1.3等aar包。
```c
并且替换
    implementation "com.baidu.adsdk:saferequest:1.2.1.3"
为：
    implementation(name: 'aidl-1.0.0', ext: 'aar')
    implementation(name: 'safehttp-1.0.29', ext: 'aar')
    implementation(name: 'saferequest-1.2.1.3', ext: 'aar')
```

* 2)Q: protubuf和宿主版本不一致导致编译失败。
* A: SDK版本用的是protubuf3.0，会用到一些新特性，但是也向下兼容，宿主版本如果之前用的是3.0以下的，先clean一下工程，把之前protubuf自动生成的代码先清掉，再用3.0重新编译即可。

* 3)Q: 使用BDGalleryView崩溃，抛出NoXXXFoundException。
* A: 这个是因为宿主用的Glide版本和SDK不一致导致的，Glide版本间差异会比较大，不同版本引入的新特性不一定能兼容。

* 4) Q: NullPointerException异常，并且堆栈在：EncryptUtilWrapper的。
* A： 看下app的Manifest文件中Application节点中是否加入了:process属性。
      一定要在application onCreate中调用我们sdk。

* 5) Q: VerifyError kotlin/UByte
* A： 需要调整gradle和gradle plugin的版本
      
   工程根目录/build.gradle
      
      buildscript {
          dependencies {
	      classpath 'com.android.tools.build:gradle:3.4.1' // 要求不低于此版本
	  }
      }
      
     gradle-wrapper.properties
     
      distributionUrl=https\://services.gradle.org/distributions/gradle-5.1.1-all.zip
     同样要求不低于此版本

* 6) Q:播放出现黑屏
* A: 出现黑屏大概有几个原因：
     ①两个单独的播放组件在做切换时没有调用controller.setViewVisibility去控制组件的显示和隐藏；
     ②宿主控制组件的时候，调用了removeView等相关的操作，此类操作会导致组件的onDetachFromWindow被回调，EGLContext被销毁，渲染的线程停止，无法再恢复，相当于整个组件被废弃。(一般会出现"invalid EGL"字样的日志)
     
* 7） Q:UnstaticLinkedError等一些Native方法找不到的错误。
  A: SDK只提供了armeabi-v7a和armeabi-v8a两种架构的库，出现这种问题可能是宿主存在armeabi的架构。
