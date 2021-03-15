[< Back to menu](../README.md)

---

# Commands

## Terminal Key Bindings

|Keys|Action|
|----|------|
|`UP Arrow`, `DOWN Arrow`|moving throught personal command history|
|`CTRL + C`|exit command|
|`CTRL + Z`|stop command|
|`CTRL + U`|cut text before cursor|
|`CTRL + K`|cut text after cursor|
|`CTRL + Y`|restore previous text|
|`CTRL + W`|cut word before cursor|
|`ALT + .`|paste last command args|

## Utils

|Command / Location|Description|
|------------------|-----------|
|`!!`|execute last command|
|`sudo`|execute as root|
|`sudo !!`|execute last command as root|
|`sudo -i`|switch to root|
|`exit`|close current context|
|`echo $PATH`|prints current path values|
|`export PATH=$PATH:/sbin`|extend path with custom directory|
|`~/.bashrc`|run commands on bash start for user (login, ssh, ...)|
|`which bash`|prints command file location|
|`watch command`|executes command in interval and shows output|

## Power commands

|Command / Location|Description|
|------------------|-----------|
|`shutdown`, `shutdown -h`|shutdown in minute|
|`shutdown -h now`|shutdown now|
|`shutdown -r`|reboot in minute|
|`shutdown -r now`, `reboot`|reboot now|
|`shutdown -c`|cancel pending shutdown|

## Hardware
|Command / Location|Description|
|------------------|-----------|
|`nvidia-smi`|Nvidia GPU info|
|`lshw`|Hardware info|
|`cat /proc/cpuinfo`|CPU info|
|`neofetch`|System info|

## Packages

|Command / Location|Description|
|------------------|-----------|
|`apt update`|update local repositories packages lists|
|`apt install package_name`|install package|
|`apt upgrade`|update packages in system|
|`apt dist-upgrade`|update system packages|

## Networking

|Command / Location|Description|
|------------------|-----------|
|`wget http://example.com`|download file from web|
|`ip a`|show current network interfaces and configuration|
|`ifup enp0s8`|activate interface|
|`ifdown enp0s8`|deactivate interface|
|`dhclient enp0s8`|run DHCP client on interface|
|`dhclient -r`|renew DHCP IP Address|
|`dhcp-lease-list`|list DHCP clients on server|
|`/etc/network/interfaces`|permament interfaces configuration ([static](files/network/interfaces_static), [dhcp](files/network/interfaces_automatic))|
|`nslookup domain [dns]`|get domain ip from dns server|

## Process management

|Command / Location|Description|
|------------------|-----------|
|`ps`|show current user processes|
|`top`|show processes|
|`htop`|show processes with gui|
|`kill -15 pid`|kill process|
|`kill -9 pid`|kill process without mercy|

## Services management

|Command / Location|Description|
|------------------|-----------|
|`service service_name start/stop/restart/reload`|start/stop/restart/reload service|
|`systemctl status service_name.service`|service status info|
|`systemctl enable service_name.service`|enable service to run|
|`systemctl daemon-reload`|reaload services configs|
|`/etc/systemd/**/*.service`|services configuration files|
|`journalctl --since yesterday -u service_name.service`|view service logs|

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

## Files

|Command / Location|Description|
|------------------|-----------|
|`cat file_name`, `more file_name`, `less file_name`|read file contents|
|`nano file_name`, `vim file_name`, `echo 'text' > file_name`|write file contents|
|`ps > file_name`|redirect output to file (`write`)|
|`ps >> file_name`|redirect output to file (`append`)|
|`less +F file_name`|read file from end|

## Directories

|Command / Location|Description|
|------------------|-----------|
|`ls -alF`|list directory content|
|`ln -s from to`|creates symbolic link to location|
|`mc`|midnight commander (file manager)|

---

[< Back to menu](../README.md)

---

<p align="center">
    <b><a href="https://github.com/MGSE97" target="_blank">MGSE97</a> 🔥 2021</b>
</div>