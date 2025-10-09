---
layout: post
title: AndroidBinding android
date: 2025-10-09 14:33:00
tags: C# android
---

## 確認目標目標 SDK

開啟專案屬性確認目標 Android Framework 版本，
假設目標版本為 35 ，最小值為 21 ，
則可使用 21~35 的 SDK。

## 創建 Android JAVA 程式庫繫結

在方案中加入 Android JAVA 程式庫繫結專案，
從 Android SDK 資料夾中複製 android.car.jar 到 Android JAVA 程式庫繫結專案資料夾底下，
android.car.jar 可能位於 "C:\Program Files (x86)\Android\android-sdk\platforms\android-34\optional" 底下，
按照 SDK 安裝方式與路徑設定不同會不一樣，
需要自己找。

複製完成後資料夾配置如下:

```
AndroidBinding_Car
 - 相依性
 - Additions/
 - Transforms/
 - Android.car.jar
```

## 編譯

對 AndroidBinding_Car 編譯後，
可在 "C:\Program Files (x86)\Android\android-sdk\platforms\android-34\optional"
底下看到 dll& arr 檔案

## 後續

在 MAUI 的專案中引用產生出來的檔案即可。

也可以將程式庫繫結專案與 MAUI 專案放在同一個方案底下，
在 MAUI 專案相依性新增專案參考勾選 AndroidBinding_Car 即可。
