<div style = "color: red;text-align: center;font-size: 14pt">
<h1>warning don't turn off your computer during the installation</br>
(except when the guide told you to)</h1>
</div>
this installation guide only work with uefi enabled motherboard<br>
</br>

# wifi
iwctl</br>

```
[iwd]# device list
// REPLACE 'DEVICE' WITH YOUR WIFI DEVICE
[iwd]# station DEVICE scan
[iwd]# station DEVICE get-networks
[iwd]# station DEVICE connect NETWORK-SSID
[iwd]# exit
```

</br>

# configuring disk partition</br>

```
/* -- drive naming scheme -- */
sd = sata drive
nvme = nvme drive
X = hard drive your
Y = partition code in that hardrive
example:
    sda1 <--- first partition of sata drive a
    nvmen1p2 <--- second partition of nvme drive 1
/* -- drive naming scheme -- */
```

lsblk #list every drive exist in machine</br> 
fdisk /dev/sdXY or nvmenXpY #accessing drive to create and erase partition</br>

```
m <--- help menu
i <--- list current partition
d <--- (delete partition if exist)
g <--- enabling gpg
n <--- new partition

#new partition
//Recommended setup (separating home directory and arch linux directory)
partition 1 = default default +300M
partition 2 = default default +(ramvalue)G
partition 3 = default default +(minimal 30)G
partition 4 = default default default

p <--- print current partition
w <--- write current config to drive
```

</br>

# formating the partitions</br>
mkfs.fat -F32 /dev/sdX1 or nvmenXp1 <--- configuring partition 1 as a FAT32 partition</br>
mkfs.ext4 /dev/sdX3 or nvmenXp3 <--- configuring partition 3 as an ext4 partition</br>
mkfs.ext4 /dev/sdX4 or nvmenXp4 <--- configuring partition 4 as an ext4 partition</br>
</br>
# Mouting</br>
mount /dev/sdX3 or nvmenXp3 /mnt <--- mounting the partition 3 as linux directory</br>
mkdir /mnt/home <--- creating directory to mount home directory</br>
mount /dev/sdX4 or nvmenXp4 /mnt/home <--- mounting the parititon 4 as home directory</br>
## Mounting other hardrive</br>
mkdir -p /mnt/media/disk1 <--- creating mounting point for one drive</br>
mount /dev/sdXY or nvmenXpY /mnt/media/disk1 <--- mounting drive to the mounting point</br>
mount | grep /sda or nvme <--- checking if the mounting positition is correct or not</br>
</br>
## setting up SWAP</br>
[What is Linux Swap](https://averagelinuxuser.com/linux-swap/)</br>
mkswap /dev/sdX2 or nvmenXp2<br>
swapon /dev/sdX2 or nvmenXp2<br>
</br>
# selecting mirror</br>
reflector --country COUNTRY --age 12 --protocol https --sort rate --save /etc/pacman.d/mirrorlist <--- change COUNTRY to your country
</br>
</br>
# installing arch linux to hard drive</br>
pacstrap -i /mnt base linux linux-firmware gvim git</br> //you can use nano if you want to

```
default <--- accept all package
default <--- continue with the intallation
#this may take a while
```

</br>

# Configuring system</br>
genfstab -U /mnt >> /mnt/etc/fstab <--- generate fstab file</br>
cat /mnt/etc/fstab <--- check if the partition mounting correct or not</br>
arch-chroot /mnt <--- change root to arch installation</br>
</br>
## time zone</br>
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime <--- selecting the time zone</br>
hwclock --systohc <--- generate adjtime file</br>
</br>
## localization</br>
vim /etc/locale.gen <--- open locale.gen file with text editor</br>

```
#uncomment
en_US.UTF-8
yourCountryCode.yourCountryTimeCode
```

</br>
locale-gen <--- to generate the changes you made to locale.gen file</br>
vim /etc/locale.conf <--- open locale.conf file with text editor</br>

```
#setting the language of the OS
LANG=en_US.UTF-8 #for american english
```

</br>

## installing packages</br>
pacman -S base-devel grub efibootmgr dosfstools os-prober mtools linux-headers <--- boot package</br>
pacman -S network-manager-applet networkmanager wireless\_tools wpa\_supplicant dialog <--- networking package</br>
pacman -S linux-lts linus-lts-headers <--- optional</br>
</br>
## EFI setup</br>
mkdir /boot/EFI <--- making boot EFI directory</br>
mount /dev/sdX1 or nvmenXp1 /boot/EFI <--- mounting the partition 1 to boot EFI directory</br>
grub-install --target=x86\_64-efi --bootloader-id=grub-uefi --recheck <--- installing grub boot manager</br>
</br>
mkdir -p /boot/grub/locale <--- making directory</br>
cp /usr/share/locale/en\\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo <--- copy file to grub directory</br>
grub-mkconfig -o /boot/grub/grub.cfg <--- generating grub config</br>
</br>
## root password</br>
passwd</br>
</br>
## finishing up the installation</br>
exit <--- exit the arch environment</br>
umount -a <--- unmount everything (if you got a warning dont worry)</br>
reboot</br>
</br>
# Customizing the installation</br>
## enabling the network</br>
systemctl enable NetworkManager</br>
systemctl start NetworkManager</br>
connect your computer to wired internet connection</br>
(you can use your phone usb tethering if your computer don't have rj45 port)</br>
</br>
## enabling multilib repository</br>
vim /etc/pacman.conf <-- opening pacman.conf file with text editor</br>

```
remove the '#' symbol
#[multilib]
#Include = /etc/pacman.d/mirrorlist

```
</br>

## installing xorg package</br>
pacman -Syyy <-- making sure repository index in up to date</br>
pacman -S xorg-server <-- installing xorg package</br>
</br>
## installing video driver</br>
lspci <--- list of every pcie device on your computer</br>
lspci | grep VGA <--- list of every graphics card / integrated gpu on your device</br>

<b>intel</b></br>
pacman -S xf86-video-intel libgl mesa</br>
<b>amd</b></br>
pacman -S mesa xf86-video-amdgpu</br>
<b>nvidia</b></br>
pacman -S nvidia nvidia-libgl mesa</br>
pacman -S lib32-nvidia-utils lib32-nvidia-libgl lib32-mesa-demos libva-vdpau-driver nvidia-settings</br>
pacman -S nvidia-lts (optional but you need it if you install lts kernel)</br>
</br>
## creating an user</br>
useradd -m -g users -G wheel USERNAME <--- creating an user</br>
passwd USERNAME <--- creating password for user</br>
vim /etc/sudoers</br>

```
** -- default configuration -- **
##
##User privilege specification
##
root ALL=(ALL) ALL
** -- default configuration -- **
USERNAME ALL=(ALL) ALL <-- add this line to your /etc/sudoers/ file 
```

</br>

## installing bluetooth</br>
pacman -S bluez bluez-libs bluez-utils pulseaudio-bluetooth</br>
systemctl enable bluetooth</br>
systemctl start bluetooth</br>
bluetoothctl</br>

```
power on
agent on
exit
```
</br>

## installing desktop mananger</br>

```
GDM - Gnome dekstop manager
LightDM - cross desktop display manager
LXDM - LXDE dekstop manager
MDM - Linux mint dekstop manager
SDDM - KDE desktop manager
```

pacman -S DesktopManagerOfChoice <--- installing desktop manager package</br>
systemctl enable DesktopManagerOfChoice <--- enabling dekstop manager</br>
</br>

## installing desktop environment</br>
<b>GNOME</b></br>
pacman -S gnome gnome-terminal nautilus gnome-tweaks gnome-control-center gnome-backgrounds</br>
<b>XFCE</b></br>
pacman -S xfce4 xfce4-goodies xfce-terminal</br>
<b>KDE Plasma</b></br>
pacman -S plasma konsole dolphin spectacle kdeconnect</br>
</br>
## installing optional packages</br>
sudo pacman -S kate gedit firefox</br>
sudo pacman -S winetricks <--- for running windows application on linux</br>
</br>
reboot</br>
