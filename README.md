# 百度聚屏广告SDK接入文档

## 产品介绍
百度聚屏广告SDK，集成了高兼容性的广告播放器，支持高清流畅播放mp4、jpg等主流图片/视频广告格式，并集成了百度安全多年积累的安全技术，具备完善的监播机制和能力，可全面保障媒体终端广告播放的流畅性、安全性和真实性。

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
   * 若接入设备的广告类型为OTT或开屏广告，则无需准备该字段；
* channel：渠道名，一般在交付邮件中将会列出；

以上信息若有疑问，可联系我们获取。


## 集成
在根gradle文件下加入：
```javascript { .theme-peacock }

allprojects {
    repositories {
        google()
        jcenter()
        maven { url 'https://dl.bintray.com/bdaiotsecurity/maven' }
    }
}
```


在app的gradle下加入：
```c
repositories {
    flatDir {
        dirs 'libs'
    }
}
dependencies {
    // 广告sdk
    implementation (name: 'adsdk_release_vxxx', ext: 'aar')
    // kotlin基础库
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.3.31"
    implementation 'com.baidu.adsdk:saferequest:1.0.0.27'
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


## 初始化(必须）
建议在Application的onCreate中初始化。
```javascript { .theme-peacock }
@Override
public void onCreate() {
    super.onCreate();
    /**
     * @param appSid : 填入聚屏分配的appSid
     * @param channel : 填入聚屏分配的channel
     * @param mediaId :
     * 1. 如果广告位类型是户外，必须填写mediaid（64位以内，仅英文或数字或下划线_）, 需要自己保证设备唯一性。
     * 另外，这个mediaid需要在聚屏方注册。
     * 2. 如果你的广告位类型是ott或者开屏广告，则可以不填写mediaid（填写为null）,sdk将会自动获取mac进行请求。
     */
    AdWrapper.INSTANCE.init(this, "JcFrL3y", "channel"， "mediaId");
}
```

## 参数设置（可选，推荐填写）
下面在AdWrapper中，可通过`AdWrapper.Instance.XXX`访问
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
```

## 身份校验（必须）
以下信息需要给到百度，百度进行入库，否则将请求不到广告。出于安全性的考虑，后台需要对客户端鉴权，防止第三方拿到SDK之后作弊，目的是为了保障客户权益。
### 需要提供应用程序包名
例如：`com.baidu.xxxx`
也就是app得gradle文件中的`applicationId`
### APP签名的md5

```
keytool -printcert -file CERT.RSA
```

签名后从APK解压出来则可以看到此文件，文件放于在META-INF文件夹。
PS：keytool工具为JDK里自带。可在jdk\bin或者jdk\jre\bin目录下找到。
如果有debug运行需求，可以把debug版的签名一并给到我们（一般一台机器一个debug签名，可以按需求加）

## 权限（必须）
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
### xml 布局接入，简单，**推荐**
在Activity/Fragment的layout中像普通view一样布局。
其中公共属性如下：
`slotid`此参数需要向聚屏申请（重要，必填）
`auto` 是否是自动播放广告（选填）
#### 视频组件
```javascript { .theme-peacock }
<com.baidu.adsdk.view.BDVideoView
        android:id="@+id/bd_video_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        BDAdView:slotid="Jy0FoE5qy"
        BDAdView:auto="true" />
```
接入后，视频组件将会一直持续播放，并自动请求广告。

#### 图片轮播组件
额外有一个独立的属性：
`intervalTime`设置x秒换一张图片，设置的时间必须在[5,30]秒中间，如果不在这个范围，默认15秒。
```javascript { .theme-peacock }
<com.baidu.adsdk.view.BDGalleryView
        android:layout_width="match_parent"
        BDAdView:auto="true"
        BDAdView:slotid="6660001"
        BDGalleryView:intervalTime="15"
        android:layout_height="match_parent"/>
```
接入后，图片轮播组件将会持续以固定的intervalTime间隔切换广告。

### 代码布局，媒体方自己通过代码控制广告请求与播放
```javascript { .theme-peacock }
/**
 * 自己控制广告流
 */
public class ManualActivity extends AppCompatActivity {

    Handler handler = new Handler();

    /**
     * 视频广告组件
     */
    BDVideoView videoView;
    /**
     * 视频广告控制器
     */
    IController videoController;

    /**
     * 图片广告组件
     */
    BDGalleryView galleryView;
    /**
     * 图片广告控制器
     */
    IController galController;

    /**
     * 图片广告组件监听器
     */
    private GalListener galListener = new GalListener();

    /**
     * 视频广告组件监听器
     */
    private VideoListener videoListener = new VideoListener();

    class VideoListener extends DefaultAdListener {
        @Override
        public void onAdPrepared(@NotNull RequestInfo info) {
            // 收到视频广告准备OK的时候，可以自己做些业务逻辑，
            // 这个例子是将图片组件隐藏，显示视频组件，然后调用showAd接口
            handler.post(new Runnable() {
                @Override
                public void run() {
                    videoView.setVisibility(View.VISIBLE);
                    galleryView.setVisibility(View.INVISIBLE);
                }
            });

            videoController.showAd();
        }

        @Override
        public void onAdFailed(@NotNull RequestInfo info, int ec, @NotNull String msg) {
            // 当广告播放失败回调，这个例子在播放视频广告失败的时候继续请求视频广告
            videoController.loadAdAsync();
        }

        @Override
        public void onAdFinish(@NotNull RequestInfo requestInfo) {
            // 当广告完成时回调，这个例子在播放视频广告播放完成的时候请求图片广告
            galController.loadAdAsync();
        }
    }

    class GalListener extends DefaultAdListener {
        @Override
        public void onAdPrepared(@NotNull RequestInfo info) {
            // 收到图片广告准备OK的时候，可以自己做些业务逻辑，
            // 这个例子是将视频组件隐藏，显示图片组件，然后调用showAd接口
            handler.post(new Runnable() {
                @Override
                public void run() {
                    videoView.setVisibility(View.INVISIBLE);
                    galleryView.setVisibility(View.VISIBLE);
                }
            });
            galController.showAd();
        }

        @Override
        public void onAdFailed(@NotNull RequestInfo info, int ec, @NotNull String msg) {
            // 当图片广告播放失败回调，这个例子在图片广告失败再请求图片广告
            galController.loadAdAsync();
        }

        @Override
        public void onAdFinish(@NotNull RequestInfo requestInfo) {
            // 当图片广告播放完成回调，这个例子在图片广告播放完成请求视频广告
            videoController.loadAdAsync();
        }
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
        videoController.setSlotid("Jy0FoE5qy");
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

监听接口的具体含义如下：
onAdPrepared  已经下载好了，但是还没有开始播放。preparedinfo里面有广告请求的唯一标识符与广告位id，此时可以通过controller调用showAd
onAdStart  已经开始播放了。（是一个时间点）
onAdFailed和onAdFinish平行，要么成功完成，要么失败
onAdClick点击广告时回调
onAdDismissed比如按了home键或者其他activtiy在这个View上面，这个方法就会被回调


## 附录
### 限制

* 不允许在播放过程中遮盖、隐藏、调整大小广告组件，否则将会影响收益。
* 广告组件大小不能小于300*300px， 否则将会影响收益。

### 错误列表（持续更新）
```javascript { .theme-peacock }
"illegal argument"
"time_out"
"permission_deny"
"url is empty"
"md5 doesn't match"
"not_support_breakpoint"
"unknown_host_exception"
"socket_exception"
"runtimeexception"
"media_file_not_fount"
"get_media_connection_error"
"media_play_unknown_error"

校验错误码：
retCode 为 0x01 包名或者md5不匹配
	   0x02 请求格式不正确或者参数错误
	   0x03 解密失败
	   0x04 鉴权失败
```
