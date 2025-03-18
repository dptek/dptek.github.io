---
layout: post
title: "为 Arch Linux 添加 CachyOS 内核，把各种游戏都支持起来"
author: DPTEK
date: 2025-03-03 07:07:07 -0700
categories: Linux gaming
image: assets/images/2025-03-03/ArchLinux-CachyOS-0.png
---

# 在Arch linux下玩游戏的思路是使用CachyOS的内核和Proton组件

[![Watch the video](https://dptek.github.io/assets/images/2025-03-03/ArchLinux-CachyOS-0.png)](https://youtu.be/nBB_byld4ZA)

## 1. 添加 CachyOS 软件源：
来源： [CachyOS软件仓库](https://wiki.cachyos.org/features/optimized_repos/)

```bash
curl https://mirror.cachyos.org/cachyos-repo.tar.xz -o cachyos-repo.tar.xz
tar xvf cachyos-repo.tar.xz && cd cachyos-repo
sudo ./cachyos-repo.sh
```

##  2. 更新系统
```bash
sudo pacman -Syyu
```
## 3.安装CachyOS Kernel Manager 以及CachyOS内核
```bash
sudo pacman -S cachyos-gaming-kernel linux-cachyos linux-cachyos-headers
```
## 4.将内核添加至Grub或者Systemd启动菜单
systemd 启动：
```bash
cd /boot/loader/entries
sudo cp your.conf linux-cachyos.conf
sudo nvim linux-cacyos.conf
```
* 按照视频编辑内核名称

* 技巧：在开机启动时，在菜单中选中CachyOS启动项然后按 d 就可以将该项设置为默认启动项

grub启动：确保你已经添加Chaotic-aur软件源
```bash
sudo pacman -S update-grub
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
## 5.安装显卡驱动Intel、AMD或者Nvidia
Intel显卡：
```bash
sudo pacman -S xf86-video-intel
```
AMD显卡：
```bash
sudo pacman -S xf86-video-amdgpu
```
Nvidia显卡：
```bash
sudo pacman -S linux-cachyos-nvidia-open nvidia-utils lib32-nvidia-utils nvidia-settings
```
因为安装了nvidia显卡的内核你还需要回到第5步添加nvidia内核到你的systemd或者grub启动菜单
## 6.安装cachyos-gaming-meta
```bash
sudo pacman -S cachyos-gaming-meta steam lutris protongplus bottles
```