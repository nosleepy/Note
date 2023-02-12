---
title: DocumentsUI分析
date: 2023-01-01 19:06:13
tags:
categories:
- 安卓
---

**源码位置：**/packages/apps/DocumentsUI

**主页面**

![主页面](https://raw.githubusercontent.com/nosleepy/picture/master/img/documentsUI_%E7%95%8C%E9%9D%A2.png)

#### 1.主体结构

![documentsUI](https://raw.githubusercontent.com/nosleepy/picture/master/img/documentsUI_%E7%BB%93%E6%9E%84.png)

如上图所示DocumentsUI 的主界面，由BaseActivity+FilesActivity+RootsFragment+DirectoryFragment 组成，数据均由Loader机制和AsycTask机制进行加载，很多Activity 自身不充当具体的UI容器，仅仅是用于Fragment的容器，具体的内容由Fragment进行展示。

#### 2.FilesActivity

布局文件是DrawerLayout，左边是侧滑菜单，右边是内容显示

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/coordinator_layout">

    <androidx.drawerlayout.widget.DrawerLayout
        android:id="@+id/drawer_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.coordinatorlayout.widget.CoordinatorLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">
			<!-- 内容显示布局 DirectoryFragment -->
        </androidx.coordinatorlayout.widget.CoordinatorLayout>

        <LinearLayout
            android:id="@+id/drawer_roots"
            android:layout_width="256dp"
            android:layout_height="match_parent"
            android:layout_gravity="start"
            android:orientation="vertical"
            android:elevation="0dp"
            android:background="?android:attr/colorBackground">
			<!-- 侧滑菜单布局 RootsFragment -->
        </LinearLayout>

    </androidx.drawerlayout.widget.DrawerLayout>

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

#### 3.RootsFragment

```java
public static RootsFragment show(FragmentManager fm, boolean includeApps, Intent intent) {
    final Bundle args = new Bundle();
    args.putBoolean(EXTRA_INCLUDE_APPS, includeApps);
    args.putParcelable(EXTRA_INCLUDE_APPS_INTENT, intent);
    final RootsFragment fragment = new RootsFragment();
    fragment.setArguments(args);
    final FragmentTransaction ft = fm.beginTransaction();
    ft.replace(R.id.container_roots, fragment);
    ft.commitAllowingStateLoss();
    return fragment;
}
```

show 方法显示出 RootsFragment 自己，RootsFragment 就是侧滑菜单部分，RootsFramgment 加载时由 Loader 机制把数据加载到 ListView 的 RootsAdapter。

```java
private final OnItemClickListener mItemListener = new OnItemClickListener() {
    @Override
    public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
        final Item item = mAdapter.getItem(position);
        item.open();
        getBaseActivity().setRootsDrawerOpen(false);
    }
};
```

选中 RootItem 时首先触发 BaseActivity#onRootPicked ，更新当前内容显示。

```java
@Override
public void onRootPicked(RootInfo root) {
    mSearchManager.cancelSearch();
    if (root.equals(getCurrentRoot()) && mState.stack.size() <= 1) {
        return;
    }
    mInjector.actionModeController.finishActionMode();
    mSortController.onViewModeChanged(mState.derivedMode);
    mState.sortModel.setDimensionVisibility(
            SortModel.SORT_DIMENSION_ID_SUMMARY,
            root.isRecents() || root.isDownloads() ? View.VISIBLE : View.INVISIBLE);
    mState.stack.changeRoot(root);
    if (mProviders.isRecentsRoot(root)) {
        refreshCurrentRootAndDirectory(AnimationView.ANIM_NONE);
    } else {
        mInjector.actions.getRootDocument(
                root,
                TimeoutTask.DEFAULT_TIMEOUT,
                doc -> mInjector.actions.openRootDocument(doc));
    }
    expandAppBar();
    updateHeaderTitle();
}
```

#### 4.DirectoryFragment

```java
public static void showDirectory(
        FragmentManager fm, RootInfo root, DocumentInfo doc, int anim) {
    if (DEBUG) {
        Log.d(TAG, "Showing directory: " + DocumentInfo.debugString(doc));
    }
    create(fm, root, doc, anim);
}

public static void showRecentsOpen(FragmentManager fm, int anim) {
    create(fm, null, null, anim);
}

public static void create(
        FragmentManager fm,
        RootInfo root,
        @Nullable DocumentInfo doc,
        @AnimationType int anim) {
    if (DEBUG) {
        if (doc == null) {
            Log.d(TAG, "Creating new fragment null directory");
        } else {
            Log.d(TAG, "Creating new fragment for directory: " + DocumentInfo.debugString(doc));
        }
    }
    final Bundle args = new Bundle();
    args.putParcelable(Shared.EXTRA_ROOT, root);
    args.putParcelable(Shared.EXTRA_DOC, doc);
    final FragmentTransaction ft = fm.beginTransaction();
    AnimationView.setupAnimations(ft, anim, args);
    final DirectoryFragment fragment = new DirectoryFragment();
    fragment.setArguments(args);
    ft.replace(getFragmentId(), fragment);
    ft.commitAllowingStateLoss();
}
```

showDirectory 和 showRecentsOpen 方法显示 DirectoryFragment 自己，DirectoryFragment 就是内容显示部分。选中 ListItem 时触发 BaseActivity#refreshCurrentRootAndDirectory ，更新层级目录和文件显示。

```java
public final void refreshCurrentRootAndDirectory(int anim) {
    mSearchManager.cancelSearch();
    if (mHasQueryContentFromIntent) {
        mHasQueryContentFromIntent = false;
        mSearchManager.setCurrentSearch(mSearchManager.getQueryContentFromIntent());
    }
    mState.derivedMode = LocalPreferences.getViewMode(this, mState.stack.getRoot(), MODE_GRID);
    mNavigator.update();
    refreshDirectory(anim);
    final RootsFragment roots = RootsFragment.get(getSupportFragmentManager());
    if (roots != null) {
        roots.onCurrentRootChanged();
    }
    String appName = getString(R.string.files_label);
    String currentTitle = getTitle() != null ? getTitle().toString() : "";
    if (currentTitle.equals(appName)) {
        getWindow().getDecorView().announceForAccessibility(appName);
    }
    String newTitle = mState.stack.getTitle();
    if (newTitle != null) {
        setTitle(newTitle);
    }
    invalidateOptionsMenu();
    mSortController.onViewModeChanged(mState.derivedMode);
    mSearchManager.updateChips(getCurrentRoot().derivedMimeTypes);
    mAppsRowManager.updateView(this);
}
```

#### 5.参考

[documentsUI源码分析](https://segmentfault.com/a/1190000010304287?utm_source=sf-similar-article)

[Android 进阶——Framework 核心之Android Storage Access Framework（SAF）存储访问框架机制详解（一）](https://crazymo.blog.csdn.net/article/details/108555094)

[Android 进阶——Framework 核心之Android Storage Access Framework（SAF）存储访问框架机制详解（二）](https://crazymo.blog.csdn.net/article/details/108555196#t16)
