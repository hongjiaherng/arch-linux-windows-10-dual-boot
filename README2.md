# Dual Boot Arch Linux & Windows 10

## Introduction

The following are my personal notes on dual booting Arch Linux with Windows 10. I have Windows 10 originally installed on my computer. This installation takes about 15 GB of spaces.

## My computer

- UEFI system 
- 512GB NVME ssd
- Wi-Fi (no ethernet)
- Intel cpu
- NVIDIA gpu

## My choices

- Bootloader : grub
- Display manager : lightdm
- Desktop environment : xfce4

## Getting started

1. Partition the disk (in my case, I allocate 100GB for my Arch)
2. Download Arch Linux ISO
3. Burn Arch Linux ISO into a usb stick
4. Boot into the usb stick

## Start installing

### *1. Keymap*
- In my case, I use the default US keymap, no change is needed
- To play with it, list all the available keymaps
```console
    # localectl list-keymaps
```

- You will see all the available keymaps and the one we use is 'us'
- There might seems alot output to scroll through, so you may want to try this (if you replace ```<some-keywords>``` with 'u', you will see all the keymaps which contain 'u')
```console
    # localectl list-keymaps | grep <some-keywords>
```

- To set the keymap (replace the ```<keymap>``` with 'us' in my case)
```console
    # loadkeys <keymap>
```

&nbsp;
### *2. Internet connection*
- Try to ping some website to see if you are connected to internet (You should get error here, indicates the internet connection is not available)
```console
    # ping google.com
```

- Check the current internet status
```console
    # ip link
```
- Refer to the second output, in my case (using wifi) the network interface named 'wlan0'
- Observe closely at the attribute 'state', I got DOWN, which means I'm not connected to Internet
- Use iwctl to connect to internet
    - To get an interactive prompt do this
    ```console
        # iwctl
    ```

    - List all Wi-Fi devices: (observe the network interface name, in my case 'wlan0')
    ```console
        # device list
    ```

    - To scan for the networks: (change ```<device>``` with the network interface name, 'wlan0')
    ```console
        # station <device> scan
    ```
    
    - List all the available networks:
    ```console
        # station <device> get-networks
    ```

    - To connect to a network: (change ```<SSID>``` with the name of the network which you wanna connect)
    ```console
        # station <device> connect <SSID>
    ```

    - exit the iwctl interactive prompt
    ```console
        # exit
    ```

- Recheck the internet connection status
```console
    # ip link
```

- You should be able to see UP at 'state' (note that this is just a temporary connection, if you restart your computer now, you need to repeat the above steps)
- Reconfirm by ping some website (you should see something work properly here, CTRL-C to stop)
```console
    # ping archlinux.com
```

&nbsp;
### *3. Synchronize network time protocol*
```console
    # timedatectl set-ntp true
```

&nbsp;
### *4. Synchronize mirror list to get the fastest server to download packages*
- Synchronize server
```console
    # pacman -Syyy
```

- Download 'reflector' package to sort server based on rate
```console
    # pacman -S reflector
```

- Set to the server to your country or nearest to your country (in my case, replace ```<country>``` with Singapore)
```console
    # reflector -c <country> -a 6 --sort rate --save /etc/pacman.d/mirrorlist
```

- Resync the server
```console
    # pacman -Syyy
```

&nbsp;
### *5. Partitioning (Crucial part)*
- To view the current available partitions on your disk
```console
    # lsblk
```

- The following is the illustration of my output (not the actual output) Observe the previous output properly and you won't see your empty partition which you created just now
```console
    nvme0n1         512G
    ├─nvme0n1p1     260M        (EFI System)
    ├─nvme0n1p2     701M        (Windows recovery environment)
    ├─nvme0n1p3     16M         (Microsoft reserved)
    └─nvme0n1p4     375.14G     (Microsoft basic data)
```

- We use 'cfdisk' to do our job (in my case i write nvme0n1 because I'm using a NVME ssd, u might use sda instead, refer to your output from lsblk closely)
```console
    # cfdisk /dev/nvme0n1
```

- Observe the output, you will be able to see your other partitions on your computer and their respective type (EFI System, Microsoft basic data, Windows recovery environment, Microsoft reserved, ...), and most importantly the free space you created just now
- Navigate to the free space and press ENTER (select 'New' option)
- In my installation, I don't want to create a SWAP partition, but I decided to create a SWAP file instead later in my linux partition
- Thus, in my case, accept the default size by hit ENTER
- Write the changes to the disk by navigate to the option 'Write' and hit ENTER, confirm by typing in 'yes'
- Navigate to 'Quit' to quit cfdisk
- View the current available partitions again
```console
    # lsblk
```
- You will see your newly created partition (the following is the illustration for my output, the actual output have some more details)
```console
    nvme0n1         512G
    ├─nvme0n1p1     260M        (EFI System)
    ├─nvme0n1p2     701M        (Windows recovery environment)
    ├─nvme0n1p3     16M         (Microsoft reserved)
    ├─nvme0n1p4     375.14G     (Microsoft basic data)
    └─nvme0n1p5     100G        (Linux filesystem)
```