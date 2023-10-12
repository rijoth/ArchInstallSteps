# Arch Linux Installation Steps

## First of all, check the internet connection
ping -c 3 archlinux.org

### for using wifi during installation

$ iwctl

#First, if you do not know your wireless device name, list all Wi-Fi devices:

[iwd]# device list

eg output: wlan0

#Then, to scan for networks:

[iwd]# station device scan

#You can then list all available networks:

[iwd]# station device get-networks

eg: station wlan0 get-networks

#Finally, to connect to a network:

[iwd]# station device connect SSID

eg: station wlan0 connect <network_name>

## If a passphrase is required, you will be prompted to enter it. Alternatively, you can supply it as a command line argument:

$ iwctl --passphrase passphrase station device connect SSID

#To disconnect from a network:

[iwd]# station device disconnect

###########################

## list the disks
fdisk -l
## run this command to create a new partition table:
cfdisk /dev/sda
## select gpt
## create 2 linux partitons  (root and home)
## create swap and efi partiton 
## write and exit

### mkfs.fat -F32 /dev/sda1 --  EFI
### mkfs.ext4 /dev/sda2 -- root
### mkfs.ext4 /dev/sda3  --home
### mkswap /dev/sda4 --swap

## Next, mount the root partition:
### mount /dev/sda2 /mnt

## Create a folder to mount the home partition and mount it:
mkdir /mnt/home
mount /dev/sda3 /mnt/home

## mount swap partition
swapon /dev/sda2

## check mounted partitions
lsblk

## installing the minimal Arch Linux system:
pacstrap -i /mnt base linux linux-firmware sudo nano

## generate the fstab file
genfstab -U -p /mnt >> /mnt/etc/fstab

## Chroot to the installed system
arch-chroot /mnt /bin/bash

## Set locale
nano /etc/locale.gen 
# uncomment  #en_US for American English

#generate the locale. Run:
locale-gen

#Create the locale.conf with corresponding language settings:
echo "LANG=en_US.UTF-8" > /etc/locale.conf

##To set the time zone, type:
ln -sf /usr/share/zoneinfo/ --  then press tab to see available options
eg: ln -sf /usr/share/zoneinfo/Asia/Calcutta /etc/localtime

## Set local time
hwclock --systohc --utc

## And check the time:
date

## Set hostname
echo rijo-pc > /etc/hostname

## need to add this name to the /etc/hosts file
nano /etc/hosts

## in the Nano editor, add this line at the end of the file:
127.0.1.1 localhost.localdomain rijo-pc

## install the network manager:
pacman -S networkmanager

systemctl enable NetworkManager
## for wifi
pacman -S iwd

### Set root password
passwd


## Install the GRUB bootloader and EFI boot manager packages:
pacman -S grub efibootmgr

## Install the bootlader on your system and generate its configuration files
mkdir /boot/efi

mount /dev/sda2 /boot/efi

lsblk -- to check if everything is mounted correctly

grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi --removable
grub-mkconfig -o /boot/grub/grub.cfg

## Reboot
exit
umount -R /mnt
reboot

## use username:root and passwd as set previously to login

## after the login, create a user account:
useradd -m -g users -G wheel -s /bin/bash <username>
passwd <username>

## enable sudo privileges for a newly created user:
EDITOR=nano visudo
%wheel ALL=(ALL) ALL -- search for this and uncomment by removing #

## save and exit 
exit

## login using username and password

## Install X window system and audio

(old)
pacman -S pulseaudio pulseaudio-alsa xorg xorg-xinit xorg-server

(new)
sudo pacman -S alsa-firmware alsa-utils pipewire pipewire-alsa pipewire-pulse pavucontrol xorg xorg-xinit xorg-server

## Install desktop environment, Iam installing XFCE desktop
pacman -S xfce4 lightdm lightdm-gtk-greeter
echo "exec startxfce4" > ~/.xinitrc
systemctl enable lightdm

## Start GUI
startx

#### Arch Linux installation is done! ######

# POST INSTALLATION

## Intel
### intel ucode
pacman -S intel-ucode
grub-mkconfig -o /boot/grub/grub.cfg

### Video Drivers
pacman -S xf86-video-intel vulkan-intel vulkan-icd-loader libva-intel-driver
##### Add your (kernel) graphics driver to your initramfs. For example, if using intel add i915:
MODULES=(i915 ...)

## AMD
sudo pacman -S amd-ucode 

sudo pacman -S --needed lib32-mesa vulkan-radeon lib32-vulkan-radeon vulkan-icd-loader lib32-vulkan-icd-loader
