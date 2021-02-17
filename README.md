# <img align="center" alt="Linux" width="32px" src="https://raw.githubusercontent.com/github/explore/80688e429a7d4ef2fca1e82350fe8e3517d3494d/topics/linux/linux.png" /> Linux commands and procedures

Usefull linux commands cheatset.

Contains some common task procedures.

If you wanna add or change something, make a pull request.

Will be updated continuously.

# Commands

## Packages

|Command / Location|Description|
|------------------|-----------|
|`apt update`|update local repositories packages lists|
|`apt install package_name`|install package|
|`apt upgrade`|update packages in system|
|`apt dist-upgrade`|update system packages|

## Utils

|Command / Location|Description|
|------------------|-----------|
|`sudo`|execute as root|
|`sudo -i`|switch to root|
|`exit`|close current context|
|`echo $PATH`|prints current path values|
|`export PATH=$PATH:/sbin`|extend path with custom directory|
|`~/.bashrc`|run commands on bash start for user (login, ssh, ...)|
|`which bash`|prints command file location|

## Disk management

|Command / Location|Description|
|------------------|-----------|
|`mount`|show current mounted directories|
|`cfdisk /dev/sdb`|partition disk|
|`blkid`|get partitions ids and types|
|`ncdu`|compute space in dir and subdirs, has gui|
|`du -sh`|compute dir size|
|`df`|show partitions usages|
|`/etc/fstab`|automatic mount on boot (use `UUID` from `blkid`)|

## Networking

|Command / Location|Description|
|------------------|-----------|
|`wget http://example.com`|download file from web|
|`ip a`|show current network interfaces and configuration|
|`dhclient enp0s8`|run DHCP client on interface|
|`/etc/network/interfaces`|permament interfaces configuration ([static](files/interfaces_static), [dhcp](files/interfaces_automatic))|

## Process management

|Command / Location|Description|
|------------------|-----------|
|`ps`|show current user processes|
|`top`|show processes|
|`htop`|show processes with gui|
|`kill -15 pid`|kill process|
|`kill -9 pid`|kill process without mercy|

## Files

|Command / Location|Description|
|------------------|-----------|
|`cat file_name`, `more file_name`, `less file_name`|read file contents|
|`nano file_name`, `vim file_name`, `echo 'text' > file_name`|write file contents|
|`ps > file_name`|redirect output to file (`write`)|
|`ps >> file_name`|redirect output to file (`append`)|

## Directories

|Command / Location|Description|
|------------------|-----------|
|`ls -alF`|list directory content|
|`ln -s from to`|creates symbolic link to location|
|`mc`|midnight commander (file manager)|

# Procedures

## Bash file

Can be executed using `./file_name`, if it has `x` attribute enabled.
To enable execution, use `chmod +x file_name`.

Basic structure
```
#!/bin/bash

commands
```
or
```
#!/usr/bin/env bash

commands
```

## Building kernel

### 1. Get kernel from web [kernel.org](https://kernel.org)

| |
|-|
|`wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.11.tar.xz`|

### 2. Unpack kernel

| | |
|-|-|
|`xz -d linux-5.11.tar.xz`|decompress archive|
|`tar -xf linux-5.11.tar`|split file to directory|
|`ln -s /usr/src/linux-5.11 linux`|make symbolic link for build|
|`cd /usr/src/linux`|go to build dir|

### 3. Create configuration

| | |
|-|-|
|`make localmodconfig`|use current config|
|`make menuconfig`|show menu to configure (don't forget to save)|

### 4. Build kernel

| | |
|-|-|
|`make -j 36`|build using 36 parallel tasks (+- 2 * Logical processors)|
|`make -j 36 modules`|build drivers & addons|

### 5. Install kernel

| | |
|-|-|
|`make -j 36 modules_install`|/lib/modules/|
|`make -j 36 install`|/boot/|
|`reboot`|finaly reboot system and boot to your new kernel|
|`cat /proc/version`|check current kernel version|


---

<p align="center">
    <b><a href="https://github.com/MGSE97" target="_blank">MGSE97</a> 🔥 2021</b>
</div>