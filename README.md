# How to install Arch Linux
by @piieboi

**THIS IS FOR UEFI SYSTEMS ONLY**

*(/dev/sda is to be replaced with the drive you're installing on, an SSD would be listed as `/dev/nvme0n1`*

## Connect to the internet:
use ethernet, setup wifi later
or
```bash
$ iwctl station [device] get-networks
$ iwctl station [device] connect [wifi name]
```
## Set Hardware Clock
```bash
$ timedatectl set-ntp true
```
## Partition your drives
this is the hard part, sorta
```bash
$ fdisk -l
``` 
Lists your drives
and
```bash
$ cfdisk /dev/nvme0n1
```
### Table
| TYPE          | SIZE               | LOCATION |
| ------------- |:-------------:     | -----:|
| EFI           | 500M               | /dev/sda1 |
| Linux SWAP    | 1/2 of RAM size*    | /dev/sda2 |
| Filesystem (x86_64) | everything else    | /dev/sda3 |

>I use the size of my ram or half the size since I run heavy apps and hibernate a lot. Older systems should definitely have more
swap and newer systems can completely exempt swap, It's whatever you feel like adding though.

## Format & Mount
another slightly hard part
### EFI partition
```bash
$ mkfs.fat -F32 /dev/sda1
```
### SWAP partition
```bash
$ mkswap /dev/sda2
$ swapon /dev/sda2
```
### FILESYSTEM partition
```bash
$ mkfs.ect4 /dev/sda3
$ mount /dev/sda3 /mnt
```

## Okay! Time for installing stuff!
> This will vary between people, I usually install everything here, you probably don't need like half of this stuff. also, I have downloaded everything I will need for a while
### Basic Install:
```bash
$ pacstrap /mnt base linux linux-devel linux-firmware linux-headers firefox nano grub networkmanager bluez bluez-utils efibootmgr dosfstools os-prober mtools sddm
```

### My Install: (Yes I know it's a lot)
```bash
$ pacstrap /mnt base linux linux-devel linux-firmware linux-headers firefox nano sudo grub wayland regreet greetd yay gdb ninja gcc cmake meson libxcb xcb-proto xcb-util xcb-util-keysyms libxfixes libx11 libxcomposite xorg-xinput libxrender pixman wayland-protocols cairo pango seatd libxkbcommon xcb-util-wm xorg-xwayland libinput libliftoff libdisplay-info cpio hyprland-nvidia-git networkmanager bluez bluez-utils xdg-desktop-portal-wlr efibootmgr dosfstools os-prober mtools
```
## Get into your install:
### Generate fstab
```bash
$ genfstab -U /mnt >> /mnt/etc/fstab
```
### Chroot
```bash
$ chroot /mnt
```
## Locales & Configs
### Locales
```bash
$ ln -sf /usr/share/zoneinfo/YOUR_REGION/YOUR_CITY /etc/localtime
$ nano /etc/locale.gen
```
**FOR LOCALE.GEN FUND YOUR LANGUAGE AND REMOVE THE "#" for `en_US.UTF-8 UTF-8`**

### Configs
```bash
$ hostnamectl system_name_here
$ nano /etc/hosts # do this to set up a local DNS server
```
### HOSTS FILE:
```
127.0.0.1      localhost
::1            localhost
127.0.1.1      system_name_here.localdomain      system_name_here
```

## Set up users
### SET YOUR ROOT PASSWORD (IN CASE YOU MESS UP, REMEMBER THIS!!!!!)
```bash
$ passwd
> SET A PASSWORD HERE
> CONFIRM IT
```
### Set up your main user
```bash
$ useradd -m username
$ passwd username
> enter usernames password (REMEMBER THIS)
> confirm it
$ usermod -aG wheel,video,audio username
```
## Setup GRUB
**THIS IS A HARD PART, DO EXACTLY AS I SHOW BECAUSE THIS CAN CAUSE YOU TO HAVE TO REMAKE YOUR ARCH INSTALL!!!**

### Install grub and it's dependencies if you didnt already:
```bash
$ pacman -S grub efibootmgr dosfstools os-prober mtools
```
### Set up bootloader folder
```bash
$ mkdir /boot/EFI
$ mount /dev/sda1 /boot/EFI
```
### INSTALL GRUB (HARD PART, DO EXACTLY LIKE I DID.)
```bash
$ grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=GRUB
```
> `--target=x86_64-efi` SETS IT UP FOR AMD64/Intel64 CPUs (x86_64 CPUs)

> `--efi-directory=/boot/EFI` SETS IT TO INSTALL AT `/boot/EFI`

> `--bootloader-id=GRUB` NAMES GRUB TO "GRUB" IN BIOS

```bash
$ grub-mkcongif -o /boot/grub/grub.cfg
```

## Turn on Networking
```bash
$ systemctl enable NetworkManager
```
## Turn on Lock Screen
```bash
$ systemctl enable sddm
```
# Congratulations! 🎉you've installed Arch Linux on your system!
## You can restart your system now OR follow along for NVIDIA drivers and KDE install, or a Hyprland Install
```lua
solo.to/
   ___  _     ___       _ 
  / ⌞ \(_)__ / ⌞ )___  (_)
 / ___/ / ⌞_) ⌞  / ⌞ \/ / 
/_/  / /\__/____/\___/ /                         
v15 /_/ 2023.06     /_/
"I AM NOT A FEMBOY"
```
____________

# NVIDIA DRIVERS
Follow the actual guide since i am not updating all of this:
[Follow it here](https://wiki.archlinux.org/title/NVIDIA#Automatic_configuration)
and
[Heres for Laptops](https://wiki.archlinux.org/title/NVIDIA_Optimus)
____________
# KDE Install
```bash
$ pacman -S xorg plasma-meta kde-applications wayland
```
done!
____________
# Hyprland Install
## Default Intsall:
```bash
$ pacman -S hyprland-git
```
## NVIDIA Install:
Install the `nvidia-dkms` driver and add it to your initramfs & kernel parameters.

Find the number [here](https://nouveau.freedesktop.org/CodeNames.html)

Follow the Guide [here](https://wiki.hyprland.org/Nvidia/) too!
```bash
$ pacman -S nvidia-dkms
```
For people using grub you can do this by adding `nvidia_drm.modeset=1` to the end of `GRUB_CMDLINE_LINUX_DEFAULT=` in `/etc/default/grub`, then run `grub-mkconfig -o /boot/grub/grub.cfg`
```bash
$ pacman -S hyprland-nvidia-git qt5-wayland qt5ct libva nvidia-vaapi-driver-git
```
> If your GPU is listed as supported by the `nvidia-open-dkms` driver, use that one instead. Note that on a laptop, it could cause problems with the suspended state when closing the lid, so you might be better off with `nvidia-dkms`.

> To get multi monitor to work properly on a hybrid graphics device (a laptop with both an Intel and an Nvidia GPU), you will need to remove the `optimus-manager` package if installed (disabling the service does not work). You also need to change your BIOS settings from hybrid graphics to discrete graphics.

> If you encounter crashes in Firefox, remove the line `env = GBM_BACKEND,nvidia-drm` 

> If you face problems with Discord windows not displaying or screen sharing not working in Zoom, remove or comment the line `env = __GLX_VENDOR_LIBRARY_NAME,nvidia.`
