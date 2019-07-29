# 百度聚屏广告SDK接入文档

## 产品介绍
百度聚屏广告SDK，集成了高兼容性的广告播放器，支持高清流畅播放mp4、jpg等主流图片/视频广告格式，并集成了百度安全多年积累的安全技术，具备完善的监播机制和能力，可全面保障媒体终端广告播放的流畅性、安全性和真实性。

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
    implementation 'com.baidu.adsdk:saferequest:1.0.0.8'
    // 视频播放器
    implementation 'com.google.android.exoplayer:exoplayer-core:2.10.1'
    // protobuf
    implementation 'com.google.protobuf:protobuf-lite:3.0.0'
    // okhttp
    releaseImplementation 'com.squareup.okio:okio:1.13.0'
    releaseImplementation 'com.squareup.okhttp3:okhttp:3.9.0'
}
```


## 初始化(必须）
建议在Application的onCreate中初始化。
```javascript { .theme-peacock }
@Override
public void onCreate() {
    super.onCreate();
    // 填入聚屏分配的appid
    AdWrapper.INSTANCE.init(this, "JcFrL3y");
}
```

## 参数设置（可选，推荐填写）
下面在AdWrapper中，可通过`AdWrapper.Instance.XXX`访问
```ruby
/**
 * @param mediaId : 可选，媒体id，要自己保证唯一性。64位以内，仅英文或数字
 * 如果填入这个mediaId,那么设备唯一性就用这个代替mac地址，并且需要联系聚屏备案。
 * 如果不填，SDK将自动获取mac
 */
fun setMediaId(mediaId: String)

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
`keytool -printcert -file CERT.RSA`
签名后从APK解压出来则可以看到此文件，文件放于在META-INF文件夹。
PS：keytool工具为JDK里自带。可在jdk\bin或者jdk\jre\bin目录下找到。
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
目前提供以下两种接入方式。
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
public class VideoViewManualActivity extends Activity implements IAdListener {

    BDVideoView videoView;
    IController controller;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_video_manual);
		
		// 也可以通过LayoutPramters手动布局。
        videoView = findViewById(R.id.bdvv_test);
        controller = videoView.getController();
        
        // 设置slotid,此id需要向聚屏申请
        controller.setSlotid("Jy0FoE5qy");
        // 增加监听器
        controller.addAdListener(this);
        // 请求广告
        controller.loadAdAsync();
    }

    @Override
    // 广告准备完毕
    public void onAdPrepared() {
        controller.showAd();
    }

    @Override
    // 广告开始播放（时间点）
    public void onAdStart() {}

    @Override
    // 广告播放失败，msg是失败原因,详见下面附录中的错误列表
    public void onAdFailed( String msg) {}

    @Override
    // 广告被点击
    public void onAdClick() {}

    @Override
    // 正常播放完毕回调
    public void onAdFinish() {
        controller.releaseAd();
    }

    @Override
    // 如果当前View被挡住，比如按了home键或者其他activtiy在这个View上面，该方法会被回调
    public void onAdDismissed() {}
}
```


## 附录
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
```
