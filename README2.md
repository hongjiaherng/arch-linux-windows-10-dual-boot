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

## 1. *Keymap*
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