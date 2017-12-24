---
layout: post
title: ADB 使用指南
date: 2017-12-03 21:49:22
tags: [Android]
categories:
- Android
---
ADB 的全称是 Android Debug Bridge,在以前仅仅是用它来安装应用、存取文件等简单操作。细细学习了解之后才发现 ADB 还能实现很多强大的功能。在此总结一下 ADB 的一些常用功能

<!-- more -->

* [语法及连接](#语法及连接)
	* [语法](#语法)
	* [连接](#连接)
		* [USB 连接](#usb-连接)
		* [无线连接](#无线连接)
* [获取设备信息](#获取设备信息)
	* [型号](#型号)
	* [屏幕分辨率](#屏幕分辨率)
	* [屏幕密度](#屏幕密度)
	* [IP 地址](#ip-地址)
	* [IMEI](#IMEI)
	* [Android 系统版本](#android-系统版本)
	* [CPU 信息](#cpu-信息)
* [文件管理](#文件管理)
	* [拉取](#拉取)
	* [推送](#推送)
* [应用管理](#应用管理)
	* [查看设备安装应用](#查看设备安装应用)
		* [所有应用](#所用应用)
		* [系统级应用](#系统级应用)
		* [用户安装应用](#用户安装应用)
		* [过滤显示](#过滤显示)
	* [应用安装](#应用安装)
	* [应用卸载](#应用卸载)
	* [清除应用数据与缓存](#清除应用数据与缓存)
	* [查看前台 Activity](#查看前台-activity)
	* [查看正在运行的 Services](#查看正在运行的-services)
	* [查看应用详细信息](#查看应用详细信息)
* [操作模拟](#操作模拟)
	* [按键模拟](#按键模拟)
	* [应用交互](#应用交互)
		* [调起 Activity](#调起-activity)
		* [调起 Service](#调起-service)
		* [发送广播](#发送广播)
		* [强制停止应用](#强制停止应用)
	* [设备操作](#设备操作)
		* [截屏](#截屏)
		* [录屏](#录屏)
		* [重启](#重启)
		* [压力测试（Monkey）](#压力测试-monkey)
		* [修改分辨率](#修改分辨率)
* [Log 日志查看](#Log-日志查看)
	* [过滤日志输出](#过滤日志输出)
		* [按级别过滤日志](#按级别过滤日志)
		* [按 Tag 过滤日志](#按-Tag-过滤日志)
* [ADB Shell 相关命令](#adb-shell-相关命令)
	* [查看进程](#查看进程)

## 语法及连接

### 语法

ADB 的基本命令语法如下：
```
adb [-d|-e|-s <serialNumber>] <command>
```
同一台电脑可以连接多个 Android 设备，当时用 adb 命令操作设备的时候，可以指定设备下发命令。具体如下

| 参数                | 含义                                             |
|---------------------|------------------------------------------------ |
| -d                  | 指定当前唯一通过 USB 连接的 Android 设备为命令目标 |
| -e                  | 指定当前唯一运行的模拟器为命令目标                 |
| `-s <Serial Number>` | 指定相应 Serial Number 号的设备/模拟器为命令目标    |

可以使用
```
adb devices
```
来查看当前连接的所有设备，如下所示
```
List of devices attached
emulator-5554   device
e8c63296        device
```
`emulator-5554`是一个模拟器，`e8c63296` 是一台真机，这些均为他们的 Serial Number。可以指定设备发送命令，当只连接一台设备的时候，Serial Number 可以忽略不写。

### 连接

上文中介绍使用
```
adb devices
```
可以查看所有连接到的设备。

#### USB 连接

使用USB 连接设备时。需要将设备
的开发者选项和 USB 调试模式开启。
可以到「设置」-「开发者选项」-「Android 调试」查看。
如果在设置里找不到开发者选项，那需要通过一个彩蛋来让它显示出来：在「设置」-「关于手机」连续点击「版本号」7 次。

#### 无线连接
无线连接可以在提前设置的情况下，免去数据线连接的烦恼，不过每次打包的时候，将 APK 发送到设备上都需要一会，数据传输自然是没有使用数据线快了，受网络干扰因素影响。
无线连接必须保证将 Android 设备与要运行 ADB 的电脑连接到同一个局域网，比如连到同一个 WiFi 即可。
具体步骤如下：
1. 将设备与电脑通过 USB 线连接并确保能够连接成功（`adb devices`指令能够看见设备）；
2. 让设备在 5555 端口监听 TCP/IP 连接；
   ```
   adb connect <device-ip-address>
   ```
3. 拔掉数据线；
4. 查找设备的 IP 地址，一般能在「设置」-「关于手机」-「状态信息」-「IP地址」找到；
5. 通过 IP 地址连接设备  
	```
	adb connect <Device IP Address>
	```
`<Device IP Address>`即是刚才获取的 IP 地址
此时再使用`adb devices`命令，如果在下面能看见设备说明连接成功。

## 获取设备信息
### 型号
命令
```
adb shell getprop ro.product.model
```
输出
```
ONEPLUS A3010
```
我手中的这台是一加 3T，型号为A3010

### 屏幕分辨率
命令
```
adb shell wm size
```
输出
```
Physical size: 1080x1920
```
我手中的这台一加 3T 的分辨率为 1080P
### 屏幕密度
命令
```
adb shell wm density
```
输出
```
Physical density: 420
```
一加 3T 的 DPI 为 420DPI

### IP 地址
命令
```
adb shell ifconfig
```
输出
```
lo        Link encap:UNSPEC
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope: Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:475045 errors:0 dropped:0 overruns:0 frame:0
          TX packets:475045 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:692847495 TX bytes:692847495

dummy0    Link encap:UNSPEC
          inet6 addr: fe80::4c8a:30ff:fec6:13fc/64 Scope: Link
          UP BROADCAST RUNNING NOARP  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:115 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 TX bytes:8050

wlan0     Link encap:UNSPEC    Driver cnss_wlan_pci
          inet addr:192.168.31.86  Bcast:192.168.31.255  Mask:255.255.255.0
          inet6 addr: fe80::9665:2dff:fe25:12f5/64 Scope: Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:2465828 errors:0 dropped:0 overruns:0 frame:0
          TX packets:138772 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:3000
          RX bytes:475187452 TX bytes:49284101

rmnet_ipa0 Link encap:UNSPEC
          UP RUNNING  MTU:2000  Metric:1
          RX packets:1897 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3233 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:2042241 TX bytes:490601

p2p0      Link encap:UNSPEC    Driver cnss_wlan_pci
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:3000
          RX bytes:0 TX bytes:0
```
我现在正使用局域网，`wlan0 inet addr:192.168.31.86`即是我的 IP 地址
### IMEI
在 Android 4.4 及以下版本可通过如下命令获取 IMEI：
```
adb shell dumpsys iphonesubinfo
```
输出
```
Phone Subscriber Info:
  Phone Type = GSM
  Device ID = 860955027785041
```
 Android 5.0 上需要 ROOT 权限才能看到
### Android 系统版本
命令
```
adb shell getprop ro.build.version.release
```
输出
```
8.0.0
```
我手中的这台一加 3T 的 Android 版本为奥利奥
### CPU 信息
命令
```
adb shell cat /proc/cpuinfo
```
输出
```
processor       : 0
BogoMIPS        : 38.40
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32
CPU implementer : 0x51
CPU architecture: 8
CPU variant     : 0x2
CPU part        : 0x201
CPU revision    : 1

processor       : 1
BogoMIPS        : 38.40
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32
CPU implementer : 0x51
CPU architecture: 8
CPU variant     : 0x2
CPU part        : 0x201
CPU revision    : 1

processor       : 2
BogoMIPS        : 38.40
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32
CPU implementer : 0x51
CPU architecture: 8
CPU variant     : 0x2
CPU part        : 0x205
CPU revision    : 1

processor       : 3
BogoMIPS        : 38.40
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32
CPU implementer : 0x51
CPU architecture: 8
CPU variant     : 0x2
CPU part        : 0x205
CPU revision    : 1

Hardware        : Qualcomm Technologies, Inc MSM8996pro
```
看到使用的硬件是 `Qualcomm MSM8996pro`，即高通 821，processor 的编号是 0 到 3，所以它是四核的。

## 文件管理
### 拉取
命令
```
adb pull <设备里的文件路径> [电脑上的目录]
```
如果省略`[电脑上的目录]`选项，则默认复制在当前目录下；
一般使用未 ROOT 的机器均是复制或拉取 `/sdcard/`目录下的文件。
eg.
```
adb pull /sdcard/demo.mp4 ~/Desktop
```
### 推送
命令
```
adb push <电脑上的文件路径> <设备里的目录>
```
eg.
```
adb push ~/Desktop/demo.mp4 /sdcard/
```
## 应用管理
### 查看设备安装应用
查看应用列表的基本命令格式是

```
adb shell pm list packages [-f]/[-d]/[-e]/[-s]/[-3]/[-i]/[-u]/[--user USER_ID]/[FILTER]
```

这些参数的含义如下所示

| 参数       | 显示列表                   |
|-------- |----------------------------|
| 无         | 所有应用                   |
| -f         | 显示应用关联的 apk 文件    |
| -d         | 只显示 disabled 的应用     |
| -e         | 只显示 enabled 的应用      |
| -s         | 只显示系统应用             |
| -3         | 只显示第三方应用           |
| -i         | 显示应用的 installer       |
| -u         | 包含已卸载应用             |
| `<FILTER>` | 包名包含 `<FILTER>` 字符串 |

下面是具体例子：
#### 所用应用
命令：

```
adb shell pm list packages
```
输出：
```
package:com.oneplus.calculator
package:net.oneplus.weather
package:com.android.cts.priv.ctsshim
package:com.oneplus.market
package:com.yulore.framework
package:com.android.providers.telephony
package:com.sony.playmemories.mobile
package:com.android.engineeringmode
package:com.android.providers.calendar
package:com.oneplus.opbugreport
package:com.oneplus.card
package:com.oneplus.note
package:com.oneplus.skin
package:com.android.providers.media
package:com.sohu.inputmethod.sogou
package:com.android.wallpapercropper
package:com.oneplus.account
package:com.quicinc.cne.CNEService
package:com.oneplus.soundrecorder
package:com.qualcomm.qti.autoregistration
...
```
#### 系统级应用
命令：

```
adb shell pm list packages -s
```
#### 用户安装应用
命令：

```
adb shell pm list packages -3
```
#### 过滤显示
比如要查看包名包含字符串 `tencent` 的应用列表，命令：
```
adb shell pm list packages tencent
```
则只会输出
```
package:com.tencent.mm
package:com.tencent.mobileqq
```
### 应用安装
命令：
```
adb install [-l]/[-r]/[-t]/[-s]/[-d]/[-g] <path_to_apk>
```
这些参数的含义如下所示

| 参数 | 含义                                                                              |
|------|-----------------------------------------------------------------------------------|
| -l   | 将应用安装到保护目录 /mnt/asec                                                    |
| -r   | 允许覆盖安装                                                                      |
| -t   | 允许安装 AndroidManifest.xml 里 application 指定 `android:testOnly="true"` 的应用 |
| -s   | 将应用安装到 sdcard                                                               |
| -d   | 允许降级覆盖安装                                                                  |
| -g   | 授予所有运行时权限                                                                |

运行命令后见到输出状态为 `Success`代表安装成功。
如果失败则会输出`Failure`及错误码，常见错误码含义如下：

| 输出                                                                | 含义                                                                     | 解决办法                                                                       |
|---------------------------------------------------------------------|--------------------------------------------------------------------------|--------------------------------------------------------------------------------|
| INSTALL\_FAILED\_ALREADY\_EXISTS                                    | 应用已经存在，或卸载了但没卸载干净                                       | `adb install` 时使用 `-r` 参数，或者先 `adb uninstall <packagename>` 再安装    |
| INSTALL\_FAILED\_INVALID\_APK                                       | 无效的 APK 文件                                                          |                                                                                |
| INSTALL\_FAILED\_INVALID\_URI                                       | 无效的 APK 文件名                                                        | 确保 APK 文件名里无中文                                                        |
| INSTALL\_FAILED\_INSUFFICIENT\_STORAGE                              | 空间不足                                                                 | 清理空间                                                                       |
| INSTALL\_FAILED\_DUPLICATE\_PACKAGE                                 | 已经存在同名程序                                                         |                                                                                |
| INSTALL\_FAILED\_NO\_SHARED\_USER                                   | 请求的共享用户不存在                                                     |                                                                                |
| INSTALL\_FAILED\_UPDATE\_INCOMPATIBLE                               | 以前安装过同名应用，但卸载时数据没有移除；或者已安装该应用，但签名不一致 | 先 `adb uninstall <packagename>` 再安装                                        |
| INSTALL\_FAILED\_SHARED\_USER\_INCOMPATIBLE                         | 请求的共享用户存在但签名不一致                                           |                                                                                |
| INSTALL\_FAILED\_MISSING\_SHARED\_LIBRARY                           | 安装包使用了设备上不可用的共享库                                         |                                                                                |
| INSTALL\_FAILED\_REPLACE\_COULDNT\_DELETE                           | 替换时无法删除                                                           |                                                                                |
| INSTALL\_FAILED\_DEXOPT                                             | dex 优化验证失败或空间不足                                               |                                                                                |
| INSTALL\_FAILED\_OLDER\_SDK                                         | 设备系统版本低于应用要求                                                 |                                                                                |
| INSTALL\_FAILED\_CONFLICTING\_PROVIDER                              | 设备里已经存在与应用里同名的 content provider                            |                                                                                |
| INSTALL\_FAILED\_NEWER\_SDK                                         | 设备系统版本高于应用要求                                                 |                                                                                |
| INSTALL\_FAILED\_TEST\_ONLY                                         | 应用是 test-only 的，但安装时没有指定 `-t` 参数                          |                                                                                |
| INSTALL\_FAILED\_CPU\_ABI\_INCOMPATIBLE                             | 包含不兼容设备 CPU 应用程序二进制接口的 native code                      |                                                                                |
| INSTALL\_FAILED\_MISSING\_FEATURE                                   | 应用使用了设备不可用的功能                                               |                                                                                |
| INSTALL\_FAILED\_CONTAINER\_ERROR                                   | 1. sdcard 访问失败;<br />2. 应用签名与 ROM 签名一致，被当作内置应用。    | 1. 确认 sdcard 可用，或者安装到内置存储;<br />2. 打包时不与 ROM 使用相同签名。 |
| INSTALL\_FAILED\_INVALID\_INSTALL\_LOCATION                         | 1. 不能安装到指定位置;<br />2. 应用签名与 ROM 签名一致，被当作内置应用。 | 1. 切换安装位置，添加或删除 `-s` 参数;<br />2. 打包时不与 ROM 使用相同签名。   |
| INSTALL\_FAILED\_MEDIA\_UNAVAILABLE                                 | 安装位置不可用                                                           | 一般为 sdcard，确认 sdcard 可用或安装到内置存储                                |
| INSTALL\_FAILED\_VERIFICATION\_TIMEOUT                              | 验证安装包超时                                                           |                                                                                |
| INSTALL\_FAILED\_VERIFICATION\_FAILURE                              | 验证安装包失败                                                           |                                                                                |
| INSTALL\_FAILED\_PACKAGE\_CHANGED                                   | 应用与调用程序期望的不一致                                               |                                                                                |
| INSTALL\_FAILED\_UID\_CHANGED                                       | 以前安装过该应用，与本次分配的 UID 不一致                                | 清除以前安装过的残留文件                                                       |
| INSTALL\_FAILED\_VERSION\_DOWNGRADE                                 | 已经安装了该应用更高版本                                                 | 使用 `-d` 参数                                                                 |
| INSTALL\_FAILED\_PERMISSION\_MODEL\_DOWNGRADE                       | 已安装 target SDK 支持运行时权限的同名应用，要安装的版本不支持运行时权限 |                                                                                |
| INSTALL\_PARSE\_FAILED\_NOT\_APK                                    | 指定路径不是文件，或不是以 `.apk` 结尾                                   |                                                                                |
| INSTALL\_PARSE\_FAILED\_BAD\_MANIFEST                               | 无法解析的 AndroidManifest.xml 文件                                      |                                                                                |
| INSTALL\_PARSE\_FAILED\_UNEXPECTED\_EXCEPTION                       | 解析器遇到异常                                                           |                                                                                |
| INSTALL\_PARSE\_FAILED\_NO\_CERTIFICATES                            | 安装包没有签名                                                           |                                                                                |
| INSTALL\_PARSE\_FAILED\_INCONSISTENT\_CERTIFICATES                  | 已安装该应用，且签名与 APK 文件不一致                                    | 先卸载设备上的该应用，再安装                                                   |
| INSTALL\_PARSE\_FAILED\_CERTIFICATE\_ENCODING                       | 解析 APK 文件时遇到 `CertificateEncodingException`                       |                                                                                |
| INSTALL\_PARSE\_FAILED\_BAD\_PACKAGE\_NAME                          | manifest 文件里没有或者使用了无效的包名                                  |                                                                                |
| INSTALL\_PARSE\_FAILED\_BAD\_SHARED\_USER\_ID                       | manifest 文件里指定了无效的共享用户 ID                                   |                                                                                |
| INSTALL\_PARSE\_FAILED\_MANIFEST\_MALFORMED                         | 解析 manifest 文件时遇到结构性错误                                       |                                                                                |
| INSTALL\_PARSE\_FAILED\_MANIFEST\_EMPTY                             | 在 manifest 文件里找不到找可操作标签（instrumentation 或 application）   |                                                                                |
| INSTALL\_FAILED\_INTERNAL\_ERROR                                    | 因系统问题安装失败                                                       |                                                                                |
| INSTALL\_FAILED\_USER\_RESTRICTED                                   | 用户被限制安装应用                                                       |                                                                                |
| INSTALL\_FAILED\_DUPLICATE\_PERMISSION                              | 应用尝试定义一个已经存在的权限名称                                       |                                                                                |
| INSTALL\_FAILED\_NO\_MATCHING\_ABIS                                 | 应用包含设备的应用程序二进制接口不支持的 native code                     |                                                                                |
| INSTALL\_CANCELED\_BY\_USER                                         | 应用安装需要在设备上确认，但未操作设备或点了取消                         | 在设备上同意安装                                                               |
| INSTALL\_FAILED\_ACWF\_INCOMPATIBLE                                 | 应用程序与设备不兼容                                                     |                                                                                |
| does not contain AndroidManifest.xml                                | 无效的 APK 文件                                                          |                                                                                |
| is not a valid zip file                                             | 无效的 APK 文件                                                          |                                                                                |
| Offline                                                             | 设备未连接成功                                                           | 先将设备与 adb 连接成功                                                        |
| unauthorized                                                        | 设备未授权允许调试                                                       |                                                                                |
| error: device not found                                             | 没有连接成功的设备                                                       | 先将设备与 adb 连接成功                                                        |
| protocol failure                                                    | 设备已断开连接                                                           | 先将设备与 adb 连接成功                                                        |
| Unknown option: -s                                                  | Android 2.2 以下不支持安装到 sdcard                                      | 不使用 `-s` 参数                                                               |
| No space left on device                                             | 空间不足                                                                 | 清理空间                                                                       |
| Permission denied ... sdcard ...                                    | sdcard 不可用                                                            |                                                                                |
| signatures do not match the previously installed version; ignoring! | 已安装该应用且签名不一致                                                 | 先卸载设备上的该应用，再安装                                                   |

**`adb install` 内部原理简介**

`adb install` 实际是分三步完成：

1. push apk 文件到 /data/local/tmp。

2. 调用 pm install 安装。

3. 删除 /data/local/tmp 下的对应 apk 文件。

所以，如果有需要也可以根据这个步骤，手动分步执行安装过程。

### 应用卸载
命令：
```
adb uninstall [-k] <packagename>
```
`<packagename>` 表示应用的包名，`-k` 参数可选，表示卸载应用但保留数据和缓存目录。

命令示例：

```sh
adb uninstall com.tencent.mobileqq
```
即卸载 QQ

### 清除应用数据与缓存
这条命令的效果相当于在设置里的应用信息界面点击了「清除缓存」和「清除数据」。
命令：
```
adb shell pm clear <packagename>
```

命令示例：

```
adb shell pm clear com.tencent.mobileqq
```
即清除 QQ 的缓存和数据

### 查看前台 Activity
命令：

```
adb shell dumpsys activity activities | grep mFocusedActivity
```

输出示例：

```sh
mFocusedActivity: ActivityRecord{8079d7e u0 com.cyanogenmod.trebuchet/com.android.launcher3.Launcher t42}
```

其中的 `com.cyanogenmod.trebuchet/com.android.launcher3.Launcher` 就是当前处于前台的 Activity。

当然，输入以下命令就可以查看所有 Activity 栈了
```
adb shell dumpsys activity activities
```

### 查看正在运行的 Services
命令：

```
adb shell dumpsys activity services [<packagename>]
```

`<packagename>` 参数不是必须的，指定 `<packagename>` 表示查看与某个包名相关的 Services，不指定表示查看所有 Services。

`<packagename>` 不一定要给出完整的包名，比如运行 `adb shell dumpsys activity services cn.artaris`，那么包名 `cn.artaris.demo1`、`cn.artaris.demo2` 和 `cn.artaris123` 等相关的 Services 都会列出来。

### 查看应用详细信息

```
adb shell dumpsys package <packagename>
```

输出中包含很多信息，包括 Activity Resolver Table、Registered ContentProviders、包名、userId、安装后的文件资源代码等路径、版本信息、权限信息和授予状态、签名版本信息等。

`<packagename>` 表示应用包名。

## 操作模拟
### 按键模拟
按键模拟主要是通过`adb shell input`来实现的，在 Android 开发的时候我们可以拦截按键事件，接受按键的函数如下
```
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
    return super.onKeyDown(keyCode, event);
}
```
其中`KeyEvent`类中定义了一系列常量，例如返回键的值为`KeyEvent.KEYCODE_BACK`，其常量值为4。按键模拟操作的值与此一致。
下表为常用按键值：

| keycode | 含义                           |
|---------|--------------------------------|
| 3       | HOME 键                        |
| 4       | 返回键                         |
| 5       | 打开拨号应用                   |
| 6       | 挂断电话                       |
| 24      | 增加音量                       |
| 25      | 降低音量                       |
| 26      | 电源键                         |
| 27      | 拍照（需要在相机应用里）       |
| 64      | 打开浏览器                     |
| 82      | 菜单键                         |
| 85      | 播放/暂停                      |
| 86      | 停止播放                       |
| 87      | 播放下一首                     |
| 88      | 播放上一首                     |
| 122     | 移动光标到行首或列表顶部       |
| 123     | 移动光标到行末或列表底部       |
| 126     | 恢复播放                       |
| 127     | 暂停播放                       |
| 164     | 静音                           |
| 176     | 打开系统设置                   |
| 187     | 切换应用                       |
| 207     | 打开联系人                     |
| 208     | 打开日历                       |
| 209     | 打开音乐                       |
| 210     | 打开计算器                     |
| 220     | 降低屏幕亮度                   |
| 221     | 提高屏幕亮度                   |
| 223     | 系统休眠                       |
| 224     | 点亮屏幕                       |
| 231     | 打开语音助手                   |
| 276     | 如果没有 wakelock 则让系统休眠 |

使用方法如下
```
adb shell input keyevent <KeyCode>
```
只要输入对应按键的按键号，则能实现对应功能。

有两个操作需要特别说明一下：
#### 模拟屏幕滑动：

```
adb shell input swipe 300 1000 300 500
```
参数 `300 1000 300 500` 分别表示`起始点x坐标 起始点y坐标 结束点x坐标 结束点y坐标`。

#### 输入文本：
在焦点处于某文本框时，可以通过 `input` 命令来输入文本。

命令：

```
adb shell input text artaris
```

现在 `artaris` 出现在文本框了。

### 应用交互
应用交互主要是通过`adb shell am <command>`来实现的,常用的`<command>`指令有以下几种：

| command                           | 用途                            |
|-----------------------------------|---------------------------------|
| `start [options] <INTENT>`        | 启动 `<INTENT>` 指定的 Activity |
| `startservice [options] <INTENT>` | 启动 `<INTENT>` 指定的 Service  |
| `broadcast [options] <INTENT>`    | 发送 `<INTENT>` 指定的广播      |
| `force-stop <packagename>`        | 停止 `<packagename>` 相关的进程 |

`<INTENT>` 即与 Android 程序时代码里的 Intent 相对应。

| 参数             | 含义                                                                                        |
|------------------|---------------------------------------------------------------------------------------------|
| `-a <ACTION>`    | 指定 action，比如 `android.intent.action.VIEW`                                              |
| `-c <CATEGORY>`  | 指定 category，比如 `android.intent.category.APP_CONTACTS`                                  |
| `-n <COMPONENT>` | 指定完整 component 名，用于明确指定启动哪个 Activity，如 `cn.artaris.demo/.MainActivity` |

`<INTENT>` 里还能带参数，与程序中的 Bundle 含义一致：

| 参数                                                          | 含义                                   |
|---------------------------------------------------------------|----------------------------------------|
| `--esn <EXTRA_KEY>`                                           | null 值（只有 key 名）                 |
| `-e/--es <EXTRA_KEY> <EXTRA_STRING_VALUE>`                    | string 值                              |
| `--ez <EXTRA_KEY> <EXTRA_BOOLEAN_VALUE>`                      | boolean 值                             |
| `--ei <EXTRA_KEY> <EXTRA_INT_VALUE>`                          | integer 值                             |
| `--el <EXTRA_KEY> <EXTRA_LONG_VALUE>`                         | long 值                                |
| `--ef <EXTRA_KEY> <EXTRA_FLOAT_VALUE>`                        | float 值                               |
| `--eu <EXTRA_KEY> <EXTRA_URI_VALUE>`                          | URI                                    |
| `--ecn <EXTRA_KEY> <EXTRA_COMPONENT_NAME_VALUE>`              | component name                         |
| `--eia <EXTRA_KEY> <EXTRA_INT_VALUE>[,<EXTRA_INT_VALUE...]`   | integer 数组                           |
| `--ela <EXTRA_KEY> <EXTRA_LONG_VALUE>[,<EXTRA_LONG_VALUE...]` | long 数组                              |

#### 调起 Activity
命令：
```
adb shell am start [options] <INTENT>
```
eg.：

```
adb shell am start -n com.tencent.mm/.ui.LauncherUI
```
即掉起微信主界面；
```
adb shell am start -n cn.artaris.demo/.MainActivity --es "toast" "hello, world"
```
表示调起 `cn.artaris.demo/.MainActivity` 并传给它 String 数据键值对 `toast - hello, world`。

#### 调起 Service
命令格式：

```
adb shell am startservice [options] <INTENT>
```

例如：

```sh
adb shell am startservice -n com.tencent.mm/.plugin.accountsync.model.AccountAuthenticatorService
```

表示调起微信的 AccountAuthenticatorService。
#### 发送广播
命令：

```
adb shell am broadcast [options] <INTENT>
```
使用该命令可以向所有组件广播，也可以只向指定组件广播；
eg.:
```
adb shell am broadcast -a android.intent.action.BOOT_COMPLETED
```
向所有应用发送开机广播。
```
adb shell am broadcast -a android.intent.action.BOOT_COMPLETED -n cn.artaris.demo/.BootReceiver
```
只向`cn.artaris.demo/.BootReceiver`发送开机广播；
以下是常用系统广播，可以模拟一些特殊场景：

| action                                          | 触发时机                                      |
|-------------------------------------------------|-----------------------------------------------|
| android.net.conn.CONNECTIVITY_CHANGE            | 网络连接发生变化                              |
| android.intent.action.SCREEN_ON                 | 屏幕点亮                                      |
| android.intent.action.SCREEN_OFF                | 屏幕熄灭                                      |
| android.intent.action.BATTERY_LOW               | 电量低，会弹出电量低提示框                    |
| android.intent.action.BATTERY_OKAY              | 电量恢复了                                    |
| android.intent.action.BOOT_COMPLETED            | 设备启动完毕                                  |
| android.intent.action.DEVICE_STORAGE_LOW        | 存储空间过低                                  |
| android.intent.action.DEVICE_STORAGE_OK         | 存储空间恢复                                  |
| android.intent.action.PACKAGE_ADDED             | 安装了新的应用                                |
| android.net.wifi.STATE_CHANGE                   | WiFi 连接状态发生变化                         |
| android.net.wifi.WIFI_STATE_CHANGED             | WiFi 状态变为启用/关闭/正在启动/正在关闭/未知 |
| android.intent.action.BATTERY_CHANGED           | 电池电量发生变化                              |
| android.intent.action.INPUT_METHOD_CHANGED      | 系统输入法发生变化                            |
| android.intent.action.ACTION_POWER_CONNECTED    | 外部电源连接                                  |
| android.intent.action.ACTION_POWER_DISCONNECTED | 外部电源断开连接                              |
| android.intent.action.DREAMING_STARTED          | 系统开始休眠                                  |
| android.intent.action.DREAMING_STOPPED          | 系统停止休眠                                  |
| android.intent.action.WALLPAPER_CHANGED         | 壁纸发生变化                                  |
| android.intent.action.HEADSET_PLUG              | 插入耳机                                      |
| android.intent.action.MEDIA_UNMOUNTED           | 卸载外部介质                                  |
| android.intent.action.MEDIA_MOUNTED             | 挂载外部介质                                  |
| android.os.action.POWER_SAVE_MODE_CHANGED       | 省电模式开启                                  |

#### 强制停止应用

命令：

```
adb shell am force-stop <packagename>
```

命令示例：

```
adb shell am force-stop cn.artaris.demo
```
表示停止 Demo 的一切进程与服务。

### 设备操作
#### 截屏
命令：

```
adb exec-out screencap -p > sc.png
```
如果该命令不奏效，可分两步，先使用
```
adb shell screencap -p /sdcard/screenshoot.png
```
命令将截图保存到设备中，再使用`adb pull`导出到电脑。
#### 录屏
命令：
```
adb shell screenrecord /sdcard/screenrecord.mp4
```
需要停止时按 <kbd>Ctrl+C</kbd>，默认录制时间和最长录制时间都是 180 秒。
再使用`adb pull`导出到电脑。
可以附加以下参数，确定录屏的参数：

| 参数                | 含义                                            |
|---------------------|-------------------------------------------------|
| --size WIDTHxHEIGHT | 视频的尺寸，比如 `1280x720`，默认是屏幕分辨率。 |
| --bit-rate RATE     | 视频的比特率，默认是 4Mbps。                    |
| --time-limit TIME   | 录制时长，单位秒。                              |
| --verbose           | 输出更多信息。                                  |
#### 重启
命令：

```
adb reboot
```
#### 压力测试 Monkey
Monkey 可以生成伪随机用户事件来模拟单击、触摸、手势等操作，可以对正在开发中的程序进行随机压力测试。

简单用法：

```sh
adb shell monkey -p <packagename> -v 500
```

表示向 `<packagename>` 指定的应用程序发送 500 个伪随机事件。

Monkey 的详细用法参考 [Monkey](https://developer.android.com/studio/test/monkey.html)。
#### 修改分辨率
命令：

```
adb shell wm size 960x540
```

表示将分辨率修改为 960px * 540px。

恢复原分辨率命令：

```
adb shell wm size reset
```

## Log日志查看
Android 的日志输出到 /dev/log。查看命令为
```
adb logcat [<option>] ... [<filter-spec>] ...
```
具体用法如下：
### 过滤日志输出
### 按级别过滤日志
Android 的日志分为如下几个优先级，我们在 Android Studio 的Logcat 选项卡中也可以过滤查看，含义如下

| 标签                | 含义                                            |
|---------------------|-------------------------------------------------|
| V  | Verbose |
| D    | Debug                    |
| I   | Info                             |
| W          | Warning                                  |
|  E          | Error|
|  F          | Fatal|
|  S         | Silent（啥也不输出）|

按某级别过滤日志则会将该级别及以上的日志全部输出。
eg.
```
adb logcat *:W
```
会将`Warning`、`Warning`、`Warning`和`Silent`日志输出。
（**注：** 在 macOS 下需要给 `*:W` 这样以 `*` 作为 tag 的参数加双引号，如 `adb logcat "*:W"`，不然会报错 `no matches found: *:W`。）


### 按Tag过滤日志
`<filter-spec>` 可以由多个 `<tag>[:priority]` 组成。

比如，命令：

```
adb logcat MainActivity:I DemoApp:D *:S
```

表示输出 tag `MainActivity` 的 Info 以上级别日志，输出 tag `DemoApp` 的 Debug 以上级别日志，及其它 tag 的 Silent 级别日志（即屏蔽其它 tag 日志）。

## ADB Shell 相关命令
### 查看进程
命令：
```
adb shell ps
```
输出：
```
USER     PID   PPID  VSIZE  RSS     WCHAN    PC        NAME
root      1     0     8904   788   ffffffff 00000000 S /init
root      2     0     0      0     ffffffff 00000000 S kthreadd
...
u0_a71    7779  5926  1538748 48896 ffffffff 00000000 S com.sohu.inputmethod.sogou:classic
u0_a58    7963  5926  1561916 59568 ffffffff 00000000 S cn.artaris.demo
...
shell     8750  217   10640  740   00000000 b6f28340 R ps
```
各列含义如下：

| 列名 | 含义      |
|------|-----------|
| USER | 所属用户  |
| PID  | 进程 ID   |
| PPID | 父进程 ID |
| NAME | 进程名    |

下载地址
[Platform-Tools](http://gofile.me/3TjWz/EF4mgGZBZ)
