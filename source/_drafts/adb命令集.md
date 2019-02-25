# ADB 命令集

## 常用的

- `adb start-server`
启动adb服务,如果它没启动的话
- `adb kill-server`
关闭服务
- `adb devices`
查看所连接的设备以及设备所对应的序列号
- `adb install -r xxxx.apk`
安装app,需要注意的是如果连接了两台设备,则会报错,此时可以添加-s <serialNumber>来处理
- `adb uninstall packagename
卸载app,有时候在手机上卸载App会出现数据清理不干净,导致App再也装不上了,这个时候可以敲命令来卸载
- `adb shell`
进入shell环境
- `adb shell pm clear packagename`
清除应用的数据,很常用吧?
- `adb shell am start -n packagename/packagename.activityname`
启动某个应用的某个Activity(以前调试老年机,那种Launcher上没有APP的机器,全靠它啊!!!!!!!)
- `adb connect <device-ip-address>`
连接到指定的ip,这个通常配合wifidebug
- `adb shell dumpsys activity top`
查看栈顶Activity,可以用来获取包名,可以用来查看其它app的包名
- `adb shell ps`
查看进程信息
- `adb shell pm list packages -f`
查看所有已安装的应用的包名
- `adb shell dumpsys activity`
dumpsys系列命令可以帮助我们查看各种信息
am的状态 Activity Manager State
- `adb shell dumpsys package`
包信息 Package Information
- `adb shell dumpsys meminfo`
内存使用情况Memory Usage
- `adb pull <remote> <local>`
从手机复制文件出来,比如把Crash日志写在SD卡上,再pull到电脑上 或者 pull ANR的trace日志
- `adb push <local> <remote>`
向手机发送文件,比如测试热修复补丁~
eg. adb push foo.txt /sdcard/foo.txt
- `adb shell cat /proc/cpuinfo`
查看手机CPU,可以看到手机架构(eg.ARMv7) 和几核处理器
可以帮助我们选择so库,排查手机cpu架构相关的问题

## 不太常用的
- `adb shell df`
获取手机磁盘空间
- `adb shell getprop ro.build.version.release`
获取手机系统版本
- `adb shell dumpsys procstats`
Memory Use Over Time
- `adb shell dumpsys gfxinfo`
Graphics State
- `adb version`
查看adb版本
- `adb help`
进入adb帮助界面