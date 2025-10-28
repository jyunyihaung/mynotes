---
layout: post
title: "MAUI Android http issue"
date: 2025-10-15 15:05:00 +0800
author: "Jyun-Yi Huang"
tags: C# android MAUI
published: true
---

# 使用 MAUI 開發 Android App 使用 HttpClient 時出現 Connection failure

從 Android 9(API 28) 開始預設停止使用 http,
開發階段要使用 http 則可以加入 config 檔案暫時開啟。

假設內網目標 IP 為 192.168.1.38，在專案中加入以下兩個檔案:

1. Platforms\Android\Resources\xml\network\_security\_config.xml

```

<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
	<domain-config cleartextTrafficPermitted="true">
		<domain includeSubdomains="true">192.168.1.38</domain>
	</domain-config>
</network-security-config>

```

2. Platforms\Android\AndroidManifest.xml

```

<manifest xmlns:android="http://schemas.android.com/apk/res/android">
	<application ...
		android:usesCleartextTraffic="true"
		android:networkSecurityConfig="@xml/network_security_config"/>
...

```

重新編譯後即可使用 HttpClient 對 192.168.1.38 連線。
