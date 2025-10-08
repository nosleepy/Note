## 一、功能简介


C03 充当 USB 摄像头，使用 HDMI - USB 转接线连接到机顶盒，机顶盒再通过显示器输出 C03 画面内容。使用安卓 Camera2 以及 AudioRecord 和 AudioTrack 完成视频和音频的传输。


## 二、SDK 使用指南


1. 导入 SDK


将 classes.jar 拷贝至 Android 工程的 libs 目录下，修改 build.gradle 文件，编译项目。


```plain
dependencies {
    implementation files('libs/classes.jar') // add
}
```


1. 主要接口


```java
interface ICameraService {
    void startC03Preview(); // 开启C03预览
    void stopC03Preview(); // 关闭C03预览
}
```


1. 添加用户权限


在工程 AndroidManifest.xml 文件中添加如下权限，注册 service。


```xml
<uses-permission android:name="android.permission.CAMERA"/>
<uses-permission android:name="android.permission.RECORD_AUDIO"/>

<service android:name="com.android.api.CameraService" android:exported="true"/>
```


1. 布局文件添加 TextureView


android:layout_width 和 android:layout_height 指定画面显示大小。


```xml
<TextureView
    android:id="@+id/tv"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```


1. 示例 Activity


```java
public class CameraActivity extends AppCompatActivity {
    private static final String TAG = "wlzhou";
    private static final String[] REQUIRED_PERMISSIONS = {Manifest.permission.CAMERA, Manifest.permission.RECORD_AUDIO};
    private static final int REQUEST_PERMISSIONS_CODE = 1;
    private TextureView textureView;
    private ICameraService cameraService;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        Log.d(TAG, "[onCreate]");
        super.onCreate(savedInstanceState);
        hideTitle();
        setContentView(R.layout.activity_camera);
        initView();
    }
    @Override
    protected void onStart() {
        super.onStart();
        Log.d(TAG, "[onStart]");
        // 绑定服务
        bindCameraService();
    }
    @Override
    protected void onStop() {
        Log.d(TAG, "[onStop]");
        super.onStop();
        try {
            cameraService.stopC03Preview();
        } catch (RemoteException e) {
            e.printStackTrace();
        }
        // 解绑服务
        unBindCameraService();
    }
    // 设置窗口没有标题
    private void hideTitle() {
        if (getSupportActionBar() != null){
            getSupportActionBar().hide();
        }
    }
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        boolean flag = true;
        if (requestCode == REQUEST_PERMISSIONS_CODE) {
            for (int i = 0; i < grantResults.length; i++) {
                if (grantResults[i] == -1) {
                    flag = false;
                    break;
                }
            }
        }
        if (flag) {
            try {
                cameraService.startC03Preview();
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        } else {
            Toast.makeText(CameraActivity.this, "no permission", Toast.LENGTH_SHORT).show();
        }
    }
    private void initView() {
        textureView = findViewById(R.id.tv);
        textureView.setSurfaceTextureListener(new TextureView.SurfaceTextureListener() {
            @Override
            public void onSurfaceTextureAvailable(@NonNull SurfaceTexture surfaceTexture, int i, int i1) {
                Toast.makeText(CameraActivity.this, "created", Toast.LENGTH_SHORT).show();
                CameraService.setTextureView(textureView);
            }
            @Override
            public void onSurfaceTextureSizeChanged(@NonNull SurfaceTexture surfaceTexture, int i, int i1) {
            }
            @Override
            public boolean onSurfaceTextureDestroyed(@NonNull SurfaceTexture surfaceTexture) {
                return false;
            }
            @Override
            public void onSurfaceTextureUpdated(@NonNull SurfaceTexture surfaceTexture) {
            }
        });
    }
    private boolean hasPermissions() {
        for (String permission : REQUIRED_PERMISSIONS) {
            if (ContextCompat.checkSelfPermission(CameraActivity.this, permission) == PackageManager.PERMISSION_DENIED) {
                return false;
            }
        }
        return true;
    }
    private void bindCameraService() {
        Intent intent = new Intent(this, CameraService.class);
        startService(intent);
        boolean b = bindService(intent, cameraServiceConnection, BIND_AUTO_CREATE);
        Log.d(TAG, "[bindCameraService] isBind -> " + b);
    }
    private void unBindCameraService() {
        Intent intent = new Intent(this, CameraService.class);
        unbindService(cameraServiceConnection);
        stopService(intent);
    }
    private ServiceConnection cameraServiceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
            Log.d(TAG, "[onServiceConnected]");
            cameraService = ICameraService.Stub.asInterface(iBinder);
            if (!hasPermissions()) {
                requestPermissions(REQUIRED_PERMISSIONS, REQUEST_PERMISSIONS_CODE);
            } else {
                try {
                    cameraService.startC03Preview();
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            }
        }
        @Override
        public void onServiceDisconnected(ComponentName componentName) {
            Log.d(TAG, "onServiceDisconnected");
            cameraService = null;
        }
    };
}
```


## 三、功能操作步骤


使用 HDMI - USB 转接线连接 C03 和机顶盒，完成 C03 到机顶盒的镜像传输。注：HDMI 端连接 C03，USB 端连接机顶盒。


![图片](data:image/png;base64,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)

## 四、完整 Demo 下载


[https://github.com/hiwlzhou/ImageTransmission/tree/master/app_camera](https://github.com/hiwlzhou/ImageTransmission/tree/master/app_camera)

[https://github.com/hiwlzhou/ImageTransmission/tree/master/app_api](https://github.com/hiwlzhou/ImageTransmission/tree/master/app_api)

