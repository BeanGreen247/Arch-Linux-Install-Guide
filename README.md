# Arch Linux Install Guide
This guide is aimed to make the instalation of Arch linux as easy as possible.

### NOTE
I am aware of the existence of the new arch installer, but it does not work in non-EFI system aka older systems and most VMs.

### Step 0

Get Arch Linux ISO file

https://archlinux.org/download/

Burn ISO to USB drive

Balena Etcher - https://www.balena.io/etcher/

dd - https://man7.org/linux/man-pages/man1/dd.1.html

Once done go into your BIOS and change boot priority to make sure that the usb drive is the first option. Then save and exit.

SUGGESTION >>> UNPLUG DRIVES THAT YOU DO NOT WANT TO INSTALL LINUX TO.

### Step 1
You might have to change your keymap. The default keymap is US. To change it list all layout by typing in 
```
ls /usr/share/kbd/keymaps/**/*.map.gz
```
To change the layout type in
```
loadkeys keymaplayoutnamehere
```

### Step 2
Check your internet connection. I should say that an ethernet connection makes the instalation much faster and easier.
```
ping -c 3 google.com
```
This will ping google.com 3 times and showing the time it took to get there and back. Usually in ms.

### Step 3
Time to setup our drives. Type in the following command to list drives in your system. 
```
fdisk -l
```
Once executed look for a string looking like /dev/sdX where X is the drive letter.

Next we need to partition our drive so type in
```
cfdisk /dev/sdX //change the X acording to your setup
```
NOTE the "//" marks a comment DO NOT type that in

Follow these steps

* select dos
* select gpt //only EFI
* select new
* setup a 128M EFI System //only for EFI
* make a linux swap partition (8GB if you can)
  * primary
  * then pick type as Linux swap / Solaris partition
* make the rest a ext4 linux filesystem
  * primary
  * then pick type as Linux if not set already
  * and set as bootable

To apply the changes select Write and type in yes followed by Quit

Format the created partitions like this
```
mkswap /dev/sda1
mkfs.ext4 /dev/sda2
```
For EFI
```
mkfs.fat -F32 /dev/sda1
mkswap /dev/sda2
mkfs.ext4 /dev/sda3
```
Now we have to mout our partitions
```
mount /dev/sda2 /mnt
swapon /dev/sda1
```
For EFI
```
mount /dev/sda3 /mnt
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
swapon /dev/sda2
```
### Step 4
Update the mirror list
```
pacman -Syy
```
### Step 5
To install Arch Linux on our drive we have to use pacstrap
```
pacstrap /mnt base base-devel linux-lts linux-firmware vim nano dhcpcd dhclient networkmanager
```
### Step 6
Grenerate fstab file
```
genfstab -U /mnt >> /mnt/etc/fstab
```
### Step 7
Enter the mounted os drive as root
```
arch-chroot /mnt
```
### Step 8
List the time zones
```
timedatectl list-timezones
```
If you know your capital city you can make it much easier by doing something like this
```
timedatectl list-timezones | grep Prague
```
NOTE: Prague is the name of my capital city, make sure to change it to yours.

To set it, type in
```
timedatectl set-timezone Europe/Prague
```
To check if you have set it correctly type in
```
timedatectl
```
My output
```
               Local time: Sun 2021-04-04 12:08:25 CEST
           Universal time: Sun 2021-04-04 10:08:25 UTC
                 RTC time: Sun 2021-04-04 10:08:25
                Time zone: Europe/Prague (CEST, +0200)
System clock synchronized: no
              NTP service: inactive
          RTC in local TZ: no
```

### Step 9
Network configuration

Setting a hostname
```
echo Zenith > /etc/hostname
```
NOTE: Zenith is just an example, a host name is the name used by your computer to make it visible to the network

Now we need to create a hosts file
```
touch /etc/hosts
```
Open the file using one of the two editors we installed
```
nano /etc/hosts
```
It should look something like this
```
127.0.0.1	localhost
::1		localhost
127.0.1.1	Zenith
```
NOTE: Zenith is just an example as metioned above

Enable NetworkManager service
```
systemctl enable NetworkManager
```

### Step 10
Set a root password
```
passwd
```
Type in the password you want to use as a root password

Adding a new user
```
useradd -m -g users -G wheel,storage,power,root beangreen247
passwd beangreen247
```
NOTE: beangreen247 is my username so make sure to change it to yours and passwd if for setting a password for the user.

### Step 11
Installing the grub bootloader
```
pacman -S grub
```
For EFI
```
pacman -S grub efibootmgr
```
To install it run

for MBR no EFI
```
grub-install --target=i386-pc --bootloader-id=GRUB /dev/sda
```
for EFI
```
grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi /dev/sda
```
NOTE: The efi-directory flags path has been created at the beginning by us

Now we need to make the grub config file
```
grub-mkconfig -o /boot/grub/grub.cfg
```



Exit arch-chroot

```
exit
```

And the finally reboot
```
reboot
```

### After reboot stuff

Add your user to the sudoers file
```
su
nano /etc/sudoers
```
uncomment the following line
```
# %wheel ALL=(ALL) ALL
```
to look like this
```
%wheel ALL=(ALL) ALL
```
That is all!
