---
title: android无障碍服务
tags:
---

# android无障碍服务



### 无障碍服务声明

```xml
<application>  
  <service android:name=".MyAccessibilityService"  
      android:label="@string/accessibility_service_label">  
    <intent-filter>  
      <action android:name="android.accessibilityservice.AccessibilityService" />  
    </intent-filter>  
  </service>  
  <uses-permission android:name="android.permission.BIND_ACCESSIBILITY_SERVICE" />  
</application> 
```

> 这些声明的所有无障碍服务需要部署在Android 1.6（API级别4）以上。

