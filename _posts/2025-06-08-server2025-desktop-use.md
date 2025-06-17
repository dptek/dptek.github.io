---
layout: post
title: "正确设置 Windows Server 2025 作为你的桌面操作系统 "
author: DPTEK
date: 2025-06-08 07:07:07 -0700
categories: Windows
image: assets/images/2025-06-08/1.png
---

## 激活 Windows Server 2025

1. 脚本来源：https://massgrave.dev/ （github的一个项目，是安全的）
2. 脚本启动：irm https://get.activated.win | iex
3. 使用 TSforge选项进行激活

## 使用 PowerShell 在 Windows Server 2025 中实现以下三项功能的脚本：

1. 禁用 **Ctrl+Alt+Del** 登录。
2. 禁用关机/重启事件跟踪。
3. 禁用 Server Manager 开机启动

---

### **完整 PowerShell 脚本**

```powershell
# 确保脚本以管理员权限运行
if (-not ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")) {
    Write-Warning "此脚本需要管理员权限，请以管理员身份运行 PowerShell！"
    exit
}

# 禁用 Ctrl+Alt+Del 登录
Write-Output "正在禁用 Ctrl+Alt+Del 登录..."
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name DisableCAD -Value 1

# 禁用关机/重启事件跟踪
Write-Output "正在禁用关机/重启事件跟踪..."
# 创建 Reliability 键（如果不存在）
if (-not (Test-Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Reliability")) {
    New-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT" -Name Reliability -Force | Out-Null
}

# 设置 ShutdownReasonOn 为 0（禁用事件跟踪）
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows NT\Reliability" -Name ShutdownReasonOn -Value 0

# 禁用 Server Manager 启动
Set-ItemProperty -Path "HKCU:\Software\Microsoft\ServerManager" -Name DoNotOpenServerManagerAtLogon -Value 1

Write-Output "设置已完成！无需重启即可生效"
```

---

### **运行脚本步骤**
最直接的方式就是复制黏贴以上脚本到你的Powershell ISE 编辑器中然后点击执行按钮。

1. 将以上脚本保存为文件，例如 `DisableFeatures.ps1`。
2. 打开 PowerShell，以管理员权限运行：

   ```powershell
   .\DisableFeatures.ps1
   ```
---

## 如何设备管理器中大量无法识别的设备以及驱动的安装

1. 设置位置：Windows update -> Advanced Options -> optional updates
2. 安装其中列出的驱动后大多数未知设备将会正常驱动

## 个别笔记本中触控板问题

1. Load a W11 Pro image, and explore the mounted folder.

2. 从Windows 11 拷贝下面的三个文件夹到你的Ventoy U盘： 来源："\Windows\System32\DriverStore\FileRepository" :

文件夹名：
amdi2c.inf_amd64_d7ae71f8eb52c084
hidi2c.inf_amd64_e94f3ca241858aef
iai2c.inf_amd64_a77c815b2999404d
3. 启动到Ventoy FirPE 环境， 拷贝上面三个文件夹到Server 2025 同样位置，并覆盖。
4. 重启Server 2025
5. 有必要的话重新安装笔记本官方的触控板驱动，重启。
6. 在设置程序中关闭触控板在重新打开，即可正常使用了

## 结论

虽然Windows Server 2025 精简但是配置麻烦还有些兼容性问题，所以目前只推荐在台式机上使用。

对于笔记本用户和台式机用户首选的还是我们的 Windows 11 iot LTSC
