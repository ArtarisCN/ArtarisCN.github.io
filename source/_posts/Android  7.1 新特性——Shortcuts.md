---
title: Android  7.1 新特性——Shortcuts
date: 2018-1-27 16:26:35
tags: [Android]
categories:
- Android
---

#### 简介
Android 7.1 （API level25、Build.VERSION_CODES.N_MR1）中加入了新的 API ——Shortcuts，它提供了一种新的快捷访问方式，类似于 iPhone 的 3D Touch （现已加入豪华名称菜单——三维触控），并且这中快捷方式可以被拖拽到桌面上单独放置。

[吐泡泡 Shortcuts Demo 下载](http://img.artaris.cn/shortcuts/ShortcutsDemo.apk)

<!-- more -->

如下图所示：

![](http://img.artaris.cn/shortcuts/img-2.jpg?imageMogr2/thumbnail/!30p)



#### 分类及作用

Shortcuts 也可以通过静态和动态方式添加。静态的方式可以作为 APP 的一个快捷入口来使用，动态的方式可以作为可变换的选项来使用。

-  **Static shortcuts**：静态shortcuts是在资源文件中定义的，所以你只能通过升级你的app来更新静态shortcuts的相关信息。

- **Dynamic shortcuts**：动态shortcuts是通过ShortcutManager相关的API来实现运行时新增、修改、移除shortcuts的。


#### 静态 Shorcuts(Static shortcuts)

静态 ShortcutsStatic Shortcuts通过在 Manifest 中声明添加。缺点是不可以修改，只能通过应用升级来添加新的静态 Shortcuts。添加主要分为两步：


1 . AndroidManifest.xml 的 Main Launcher 对应的 Activity 内添加 `meta-data meta-data name` 为`android.app.shortcuts`，如下：

```
<activity android:name=".MainActivity">
	<intent-filter>
		<action android:name="android.intent.action.MAIN" />
		<category android:name="android.intent.category.LAUNCHER" />
	</intent-filter>

	<meta-data
		android:name="android.app.shortcuts"
		android:resource="@xml/shortcuts"/>   
</activity>
```
必须在启动界面对应的 Activity 定义 shortcuts ，`android:resource`指向了 shortcuts 的资源文件。

2. 资源文件中定义具体的 shortcuts
res 目录下新建 xml 文件夹，并新建 shortcuts.xml 文件，内容如下：

```
<shortcuts xmlns:android="http://schemas.android.com/apk/res/android">
    <shortcut
        android:shortcutId="settings"
        android:enabled="true"
        android:icon="@mipmap/ic_star_border_black_24dp"
        android:shortcutShortLabel="@string/settings_short_name"
        android:shortcutLongLabel="@string/settings_long_name"
        android:shortcutDisabledMessage="@string/settings_disable_msg">

        <intent
            android:action="android.intent.action.VIEW"
            android:targetPackage="cn.artaris.shortcutsdemo"
            android:targetClass="cn.artaris.shortcutsdemo.MainActivity" />
        <categories android:name="android.shortcut.conversation"/>
    </shortcut>
</shortcuts>
```
以 `<shortcuts>`元素为根，可以包含多个 shortcut 元素，每个 shortcut 元素表示一个 shortcut。其中不同字段的含义为：
*   shortcutId表示 shortcut 唯一标识符，相同的 shortcutId 会被覆盖。必须字段。
*   shortcutShortLabel为将 shortcut 拖动到桌面时显示的名字，官方建议不超过 10 个字符，必须字段。
*   shortcutLongLabel为 shortcut 列表中每个 shortcut 的名字，不宜过长，如果过长或未设置默认会显示 ShortLabel，官方建议不超过 25 个字符。可选字段。
*   icon为 shortcut 的 icon，在列表展示和拖动到桌面时显示需要，可选字段。
*   enabled表示 shortcut 是否可用，false 表示禁用。xml 中这个属性几乎没有被设置为 false 的实际场景，具体原因可见**6.7 如何更好的删除(废弃)老的 Shortcut**中介绍。
*   shortcutDisabledMessage为已固定在桌面的 shortcut 被 Disabled 后点击时的 Toast 提示内容。可选字段。
*   intent为点击 shortcut 时响应的 Intent，必须字段。这里可以添加多个 Intent，成为一个 Intent 栈，点击会启动最后一个 Intent，在这个 Intent 回退时会启动它前面一个 Intent，相当于自动将所有 Intent 添加到了堆栈。对于先跳转到某个页面，Back 键希望退回主页而不是结束 App 这类场景，多个 Intents 挺实用的。

> **intent可设置属性包括：**
`android:action`、`android:data`、`android:mimeType`、`android:targetClass`、`android:targetPackage`。其中`android:action`为必须属性。

#### 动态 Shorcuts(Dynamic shortcuts)

动态 ShortcutsDynamic Shortcuts 通过 ShortcutManager API 进行操作。可以动态添加、修改、删除。


```
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N_MR1) {
            /*
            * 获取 ShortcutManager 管理 shortcuts
            */
            mShortcutManager = getSystemService(ShortcutManager.class);
            /*
            * 获取 ShortcutManager 支持的最多 shortcuts 个数
            */
            mMaxShortcutCount = mShortcutManager.getMaxShortcutCountPerActivity();
        } else {
            new AlertDialog.Builder(mContext)
                    .setTitle(getString(R.string.main_dialog_simple_title))
                    .setMessage(getString(R.string.main_dialog_simple_message))
                    .setNeutralButton(getString(R.string.dialog_confirm), new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            finish();
                        }
                    })
                    .show();
        }
```
创建新的 shortcut 的方法为：

```
Icon icon = Icon.createWithResource(mContext, R.mipmap.ic_favorite_black_24dp);   Intent intent = new Intent(mContext, ShowShortcutsActivity.class); intent.setAction(Intent.ACTION_VIEW); intent.putExtra("msg", singleChoiceItems[itemSelected]);

ShortcutInfo shortcut = new ShortcutInfo.Builder(mContext, String.valueOf(itemSelected))
        .setShortLabel(singleChoiceItems[itemSelected])
        .setLongLabel(singleChoiceItems[itemSelected])
        .setIcon(icon)
        .setIntent(intent)
        .build();
```
添加的方法有：
*   `setDynamicShortcuts(List<ShortcutInfo>)`可以替换并添加所有 shortcut 列表；
*   `addDynamicShortcuts(List<ShortcutInfo>)`可以添加新的 shortcut 到列表，超过最大个数会报异常；
*   `updateShortcuts(List<ShortcutInfo>)`可以更新一组 shortcuts；
*   `removeDynamicShortcuts(List<ShortcutInfo>)`和removeAllDynamicShortcuts() 可以删除部分或所有 shortcuts。

ShortcutInfo 的属性与 xml 中定义字段含义一致，shortcutId shortcutShortLabel intent 是必须设置的字段，并且intent必须设置Action。

#### 固定的 Shortcuts(Pinned Shortcuts)
Pinned Shortcuts 是指将 Shortcut 固定到桌面的功能，如下图所示：
![](http://img.artaris.cn/shortcuts/img-1.jpg?imageMogr2/thumbnail/!30p)
由于固定到了桌面，APP 不能再添加、修改、删除这些这些 Shortcut ，只能禁用这些 Shortcut 。即便 App 内删除了某个 Shorcut，对应的已固定到桌面的 Shortcuts 也不会被删除。

可以通过：

1.  `getPinnedShortcuts()`得到所有固定的 Shortcuts 的信息。
2.  `disableShortcuts(List)`或`disableShortcuts(List, CharSequence)`禁用动态的 Shortcuts。

对于静态的 Shortcuts 需要在资源文件中设置`android:enabled="false"`进行禁用，不过静态 Shortcuts 可直接通过删除达到禁用的效果.

#### 其他

- 动态 Shortcuts 与静态 Shortcuts 区别

1.  静态 Shortcuts 只能通过升级应用修改，动态 Shortcuts 随时可以修改；
2.  静态 Shortcuts 的 Intent 无法设置 Flag，默认为`FLAG_ACTIVITY_NEW_TASK`和`FLAG_ACTIVITY_CLEAR_TASK Flag`，即若应用运行中会清除所有已存在的 Activity。动态 Shortcuts 的 Intent 可以设置 Flag；
3.  静态 Shortcuts 的rank系统默认根据声明顺序设置，动态 Shortcuts 的rank可以通过setRank(int rank)接口主动设置，rank 不能小于 0，值越大表示在 shortcut 列表展示时离 App Icon 越远。静态 Shortcuts 默认比动态 Shortcuts 离 App Icon 更近。
4.  静态 Shortcuts 删除可以直接删除，动态 Shortcuts 建议通过禁用删除；
