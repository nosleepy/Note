---
title: Activity应用启动
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

#### Activity启动流程

![](https://raw.githubusercontent.com/nosleepy/picture/master/img/android_app_startup_process.png)

Zygote 进程 fork 出 App 进程之后，会调用 ActivityThread 的 main 方法。

```java
//ActivityThread.java
    public static void main(String[] args) {
        Looper.prepareMainLooper();
        ActivityThread thread = new ActivityThread();
        thread.attach(false, startSeq);
        Looper.loop();
    }
```

`attach` 方法调用 AMS 的 `attachApplication` 方法通知 AMS 创建 Application

```java
//ActivityThread.java
private void attach(boolean system, long startSeq) {
            final IActivityManager mgr = ActivityManager.getService();
            try {
                mgr.attachApplication(mAppThread, startSeq);
            } catch (RemoteException ex) {
                throw ex.rethrowFromSystemServer();
            }
        }
```

mAppThread 是一个 ApplicationThread 类型的 Binder 对象，用于 AMS 处理完结果向 App 进程发送信息

```java
//ActivityThread.java
private class ApplicationThread extends IApplicationThread.Stub {
	
}
```

AMS 处理 App 进程的 attachApplication 请求

```java
//ActivityManagerService.java
    public final void attachApplication(IApplicationThread thread, long startSeq) {
        if (thread == null) {
            throw new SecurityException("Invalid application interface");
        }
        synchronized (this) {
            int callingPid = Binder.getCallingPid();
            final int callingUid = Binder.getCallingUid();
            final long origId = Binder.clearCallingIdentity();
            attachApplicationLocked(thread, callingPid, callingUid, startSeq);
            Binder.restoreCallingIdentity(origId);
        }
    }
```

最后会调用到 thread.bindApplication 方法通知 App 进程创建 Application 对象

```
//ApplicationThread.java
public final void bindApplication() {           
    sendMessage(H.BIND_APPLICATION, data);
}
```

通过 Handler 发送一条创建 Application 的 BIND_APPLICATION 消息

```java
//H.java
class H extends Handler {
    public void handleMessage(Message msg) {
            switch (msg.what) {
                case BIND_APPLICATION:
                    AppBindData data = (AppBindData)msg.obj;
                    handleBindApplication(data);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
            }
    }
}
```

在 handleBindApplication 方法中反射创建 Application，并且回调 onCreate 方法

```java
//ActivityThread.java
private void handleBindApplication(AppBindData data) {
		Application app;
        final StrictMode.ThreadPolicy savedPolicy = StrictMode.allowThreadDiskWrites();
        final StrictMode.ThreadPolicy writesAllowedPolicy = StrictMode.getThreadPolicy();
        try {
            app = data.info.makeApplication(data.restrictedBackupMode, null);
            app.setAutofillOptions(data.autofillOptions);
            app.setContentCaptureOptions(data.contentCaptureOptions);
            mInitialApplication = app;
            try {
                mInstrumentation.callApplicationOnCreate(app);
            } catch (Exception e) {
                if (!mInstrumentation.onException(app, e)) {
                    throw new RuntimeException(
                      "Unable to create application " + app.getClass().getName()
                      + ": " + e.toString(), e);
                }
            }
        } finally {
            if (data.appInfo.targetSdkVersion < Build.VERSION_CODES.O_MR1
                    || StrictMode.getThreadPolicy().equals(writesAllowedPolicy)) {
                StrictMode.setThreadPolicy(savedPolicy);
            }
        }
}
```

同理 Activity 的创建也是 AMS 通知 App 进程调用 handleLaunchActivity 方法

```java
//ActivityThread.java
public Activity handleLaunchActivity(ActivityClientRecord r,
            PendingTransactionActions pendingActions, Intent customIntent) {
        final Activity a = performLaunchActivity(r, customIntent);
        return a;
}
    
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        ContextImpl appContext = createBaseContextForActivity(r);
        Activity activity = null;
        try {
            java.lang.ClassLoader cl = appContext.getClassLoader();
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
        } catch (Exception e) {
        }
        try {
            Application app = r.packageInfo.makeApplication(false, mInstrumentation);
            if (activity != null) {
                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback,
                        r.assistToken);
                if (r.isPersistable()) {
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
            }
            r.setState(ON_CREATE);
            }
        } catch (SuperNotCalledException e) {
            throw e;
        } 
        return activity;
    }
```

#### Activity、Window、View的关系

从 setContentView 开始分析

```java
//Activity.java
    public void setContentView(@LayoutRes int layoutResID) {
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
    }
    
    public Window getWindow() {
        return mWindow;
    }
```

Activity 的 setContentView 方法会调用 getWindow() 的 setContentView 方法，mWindow 在 Activity 的回调 attach 方法中创建

```java
//Activity.java
final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken,
            IBinder shareableActivityToken) {
        mWindow = new PhoneWindow(this, window, activityConfigCallback);
        mWindow.setWindowControllerCallback(mWindowControllerCallback);
        mWindow.setCallback(this);
        mWindow.setOnWindowDismissedCallback(this);
        mWindow.getLayoutInflater().setPrivateFactory(this);
        mToken = token;
        mLastNonConfigurationInstances = lastNonConfigurationInstances;
        mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
        if (mParent != null) {
            mWindow.setContainer(mParent.getWindow());
        }
        mWindowManager = mWindow.getWindowManager();
        mCurrentConfig = config;
        mWindow.setColorMode(info.colorMode);
        mWindow.setPreferMinimalPostProcessing(
                (info.flags & ActivityInfo.FLAG_PREFER_MINIMAL_POST_PROCESSING) != 0);
        getAutofillClientController().onActivityAttached(application);
        setContentCaptureOptions(application.getContentCaptureOptions());
    }
```

mWindow 和 WindowManager 建立关系

```java
//Window.java
    public void setWindowManager(WindowManager wm, IBinder appToken, String appName,
            boolean hardwareAccelerated) {
        mAppToken = appToken;
        mAppName = appName;
        mHardwareAccelerated = hardwareAccelerated;
        if (wm == null) {
            wm = (WindowManager)mContext.getSystemService(Context.WINDOW_SERVICE);
        }
        mWindowManager = ((WindowManagerImpl)wm).createLocalWindowManager(this);
    }
```

```java
//WindowManagerImpl.java
    public WindowManagerImpl createLocalWindowManager(Window parentWindow) {
        return new WindowManagerImpl(mContext, parentWindow, mWindowContextToken);
    }
    
  private WindowManagerImpl(Context context, Window parentWindow,
            @Nullable IBinder windowContextToken) {
        mContext = context;
        mParentWindow = parentWindow;
        mWindowContextToken = windowContextToken;
    }
```

Window 是一个抽象类，唯一实现是 PhoneWindow

```java
public class PhoneWindow extends Window implements MenuBuilder.Callback {
	    public void setContentView(int layoutResID) {
        if (mContentParent == null) {
            installDecor();
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        mContentParent.requestApplyInsets();
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
    }
    
    private void installDecor() {
        mForceDecorInstall = false;
        if (mDecor == null) {
            mDecor = generateDecor(-1);
            mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
            mDecor.setIsRootNamespace(true);
        } else {
            mDecor.setWindow(this);
        }
        if (mContentParent == null) {
            mContentParent = generateLayout(mDecor);
            }
        }
    }
    
   protected DecorView generateDecor(int featureId) {
        return new DecorView(context, featureId, this, getAttributes());
    }
    
    protected ViewGroup generateLayout(DecorView decor) {
        ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
        if (contentParent == null) {
            throw new RuntimeException("Window couldn't find content container view");
        }
        return contentParent;
    }
}
```

PhoneWindow 的 setContentView 方法内部会实例化 DecorView，并且为 mContentParent 变量赋值，mContentParent 是 DecorView 中 id 为 ID_ANDROID_CONTENT 的一个 View，然后使用 mLayoutInflater 将布局文件加载到 mContentParent 中

setContentView 是在 onCreate 中调用的，此时视图还未呈现，回到 ActivityThread 的 handleResumeActivity 中

```java
//ActivityThread.java
public void handleResumeActivity(IBinder token, boolean finalStateRequest, boolean isForward,
            String reason) {
        final ActivityClientRecord r = performResumeActivity(token, finalStateRequest, reason); //回调Activity的onResume方法
        final Activity a = r.activity;
        if (r.window == null && !a.mFinished && willBeVisible) {
            r.window = r.activity.getWindow();
            View decor = r.window.getDecorView();
            decor.setVisibility(View.INVISIBLE);
            ViewManager wm = a.getWindowManager();
            WindowManager.LayoutParams l = r.window.getAttributes();
            a.mDecor = decor;
            l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;
            l.softInputMode |= forwardBit;
            if (r.mPreserveWindow) {
                a.mWindowAdded = true;
                r.mPreserveWindow = false;
                ViewRootImpl impl = decor.getViewRootImpl();
                if (impl != null) {
                    impl.notifyChildRebuilt();
                }
            }
        } else if (!willBeVisible) {
            if (localLOGV) Slog.v(TAG, "Launch " + r + " mStartedActivity set");
            r.hideForNow = true;
        }
    }

public ActivityClientRecord performResumeActivity(IBinder token, boolean finalStateRequest,
            String reason) {
        final ActivityClientRecord r = mActivities.get(token);
  		r.activity.performResume(r.startsNotResumed, reason);
        return r;
    }
```

关键代码 `wm.addView(decor, l)` 中调用 WindowManager 的 addView 方法

```java
public interface ViewManager {
    void addView(View var1, LayoutParams var2);

    void updateViewLayout(View var1, LayoutParams var2);

    void removeView(View var1);
}

public interface WindowManager extends ViewManager {

}
```

WindowManager 的实现为 WindowManagerImpl

```java
public final class WindowManagerImpl implements WindowManager {
		private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();
		
	    public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
        applyDefaultToken(params);
        mGlobal.addView(view, params, mContext.getDisplayNoVerify(), mParentWindow,
                mContext.getUserId());
    }
}
```

委托给 mGlobal 来调用，WindowManagerGlobal 是一个单例对象

```java
public final class WindowManagerGlobal {
	public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow, int userId) {
        final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
        ViewRootImpl root;
        View panelParentView = null;
        synchronized (mLock) {
            root = new ViewRootImpl(view.getContext(), display);
            view.setLayoutParams(wparams);
            mViews.add(view);
            mRoots.add(root);
            mParams.add(wparams);
            try {
                root.setView(view, wparams, panelParentView, userId);
            } catch (RuntimeException e) {
                if (index >= 0) {
                    removeViewLocked(index, true);
                }
                throw e;
            }
        }
    }
}
```

创建 ViewRootImpl 对象 root，将 view、root、wparams 等信息存到集合中，最后调用 root.setView 方法

```java
//ViewRootImpl.java
public final class ViewRootImpl implements ViewParent,
        View.AttachInfo.Callbacks, ThreadedRenderer.DrawCallbacks {
    final W mWindow;
            
   public ViewRootImpl(Context context, Display display, IWindowSession session,
            boolean useSfChoreographer) {
       mWindow = new W(this); //和ActivityThread中的H类一样用于WMS和ViewRootImpl通信
   }
            
    public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView,
            int userId) {
        synchronized (this) {
            if (mView == null) {
                mView = view;
                requestLayout(); // 开始View的测量、布局、绘制
                try {
                    res = mWindowSession.addToDisplayAsUser(mWindow, mSeq, mWindowAttributes,
                            getHostVisibility(), mDisplay.getDisplayId(), userId, mTmpFrame,
                            mAttachInfo.mContentInsets, mAttachInfo.mStableInsets,
                            mAttachInfo.mDisplayCutout, inputChannel,
                            mTempInsets, mTempControls);
                    setFrame(mTmpFrame);
                } catch (RemoteException e) {
                    throw new RuntimeException("Adding window failed", e);
                } finally {
                    if (restore) {
                        attrs.restore();
                    }
                }
            }
        }
    }
            
static class W extends IWindow.Stub {
	private final WeakReference<ViewRootImpl> mViewAncestor;
	private final IWindowSession mWindowSession;

	 W(ViewRootImpl viewAncestor) {
		 mViewAncestor = new WeakReference<ViewRootImpl>(viewAncestor);
		mWindowSession = viewAncestor.mWindowSession;
	}
 }
}
```

内部调用 requestLayout 方法完成 View 的测量、布局、绘制，然后调用 mWindowSession 的 addToDisplayAsUser 方法通知 WMS 添加 Window 视图

```java
//WindowManagerGlobal.java
    public static IWindowSession getWindowSession() {
        synchronized (WindowManagerGlobal.class) {
            if (sWindowSession == null) {
                try {
                    InputMethodManager.ensureDefaultInstanceForDefaultDisplayIfNecessary();
                    IWindowManager windowManager = getWindowManagerService();
                    sWindowSession = windowManager.openSession(
                            new IWindowSessionCallback.Stub() {
                                @Override
                                public void onAnimatorScaleChanged(float scale) {
                                    ValueAnimator.setDurationScale(scale);
                                }
                            });
                } catch (RemoteException e) {
                    throw e.rethrowFromSystemServer();
                }
            }
            return sWindowSession;
        }
    }

    public static IWindowManager getWindowManagerService() {
        synchronized (WindowManagerGlobal.class) {
            if (sWindowManagerService == null) {
                sWindowManagerService = IWindowManager.Stub.asInterface(
                        ServiceManager.getService("window"));
                try {
                    if (sWindowManagerService != null) {
                        ValueAnimator.setDurationScale(
                                sWindowManagerService.getCurrentAnimatorScale());
                        sUseBLASTAdapter = sWindowManagerService.useBLAST();
                    }
                } catch (RemoteException e) {
                    throw e.rethrowFromSystemServer();
                }
            }
            return sWindowManagerService;
        }
    }
```

mWindowSession 是 WindowManagerGlobal 的成员，是一个 Binder 对象

```java
//Sesstion.java
class Session extends IWindowSession.Stub implements IBinder.DeathRecipient {
	    public int addToDisplayAsUser(IWindow window, int seq, WindowManager.LayoutParams attrs,
            int viewVisibility, int displayId, int userId, Rect outFrame,
            Rect outContentInsets, Rect outStableInsets,
            DisplayCutout.ParcelableWrapper outDisplayCutout, InputChannel outInputChannel,
            InsetsState outInsetsState, InsetsSourceControl[] outActiveControls) {
        return mService.addWindow(this, window, seq, attrs, viewVisibility, displayId, outFrame,
                outContentInsets, outStableInsets, outDisplayCutout, outInputChannel,
                outInsetsState, outActiveControls, userId);
    }
}
```

调用 WMS 的 addWindow 方法来添加 Window

```java
//WindowManagerService.java
public class WindowManagerService extends IWindowManager.Stub
        implements Watchdog.Monitor, WindowManagerPolicy.WindowManagerFuncs {
            public int addWindow(Session session, IWindow client, int seq,
            LayoutParams attrs, int viewVisibility, int displayId, Rect outFrame,
            Rect outContentInsets, Rect outStableInsets,
            DisplayCutout.ParcelableWrapper outDisplayCutout, InputChannel outInputChannel,
            InsetsState outInsetsState, InsetsSourceControl[] outActiveControls,
            int requestUserId) {
            	//...
            }
 }
```

#### 总结

![](https://raw.githubusercontent.com/nosleepy/picture/master/img/window_manager_service_process.png)

+ Window表示一个窗口，这是一个抽象类，具体的实现是PhoneWindow，可以通过WindowManager来创建一个Window。
+ Activity是Android四大组件，主要和用户进行交互，View是视图对象。
+ Android中所有的视图都是通过Window来呈现的，不管是Activity还是Dialog都是依附在Window上的。
+ 当启动Activity通过setContentView()方法来添加布局的时候实际上是调用PhoneWindow的setContentView方法将布局添加到DecorView中，DecorView是Activity的顶级View，是一个FrameLayout，内部包含了标题栏和内容栏，而内容栏的id就是content，这也是setContentView方法命名的原因。最后通过布局加载器LayoutInflater生成View对象然后通过addView()方法添加到Window上。而Window内部有一个ViewGroup容器，addView()方法就是将View对象添加到容器中。
