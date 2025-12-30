---
layout: post
title: "Linux需要杀毒软件吗？我可以不装，但是你不能没有！"
author: DPTEK
date: 2025-12-26 07:07:07 -0700
categories: 开源软件
image: assets/images/2025-12-26/clamav.webp
---

### ClamAV 简介

ClamAV 是一款流行的开源（GPLv2）抗病毒引擎，专用于检测木马、病毒、恶意软件及其他恶意威胁。
它最初为类 Unix 系统（尤其是 Linux）开发，在邮件网关的邮件扫描方面表现出色，广泛应用于服务器端防护、网络扫描和终端安全。
主要特性包括：

命令行扫描器（clamscan）
多线程守护进程（clamd），支持更快扫描和实时访问扫描
通过 freshclam 自动更新病毒签名数据库

ClamAV 支持跨平台（Linux、macOS、Windows、BSD 等），由 Cisco Talos 维护。
虽然不像商业抗病毒软件那样提供完整的实时端点保护套件，但它轻量级、免费，且在按需扫描或服务器端恶意软件检测方面非常有效。

For more details, visit the official site at [clamav.net](https://www.clamav.net/).

### 安装 
```bash
sudo pacman -Syu clamav
```

### 初始设置
Update the virus database (required before first use or enabling services):
```bash
sudo freshclam
```

### ClamAV 系统服务
Enable and start automatic database updates:
```bash
sudo systemctl enable --now clamav-freshclam.service
```

Enable and start the ClamAV daemon (for faster scans with clamdscan or on-access scanning):
```bash
sudo systemctl enable --now clamav-daemon.service
```

For on-access real-time scanning (requires configuration in `/etc/clamav/clamd.conf`):
```bash
sudo systemctl enable --now clamav-daemon@scan.service  # or clamonacc if configured
```

Check service status:
```bash
systemctl status clamav-freshclam.service
systemctl status clamav-daemon.service
```

### ClamAV 主要命令
- **手动更新病毒特征库**:
  ```bash
  freshclam
  ```

- **扫描单独文件**:
  ```bash
  clamscan /path/to/file
  ```

- **嵌套扫描目录 (只显示感染文件)**:
  ```bash
  clamscan --recursive --infected /path/to/directory
  ```

- **全面系统扫描 (排除系统文件夹，例子)**:
  ```bash
  clamscan --recursive --infected --exclude-dir='^/sys|^/proc|^/dev' /
  ```

- **删除感染文件**:
  ```bash
  clamscan --recursive --infected --remove /path/to/directory
  ```

- **移动感染文件到隔离目录**:
  ```bash
  clamscan --recursive --infected --move=/path/to/quarantine /path/to/directory
  ```

- **平行同步扫描大目录 (使用所有 CPU 核心)**:
  ```bash
  find /path/to/directory -type f -print0 | xargs -0 -P $(nproc) clamscan
  ```

- **使用服务 daemon 进行扫描 (更快, 需要 clamd 已经在后台运行)**:
  ```bash
  clamdscan /path/to/file_or_directory
  ```

- **Monitor clamd resources (like top for ClamAV)**:
  ```bash
  clamdtop
  ```

- **Generate or check configuration files**:
  ```bash
  clamconf
  ```

### 病毒样本测试 ClamAV
Download the harmless EICAR test file and scan it:
```bash
curl -o eicar.com https://www.eicar.org/download/eicar.com
clamscan eicar.com
```
It should detect `EICAR-Test-File` as infected.

For more options, use `man clamscan`, `man freshclam`, `man clamdscan`, etc. Configuration files are in `/etc/clamav/`. Always run scans as a non-root user if possible, or ensure proper permissions on `/var/lib/clamav/`.