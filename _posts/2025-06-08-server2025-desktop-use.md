---
layout: post
title: "正确设置 Windows Server 2025 作为你的桌面操作系统 "
author: DPTEK
date: 2025-06-08 07:07:07 -0700
categories: Windows
image: 
---

以下是调整 Windows Server 2025 为桌面操作系统的指南，**将命令行方法优先列出**，随后补充对应的 GUI 操作说明：

---

### **1. 安装 Desktop Experience**

#### **命令行**

##### PowerShell

```powershell
Install-WindowsFeature Desktop-Experience
Restart-Computer
```

#### **GUI**

1. 打开 **Server Manager**。
2. 点击 **Add Roles and Features**。
3. 在向导中，选择 **Desktop Experience** 功能（在 **User Interfaces and Infrastructure** 下）。
4. 安装完成后，重启服务器。

---

### **2. 启用主题**

#### **命令行**

##### CMD

```cmd
sc config Themes start= auto
net start Themes
```

##### PowerShell

```powershell
Set-Service -Name Themes -StartupType Automatic
Start-Service Themes
```

#### **GUI**

1. 打开 **Services**（运行 `services.msc`）。
2. 找到 **Themes** 服务。
3. 设置为 **Automatic** 并启动。
4. 右键单击桌面，选择 **Personalize**，选择一个主题。

---

### **3. 启用音频**

#### **命令行**

##### CMD

```cmd
sc config Audiosrv start= auto
net start Audiosrv
```

##### PowerShell

```powershell
Set-Service -Name Audiosrv -StartupType Automatic
Start-Service Audiosrv
```

#### **GUI**

1. 打开 **Services**。
2. 找到 **Windows Audio** 服务。
3. 设置为 **Automatic** 并启动。

---

### **4. 简化登录/注销**

#### **命令行**

##### 禁用 Ctrl+Alt+Del 登录

###### CMD

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v DisableCAD /t REG_DWORD /d 1 /f
```

###### PowerShell

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name DisableCAD -Value 1
```

##### 启用自动登录

###### CMD

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName /t REG_SZ /d "<YourUsername>" /f
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword /t REG_SZ /d "<YourPassword>" /f
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AutoAdminLogon /t REG_SZ /d "1" /f
```

###### PowerShell

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name DefaultUserName -Value "<YourUsername>"
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name DefaultPassword -Value "<YourPassword>"
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" -Name AutoAdminLogon -Value "1"
```

#### **GUI**

1. 打开 **Local Group Policy Editor** (`gpedit.msc`)。
2. 导航到 **Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options**。
3. 启用 **Interactive Logon: Do not require CTRL+ALT+DEL**。
4. 运行 `netplwiz`，取消勾选 **Users must enter a username and password to use this computer**，输入自动登录的凭据。

---

### **5. 调整电源设置**

#### **命令行**

##### CMD/PowerShell

```cmd
powercfg /setactive SCHEME_BALANCED
```

#### **GUI**

1. 打开 **Control Panel > Power Options**。
2. 选择 **Balanced** 电源计划或创建自定义计划。

---

### **6. 优化启动和关机**

#### **命令行**

##### 禁用 Server Manager 启动

###### PowerShell

```powershell
Set-ItemProperty -Path "HKCU:\Software\Microsoft\ServerManager" -Name DoNotOpenServerManagerAtLogon -Value 1
```

##### 管理启动项

###### PowerShell

```powershell
Get-Service | Where-Object {$_.StartType -eq "Automatic"} | Out-GridView -PassThru | Set-Service -StartupType Manual
```

#### **GUI**

1. 打开 **Task Manager** (`Ctrl+Shift+Esc`)。
2. 在 **Startup** 选项卡中禁用不必要的启动服务。
3. 在 **Server Manager** 中，转到 **Manage > Server Manager Properties**，禁用启动时自动打开。

---

### **7. 添加缺失的桌面功能**

#### **命令行**

##### 启用 Windows Search

###### CMD

```cmd
sc config WSearch start= auto
net start WSearch
```

###### PowerShell

```powershell
Set-Service -Name WSearch -StartupType Automatic
Start-Service WSearch
```

#### **GUI**

1. 打开 **Services**。
2. 找到 **Windows Search** 服务并设置为 **Automatic**。
3. 手动安装缺失的功能（如 Microsoft Store）。

---

### **8. 调整性能设置**

#### **命令行**

###### CMD

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\PriorityControl" /v Win32PrioritySeparation /t REG_DWORD /d 26 /f
```

###### PowerShell

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\PriorityControl" -Name Win32PrioritySeparation -Value 26
```

#### **GUI**

1. 右键单击 **This PC**，选择 **Properties > Advanced System Settings**。
2. 在 **Performance** 中点击 **Settings**，选择 **Adjust for best performance** 或自定义。

---

### **9. 安装桌面软件**

#### **命令行**

##### 使用 Chocolatey 快速安装

###### PowerShell

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
choco install googlechrome vlc -y
```

#### **GUI**

1. 手动从网络下载和安装常用的桌面软件。

---

### **10. 配置更新与安全性**

#### **命令行**

###### 启用 Windows Update

CMD:

```cmd
sc config wuauserv start= auto
net start wuauserv
```

PowerShell:

```powershell
Set-Service -Name wuauserv -StartupType Automatic
Start-Service wuauserv
```

#### **GUI**

1. 打开 **Windows Update**，检查更新并设置自动更新。

---

希望这些改进帮助您更高效地将 Windows Server 调整为桌面环境！如需进一步帮助，请随时联系我。


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