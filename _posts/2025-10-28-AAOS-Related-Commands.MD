---
layout: post
title: "AAOS Related Command"
date: 2025-10-28 16:03:00 +0800
categories: Android AAOS
tags: Android AAOS
published: false
---

#查詢當前 Android 虛擬機權限

ro.build.type 如果為 user 則無法 root，
如果為 userdebug 則可以 root

```bash

.\adb.exe -s emulator-5554 shell getprop ro.build.type

```

#Root 進入 Android 系統

```bash

.\adb.exe -s emulator-5554 root

```

#重新掛載取得讀寫系統權限

```bash

.\adb.exe -s emulator-5554 remount

```


