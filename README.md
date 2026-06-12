# Panter-OS: Custom Kali Linux Live Builder

An automation project designed to create a customized, standalone, and portable Kali Linux Live ISO for cybersecurity laboratory environments.

**This project was created to learn Linux distribution customization and rebuilding processes.**

## 🎯 Project Purpose

This project produces a customized Live ISO based on Kali Linux. The hostname, MOTD, terminal appearance, wallpaper, and package selection are automatically applied during the build process.

All customizations are applied automatically through live-build configurations and hook scripts.

---

### 🌍 Building on Different Operating Systems

This project requires a Debian/Kali-based Linux environment and the `apt` package manager to function.

If you are using Windows or macOS, you should perform the build process inside a Linux virtual machine (Debian/Ubuntu/Kali) running on VirtualBox or VMware. It does not run directly on Windows or macOS.

---

## 💻 System Requirements and Recommended Environment

To avoid disk and memory bottlenecks (`xorriso` and `tmpfs` errors) during the build process, the following requirements are **critical**.

* **Recommended Development Environment:** It is recommended to perform build and testing operations in a clean virtual machine running on **VirtualBox** rather than directly on the host system.
* **Operating System:** Debian, Ubuntu, or Kali Linux (inside a virtual machine).
* **Free Disk Space:** At least **30 GB** (required for temporary chroot and rootfs files generated during the build process).
* **RAM:** At least **4 GB** (8 GB recommended).

---

## Features / Project Output

* Custom hostname (`cyber-node-01`)
* Custom MOTD (`PANTER-OS`)
* Custom terminal prompt (`[SEC-LAB]`)
* Custom wallpaper
* XFCE-based desktop environment
* Live ISO support
* VirtualBox compatibility
* Pre-installed lab tools (`nmap`, `wireshark`, `tmux`)

---

## 🚀 Step-by-Step Installation and Build Guide

Follow the steps below to build your own Panter-OS ISO image.

### Step 1: Install Required Packages

Install the build engine and required dependencies:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git live-build cdebootstrap xorriso
```

### Step 2: Clone the Project Skeleton

Download Kali's official build infrastructure and create the customization directories:

```bash
cd ~
git clone https://gitlab.com/kalilinux/build-scripts/live-build-config.git panter-os
cd panter-os

mkdir -p kali-config/common/includes.chroot/etc/skel/
mkdir -p kali-config/common/includes.chroot/root/
mkdir -p kali-config/common/includes.chroot/usr/share/panter-images/
mkdir -p kali-config/common/package-lists/
mkdir -p kali-config/common/hooks/live/
```

### Step 3: Prepare the Images

Before running the commands, make sure the images you plan to use are available on your Desktop (`~/Desktop/`) with the following names:

* `desktop.png` (for the XFCE desktop background)
* `login.png` (for the LightDM login screen background)

After confirming that the images are ready, run the following commands to copy them into the project directory:

```bash
cp ~/Desktop/desktop.png kali-config/common/includes.chroot/usr/share/panter-images/desktop.png
cp ~/Desktop/login.png kali-config/common/includes.chroot/usr/share/panter-images/login.png
```

### Step 4: System Identity (Hostname & MOTD)

Configure the welcome message displayed in the terminal and the system hostname:

```bash
cat << 'EOF' > kali-config/common/includes.chroot/etc/motd
============================================================
* DISTRIBUTION: PANTER-OS CYBERSECURITY LAB OS            *
* STATUS: SYSTEM SUCCESSFULLY LOADED...                   *
============================================================
EOF
```

```bash
echo "cyber-node-01" > kali-config/common/includes.chroot/etc/hostname
```

```bash
cat << 'EOF' > kali-config/common/includes.chroot/etc/hosts
127.0.0.1   localhost
127.0.1.1   cyber-node-01
::1         localhost ip6-localhost ip6-loopback
ff02::1     ip6-allnodes
ff02::2     ip6-allrouters
EOF
```

### Step 5: Terminal Customization and Package List

Define the custom `[SEC-LAB]` terminal prompt and the lab tools and VirtualBox X11 drivers to be included in the ISO:

```bash
# Terminal Prompt for the default user (kali)
cat << 'EOF' > kali-config/common/includes.chroot/etc/skel/.bashrc
export PS1="\[\e[1;31m\][SEC-LAB]\[\e[0m\]-\[\e[1;32m\]\u@\h\[\e[0m\]:\[\e[1;34m\]\w\[\e[0m\]\$ "
EOF
```

```bash
# Terminal Prompt for the root user
cat << 'EOF' > kali-config/common/includes.chroot/root/.bashrc
export PS1="\[\e[1;31m\][SEC-LAB]\[\e[0m\]-\[\e[1;32m\]\u@\h\[\e[0m\]:\[\e[1;34m\]\w\[\e[0m\]\$ "
EOF
```

```bash
# Required Tools and VirtualBox Drivers
cat << 'EOF' > kali-config/common/package-lists/panter-tools.list.chroot
nmap
wireshark
tmux
virtualbox-guest-x11
virtualbox-guest-utils
EOF
```

### Step 6: Automation Script (Hook Script)

Create a hook script that applies the customizations during the final stage of the build process.

```bash
cat << 'EOF' > kali-config/common/hooks/live/99-panter-os-setup.chroot
#!/bin/sh
echo ">>> Applying Panter-OS customizations..."

# 1. Replace the default wallpapers
cp -f /usr/share/panter-images/desktop.png /usr/share/backgrounds/kali/kali-wallpaper_16x9.png
cp -f /usr/share/panter-images/desktop.png /usr/share/backgrounds/kali/kali-wallpaper_16x9.svg

# 2. Replace the LightDM login screen background
cp -f /usr/share/panter-images/login.png /usr/share/desktop-base/kali-theme/login/background
cp -f /usr/share/panter-images/login.png /usr/share/desktop-base/kali-theme/login/background.svg

chmod -R 755 /usr/share/backgrounds/kali/
chmod -R 755 /usr/share/desktop-base/kali-theme/login/

# 3. Set Bash as the default shell instead of ZSH
chsh -s /bin/bash kali
chsh -s /bin/bash root

echo ">>> Panter-OS customization completed."
EOF
```

```bash
# Make the script executable
chmod +x kali-config/common/hooks/live/99-panter-os-setup.chroot
```

### Step 7: Start the Build Process

After completing all configurations, start the ISO build process using the XFCE desktop variant:

```bash
sudo lb clean
sudo ./build.sh --variant xfce && mv images/*.iso images/panter-os.iso
```

## ⚠️ Critical Checklist

* **RAM:** At least 4 GB
* **Video Memory:** 128 MB
* **Graphics Controller:** VBoxSVGA
* **3D Acceleration:** Disabled
* **Hook Script:** Must have executable permissions
