# Arch Installation  

After hopping from a few Linux distrubutions for awhile I have decided to try out Arch. I believe that I have enough knowledge and experience using Linux to set up a customized and minimal installation which Arch should be able to give me.  

## End Goal

The end goal of this project is:  
1. Install Arch Linux.  
2. Installa graphical environment on top of Linux.  
3. Ensure that all components of the laptop are working under Linux.  
4. Customize the system to be my daily driver for a couple of years (or months if I get bored again).  

## Preparation  

I have backed up my .config folder into my Github account. Most of my files are synched up on OneDrive. I also have my /home partition on a separate hard drive, so I do not expect any loss of files during the process.  

I have downloaded the October 2020 image from the Arch Linux [download page][1]. Since I am on POP!_OS, I used Popsicle to burn the image to a USB drive.  

To be able to install Arch on the laptop, I have disabled secure boot in BIOS before booting into the installer.  

Using Popsicle is pretty straight forward. Just run the application, point it to the donwloaded image then click on Next. Then select your USB drive then click on Next. Enter your root credentials then wait for it to complete.  


## Booting The Installer

For the Machreator LF35, I have to press the ESC key during the start up secquence to get to the boot menu. Once the boot menu is up, it is just a matter of selecting my USB drive. Once the installer fully loads I am greeted with this screen.

## Setting Up WiFi  

The latop does not come with an ethernet port, so I will have to connect to my WiFi to get internet. When you boot into the Arch installer you get a message on how to connect to your WiFi. As indicated in the screen, we will use 'iwctl' to connect. Running the command drops you to a prompt:

>[iwd]#  
  
Here we type in device list to get a list wireless devices. Hopefully your WiFi card will be detected and listed. In my case, the Intel WiFi card got detected and was assigned to wlan0. We can then run the following command to get a list of available networks.

>[iwd]# station <network device> scan  
>[iwd]# station <network device> get-networks  

Once you see your WiFi network run the following to connect to it:  

>[iwd]# station <network device> connect <SSID>  

You will be prompted for the password. Once connected you can type in exit to return to the command prompt.  

We can confirm that we are connected by checking if we recieved an IP address. You can run the command ip address to check this:


We can also ping the [Arch Linux][2] website to confirm that we have internet connection.  
 
## Partitioning Harddrive

I have 2 hard drives installed on the system. An NVME SSD and a SATA SSD. Below is how they are setup:

As can be seen, the NVME drive is on /dev/nvme0n1 while my SATA drive is on /dev/sda/. The device /dev/sdb is the USB drive where the Arch installer is located.  

Since I already had Pop!_OS installed on this drive, you can see that we have several partitions already. I will be wiping all the partitions on the NVME drive and keeping the partitions on the SATA drive. I will also be setting up a swap partition on the SATA drive.  

For the NVME drive I will creating a new GPT partition. This partition will be split into a /boot and / partitions. I think 500MB should be ok for /boot. The remainder will be for the / partition.  

Since we will be partitioning the NVME drive we will issue the command:  

fdisk /dev/nvme0n1

We will then issue the command 'g' since we want to create a new gpt partition. We will then create a new partition for /boot, so we enter the command 'n' for new then accept the default partition number and default first sector. For the last sector I will input '+500M'. This will create a partition of type 'Linux filesystem'. Since this will be for EFI, I will change this by issuing the command 't'. Then for partition type I will use '1'. You can type in 'L' to see a list of available partition types if you want to.  

Next I will allocate the remaining drive to my / partition. So I issue the command 'n' for new partition, then use the defaults for partition number, first sector and last sector.

So if I type in 'p' to print my settings I will get the following:  

I will then need to write these settings to the disk by typing in 'w'.  

I will then need to format the partitions that I created. As can be seen in the lsblk command I have created 2 partitions:

-/dev/nvme0n1p1 with 500MB
-/dev/nvme0n1p2 with 931GB  

The EFI partition will need to be formatted as a FAT32 drive. For the / partition I will be using EXT4. I have thought about using BTRFS, but I guess I will wait a little bit more for the technology to mature before I use on my daily driver.  

To format the EFI partition I issue the command:  

mkfs.fat -F32 /dev/nvme0n1p1  

For the root partition I will issue the command:  

mkfs.ext4 /dev/nvme0n1p2  

Next I will be creating my swap partition and make it active. I already have the partition created by my Pop!_OS install so I can skip the partitioning of that. To set my swap drive I will use the following commands:  

mkswap /dev/sda2  
swapon /dev/sda2  

Once my partitions are created, I will need to mount them. First I need to mount /.  

mount /dev/nvme0n1p2 /mnt  

Next I will mount /dev/nvme0n1p1 to /boot, but first I will need to create the directory.  

mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot  

Next I will mount my home drive.  

mkdir /mnt/home  
mount /dev/sda1 /mnt/home  

With my drives mounted I can create the fstab file before I install Arch on the hard drive.  

mkdir /mnt/etc  
genfstab -U -p /mnt >> /mnt/etc/fstab 

Before I proceed with the installation, I will be cleaning up my /home folder as I do not want any of my settings from Pop!_OS to carry over into Arch.

## Installation 
  
So now we can start installing Arch Linux. I will installing the following packages:  
- base  
- base_devel  
- linux-lts  
- linux-lts-headers  
- linux  
- linux-headers  
- nvidia-lts  
- nvidia  
- mesa  
- vim  
- linux-firmware  
- intel-ucode

With our base installed, we can now switch to our installed system to finish the initial configuration.  
arch-chroot /mnt  

Next we set our localizations. So we first set up our timezone.  

ln -sf /usr/share/zoneinfo/Asia/Manila /etc/localtime  

Next we synch our clock.  

hwclock --systohc

Next we configure our lacale file.  

vim /etc/locale.gen  

Now I will uncomment en_PH.UTF-8 and save the file. Next I will run:  

locale-gen

Next I will set the language in the locale.conf file.  

echo "LANG=en_PH.UTF-8" >> /etc/locale.conf  

Next we set up the hostname for the laptop. So I create a hostname file in /etc with the name that I want for the laptop"

echo "devBox" >> /etc/hostname  

We will also need to edit the hosts file in /etc.  

vim /etc/hosts

We set the following:  

127.0.0.1	localhost
::1		localhost
127.0.1.1	devBox.localdomain	devBox  

Next we set up our root password and create a local user. To set the root password I type in 'passwd' and type in the root password that I want.  

To create a new user I type in:  

useradd -m -g users -G wheel eric  
passwd eric  

Next I want my local user to be able to run sudo. In the user creation command, I specified that the user should be a member of the wheel group. We will enable this group in our sudoers file.  

EDITOR=vim visudo  

I then scroll down and uncomment the first %wheel ALL=(ALL) ALL line. The second %wheel line tells thatmembers of the wheel group do not need to enter their password to use sudo, we do not want this. So keep that line commented.  

Next we install our WiFi tools so that we can reconnect to WiFi once we reboot.  

pacman -S networkmanager network-manager-applet wireless_tools wpa_supplicant dialog  

We then need to enable network manager.

systemctl enable NetworkManager  

Next we install the files we need to configure our bootmanager. For this installation I am planning on using systemd-boot.  

pacman -S efibootmgr os-prober mtools dosfstools  

Now we will install out bootloader.  

bootctl --path=/boot install  

Next we need to configure it. So we go into /boot/loader. Inside this directory we have several files, we will first edit loader.conf.  

vim loader.conf  

So here I can enable the timeout. I will also edit out the default entry to read arch-*  

Then we go into the /boot/loader/entries directory. Then we create a file here called arch.conf


When done, we can then exit out of the installer by typing in exit, unmounting our drives then rebooting the laptop. With everything done we are greeted with the systemd-boot menu and a few seconds later we are in our base Arch Linux installation.  

## XOrg and Window Manager  

For this installation, I plan on using i3-gaps as I really want a very minimal install of Linux running. So first we need to install Xorg, which it looks like got installed when we included mesa earlier. So we will also need a display manager so that our login screen will be graphical. I will use lightdm for this.  

sudo pacman -S lightdm lightdm-webkit2-greeter lightdm-webkit-theme-litarvan lightdm-gtk-greeter

Finaly we install i3-gaps.

sudo pacman -S i3-gaps

I will also need a terminal emulator before I restart. Personally I prefer tilix.

sudo pacman -S tilix

Lastly, lets enable lightdm before rebooting the laptop.

sudo systemctl enable lightdm.service

Once rebooted we are now greeted with our GTK lightdm screen. This is where I will end this section. I will create different sections for the configuration of each component starting with i3-gaps.


[1]: https://www.archlinux.org/download/
[2]: https://www.archlinux.org

