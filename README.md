<h1 align="center"> Termux Guide Tutorial on How to <br> Install Ubuntu using chroot on Termux </h1>
<p align="center">
</p>

## CMD INSTALL 
```
pkg update
pkg install tsu pulseaudio
su
```

```
mkdir /data/local/tmp/chrootubuntu
cd /data/local/tmp/chrootubuntu
```

```
curl https://cdimage.ubuntu.com/ubuntu-base/releases/24.04/release/ubuntu-base-24.04.3-base-arm64.tar.gz -o ubuntu.tar.gz
```

```
tar xpvf ubuntu.tar.gz --numeric-owner
mkdir sdcard
```

```
cd ..
vi start_ubuntu.sh
```

```
#!/bin/sh

# The path of Ubuntu rootfs
UBUNTUPATH="/data/local/tmp/chrootubuntu"

# Fix setuid issue
busybox mount -o remount,dev,suid /data

busybox mount --bind /dev $UBUNTUPATH/dev
busybox mount --bind /sys $UBUNTUPATH/sys
busybox mount --bind /proc $UBUNTUPATH/proc
busybox mount -t devpts devpts $UBUNTUPATH/dev/pts

# /dev/shm for Electron apps
busybox mount -t tmpfs -o size=256M tmpfs $UBUNTUPATH/dev/shm

# Mount sdcard
busybox mount --bind /sdcard $UBUNTUPATH/sdcard

# chroot into Ubuntu
busybox chroot $UBUNTUPATH /bin/su - root

# Umount everything after exiting the shell. Because the graphical environment will be installed later, they are commented. If you do not want to install a graphics environment, uncomment the following commands.
#busybox umount $UBUNTUPATH/dev/shm
#busybox umount $UBUNTUPATH/dev/pts
#busybox umount $UBUNTUPATH/dev
#busybox umount $UBUNTUPATH/proc
#busybox umount $UBUNTUPATH/sys
#busybox umount $UBUNTUPATH/sdcard
```

```
chmod +x start_ubuntu.sh
sh start_ubuntu.sh
```

```
echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "127.0.0.1 localhost" > /etc/hosts

groupadd -g 3003 aid_inet
groupadd -g 3004 aid_net_raw
groupadd -g 1003 aid_graphics
usermod -g 3003 -G 3003,3004 -a _apt
usermod -G 3003 -a root

apt update && apt upgrade

apt install nano vim net-tools sudo git
```

```
dpkg-reconfigure tzdata
```

```
sudo apt install locales
sudo locale-gen en_US.UTF-8
```

```
sudo apt install xubuntu-desktop
```

```
sudo apt install software-properties-common
sudo add-apt-repository ppa:mozillateam/ppa
sudo apt-get update
sudo apt-get install firefox-esr
```

```
apt-get autopurge snapd

cat <<EOF | sudo tee /etc/apt/preferences.d/nosnap.pref
# To prevent repository packages from triggering the installation of Snap,
# this file forbids snapd from being installed by APT.
# For more information: https://linuxmint-user-guide.readthedocs.io/en/latest/snap.html
Package: snapd
Pin: release a=*
Pin-Priority: -10
EOF
```

```
vim .shortcuts/start_chrootubuntu.sh
```

```
#!/bin/bash

# Kill all old prcoesses
killall -9 termux-x11 Xwayland pulseaudio virgl_test_server_android termux-wake-lock

## Start Termux X11
am start --user 0 -n com.termux.x11/com.termux.x11.MainActivity

sudo busybox mount --bind $PREFIX/tmp /data/local/tmp/chrootubuntu/tmp

XDG_RUNTIME_DIR=${TMPDIR} termux-x11 :0 -ac &

sleep 3

# Start Pulse Audio of Termux
pulseaudio --start --exit-idle-time=-1
pacmd load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1 auth-anonymous=1

# Start virgl server
virgl_test_server_android &

# Execute chroot Ubuntu script
su -c "sh /data/local/tmp/startu.sh"
```

```
touch .shortcuts/start_chrootubuntu.sh
chmod +x .shortcuts/start_chrootubuntu.sh
```

```
pkg install tur-repo
pkg update -y && pkg upgrade -y
pkg install mesa-zink virglrenderer-mesa-zink vulkan-loader-android
```

```
MESA_LOADER_DRIVER_OVERRIDE=zink GALLIUM_DRIVER=zink ZINK_DESCRIPTORS=lazy virgl_test_server --use-egl-surfaceless &
```
