# Android Attr

## AndroidManifest
### android:configChanges

- mcc:
The IMSI mobile country code (MCC) has changed — a SIM has been detected and updated the MCC.
IMSI(国际移动用户识别码)发生改变，检测到SIM卡，或者更新MCC
- mnc:
The IMSI mobile network code (MNC) has changed — a SIM has been detected and updated the MNC.
IMSI网络发生改变,检测到SIM卡，或者更新MCC
其中mcc和mnc理论上不可能发生变化
- locale:
The locale has changed — the user has selected a new language that text should be displayed in.
语言发生改变，用户选择了一个新的语言，文字应该重新显示
- touchscreen:
The touchscreen has changed. (This should never normally happen.)
触摸屏发生改变，这通常是不应该发生的
- keyboard:
The keyboard type has changed — for example, the user has plugged in an external keyboard.
键盘类型发生改变，例如，用户使用了外部键盘
- keyboardHidden:
The keyboard accessibility has changed — for example, the user has revealed the hardware keyboard.
键盘发生改变，例如，用户使用了硬件键盘
- navigation:
The navigation type (trackball/dpad) has changed. (This should never normally happen.)
导航发生改变，（这通常不应该发生） 举例：连接蓝牙键盘，连接后确实导致了navigation的类型发生变化。因为连接蓝牙键盘后，我可以使用方向键来navigate了
- screenLayout：
The screen layout has changed — this might be caused by a different display being activated.
屏幕的布局发生改变，这可能导致激活不同的显示
- fontScale：
The font scaling factor has changed — the user has selected a new global font size.
全局字体大小缩放发生改变
- orientation：
The screen orientation has changed — that is, the user has rotated the device.设备旋转，横向显示和竖向显示模式切换。
- screenSize: 
屏幕大小改变了
- smallestScreenSize: 
屏幕的物理大小改变了，如：连接到一个外部的屏幕上