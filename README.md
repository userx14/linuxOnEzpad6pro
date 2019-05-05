# linuxOnEzpad6pro
tutorial on how to get Debian running on Ezpad 6 pro

# configure packet management
configure /etc/apt/sources.list

```
deb http://security.debian.org/debian-security stretch/updates main contrib
deb-src http://security.debian.org/debian-security stretch/updates main contrib

deb http://ftp.de.debian.org/debian/ stretch main contrib non-free
deb-src http://ftb.de.debian.org/debian/ stretch main contrib non-free

deb http://deb.debian.org/debian/ stretch-updates main contrib
deb-src http://deb.debian.org/debian/ stretch-updates main contrib
```

# do after installation
```apt update```

```apt install git```

# compile kernel for silead
1) get the requiered tools

```sudo apt-get install dmidecode build-essential libncurses-dev bison flex libssl-dev libelf-dev```

2) get latest linux kernel from kernel.org

```wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.0.11.tar.xz -P ./Downloads```

3) extract the archive's content

```tar -xf Downloads/linux-4.20.2.tar.xz```

4) enter the directory of the kernel

```cd Downloads/linux-4.20.2```

5) clean the repository

```sudo make clean```

6.1) get dmi parameters of your device with dmidecode, viewing only the first 20 lines (which should contain the BIOS information, if not search through the complete output)

```sudo dmidecode | head -n 20```

6.2) edit the touchscreen_dmi.c file and add entries as described in section silead_ts

```sudo nano ./drivers/platform/x86/touchscreen_dmi.c```

7.1) start the terminal-gui based configuration

```sudo make menuconfig```

7.2) enable the following option

Kernal 4.x

Device Drivers  --->X86 Platform Specific Device Drivers  --->DMI based touchscreen configuration info

Kernel 5.x

Device Drivers  ---Input device support ---- touchscreens ----silead I2C touchscreen

Device Drivers  --->X86 Platform Specific Device Drivers  --->DMI based touchscreen configuration info

8) save .config and exit

9) compilation will take a long time - 3 hours+ on my tablet with celeron n3450, using only two threads because otherwise it tends to overheat
```sudo make -j2 deb-pkg```
10)
```cd ..```
11) install the debian packages of the compiled kernel
```
sudo dpkg -i linux-libc-dev_4.20.2-1_amd64.deb
sudo dpkg -i linux-headers-4.20.2_4.20.2-1_amd64.deb
sudo dpkg -i linux-image-4.20.2_4.20.2-1_amd64.deb
sudo dpkg -i linux-image-4.20.2-dbg_4.20.2-1_amd64.deb
```

12) add entry for the new kernel version in grub

```sudo update-grub```

13) copy driver for your device from this repository to

```cp ./tablet-name-driver.fw /lib/firmware/silead/tablet-name-driver.fw```

14) reboot and select linux 4.20 in grub

```reboot```

15) check for possible errors

```dmesg | grep silead```

16) if you have modified touchscreen_dmi.c then please consider creating a patch for the mainline kernel.
see this youtube video by FOSDEM - Greg Kroah-Hartman "Write and Submit your first Linux kernel Patch": https://www.youtube.com/watch?v=LLBrBBImJt4

# get wifi working on rtl8723bu
git clone https://github.com/lwfinger/rtl8723bu
cd rtl8723bu
edit Makefile and comment EXTRA_CFLAGS += -DCONFIG_CONCURRENT_MODE
apt instal make
make
make install
echo "blacklist rtl8xxxu"> /etc/modprobe.d/rtl8xxxu-blacklist.conf

# fix diagonal screen tearing
```touch /usr/share/X11/xorg.conf.d/20-intel.conf```

```nano /usr/share/X11/xort.conf.d/20-intel.conf```

fill file with:

```
Section "Device"
   Identifier  "Intel Graphics"
   Driver      "intel"
   Option      "TearFree"    "true"
EndSection
```

# logical scrolling direction:
## to test input
```apt install evtest xinput```
### find out which event id is the trackpad (on my tablet it is event15)
```cat /dev/input/event15```

and watch for incoming data it you touch the touchpad
## setup scroll direction
```cd /usr/share/X11/xorg.conf.d/```

```nano 20-natural-scrolling.conf```

fill file with:

```
Section "InputClass"
        Identifier "Natural Scrolling"
        MatchDevicePath "/dev/input/event15"
        Driver "libinput"
        Option "ButtonMapping" "1 2 3 4 5 7 6"
EndSection
```



# Adonit stylus support
```git clone https://github.com/GeReV/adonit_linux```

```cd adonit_linux```

```apt install libglib2.0-dev libdbus-1-dev```

```make```

## get Blueman to find out mac address of the pen
```apt install blueman```

find out bluetooth id
## to test input
```apt install evtest xinput```
### find out which event id is touchscreen (probably the last one)
```cat /dev/input/event19```
### watch for output when you touch the screen

```./adonit --pen 00:17:53:51:DD:5D --touchscreen /dev/input/event19```

