---
layout: post
title: "Arka Linux GUI 让你像安装 Windows 一样在图形界面下简单安装 ArchLinux 无需任何命令行"
author: DPTEK
date: 2025-03-18 07:07:07 -0700
categories: 浏览器
image: assets/images/2025-03-18/ArkaLinuxGUI-install-arch-linux.png
---

## Arka Linux Gui 简介
Arka Linux GUI 是一个项目，旨在通过提供图形化安装程序来简化 Arch Linux 的安装过程，使其对那些觉得传统 Arch Linux 安装过于复杂的用户更友好。相比标准的 Arch Linux 安装方式（需要较强的命令行操作能力），Arka Linux GUI 提供了一个直观的界面，引导用户完成安装，同时保留了纯正的 Arch Linux 体验。

这个项目提供了多个版本，每个版本使用不同的桌面环境，比如 GNOME、KDE Plasma、XFCE、Cinnamon 或 i3 窗口管理器。这些版本分为两种类型：“纯净版”（pure editions），提供最基本的 Arch Linux 设置，几乎没有预装软件；以及“主题版”（themed editions），包含一些轻量级的定制、额外的软件包（如打印或蓝牙支持）以及更精美的外观，帮助用户快速上手。它的安装程序基于 Calamares，这是一个在许多 Linux 发行版中广泛使用的图形化安装工具，以简单易用著称。

***Arka Linux GUI 并不是一个独立的 Linux 发行版***，而是一个离线安装 Arch Linux 的工具，用户可以在安装前启动到一个实况（live）环境中体验。它遵循定期的发布计划，通常会更新 ISO 文件，以跟上 Arch Linux 滚动更新的步伐。这个项目的前身是 Arch Linux GUI (ALG)，但由于与官方 Arch Linux 团队的商标争议，后来更名为 Arka Linux GUI。

值得一提的是，这个项目的发展历程有些波折。它曾一度停止开发，但后来被社区贡献者重新拾起，目前仍在持续改进，重点放在稳定性和易用性上。对于想尝试 Arch Linux 但不愿从头开始面对命令行的用户来说，Arka Linux GUI 是一个不错的选择。不过安装完成后，它依然秉持 Arch Linux 的极简和“自己动手”哲学，不会偏离太远。

## Arka Linux 安装后的更新问题
由于 Arka 的镜像发布于去年的8月，如果他执行的是离线安装，那我们新安装好的Arch Linux就需要更新系统。旧系统更新是会遇到一些问题的。但是Arch本身就是个滚动更新的发行版，所以这些问题很好解决。

在使用 Arka Linux GUI 安装 GNOME 桌面环境后，如果更新过程中出现错误，可能是由于多种原因导致的，比如软件包冲突、镜像源问题、系统配置错误等。以下是一些常见的解决方法，供您逐步排查和解决问题：

### 1. 检查系统更新命令
确保您使用正确的命令更新系统。Arch Linux（包括通过 Arka Linux GUI 安装的系统）使用 `pacman` 包管理器。运行以下命令来更新系统：
```bash
sudo pacman -Syu
```
- `-S` 表示同步软件包。
- `-y` 刷新本地包数据库。
- `-u` 更新所有已安装的包。

如果遇到错误，请记下具体的错误信息，这对后续排查非常重要。

### 2. 检查网络连接和镜像源
更新出错可能是因为网络问题或镜像源不可用。Arka Linux GUI 默认使用 Arch Linux 的官方镜像源，但有时某些镜像可能响应缓慢或失效。
- 测试网络连接：
  ```bash
  ping archlinux.org
  ```
  如果无法 ping 通，请检查您的网络设置。
- 检查 `/etc/pacman.d/mirrorlist` 文件，确保镜像源有效。您可以手动编辑该文件，将靠近您所在地区的镜像移到顶部，或者使用以下命令生成新的镜像列表：

执行此命令找到国家代码：
```bash
  reflector --list-countries
```

  ```bash
  sudo pacman -Syy
  sudo reflector --country 'US' --latest 10 --sort rate --save /etc/pacman.d/mirrorlist
  ```
  如果您在中国，可以用 `--country CN`。

### 3. 解决依赖冲突
如果更新时提示依赖冲突（例如某个包需要特定版本的依赖），可以尝试以下方法：
- 清理包缓存并重新同步：
  ```bash
  sudo pacman -Scc  # 清理缓存
  sudo pacman -Syyu  # 重新同步并更新
  ```
- 如果仍然有冲突，手动安装冲突的包。例如，错误信息中提到的包可以用以下命令安装：
  ```bash
  sudo pacman -S 包名
  ```

### 4. 处理常见错误
以下是一些常见的更新错误及解决方法：
- **“GPG 签名无效”或“密钥错误”**：
  更新密钥环：
  ```bash
  sudo pacman -S archlinux-keyring
  sudo pacman-key --init
  sudo pacman-key --populate archlinux
  sudo pacman -Syu
  ```
- **“文件冲突”**：
  如果提示某个文件冲突，可以尝试强制覆盖：
  ```bash
  sudo pacman -Syu --overwrite '*'
  ```
  但请谨慎使用，确保不会覆盖重要配置文件。

## 更新未完成之后重启电脑
### 1. 添加Chaotic AUR
Chaotic AUR 可以实现用 pacman 直接安装已经编译好的软件包，减少从yay来安装AUR软件需要编译的机会。

```bash
sudo pacman-key --recv-key 3056513887B78AEB --keyserver keyserver.ubuntu.com
sudo pacman-key --lsign-key 3056513887B78AEB
```
```bash
sudo pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-keyring.pkg.tar.zst'
sudo pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-mirrorlist.pkg.tar.zst'
```

Then, we append (adding at the end) the following to /etc/pacman.conf:
```bash
[chaotic-aur]
Include = /etc/pacman.d/chaotic-mirrorlist
```
再次更新软件源
```bash
sudo pacman -Syu
```

### 2. 安装gnome的中文输入法ibus

如果你在安装时已经选择了zh_CN作为系统语言，你的Arch就已经显示中文了，你就可以直接安装输入法了。

如果你没在安装时选择中文你需要：
```bash
sudo vim /etc/locale.gen
```
在这个文本文件中去除zh_CN.UTF-8的#号
```bash
sudo locale-gen
```
ibus适合安装在gnome上
```bash
sudo pacman -S ibus  ibus-sunpinyin ibus-libpinyin ibus-table-chinese
```

## 完成了Arch Linux的安装和基本配置

这种安装方式比安装Windows和其他简单的发行版比你觉得怎样呢？