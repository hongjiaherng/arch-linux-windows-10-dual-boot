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
    - To get an interactive prompt do:
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


