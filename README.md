# <img align="center" alt="Linux" width="50px" src="https://raw.githubusercontent.com/github/explore/80688e429a7d4ef2fca1e82350fe8e3517d3494d/topics/linux/linux.png" /> Linux commands and procedures

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

<table>
    <tr><td><code>wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.11.tar.xz</code></td></tr>
</table>

### 2. Unpack kernel

<table>
    <tr><td><code>xz -d linux-5.11.tar.xz</code></td><td>decompress archive</td></tr>
    <tr><td><code>tar -xf linux-5.11.tar</code></td><td>split file to directory</td></tr>
    <tr><td><code>ln -s /usr/src/linux-5.11 linux</code></td><td>make symbolic link for build</td></tr>
    <tr><td><code>cd /usr/src/linux</code></td><td>go to build dir</td></tr>
</table>

### 3. Create configuration

<table>
    <tr><td><code>make localmodconfig</code></td><td>use current config</td></tr>
    <tr><td><code>make menuconfig</code></td><td>show menu to configure (don't forget to save)</td></tr>
</table>

### 4. Build kernel

<table>
    <tr><td><code>make -j 36</code></td><td>build using 36 parallel tasks (+- 2 * Logical processors)</td></tr>
    <tr><td><code>make -j 36 modules</code></td><td>build drivers & addons</td></tr>
</table>

### 5. Install kernel

<table>
    <tr><td><code>make -j 36 modules_install</code></td><td>/lib/modules/</td></tr>
    <tr><td><code>make -j 36 install</code></td><td>/boot/</td></tr>
    <tr><td><code>reboot</code></td><td>finaly reboot system and boot to your new kernel</td></tr>
    <tr><td><code>cat /proc/version</code></td><td>check current kernel version</td></tr>
</table>

---

<p align="center">
    <b><a href="https://github.com/MGSE97" target="_blank">MGSE97</a> 🔥 2021</b>
</div>
