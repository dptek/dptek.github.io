---
layout: post
title: "把 ArchLinux 安装到U盘，支持 BIOS 和 EFI 的电脑启动，带有 LUKS 全盘加密，U盘丢失也不怕个人敏感数据泄露！"
author: DPTEK
date: 2025-03-14 07:07:07 -0700
categories: 浏览器
image: assets/images/2025-03-14/安全浏览器推荐.png
---

Here's an updated guide that incorporates using `wipefs` to prepare the USB drive for full disk encryption during the Arch Linux installation process:

---

### **Preparation: Wipe the USB Drive**

1. **Identify the USB drive:**
   ```bash
   lsblk
   ```
   Confirm the device name for the USB (e.g., `/dev/sdX`).

2. **Wipe all file system signatures:**
   ```bash
   wipefs --all --force /dev/sdX
   ```

This step removes all existing file system signatures on the device while leaving the device itself intact, preparing it for new partitions.

---

### **Partition and Encrypt the USB**

1. **Create partitions:**
   - Use `sgdisk` to partition the device:
     - 10MB BIOS boot partition (unformatted)
     - 500MB EFI partition (FAT32)
     - Remaining space for the encrypted root partition
   ```bash
   sgdisk -o -n 1:0:+10M -t 1:EF02 -n 2:0:+500M -t 2:EF00 -n 3:0:0 -t 3:8304 /dev/sdX
   ```

2. **Format the EFI partition:**
   ```bash
   mkfs.fat -F32 /dev/sdX2
   ```

3. **Set up LUKS encryption on the root partition:**
   - Initialize LUKS on the root partition:
     ```bash
     cryptsetup luksFormat /dev/sdX3
     ```
   - Open the encrypted partition:
     ```bash
     cryptsetup open /dev/sdX3 cryptroot
     ```
   - Format the decrypted partition:
     ```bash
     mkfs.ext4 /dev/mapper/cryptroot
     ```

4. **Mount the encrypted partition:**
   ```bash
   mount /dev/mapper/cryptroot /mnt/usb
   ```

5. **Mount the EFI partition:**
   ```bash
   mkdir /mnt/usb/boot
   mount /dev/sdX2 /mnt/usb/boot
   ```

---

### **Proceed with Base Installation**

Follow the original steps for:
1. **Installing the base system:**
   ```bash
   pacstrap /mnt/usb linux linux-firmware base vim
   ```
2. **Generating an `fstab`:**
   ```bash
   genfstab -U /mnt/usb > /mnt/usb/etc/fstab
   ```

---

### **Encryption Configuration**

1. **Chroot into the new system:**
   ```bash
   arch-chroot /mnt/usb
   ```

2. **Edit `mkinitcpio` hooks for encryption:**
   - Open the configuration file:
     ```bash
     vim /etc/mkinitcpio.conf
     ```
   - Modify the `HOOKS` line to include `encrypt`:
     ```
     HOOKS=(base udev autodetect modconf block encrypt filesystems keyboard fsck)
     ```
   - Generate the new initramfs:
     ```bash
     mkinitcpio -P
     ```

3. **Configure GRUB for LUKS:**
   - Edit `/etc/default/grub`:
     ```bash
     vim /etc/default/grub
     ```
   - Add the `cryptdevice` parameter to the `GRUB_CMDLINE_LINUX_DEFAULT` line:
     ```
     GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 cryptdevice=UUID=<UUID-of-/dev/sdX3>:cryptroot root=/dev/mapper/cryptroot"
     ```
     Replace `<UUID-of-/dev/sdX3>` with the actual UUID:
     ```bash
     blkid /dev/sdX3
     ```
   - Install GRUB for BIOS and UEFI:
     ```bash
     grub-install --target=i386-pc --recheck /dev/sdX
     grub-install --target=x86_64-efi --efi-directory=/boot --recheck --removable
     ```
   - Generate the GRUB configuration:
     ```bash
     grub-mkconfig -o /boot/grub/grub.cfg
     ```

---

### **Complete the Installation**

Follow the original tutorial for:
- Setting up locale, time, hostname, and networking.
- Creating a user account and configuring `sudo`.

---

### **Finalize and Test**

1. **Exit the chroot:**
   ```bash
   exit
   ```

2. **Unmount all partitions:**
   ```bash
   umount -R /mnt/usb
   cryptsetup close cryptroot
   ```

3. **Reboot and test:**
   Upon booting, you will be prompted to enter the LUKS passphrase to unlock the root partition.

---

By using `wipefs` to clean the USB device and incorporating full disk encryption, this guide ensures a secure and clean setup for Arch Linux on a USB drive. Let me know if you have any further questions!