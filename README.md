<div style = "color: red;text-align: center;font-size: 14pt">
<h1>warning don't turn off your computer during the installation</br>
(except when the guide told you to)</h1>
</div>
this installation guide only work with uefi enabled motherboard<br>
</br>

# Wi-Fi Configuration 
- `iwctl`</br>

```
[iwd]# device list
// REPLACE 'DEVICE' WITH YOUR WIFI DEVICE
[iwd]# station DEVICE scan
[iwd]# station DEVICE get-networks
[iwd]# station DEVICE connect NETWORK-SSID
[iwd]# exit
```

</br>

# Configuring Disk Partition</br>

Drive naming Scheme  =  `sdXY` for SATA drive and `nvmenXpY` for NVME drive<br>

where,

- `sd` - SATA drive
- `nvme` - NVMe drive
- `X` - Replace `X` with your hard drive
- `Y` - Replace `Y` with the partition code in hard drive `X`

Example: 

- `sda1` - first partition of SATA drive `a`
- `nvmen1p2` - second partition of NVMe drive `1`

Commands:

- `lsblk`  
	lists every drive exist in machine</br> 
- `fdisk /dev/sdXY` or `nvmenXpY`  
	accesses drive to create and erase partition</br>

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

# Formatting Partitions</br>
- Configuring partition `1` of drive `X` as a FAT32 partition:

	​	`mkfs.fat -F32 /dev/sdX1` or `mkfs.fat -F32 /dev/nvmenXp1`

- Configuring partition `3` of drive `X` as a ext4 partition:

	​	`mkfs.ext4 /dev/sdX3` or `mkfs.ext4 /dev/nvmenXp3`

- Configuring partition `4` of drive `X` as a ext4 partition:

	​	`mkfs.ext4 /dev/sdX4` or `mkfs.ext4 /dev/nvmenXp4`



# Mounting</br>
- Mounting partition 3 as a Linux directory.

	`mount /dev/sdX3 or nvmenXp3 /mnt` </br>

- Creating a directory to mount home directory

	`	mkdir /mnt/home` </br>

- Mounting partition 4 as home directory

	`mount /dev/sdX4` or `nvmenXp4 /mnt/home`</br>

## Mounting Other Hard Drive</br>
- Creating mounting point for one drive

	​	`mkdir -p/mnt/media/disk1`

- Mounting drive to the mounting point

	​	`mount /dev/sdXY` or `nvmenXpY /mnt/media/disk1`

- Checking if the mounting position is correct

	​	`mount | grep /sda` or `mount | grep /nvmen1`



## Setting up SWAP</br>

[What is Linux Swap](https://averagelinuxuser.com/linux-swap/)</br>
`mkswap /dev/sdX2` or `mkswap /dev/nvmenXp2`<br>`swapon /dev/sdX2` or `swapon /dev/nvmenXp2`<br>
</br>

# Selecting Mirror Server</br>
Change the mirror server used for package manager

```shell
reflector --country COUNTRY --age 12 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

change `COUNTRY` to your country
</br>
</br>

# Installing Arch Linux to Hard Drive</br>
```shell
pacstrap -i /mnt base linux linux-firmware gvim git
```

You can use `nano` if you want to

```
default <--- accept all package
default <--- continue with the intallation
#this may take a while
```

</br>

# Configuring System</br>
Generating fstab file

```shell
genfstab -U /mnt >> /mnt/etc/fstab
```

Check if the partition mounting is correct<br>
	`cat /mnt/etc/fstab`

Change root to arch installation<br>
	`arch-chroot /mnt`



## Setting System Time Zone</br>
Selecting the time zone <br>
`ln -sf /usr/share/zoneinfo/Region/City /etc/localtime` 

Generate adjtime file<br>
`hwclock --stohc`

Generate adjtime file<br>
`hwclock --systohc`
</br>

## Localization</br>

Open locale.gen file with text editor - `vim /etc/locale.gen`

```
#uncomment
en_US.UTF-8
yourCountryCode.yourCountryTimeCode
```
</br>
Generate the changes made to locale.gen file</br>

`locale-gen` 

Open locale.conf file with text editor</br>
`vim /etc/locale.conf`

```
#setting the language of the OS
LANG=en_US.UTF-8 #for american english
```

</br>

## Installing Packages</br>
Boot packages:

```shell
pacman -S base-devel grub efibootmgr dosfstools os-prober mtools linux-headers
```

Networking packages:

```shell
pacman -S network-manager-applet networkmanager wireless_tools wpa_supplicant dialog iwd
```

Optional:

```shell
pacman -S linux-lts linus-lts-headers
```



## EFI Setup</br>
1. Making boot EFI directory</br>
	`mkdir /boot/EFI` <--- making boot EFI directory</br>

2. Mounting partition `1` to boot EFI directory</br>
	`mount /dev/sdX1 or nvmenXp1 /boot/EFI` 

3. Installing grub boot manager</br>
	`grub-install --target=x86\_64-efi --bootloader-id=grub-uefi --recheck`

4. Making locale directory</br>
	`mkdir -p /boot/grub/locale` 

5. Copy file to grub directory</br>
	`cp /usr/share/locale/en\\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo`
	
6. Generate grub config</br>
	`grub-mkconfig -o /boot/grub/grub.cfg`

## Root Password</br>
`passwd`</br>
</br>

## Finishing Up the Installation</br>
1. Exit arch environment</br>
	`exit`
2. Unmount everything (don't worry if you got a warning)</br>
	`umount -a` 
3. `reboot`</br>
	</br>

# Customizing the Installation</br>
## Enabling Network</br>
`systemctl enable NetworkManager`</br>
`systemctl start NetworkManager`</br>
connect your computer to wired internet connection</br>
(you can use your phone usb tethering if your computer don't have rj45 port)</br>
</br>

## Enabling multilib Repository</br>
Open pacman.conf file with text editor
`vim /etc/pacman.conf` 

```
remove the '#' symbol
#[multilib]
#Include = /etc/pacman.d/mirrorlist
```
</br>

## Installing xorg Package</br>
Make sure repository index is up to date</br>
`pacman -Syyy`

Install xorg package</br>
`pacman -S xorg`

## Installing Video Driver</br>
List every PCIE device on your computer</br>
`lspci`

List every graphic card/integrated gpu on your device</br>
`lspci | grep VGA` 

<b>intel</b></br>
`pacman -S xf86-video-intel libgl mesa`</br>

<b>amd</b></br>
`pacman -S mesa xf86-video-amdgpu`</br>

<b>nvidia</b></br>
`pacman -S nvidia-dkms nvidia-utils lib32-nvidia-utils`
</br>

## Creating a User</br>
Creating a user</br>
`useradd -m -g users -G wheel USERNAME` </br>
Replace USERNAME with your username</br>

Creating password for user</br>
`passwd USERNAME`


`EDITOR=vim visudo`</br>

```
# %wheel ALL=(ALL) ALL
```
uncomment this line</br>

</br>

## Installing Bluetooth</br>
`pacman -S bluez bluez-libs bluez-utils pulseaudio-bluetooth`</br>
`systemctl enable bluetooth`</br>
`systemctl start bluetooth`</br>
`bluetoothctl`</br>

```
power on
agent on
exit
```
</br>

## Installing Desktop Manager</br>

Desktop Manager (choose one)

- GDM - Gnome desktop manager
- LightDM - cross desktop display manager
- LXDM - LXDE desktop manager
- MDM - Linux mint desktop manager
- SDDM - KDE desktop manager

Installing desktop manager package</br>
`pacman -S DesktopManagerofChoice`


Enabling desktop manager</br>
`systemctl enable DesktopManagerofChoice`


</br>

## Installing Desktop Environment</br>
<b>GNOME</b></br>
`pacman -S gnome gnome-terminal nautilus gnome-tweaks gnome-control-center gnome-backgrounds`</br>
<b>XFCE</b></br>
`pacman -S xfce4 xfce4-goodies xfce-terminal`</br>
<b>KDE Plasma</b></br>
`pacman -S plasma konsole dolphin spectacle kdeconnect`</br>
</br>

## Installing Optional Packages</br>
`sudo pacman -S kate gedit firefox`</br>
`sudo pacman -S winetricks` <--- for running windows application on linux</br>
</br>


## Reboot</br>
`reboot`</br>
