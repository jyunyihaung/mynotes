---
layout: post
title: rpi 4b otg study
date: 2025-10-02 14:33:00
tags: rpi
---

## 安裝必要套件
sudo modprobe libcomposite


## 鍵盤、滑鼠模擬啟動腳本
以管理員權限建立以下腳本:

/usr/local/sbin/hid-gadget-up.sh
```
#!/usr/bin/env bash
set -euo pipefail

UDC_NAME=$(ls /sys/class/udc | head -n1 || true)  # 例如 fe980000.usb
GROOT="/sys/kernel/config/usb_gadget"
G="$GROOT/kbdmouse"

# 確保 configfs 已掛載
if ! mountpoint -q /sys/kernel/config; then
  mount -t configfs none /sys/kernel/config || true
fi

# 載入必要模組
modprobe dwc2 || true
modprobe libcomposite || true

# 若先前有殘留未完整建立的目錄，先做最小清理
if [[ -e "$G/UDC" ]]; then
  # 若已綁定，先解除
  if [[ -s "$G/UDC" ]]; then echo "" > "$G/UDC" || true; fi
fi

mkdir -p "$G"
cd "$G"

# 基本屬性（若已存在就覆蓋）
echo 0x1d6b > idVendor
echo 0x0104 > idProduct
echo 0x0100 > bcdDevice
echo 0x0200 > bcdUSB

mkdir -p strings/0x409
echo "1234567890" > strings/0x409/serialnumber
echo "RaspberryPi" > strings/0x409/manufacturer
echo "Pi4B HID Gadget" > strings/0x409/product

mkdir -p configs/c.1/strings/0x409
echo "Config 1: HID" > configs/c.1/strings/0x409/configuration
echo 120 > configs/c.1/MaxPower

# Keyboard function
mkdir -p functions/hid.usb0
echo 1 > functions/hid.usb0/protocol
echo 1 > functions/hid.usb0/subclass
echo 8 > functions/hid.usb0/report_length
# HID report descriptor (keyboard)
echo -ne \
'\x05\x01\x09\x06\xa1\x01\x05\x07\x19\xe0\x29\xe7\x15\x00\x25\x01\x75\x01\x95\x08\x81\x02\x95\x01\x75\x08\x81\x03\x95\x05\x75\x01\x05\x08\x19\x01\x29\x05\x91\x02\x95\x01\x75\x03\x91\x03\x95\x06\x75\x08\x15\x00\x25\x65\x05\x07\x19\x00\x29\x65\x81\x00\xc0' \
  > functions/hid.usb0/report_desc

# Mouse function
mkdir -p functions/hid.usb1
echo 2 > functions/hid.usb1/protocol
echo 1 > functions/hid.usb1/subclass
echo 3 > functions/hid.usb1/report_length
# HID report descriptor (mouse)
echo -ne \
'\x05\x01\x09\x02\xa1\x01\x09\x01\xa1\x00\x05\x09\x19\x01\x29\x03\x15\x00\x25\x01\x95\x03\x75\x01\x81\x02\x95\x01\x75\x05\x81\x03\x05\x01\x09\x30\x09\x31\x15\x81\x25\x7f\x75\x08\x95\x02\x81\x06\xc0\xc0' \
  > functions/hid.usb1/report_desc

# 將 function 連到 config（若連過就略過）
ln -sf functions/hid.usb0 configs/c.1/hid.usb0
ln -sf functions/hid.usb1 configs/c.1/hid.usb1

# 綁定到 UDC
if [[ -z "$UDC_NAME" ]]; then
  echo "找不到 UDC（/sys/class/udc 空）" >&2
  exit 1
fi
echo "$UDC_NAME" > UDC

echo "HID gadget up 完成：$UDC_NAME"

```

## 鍵盤、滑鼠模擬停止腳本
以管理員權限建立以下腳本:

/usr/local/sbin/hid-gadget-down.sh
```
#!/usr/bin/env bash
set -euo pipefail
GROOT="/sys/kernel/config/usb_gadget"
G="$GROOT/kbdmouse"
[ -d "$G" ] || { echo "gadget 不存在，略過"; exit 0; }
cd "$G"
# 1) 解除綁定
[ -e UDC ] && echo "" > UDC || true
# 2) 拆 symlink
rm -f configs/c.1/hid.usb0 configs.c.1/hid.usb1 2>/dev/null || true
rm -f configs/c.1/os_desc 2>/dev/null || true
# 3) 關 os_desc/webusb
[ -e os_desc/use ] && echo 0 > os_desc/use || true
rm -f os_desc/* 2>/dev/null || true; rmdir os_desc 2>/dev/null || true
[ -e webusb/use ] && echo 0 > webusb/use || true
rm -f webusb/* 2>/dev/null || true; rmdir webusb 2>/dev/null || true
# 4) 移 functions
rmdir functions/hid.usb1 2>/dev/null || true
rmdir functions/hid.usb0 2>/dev/null || true
rm -rf functions 2>/dev/null || true
# 5) 移 strings/config
rm -rf configs/c.1/strings/0x409 2>/dev/null || true
rmdir --ignore-fail-on-non-empty configs/c.1 2>/dev/null || true
rm -rf strings/0x409 2>/dev/null || true
rmdir strings 2>/dev/null || true
# 6) 刪 gadget 根
cd "$GROOT"; rmdir kbdmouse 2>/dev/null || rm -rf kbdmouse || true
echo "HID gadget 已清除（含 os_desc/webusb）"
```



## Change bash property

sudo chmod +x /usr/local/sbin/hid-gadget-up.sh
sudo chmod +x /usr/local/sbin/hid-gadget-down.sh


## Test 

### Send keyboard "A"

```
echo -ne "\0\0\x04\0\0\0\0\0" | sudo tee /dev/hidg0
echo -ne "\0\0\0\0\0\0\0\0"   | sudo tee /dev/hidg0
```

### Move mouse to right of 10 pixel 
echo -ne "\0\x0a\0" | sudo tee /dev/hidg1


## Register hid-gadget
sudo vim /etc/systemd/system/hid-gadget.service

/etc/systemd/system/hid-gadget.service
```
[Unit]
Description=USB HID Gadget (Keyboard + Mouse)
After=local-fs.target sys-kernel-config.mount
Wants=sys-kernel-config.mount

[Service]
Type=oneshot
ExecStartPre=/usr/sbin/modprobe dwc2
ExecStartPre=/usr/sbin/modprobe libcomposite
ExecStart=/usr/local/sbin/hid-gadget-up.sh
ExecStop=/usr/local/sbin/hid-gadget-down.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Register command:
```
sudo systemctl daemon-reload
sudo systemctl enable hid-gadget.service
```

## Change device promission
```
/dev/hidg0 寫入權限
預設常需 root。你可用 udev 規則放寬：

echo 'KERNEL=="hidg0", MODE="0666"' | sudo tee /etc/udev/rules.d/99-hidg.rules
sudo udevadm control --reload-rules && sudo udevadm trigger

echo 'KERNEL=="hidg1", MODE="0666"' | sudo tee /etc/udev/rules.d/99-hidg.rules
sudo udevadm control --reload-rules && sudo udevadm trigger
```

## Reboot
```
sudo reboot
```


## Problem
權限不足 bug:

```
10月 01 16:57:51 ubuntu systemd[1]: rm-kb-ms-web.service: Scheduled restart job, restart counter is at 100.
10月 01 16:57:51 ubuntu systemd[1]: Started rm-kb-ms-web.service - RM KB MS Web.
10月 01 16:57:51 ubuntu RM_KB_MS_Web[3071]: Unhandled exception. System.UnauthorizedAccessException: Access to the path '/dev/hidg0' is denied.
10月 01 16:57:51 ubuntu RM_KB_MS_Web[3071]:  ---> System.IO.IOException: Permission denied
10月 01 16:57:51 ubuntu RM_KB_MS_Web[3071]:    --- End of inner exception stack trace ---
10月 01 16:57:51 ubuntu RM_KB_MS_Web[3071]:    at Interop.ThrowExceptionForIoErrno(ErrorInfo errorInfo, String path, Boolean isDirError)
10月 01 16:57:51 ubuntu RM_KB_MS_Web[3071]:    at Microsoft.Win32.SafeHandles.SafeFileHandle.Open(String path, OpenFlags flags, Int32 mode, Boolean failForSymlink, Boolean& wasSymlink, Func`4 createOpenException)
10月 01 16:57:51 ubuntu RM_KB_MS_Web[3071]:    at Microsoft.Win32.SafeHandles.SafeFileHandle.Open(String fullPath, FileMode mode, FileAccess access, FileShare share, FileOptions options, Int64 preallocationSize, UnixFileMode openPermissions, Int64& fileLength, UnixFileMode& filePermissions, Boolean failForSymlink, Boolean& wasSymlink, Func`4 createOpenException)
10月 01 16:57:51 ubuntu RM_KB_MS_Web[3071]:    at System.IO.Strategies.OSFileStreamStrategy..ctor(String path, FileMode mode, FileAccess access, FileShare share, FileOptions options, Int64 preallocationSize, Nullable`1 unixCreateMode)
10月 01 16:57:51 ubuntu RM_KB_MS_Web[3071]:    at System.IO.FileStream..ctor(String path, FileMode mode, FileAccess access, FileShare share, Int32 bufferSize, FileOptions options, Int64 preallocationSize)
10月 01 16:57:51 ubuntu RM_KB_MS_Web[3071]:    at HidKeyboard..ctor(String devicePath) in C:\Users\jy.jy.huang\source\repos\RM_KB_MS_Web\RM_KB_MS_Web\Program.cs:line 88
10月 01 16:57:51 ubuntu RM_KB_MS_Web[3071]:    at Program.<Main>$(String[] args) in C:\Users\jy.jy.huang\source\repos\RM_KB_MS_Web\RM_KB_MS_Web\Program.cs:line 11
10月 01 16:57:51 ubuntu RM_KB_MS_Web[3071]:    at Program.<Main>(String[] args)
10月 01 16:57:52 ubuntu systemd[1]: rm-kb-ms-web.service: Main process exited, code=dumped, status=6/ABRT
10月 01 16:57:52 ubuntu systemd[1]: rm-kb-ms-web.service: Failed with result 'core-dump'.
10月 01 16:57:55 ubuntu systemd[1]: rm-kb-ms-web.service: Scheduled restart job, restart counter is at 101.
10月 01 16:57:55 ubuntu systemd[1]: Started rm-kb-ms-web.service - RM KB MS Web.
10月 01 16:57:55 ubuntu RM_KB_MS_Web[3085]: Unhandled exception. System.UnauthorizedAccessException: Access to the path '/dev/hidg0' is denied.
10月 01 16:57:55 ubuntu RM_KB_MS_Web[3085]:  ---> System.IO.IOException: Permission denied
10月 01 16:57:55 ubuntu RM_KB_MS_Web[3085]:    --- End of inner exception stack trace ---
10月 01 16:57:55 ubuntu RM_KB_MS_Web[3085]:    at Interop.ThrowExceptionForIoErrno(ErrorInfo errorInfo, String path, Boolean isDirError)
10月 01 16:57:55 ubuntu RM_KB_MS_Web[3085]:    at Microsoft.Win32.SafeHandles.SafeFileHandle.Open(String path, OpenFlags flags, Int32 mode, Boolean failForSymlink, Boolean& wasSymlink, Func`4 createOpenException)
10月 01 16:57:55 ubuntu RM_KB_MS_Web[3085]:    at Microsoft.Win32.SafeHandles.SafeFileHandle.Open(String fullPath, FileMode mode, FileAccess access, FileShare share, FileOptions options, Int64 preallocationSize, UnixFileMode openPermissions, Int64& fileLength, UnixFileMode& filePermissions, Boolean failForSymlink, Boolean& wasSymlink, Func`4 createOpenException)
10月 01 16:57:55 ubuntu RM_KB_MS_Web[3085]:    at System.IO.Strategies.OSFileStreamStrategy..ctor(String path, FileMode mode, FileAccess access, FileShare share, FileOptions options, Int64 preallocationSize, Nullable`1 unixCreateMode)
10月 01 16:57:55 ubuntu RM_KB_MS_Web[3085]:    at System.IO.FileStream..ctor(String path, FileMode mode, FileAccess access, FileShare share, Int32 bufferSize, FileOptions options, Int64 preallocationSize)
10月 01 16:57:55 ubuntu RM_KB_MS_Web[3085]:    at HidKeyboard..ctor(String devicePath) in C:\Users\jy.jy.huang\source\repos\RM_KB_MS_Web\RM_KB_MS_Web\Program.cs:line 88
10月 01 16:57:55 ubuntu RM_KB_MS_Web[3085]:    at Program.<Main>$(String[] args) in C:\Users\jy.jy.huang\source\repos\RM_KB_MS_Web\RM_KB_MS_Web\Program.cs:line 11
10月 01 16:57:55 ubuntu RM_KB_MS_Web[3085]:    at Program.<Main>(String[] args)
10月 01 16:57:56 ubuntu systemd[1]: rm-kb-ms-web.service: Main process exited, code=dumped, status=6/ABRT
10月 01 16:57:56 ubuntu systemd[1]: rm-kb-ms-web.service: Failed with result 'core-dump'.
```

解法:
```
sudo groupadd -f hidg
sudo usermod -aG hidg ubuntu
sudo udevadm control --reload-rules
sudo systemctl stop hid-gadget.service 2>/dev/null || true
sudo systemctl start hid-gadget.service
```

```
ls -l /dev/hidg*
# 期待看到：crw-rw---- root hidg ... /dev/hidg0
```

/etc/systemd/system/rm-kb-ms-web.service:
```
[Unit]
Description=RM KB MS Web
After=network-online.target hid-gadget.service
Wants=network-online.target
Requires=hid-gadget.service

[Service]
# 用非 root 帳號執行（請換成你的帳號）
User=ubuntu
# 讓行程擁有 hidg 群組存取權
SupplementaryGroups=hidg
# 你的應用啟動指令（請換成實際路徑/命令）
WorkingDirectory=/opt/rm-kb-ms-web
ExecStart=/usr/bin/dotnet /opt/rm-kb-ms-web/RM_KB_MS_Web.dll
Restart=always
RestartSec=2

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl daemon-reload
sudo systemctl restart rm-kb-ms-web.service
