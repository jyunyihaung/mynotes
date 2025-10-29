---
layout: post
title: "AAOS Related Command"
date: 2025-10-29 11:49:00 +0800
categories: Android AAOS
tags: Android AAOS
author: "Jyun-Yi Huang"
published: true
---

#查詢當前 Android 虛擬機權限

ro.build.type 如果為 user 則無法 root，
如果為 userdebug 則可以 root

````powershell

.\adb.exe -s emulator-5554 shell getprop ro.build.type

```

#Root 進入 Android 系統

```powershell

.\adb.exe -s emulator-5554 root

```

#重新掛載取得讀寫系統權限

```powershell

.\adb.exe -s emulator-5554 remount

```

#使用指令查詢 AVD(android virtual device)

```powershell

& "$env:LOCALAPPDATA\Android\Sdk\emulator\emulator.exe" -list-avds
# 假設輸出：MyAutomotiveApi35

AppData\Local\Android\Sdk> .\emulator\emulator.exe -list-avds
Honda_IVI-SYSTEM
Medium_Phone_API_36.1

```

#使用指令開啟 AVD(android virtual device)

-writable-system 會用 overlayfs 讓 /system 可寫；-no-snapshot* 避免載入唯讀快照

```powershell

& "$env:LOCALAPPDATA\Android\Sdk\emulator\emulator.exe" -avd MyAutomotiveApi35 `
  -writable-system -no-snapshot -no-snapshot-load -no-snapshot-save `
  -selinux permissive

```

#建立系統預設資料夾並推送APK檔

````powershell

$APPNAME = "Bluetooth_Scan_Test"
$PKG     = "com.companyname.bluetooth_scan_test"
$SYS_DIR = "/system/priv-app/$APPNAME"

.\adb.exe -s emulator-5554 shell "mkdir -p $SYS_DIR && chmod 755 $SYS_DIR"
.\adb.exe -s emulator-5554 push "...\Bluetooth_Scan_Test\Bluetooth_Scan_Test\bin\Debug\net9.0-android\com.companyname.bluetooth_scan_test-Signed.apk" "$SYS_DIR/$APPNAME.apk"
.\adb.exe -s emulator-5554 shell "chmod 644 $SYS_DIR/$APPNAME.apk"
.\adb.exe -s emulator-5554 shell "ls -l $SYS_DIR"

```

如需特殊權限則還需要建立權限白名單，
建立 .\privapp-permissions-com.companyname.bluetooth\_scan\_test.xml 內容例如：

```xml

<permissions>
  <privapp-permissions package="com.companyname.bluetooth_scan_test">
    <permission name="android.car.permission.CAR_SPEED"/>
  </privapp-permissions>
</permissions>

```

推送指令:

````powershell

.\adb.exe -s emulator-5554 push ".\privapp-permissions-$PKG.xml" "/system/etc/permissions/"

```

#檢查特殊權限相關訊息

````powershell

.\adb.exe -s emulator-5554 logcat -d | findstr /i "Privileged Permission PackageManager com.companyname.bluetooth_scan_test"

```
