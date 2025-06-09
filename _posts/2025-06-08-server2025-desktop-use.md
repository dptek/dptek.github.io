---
layout: post
title: "正确设置 Windows Server 2025 作为你的桌面操作系统 "
author: DPTEK
date: 2025-06-08 07:07:07 -0700
categories: Windows
image: assets/images/2025-06-08/1.png
---



---

### **1. Install Desktop Experience**

To enable Desktop Experience:

#### **PowerShell**

```powershell
Install-WindowsFeature Desktop-Experience
```

After installation, restart the server:

```powershell
Restart-Computer
```

---

### **2. Enable Themes**

To enable and start the **Themes** service:

#### **CMD**

```cmd
sc config Themes start= auto
net start Themes
```

#### **PowerShell**

```powershell
Set-Service -Name Themes -StartupType Automatic
Start-Service Themes
```

---

### **3. Enable Audio**

To enable and start the **Windows Audio** service:

#### **CMD**

```cmd
sc config Audiosrv start= auto
net start Audiosrv
```

#### **PowerShell**

```powershell
Set-Service -Name Audiosrv -StartupType Automatic
Start-Service Audiosrv
```

---

### **4. Simplify Login/Logout**

#### Disable Ctrl+Alt+Del for Login

#### **CMD**

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v DisableCAD /t REG_DWORD /d 1 /f
```

#### **PowerShell**

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name DisableCAD -Value 1
```

#### Enable Auto-login

#### **CMD**

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName /t REG_SZ /d "<YourUsername>" /f
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword /t REG_SZ /d "<YourPassword>" /f
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AutoAdminLogon /t REG_SZ /d "1" /f
```

#### **PowerShell**

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name DefaultUserName -Value "<YourUsername>"
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name DefaultPassword -Value "<YourPassword>"
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name AutoAdminLogon -Value "1"
```

---

### **5. Adjust Power Settings**

Switch to the **Balanced** power plan:

#### **CMD**

```cmd
powercfg /setactive SCHEME_BALANCED
```

#### **PowerShell**

```powershell
powercfg /setactive SCHEME_BALANCED
```

---

### **6. Optimize Startup and Shutdown**

#### Disable Server Manager at Startup

#### **PowerShell**

```powershell
Set-ItemProperty -Path "HKCU:\Software\Microsoft\ServerManager" -Name DoNotOpenServerManagerAtLogon -Value 1
```

#### Manage Startup Items

To disable specific startup services, use:

#### **PowerShell**

```powershell
Get-Service | Where-Object {$_.StartType -eq "Automatic"} | Out-GridView -PassThru | Set-Service -StartupType Manual
```

---

### **7. Add Missing Desktop Features**

#### Enable Windows Search

#### **CMD**

```cmd
sc config WSearch start= auto
net start WSearch
```

#### **PowerShell**

```powershell
Set-Service -Name WSearch -StartupType Automatic
Start-Service WSearch
```

#### Install Microsoft Store (Sideload)

Sideloading the Microsoft Store requires downloading and installing the required packages manually or through Chocolatey:

```powershell
choco install microsoft-store
```

---

### **8. Tweak Performance Settings**

Switch to prioritize applications instead of background services:

#### **CMD**

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\PriorityControl" /v Win32PrioritySeparation /t REG_DWORD /d 26 /f
```

#### **PowerShell**

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\PriorityControl" -Name Win32PrioritySeparation -Value 26
```

---

### **9. Install Desktop Software**

#### Use Chocolatey for quick software installations:

#### **PowerShell**

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
choco install googlechrome vlc -y
```

---

### **10. Regular Updates and Security**

#### Set Windows Update to Auto:

#### **CMD**

```cmd
sc config wuauserv start= auto
net start wuauserv
```

#### **PowerShell**

```powershell
Set-Service -Name wuauserv -StartupType Automatic
Start-Service wuauserv
```

### 以下是一键执行脚本实现以上所有任务
请将以下脚本保存为 DesktopOptimization.ps1 
以管理员身份执行 .\DesktopOptimization.ps1

```powershell
# 提升权限执行
if (-not ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")) {
    Write-Warning "此脚本需要管理员权限，请以管理员身份运行 PowerShell！"
    exit
}

# 1. 设置执行策略为 Unrestricted
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope LocalMachine -Force

# 2. 安装 Desktop Experience
Write-Output "正在安装 Desktop Experience..."
Install-WindowsFeature Desktop-Experience
Restart-Computer -Force

# 脚本将在重启后继续运行
if (Test-Path -Path "$env:TEMP\DesktopOptimization-PostRestart.flag") {
    Remove-Item -Path "$env:TEMP\DesktopOptimization-PostRestart.flag"
} else {
    New-Item -Path "$env:TEMP\DesktopOptimization-PostRestart.flag" -ItemType File
    Write-Output "完成重启后再次运行脚本以完成后续配置。"
    exit
}

# 3. 启用 Themes 服务
Write-Output "启用 Themes 服务..."
Set-Service -Name Themes -StartupType Automatic
Start-Service Themes

# 4. 启用 Windows Audio 服务
Write-Output "启用 Windows Audio 服务..."
Set-Service -Name Audiosrv -StartupType Automatic
Start-Service Audiosrv

# 5. 禁用 Ctrl+Alt+Del 登录
Write-Output "禁用 Ctrl+Alt+Del 登录..."
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name DisableCAD -Value 1

# 6. 设置自动登录
Write-Output "配置自动登录..."
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name DefaultUserName -Value "<YourUsername>"
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name DefaultPassword -Value "<YourPassword>"
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name AutoAdminLogon -Value "1"

# 7. 设置电源模式为平衡
Write-Output "设置电源模式为平衡..."
powercfg /setactive SCHEME_BALANCED

# 8. 禁用 Server Manager 启动
Write-Output "禁用 Server Manager 启动..."
Set-ItemProperty -Path "HKCU:\Software\Microsoft\ServerManager" -Name DoNotOpenServerManagerAtLogon -Value 1

# 9. 启用 Windows Search 服务
Write-Output "启用 Windows Search 服务..."
Set-Service -Name WSearch -StartupType Automatic
Start-Service WSearch

# 10. 调整性能设置优先考虑程序
Write-Output "调整性能设置..."
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\PriorityControl" -Name Win32PrioritySeparation -Value 26

# 11. 安装 Chocolatey 并安装常用软件
Write-Output "安装 Chocolatey..."
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.SecurityProtocolType]::Tls12
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

Write-Output "安装常用软件 (Google Chrome, VLC)..."
choco install googlechrome vlc -y

# 12. 启用 Windows Update
Write-Output "启用 Windows Update 服务..."
Set-Service -Name wuauserv -StartupType Automatic
Start-Service wuauserv

Write-Output "所有配置已完成，您的系统已优化为桌面使用！"
```