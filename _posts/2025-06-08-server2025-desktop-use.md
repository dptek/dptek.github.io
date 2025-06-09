---
layout: post
title: "正确设置 Windows Server 2025 作为你的桌面操作系统 "
author: DPTEK
date: 2025-06-08 07:07:07 -0700
categories: Windows
image: assets/images/2025-06-08/1.png
---

以下是使用 PowerShell 在 Windows Server 2025 中实现以下两项功能的脚本：

1. 禁用 **Ctrl+Alt+Del** 登录。
2. 禁用关机/重启事件跟踪。

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

Write-Output "设置已完成！无需重启即可生效。"
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

