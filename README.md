# termux-chroot-ubuntu
A guide to install chroot ubuntu on your Android devices

## Installation

#### In Termux: Setting Up termux

```
termux-setup-storage
termux-change-repo

pkg update
pkg install x11-repo root-repo
pkg install tsu pulseaudio wget nano
```

#### In Termux: Install rootfs

```
tsu
mkdir /data/rootfs
cd /data/rootfs
wget http://cdimage.ubuntu.com/ubuntu-base/releases/22.04/release/ubuntu-base-22.04.1-base-arm64.tar.gz
tar -xvpf ubuntu-base-22.04.1-base-arm64.tar.gz --numeric-owner
mkdir sdcard dev/shm
```

#### File: /data/data/com.termux/files/usr/bin/pulse

From: https://github.com/ThieuMinh26/Proot-Setup/blob/main/Install_Script

```
#!/data/data/com.termux/files/usr/bin/sh

pulseaudio --start --exit-idle-time=-1
pacmd load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1 auth-anony
```

#### File: /data/data/com.termux/files/usr/bin/start 

From https://github.com/ThieuMinh26/Proot-Setup/blob/main/Chroot/start.sh

```
#!/data/data/com.termux/files/usr/bin/sudo sh

# fix /data mount options
# mount -o remount,dev,suid /data

mount --bind /dev ./chroot/dev
mount --bind /sys ./chroot/sys
mount --bind /proc ./chroot/proc
mount --bind /dev/pts ./chroot/dev/pts
mount --bind /dev/shm ./chroot/dev/shm
mount --bind /sdcard ./chroot/sdcard

# disable termux-exec
unset LD_PRELOAD

export PATH=/bin:/sbin:/usr/bin:/usr/sbin
export TERM=$TERM
export TMPDIR=/tmp

# change root to your username later
chroot ./chroot /bin/su - root
```

#### File: ../usr/bin/stop

From https://github.com/ThieuMinh26/Proot-Setup/blob/main/Chroot/stop.sh

```
#!/data/data/com.termux/files/usr/bin/sudo sh

umount ./chroot/dev/pts
umount ./chroot/dev/shm
umount ./chroot/dev
umount ./chroot/sys
umount ./chroot/proc
umount ./chroot/sdcard
```

#### In Termux: Start Ubuntu

```
cd ../usr/bin
chmod +x start stop pulse
start
```

#### In Ubuntu: Configure network and user group

From https://github.com/ThieuMinh26/Proot-Setup/blob/main/Chroot/Chroot-Termux

```
echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "127.0.0.1 localhost" > /etc/hosts

groupadd -g 3001 aid_bt
groupadd -g 3002 aid_bt_net
groupadd -g 3003 aid_inet
groupadd -g 3004 aid_net_raw
groupadd -g 3005 aid_admin

usermod -a -G 3001,3002,3003,3004,3005 root
usermod -a -G 3003 _apt
usermod -g 3003 _apt
```

#### In Ubuntu: Add new user (replace `cortmiem`)

```
apt update
apt upgrade
apt install nano sudo
adduser cortmiem
usermod -a -G 3001,3002,3003,3004,3005 cortmiem
echo "cortmiem ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/cortmiem
```

#### In Ubuntu: Install desktop environment

```
# 1.mate desktop and Gnome Flashback support HiDPI, look better on 2k tablet. 
apt install ubuntu-mate-desktop # full
apt install mate-desktop-environment-core

# 2.icewm takes less space provides windows-like interface
apt install icewm

# 3.no desktop environment, start xorg directly with wine
```

#### In Ubuntu: Install desktop environment

```
# 1.mate desktop and Gnome Flashback support HiDPI, look better on 2k tablet. 
apt install ubuntu-mate-desktop # full
apt install mate-desktop-environment-core

# 2.icewm takes less space provides windows-like interface
apt install icewm

# 3.no desktop environment, start xorg directly with wine
```

#### In Ubuntu: Install Xorg and Pulseaudio

```
apt install xorg tigervnc-standalone-server pulseaudio
echo "export PULSE_SERVER=127.0.0.1" >> /etc/profile
```

#### In Ubuntu: Configure VNC

```
su - cortmiem
# just type vncserver when use icewm with root.
LD_PRELOAD=/lib/aarch64-linux-gnu/libgcc_s.so.1 vncserver -xstartup mate-session -geometry=2560x1600
```

#### In Ubuntu: Stop VNC

```
# alias exit="vncserver -kill&exit"
```

#### In Ubuntu: Box64 Wine64

```
sudo apt install wget xz-utils sudo
wget https://github.com/ThieuMinh26/Proot-Setup/raw/main/Box86-64_Wine86-64.sh
sudo bash Box86-64_Wine86-64.sh


sudo apt install locales
sudo sh -c "echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen"
sudo sh -c "echo 'ja_JP.UTF-8 UTF-8' >> /etc/locale.gen"
sudo sh -c "echo 'zh_CN.UTF-8 UTF-8' >> /etc/locale.gen"

# CJK users install font pack 
# https://pan.baidu.com/s/1SWTe1Dj485FTJSdKqI6QCA?pwd=4abj
# https://1drv.ms/u/s!AiAVZceokcQYqUbCX5CqixCy0udp?e=7e5OEm

LANG=ja_JP box64 wine explorer
# LD_PRELOAD=/lib/aarch64-linux-gnu/libgcc_s.so.1 vncserver -xstartup "LANG=ja_JP box64 wine explorer /desktop=shell,1920x1200 explorer" -geometry=2560x1600
```

#### In Ubuntu: Zink, check [here](https://github.com/ThieuMinh26/Proot-Setup/blob/main/Zink)

```
ZINK: warning, this is *cpu-based conditional rendering*, say bye-bye to fps
```

shows on A13 Snapdragon 865+ and A13 Tensor G1. Help needed.

#### In Ubuntu: Visual Studio Code

```
sudo apt-get install wget gpg
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
rm -f packages.microsoft.gpg
```

