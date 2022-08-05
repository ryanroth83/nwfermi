# NextWindow Touchscreen on Ubuntu 18.04

This is a guide for installing / configuring a NextWindow Touchscreen on Ubuntu 18.04.

It is completely based on the work done by Daniel Newton / djp (https://launchpad.net/nwfermi/).
The driver consists of a GPL'd kernel part and a proprietary / closed source user space daemon, without this daemon it will not work.

Compared to the latest release this version has following enhancements:

- Fix driver compilation for recent kernels, basically it replaces every err() call with pr_err() as err() macro was removed from usb.h back in 2012
- Fix startup of user space deamon via udev/systemd implementation as you cannot launch 32 bit applications on 64 bits OSes via udev (blocked via seccomp in systemd-udevd)
- Xorg config example

Validated on a Dell SX2210TB, indentified as "1926:0064 NextWindow 1950 HID Touchscreen" by lsusb.
Confirmed to be working by user devkicks on a VAIO All in one VPCL11M1E, identified as "1926:0065 NextWindow 1950 HID Touchscreen" (See issue #1) 

# Install required packages

```
sudo apt-get install dkms build-essential autoconf2.13 xorg-dev xserver-xorg-dev xutils-dev libgrail-dev autoconf libtool lib32z1 lib32ncurses5 libc6-i386
```

# Build and patch original nwfermi driver

- Get latest deb from https://launchpad.net/nwfermi/trunk/0.6.5/+download/nwfermi-0.6.5.0_amd64.deb
- Extract deb
```
sudo dpkg -x nwfermi-0.6.5.0_amd64.deb nwfermi
```
- Copy nwfermi source to /usr/src
```
sudo cp -p -r nwfermi/usr/src/nwfermi-0.6.5.0 /usr/src/
```
- Apply patch
```
sudo cp -p nw-fermi.c.patch /usr/src/nwfermi-0.6.5.0/
sudo cd /usr/src/nwfermi-0.6.5.0/
sudo patch -p1 < nw-fermi.c.patch
```
- Make module
```
sudo dkms build nwfermi/0.6.5.0
```
- Install module
```
sudo dkms install nwfermi/0.6.5.0
```

# nwfermi user space daemon

- Copy daemon from deb to /usr/sbin
```
sudo cp nwfermi/usr/sbin/nwfermi_daemon /usr/sbin
```
- Install udev rules from this git repo
```
sudo cp etc/udev/rules.d/40-nw-fermi.rules /etc/udev/rules.d/
```
- Install systemd service file from this git repo
```
sudo cp etc/systemd/system/nwfermi@.service /etc/systemd/system/
```
- Reload systemd
```
sudo systemctl daemon-reload
```

# Rebuild xf86-input-nextwindow Xorg module

- Download xf86-input-nextwindow_0.3.4~precise1.tar.gz from https://launchpad.net/~djpnewton/+archive/ubuntu/xf86-input-nextwindow/+packages
- Extract and compile
```
sudo chmod +x autogen.sh ; ./autogen.sh
sudo make
sudo make install
```
- Install module
```
sudo cp /usr/local/lib/xorg/modules/input/nextwindow_drv.* /usr/lib/xorg/modules/input/
```

# Add Xorg configuration
- Copy Xorg config from this git repo
```
sudo cp etc/X11/xorg.conf.d/10-nwfermi.conf /etc/X11/xorg.conf.d/
```

# Add your local user to the input group

To be able to read the input device your local user should be part of the *input* group.
Optionally, you should/can add the gdm user to this group as well

```
sudo usermod -a -G input gdm
sudo usermod -a -G input ryan
```

# Reboot

Reboot and everything should work! Enjoy.
