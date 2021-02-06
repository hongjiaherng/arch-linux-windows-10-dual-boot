Dual Boot Arch Linux & Windows 10
Introduction
- The following are my personal notes on dual booting Arch Linux with Windows 10. I have Windows 10 originally installed on my computer. This installation takes about 15 GB of spaces.

My computer
- UEFI system
- 512GB NVME ssd
- Wi-Fi (no ethernet)
- Intel cpu
- NVIDIA gpu

My choices
- Bootloader: grub
- Display manager: lightdm
- Desktop environment: xfce4

Getting started
1. Partition the disk (in my case, I allocate 100GB for my Arch)
2. Download Arch Linux ISO
3. Burn Arch Linux ISO into a usb stick
4. Boot into the usb stick

Start installing
1. Keymap
- In my case use the default US keymap, no change is needed
- To play with it, list all the available keymaps
    -> localectl list-keymaps
- You will see all the available keymaps and the one we use is 'us'
- There might seems alot to scroll through, so you may want to try this (if you replace <some-keywords> with 'u', you will see all the keymaps which contain 'u')
    -> localectl list-keymaps | grep <some-keywords>
- To set the keymap (replace the <keymap> with 'us' in my case)
    -> loadkeys <keymap>

2. Internet connection
- Try to ping some website to see if you are connected to internet (You should get error here, indicates the internet connection is not available)
    -> ping google.com
- Check the current internet status
    -> ip link
- Refer to the second output, in my case (using wifi) the network interface named 'wlan0'
- Observe closely at the attribute 'state', I got DOWN, which means I'm not connected to Internet
- Use iwctl to connect to internet
    - To get an interactive prompt do:
    -> iwctl
    - List all Wi-Fi devices: (observe the network interface name, in my case 'wlan0')
    -> device list
    - To scan for the networks: (change <device> with the network interface name, 'wlan0')
    -> station <device> scan
    - List all the available networks:
    -> station <device> get-networks
    - To connect to a network: (change <SSID> with the name of the network which you wanna connect)
    -> station <device> connect <SSID>
    - exit the iwctl interactive prompt
    -> exit
- Recheck the internet connection status
    -> ip link
- You should be able to see UP at 'state' (note that this is just a temporary connection, if you restart your computer now, you need to repeat the above steps)
- Reconfirm by ping some website (you should see something work properly here, CTRL-C to stop)
    -> ping archlinux.com

3. Synchronize network time protocol first
    -> timedatectl set-ntp true

4. Synchronize mirror list to get the fastest server to download packages
- Synchronize server
    -> pacman -Syyy
- Download 'reflector' package to sort server based on rate
    -> pacman -S reflector
- Set to the server to your country or nearest to your country (in my case, replace <country> with Singapore)
    -> reflector -c <country> -a 6 --sort rate --save /etc/pacman.d/mirrorlist
- Resync the server
    -> pacman -Syyy

5. Partitioning (Crucial part)
- To view the current available partitions on your disk
    -> lsblk
- The following is the illustration of my output (not the actual output) Observe the previous output properly and you won't see your empty partition which you created just now

nvme0n1         512G
├─nvme0n1p1     260M        (EFI System)
├─nvme0n1p2     701M        (Windows recovery environment)
├─nvme0n1p3     16M         (Microsoft reserved)
└─nvme0n1p4     375.14G     (Microsoft basic data)

- We use 'cfdisk' to do our job (in my case i write nvme0n1 because I'm using a NVME ssd, u might use sda instead, refer to your output from lsblk closely)
    -> cfdisk /dev/nvme0n1
- Observe the output, you will be able to see your other partitions on your computer and their respective type (EFI System, Microsoft basic data, Windows recovery environment, Microsoft reserved, ...), and most importantly the free space you created just now
- Navigate to the free space and press ENTER (select 'New' option)
- In my installation, I don't want to create a SWAP partition, but I decided to create a SWAP file instead later in my linux partition
- Thus, in my case, accept the default size by hit ENTER
- Write the changes to the disk by navigate to the option 'Write' and hit ENTER, confirm by typing in 'yes'
- Navigate to 'Quit' to quit cfdisk
- View the current available partitions again
    -> lsblk
- You will see your newly created partition (the following is the illustration for my output, the actual output have some more details)

nvme0n1         512G
├─nvme0n1p1     260M        (EFI System)
├─nvme0n1p2     701M        (Windows recovery environment)
├─nvme0n1p3     16M         (Microsoft reserved)
├─nvme0n1p4     375.14G     (Microsoft basic data)
└─nvme0n1p5     100G        (Linux filesystem)

6. Format the newly created partition
- Format the newly created partition (nvme0n1p5) to EXT4 filesystem
    -> mkfs.ext4 /dev/nvme0n1p5
- Feel free to enter cfdisk again to check the formatted partition filesystem type
    -> cfdisk /dev/nvme0n1

7. Mount the file systems
- Mount the directory '/mnt' to the linux partition (in my case, nvme0n1p5 is my linux partition, replace <partition> with nvme0n1p5)
    -> mount /dev/<partition> /mnt
- Make a directory for '/boot' in '/mnt' directory (this is to keep the arch linux boot entry to boot up the OS)
    -> mkdir /mnt/boot
- Mount it to the EFI partition (in my case, nvme0n1p1 is the EFI partition, replace <partition> with nvme0n1p1, EFI partition is the partition which contains the bootloader for our OS -> windows 10, now we have Arch linux)
    -> mount /dev/<partition> /mnt/boot
- Here we are making a special thing, we want our Arch linux OS able to access the windows 10 partition, so we make a directory inside '/mnt' which named '/windows10' and it will provide an entry point to the Windows 10 in Arch Linux
    -> mkdir /mnt/windows10
- So, we must mount the directory to the Windows 10's partition to be able to achieve that (in my case, nvme0n1p4 is my Windows 10's partition, replace <partition> with nvme0n1p4)
    -> mount /dev/<partition> /mnt/windows10
- View the changes we make so far (you will see the mount points are being displayed)
    -> lsblk
- My illustration for output
nvme0n1         512G
├─nvme0n1p1     260M        /mnt/boot       (EFI System)
├─nvme0n1p2     701M                        (Windows recovery environment)
├─nvme0n1p3     16M                         (Microsoft reserved)
├─nvme0n1p4     375.14G     /mnt/windows10  (Microsoft basic data)
└─nvme0n1p5     100G        /mnt            (Linux filesystem)

8. Base system install
- 'pacstrap' - the script use to install base sytem
- '/mnt' - location where the packages will be installed on
- 'base' - base system (but not enough for a fully functional base system, so it's necessary to install other packages)
- 'linux' - latest linux kernel
- 'linux-firmware' - linux firmware
- <text-editor> - replace it with the text editor you wanna use, most common one is 'nano' or 'vim', I use 'nano' here, replace <text-editor> with 'nano'
- <firmware-for-processor> - replace it with either 'intel-ucode' (if use intel processor), or 'amd-ucode' (if use amd processor), or don't need to install (if use virtual machine), I use intel processor, replace <firmware-for-processor> with 'intel-ucode'
    -> pacstrap /mnt base linux linux-firmware <text-editor> <firmware-for-processor> 

9. FSTAB 
- Generate a fstab file (append the UUID of of the partitions which contains '/mnt' directory to fstab file in this directory '/mnt/etc/fstab', in my case I have 'nvme0n1p1' (/mnt/boot), 'nvme0n1p4' (/mnt/windows10) and 'nvme0n1p5' (/mnt) which set '/mnt' as a mount point)
    -> genfstab -U /mnt >> /mnt/etc/fstab
- To view the changes
    -> cat /mnt/etc/fstab
- Or if you want to play around or edit it in case of errors (CTRL-X to exit)
    -> nano /mnt/etc/fstab

10. Enter the chroot
-> arch-chroot /mnt
- You will see the prompt has changed too [root@archiso /]#

11. Create swapfile (as we don't create SWAP partition just now)
- Create swapfile (allocate it with a length/ size of 2GB and name it '/swapfile')
    -> fallocate -l 2GB /swapfile
- Change the permission of the swapfile to full read and write access for root (Permissions of 600 mean that the owner has full read and write access to the file, while no other user can access the file)
    -> chmod 600 /swapfile
- Make it a swap file
    -> mkswap /swapfile
- Activate the swap file
    -> swapon /swapfile
- Put swap file to fstab file (replace <text-editor> with text editor which you wanna use, in my case 'nano')
    -> <text-editor> /etc/fstab
- You will enter the text editor, go to the new line below the available text, type in
    -> /swapfile none swap defaults 0 0
- the above line represent this:
    - '/swapfile' -> name of the file
    - 'none'      -> mount point is none
    - 'swap'      -> file type is swap
    - 'defaults'  -> options are set to defaults
    - '0'         -> set dump to 0
    - '0'         -> set pass to 0


12. Set timezone
- View the current status of timedatectl setting
    -> timedatectl status
- Observe the 'Local time', it should be corrected to your own timezone
- List all the available timezone (CTRL-C to quit out)
    -> timedatectl list-timezone
- So you want to filter out the timezone based on your region as there are too many to choose from, do this (replace <region> with your region name, in my case, 'Asia')
    -> timedatectl list-timezone | grep <region>
- After you got your proper timezone, set it (replace <region> and <city> with your corresponding region and city, in my case '.../Asia/Kuala_Lumpur')
    -> ln -sf /usr/share/zoneinfo/<region>/<city> /etc/localtime
- Synchronize system clock to hardware clock
    -> hwclock --systohc

13. Set locale
- Open the locale.gen file to edit
    -> nano /etc/locale.gen
- We want to use the locale (en_US.UTF-8 UTF-8), so find this line and uncomment it (delete the '#' in front of the line you want)
- CTRL-O to write changes, and CTRL-X to exit
- Generate locale
    -> locale-gen
- Add a this line "LANG=en_US.UTF-8" to the /etc/locale.conf file (can use text editor to add it too, the added line depends on the locale you use)
    -> echo "LANG=en_US.UTF-8" >> /etc/locale.conf
- If you change the keymap at the beginning (not using the default us keymap), you need to add the keymap to /etc/vconsole.conf file, replace <keymap> with the keymap you selected, if no, don't need to do this
    -> echo "KEYMAP=<keymap>" >> /etc/vconsole.conf

14. Set hostname
- Create and open the file '/etc/hostname' to set hostname (it's originally empty)
    -> nano /etc/hostname
- Enter hostname into the /etc/hostname file, in my case, I wanna call it 'arch'
- CTRL-O to write changes, CTRL-X to exit
- Open to add matching entries to the file '/etc/hosts' (it contains something originally)
    -> nano /etc/hosts
- Enter the following in the '/etc/hosts' file, replaced <hostname> with your previously set hostname, in my case 'arch'

127.0.0.1   localhost
::1         localhost
127.0.1.1   <hostname>.localdomain      <hostname>

- CTRL-O to write changes, CTRL-X to exit

15. Set the root password
-> passwd
- Enter the password and reconfirm it

16. Install bootloader named 'GRUB' for booting into Arch linux
- Install bootloader and some useful packages for later use
    - 'grub'                    - Bootloader we choose to use
    - 'efibootmgr'              - Linux user-space application to modify the EFI Boot Manager
    - 'os-prober'               - Utility to detect other OSes on a set of drives
    - 'ntfs-3g'                 - NTFS filesystem driver and utilities, install this because we wanna access Windows 10 partition later on the system, Windows 10 uses NTFS filesystem
    - 'networkmanager'          - Network connection manager and user applications
    - 'network-manager-applet'  - Applet for managing network connections
    - 'wireless_tools'          - Tools allowing to manipulate the Wireless Extensions
    - 'wpa_supplicant'          - A utility providing key negotiation for WPA wireless networks
    - 'dialog'                  - A tool to display dialog boxes from shell scripts
    - 'mtools'                  - A collection of utilities to access MS-DOS disks
    - 'dosfstools'              - DOS filesystem utilities
    - 'base-devel'              - A package group that includes tools needed for building packages (compiling and linking)
    - 'linux-headers'           - Headers and scripts for building modules for the Linux kernel	
    - 'git'                     - Distributed version control system
    - 'bluez'                   - Daemons for the bluetooth protocol stack
    - 'bluez-utils'             - Development and debugging utilities for the bluetooth protocol stack	
    - 'pulseaudio-bluetooth'    - Bluetooth support for PulseAudio
    - 'cups'                    - The CUPS Printing System - daemon package
    - 'openssh'                 - Premier connectivity tool for remote login with the SSH protocol
    -> pacman -S grub efibootmgr os-prober ntfs-3g networkmanager network-manager-applet wireless_tools wpa_supplicant dialog mtools dosfstools base-devel linux-headers git bluez bluez-utils pulseaudio-bluetooth cups openssh
- Install grub after downloading the package
    - 'x86_64-efi'        - target is 64 bit system
    - '/boot'             - the efi directory of Arch linux ('/mnt/boot')
    - 'GRUB'              - the bootloader id
    -> grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
- Generate configuration file for grub bootloader
    -> grub-mkconfig -o /boot/grub/grub.cfg

17. Enable some services to use it automatically when booting up the OS
- NetworkManager to automatically connect to internet
    -> systemctl enable NetworkManager
- bluetooth to automatically enable bluetooth service
    -> systemctl enable bluetooth
- org.cups.cupsd to automatically enable cups printing service
    -> systemctl enable cups
- sshd to automatically enable ssh
    -> systemctl enable sshd

18. Create new user
- Add user with 'wheel' which indicates user with sudo privilege, replace <username> with your own desired username
    -> useradd -mG wheel <username>
- Add password for the user you've created, replace <username> with your own desired username
    -> passwd <username>
- Edit the entries in visudo file to give the user sudo privilege
    -> EDITOR=nano visudo
- uncomment the line '%wheel ALL=(ALL) ALL
- CTRL-O to write changes, CTRL-X to exit

19. Leave the Arch ISO installer
- Exit the from root
    -> exit
- Unmount all the partitions
    -> umount -a
- Reboot the machine
    -> reboot
- You can remove the usb stick now
- Boot into Arch Linux by select Arch Linux in the GRUB bootloader
- Log into the user you've created just now (enter username and password)

20. Connect to the internet again
- Check the current internet connection
    -> ping google.com
- You should see error message here, to reconfirm check this
    -> ip link
- Refer to the second output, in my case (using wifi) the network interface named 'wlan0'
- Observe closely at the attribute 'state', it should be DOWN, which means I'm not connected to Internet
- Connect to Wi-Fi using networkmanager's curses-based interface (nmtui)
    -> nmtui
- Go to 'Activate a connection' and enter, select the Wi-Fi you wanna connect to
- After connecting, ESC to quit nmtui (you should connect to internet now)
- Feel free to refresh the server again
    -> sudo pacman -Syyy

21. Graphics driver
- Install graphics driver
    - If you are using virtual machine, replace <graphics-driver> with 'xf86-video-qxl'
    - If you have a Intel card, replace <graphics-driver> with 'xf86-video-intel'
    - If you have a AMD card, replace <graphics-driver> with 'xf86-video-amdgpu
    - If you have a NVIDIA card, replace <graphics-driver> with 'nvidia' and 'nvidia-utils'
    -> sudo pacman -S <graphics-driver>

22. Display server
- Install the group of display server, 'xorg'
    -> sudo pacman -S xorg

23. Display manager (depends on desktop environment you want to use) and Desktop environment
- I chose to use XFCE desktop environment, and thus lightdm display manager
- You can install your favourite display manager & desktop environment, kindly explore around from here:
    - [Display manager] https://wiki.archlinux.org/index.php/Display_manager
    - [Desktop environment] https://wiki.archlinux.org/index.php/Desktop_environment
- Install 'lightdm' related packages
    - 'lightdm' - display manager
    - 'lightdm-gtk-greeter' - greeter for the display manager (display when user login)
    - 'lightdm-gtk-greeter-settings' - settings tab to customize the lightdm greeter
    -> sudo pacman -S lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings

- Install XFCE desktop environment
    - 'xfce4' - desktop environment
    - 'xfce4-goodies' - extra packages for XFCE
    -> sudo pacman -S xfce4 xfce4-goodies

24. Enable display manager
- Enable lightdm so that it will automatically execute when we boot up the machine, you should replace 'lightdm.service' with other display manager's service if you are not using the same display manager with me
    -> sudo systemctl enable lightdm

25. Install yay (AUR (Arch User Repository) helper which helps to install packages from AUR)
- Clone the yay package using git
    -> git clone https://aur.archlinux.org/yay.git
- Change directory into 'yay/'
    -> cd yay/
- Make the package with the PKGBUILD file defined in yay
    -> makepkg -si PKGBUILD
- After installing yay, back to the original directory
    -> cd ..

26. Some extra things to install (browser, office tools, font, backup solution)
- Install chromium web browser, you can install other web browser such as firefox as well
    -> sudo pacman -S chromium
- Install wps office and its required font
    -> yay -S wps-office ttf-wps-fonts
- Install microsoft font 
    -> yay -S ttf-ms-fonts
- Install backup solution for the system which is 'timeshift'
    -> yay -S timeshift


27. Reboot
- Reboot the machine to boot into graphical desktop environment
    -> reboot
- If nothing goes wrong, you should be greeted by 'lightdm' after rebooting

28. Download some useful packages
- pamac - A graphical package manager
    -> yay -S pamac-all
- grub-customizer - A graphical application to customize grub bootloader
    -> sudo pacman -S grub-customizer
- xarchiver - Archive tools
    -> yay -S xarchiver
- vlc - VLC media player 
    -> sudo pacman -S vlc
- java
    -> sudo pacman -S jdk-openjdk
- Visual studio code
    -> yay -S visual-studio-code-bin

29. Solve some bugs
- Thunar file manager not showing 'trash'
    - Install gvfs
        -> sudo pacman -S gvfs

30. Start customizing