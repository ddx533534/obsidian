# 1.简介
Android 调试桥 (Android debug bridge) 是一种功能多样的命令行工具，可与设备进行通信。同时是一种客户端-服务器程序，包括以下三个组件：

-   **客户端**：用于发送命令。客户端在开发机器上运行。可以通过发出 adb 命令从命令行终端调用客户端。
-   **守护程序 (adbd)**：用于在设备上运行命令。守护程序在每个设备上作为后台进程运行。
-   **服务器**：用于管理客户端与守护程序之间的通信。服务器在开发机器上作为后台进程运行。

# 2.安装
1. `adb` 包含在 Android SDK 平台工具软件包中。使用 [SDK 管理器](https://developer.android.com/studio/intro/update#sdk-manager)下载此软件包，该管理器会将其安装在 `android_sdk/platform-tools/` 下。
2. 如果不想安装Android stdio则可 [点击此处进行下载](https://developer.android.com/studio/releases/platform-tools)。也可以通过命令行单独下载adb：
    brew cask install android-platform-tools   

# 3.使用
如果有多个设备/模拟器连接需要通过 **-s** 参数指定目标设备/模拟器，单个设备可以省略该参数。
## 3.1 调试
1.设备
-查看已经连接的设备
``` shell
$ adb devices 
List of devices attached
0123456789ABCDEF	device
emulator-5554	device
```

-使用特定设备操作
``` shell
$ adb -s 0123456789ABCDEF <command> 
```

2.重启
-正常重启
``` shell
$ adb -s 0123456789ABCDEF reboot
```

-重启到 bootloader (刷机模式)  
   ``` shell
$ adb -s 0123456789ABCDEF reboot bootloader
```

-重启到 recovery (恢复模式)  
``` shell
$ adb -s 0123456789ABCDEF reboot recovery
```

2.进程
-查看指定设备进程
``` shell
$ adb -s 0123456789ABCDEF shell ps
USER           PID  PPID     VSZ    RSS WCHAN            ADDR S NAME
root             1     0   32568   5564 0                   0 S init
root             2     0       0      0 0                   0 S [kthreadd]
root             4     2       0      0 0                   0 I [kworker/0:0H]
root             6     2       0      0 0                   0 I [mm_percpu_wq]
root             7     2       0      0 0                   0 S [ksoftirqd/0]
root             8     2       0      0 0                   0 I [rcu_preempt]
root             9     2       0      0 0                   0 I [rcu_sched]
root            10     2       0      0 0                   0 I [rcu_bh]
......
```

-杀死指定pid的进程
``` shell
$ adb -s 0123456789ABCDEF shell kill <pid>
```

3.内存
-查看系统当前内存使用情况
``` shell
$ adb -s 0123456789ABCDEF shell cat /proc/meminfo
```

-查看指定包名应用内存使用情况(运行中)
``` shell
$ adb -s 0123456789ABCDEF shell dumpsys meminfo <package>
```

4.Activity
-启动Activity
``` shell
$ adb -s 0123456789ABCDEF shell am start -n <package>/<package-activity>
```

-停止Activity
``` shell
$ adb -s 0123456789ABCDEF shell am force-stop <package>
```

-**查看当前获取焦点的Window所包含的Activity，一般用来获取当前设备/模拟器正在展示的Activity**
``` shell
$ adb -s 0123456789ABCDEF shell dumpsys window | grep mCurrentFocus
```
`
## 3.2 应用管理

1.**安装apk**
``` shell
$ adb -s emulator-5554 install hello_world.apk
Performing Streamed Install
Success
```

2.**查看应用**
-查看设备所有的应用包名
``` shell
$ adb -s emulator-5554 shell pm list packages
......
```

-查看指定包名对应的apk路径
``` shell
$ adb -s emulator-5554 shell pm path <package>
package:/data/app/......
```

3.**卸载应用**
卸载指定包名应用
``` shell
$ adb -s emulator-5554 unstall <package>
Success
```

## 3.3 shell
1.启动和退出交互式shell
``` shell
$ adb -s 0123456789ABCDEF shell
sl8541e_su806:/ $ exit/Ctrl+D
```

2.文件读写
-读取
``` shell
$ adb -s 0123456789ABCDEF pull <remote> <local>
```

-写入
``` shell
$ adb -s 0123456789ABCDEF push <local> <remote> 
```

2.屏幕截图并从设备下载
``` shell
$ adb -s 0123456789ABCDEF shell
sl8541e_su806:/ $ screencap <filename> 
sl8541e_su806:/ $ exit/Ctrl+D
$ adb -s 0123456789ABCDEF pull <filename>
```

3.屏幕录制
``` shell
$ adb -s 0123456789ABCDEF shell
sl8541e_su806:/ $ screenrecord --time-limit=<time> <filename> 
sl8541e_su806:/ $ exit/Ctrl+D
$ adb -s 0123456789ABCDEF pull <filename>
```
screenrecord程序局限性：
-   音频不与视频文件一起录制。
-   不支持在录制时旋转屏幕。如果在录制期间屏幕发生了旋转，则部分屏幕内容在录制时将被切断。