# tryfix
Android热修复框架

# 使用实例

## app build.gradle

```
repositories {
    jcenter()
    ...
    maven { url "https://github.com/johanmans/tryfix/raw/master" }
}
dependencies {
    ...
    compile('com.johan:tryfix:1.0')
}
```

## Application

```
public class App extends Application {

    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
        // 开启调试模式
        TryFix.debug();
        // 打补丁
        TryFix.patch(base);
    }

}
```

## Activity

```
public class MainActivity extends AppCompatActivity implements PermissionHelper.OnPermissionCallback {

    private TextView testView;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        testView = (TextView) findViewById(R.id.test_view);
        testView.setText("=============>" + Test.getInfo() + "\n" + new Error().error());
        PermissionHelper.requestPermission(this, new String[]{Manifest.permission.READ_EXTERNAL_STORAGE, Manifest.permission.WRITE_EXTERNAL_STORAGE}, this);
        // 务必要注册FixReceiver
        FixReceiver.register(this, fixReceiver);
    }

    @Override
    protected void onDestroy() {
        FixReceiver.unregister(this, fixReceiver);
        super.onDestroy();
    }

    private FixReceiver fixReceiver = new FixReceiver() {
        @Override
        protected void onReceiveResult(int version, int code, String message) {

        }
    };

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        PermissionHelper.handlePermissionResult(requestCode, permissions, grantResults, this);
    }

    @Override
    public void onPermissionAccept(int requestCode, String... permissions) {
        // 下载新的Patch文件并进行修复
        // 参数1 Context
        // 参数2 要Patch版本号
        // 参数3 Patch文件下载网址
        TryFix.fix(this, 1, "http://99.56888.net/wlapp/download/test/patch.apk");
    }

    @Override
    public void onPermissionRefuse(int requestCode, String... permission) {

    }

}
```

 ## AndroidManifest
 
 ```
 <manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.johan.fix">

    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:name=".App"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">

        ...

        <!-- 最好放在新的进程中 android:process=":patch" -->
        <service android:name="com.johan.tryfix.service.FixService" android:process=":patch" />

    </application>

</manifest>
 ```
