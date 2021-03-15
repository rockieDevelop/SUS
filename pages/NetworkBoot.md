[< Back to menu](../README.md)

---

# Network boot

## Boot server

Will be used for DHCP and for storing/distributing boot image to clients.
![Network schema](images/network-boot-schema.png)

## 1. Configure static IP Address

<table>
    <tr><td><code>nano /etc/network/interfaces</code></td><td>edit network interfaces file</td></tr>
</table>

Append folowing lines to [file](files/network/interfaces_static). Change to fit your system and network.

This configures `enp0s8` interface to:

- use static IP address `192.168.26.2`
- with `netmask 255.255.255.0`
- and `broadcast 192.168.26.255`.

```
auto enp0s8
iface enp0s8 inet static
  address 192.168.26.2/24
```

<table>
    <tr><td><code>ifup enp0s8</code></td><td>activate interface</td></tr>
    <tr><td>
        <code>ifdown enp0s8</code> then <code>ifup enp0s8</code>
    </td><td>if interface is allready activated, deactivate interface and reactivate interface, to load changes</td></tr>
    <tr><td><code>ip a</code></td><td>check current status</td></tr>
</table>

## 2. Setting up DHCP Server

Configuration files [/etc/default/isc-dhcp-server](files/dhcp/isc-dhcp-server) and [/etc/dhcp/dhcpd.conf](files/dhcp/dhcpd.conf).

<table>
    <tr><td><code>apt install isc-dhcp-server</code></td><td>install dhcp server package</td></tr>
    <tr><td>---</td><td>---</td></tr>
    <tr><td><code>nano /etc/default/isc-dhcp-server</code></td><td>configure dhcp server</td></tr>
    <tr><td><code>INTERFACESv4="enp0s8"</code></td><td>add your interface to property</td></tr>
    <tr><td>---</td><td>---</td></tr>
    <tr><td><code>nano /etc/dhcp/dhcpd.conf</code></td><td>configure dhcp server</td></tr>
    <tr><td><code>option domain-name "example.com";</code></td><td>set domain</td></tr>
    <tr><td><code>option domain-name-servers 8.8.8.8, 1.1.1.1;</code></td><td>set name servers (DNS Servers)</td></tr>
    <tr><td><code>default-lease-time 600;</code></td><td>set time (seconds) between checks for ip by client</td></tr>
    <tr><td><code>subnet ...</code></td><td>set your subnet for dhcp</td></tr>
    <tr><td>---</td><td>---</td></tr>
    <tr><td><code>service isc-dhcp-server start</code></td><td>start dhcp service</td></tr>
    <tr><td><code>service isc-dhcp-server status</code></td><td>check dhcp service status</td></tr>
</table>

## 3. Setup NAT request translation

Allow forwarding requests from DHCP Clients to internet or other networks.

### Temporary configuration:
<table>
    <tr><td><code>iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE</code></td><td>setup forwarding on interface <b>enp0s3</b> in table <b>nat</b></td></tr>
    <tr><td><code>echo 1 > /proc/sys/net/ipv4/ip_forward</code></td><td>enable ip forwarding</td></tr>
    <tr><td><code>iptables -t nat --list</code></td><td>list all rules in <b>nat</b> table</td></tr>
</table>

### Persistent configuration:
<table>
    <tr><td><code>nano /etc/sysctl.conf</code> then uncomment <code>net.ipv4.ip_forward</code></td><td>enable ip forwarding</td></tr>
    <tr><td><code>apt install iptables-persistent</code></td><td>utility to manage iptables, copies config on install</td></tr>
    <tr><td><code>nano /etc/iptables/rules.v4</code> should contain configuration <code>-A POSTROUTING -o enp0s3 -j MASQUERADE</code></td><td>these rules are applied on reboot</td></tr>
    <tr><td><code>iptables -t nat --list</code></td><td>list all rules in <b>nat</b> table</td></tr>
</table>

## 4. Test DHCP Client

Ensure there is only one DHCP Server on network.

Connect with your client to network and check with `ip a`.

Use `dhclient -r` on client to renew IP Address;

Test outgoing connections to different networks using `ping`.

Use `dhcp-lease-list` to list all dhcp clients on server.

## 5. Install TFTP Server

Trivial file transfer protocol server. Used to transfer files to clients.

<table>
    <tr><td><code>apt install tftpd-hpa</code></td><td>install TFTP server package</td></tr>
    <tr><td><code>nano /etc/default/tftpd-hpa</code></td><td>configure TFTP server (optional)</td></tr>
    <tr><td><code>echo 'Test' > /srv/tftp/test</code></td><td>create test file</td></tr>
</table>

## 6. Test TFTP Client

<table>
    <tr><td><code>apt install tftp-hpa</code></td><td>install TFTP client package</td></tr>
    <tr><td><code>tftp 192.168.26.2</code></td><td>connect to TFTP server</td></tr>
    <tr><td><code>binary</code></td><td>switch to binary mode</td></tr>
    <tr><td><code>get test</code></td><td>download test file</td></tr>
    <tr><td><code>quit</code></td><td>close connection</td></tr>
    <tr><td><code>cat test</code></td><td>read downloaded file</td></tr>
</table>

## 7. Configure network boot on server

Clients will download `pxelinux.0` from server `192.168.26.2` using TFTP.

<table>
    <tr><td><code>nano /etc/dhcp/dhcpd.conf</code></td><td>configure DHCP to use network boot</td></tr>
    <tr><td>Add <code>next-server 192.168.26.2;</code> and <code>filename "pxelinux.0";</code> to subnet</td><td>add network server and image filename</td></tr>
    <tr><td><code>dhcpd -t</code></td><td>check DHCP configuration</td></tr>
    <tr><td><code>service isc-dhcp-server restart</code></td><td>restart DHCP service</td></tr>
    <tr><td><code>cd /srv/tftp</code></td><td>move to TFTP directory</td></tr>
    <tr><td><code>wget http://ftp.debian.org/debian/dists/stretch/main/installer-amd64/current/images/netboot/netboot.tar.gz</code></td><td>download netboot</td></tr>
    <tr><td><code>gzip -d netboot.tar.gz</code></td><td>decompress netboot</td></tr>
    <tr><td><code>tar -xf netboot.tar</code></td><td>split netboot to files</td></tr>
</table>

## 8. Test network boot on client

Connect client to network, and choose to boot from network.

Client will download `pxelinux.0` from server `192.168.26.2` using TFTP and show menu.

## 9. Setup NFS server

Map directories to clients.

<table>
    <tr><td><code>apt install nfs-kernel-server</code></td><td>install NFS server</td></tr>
    <tr><td><code>mkdir /remote_root</code></td><td>create remote boot directory</td></tr>
    <tr><td><code>nano /etc/exports</code></td><td>configure NFS server</td></tr>
    <tr><td><code>/remote_root&nbsp;192.168.26.0/24(rw,async,no_root_squash)</code></td><td>boot from <b>/remote_root</b> directory for subnet <b>192.168.26.0/24</b></td></tr>
    <tr><td><code>/home&nbsp;192.168.57.0/24(rw,async)</code></td><td>map home directory for subnet <b>192.168.26.0/24</b></td></tr>
    <tr><td><code>service nfs-kernel-server restart</code></td><td>restart NFS server</td></tr>
    <tr><td><code>echo 'Test' > /remote_root/test</code></td><td>create test file</td></tr>
</table>

## 10. Test mount on client

<table>
    <tr><td><code>apt install nfs-client</code></td><td>install NFS client</td></tr>
    <tr><td><code>mount 192.168.26.2:/remote_root /mnt/</code></td><td>mount NFS server to /mnt</td></tr>
    <tr><td><code>cat /mnt/test</code></td><td>read test file</td></tr>
</table>

## 11. Create our network boot system

Map parts our system for client to boot from.

Copy directories to `/remote_boot`. Create others.

```
bin -> cp
boot -> cp
dev
etc -> cp
home
lib -> cp
lib64 -> cp
media
mnt
opt -> cp
proc
root -> cp
run
sbin -> cp
srv -> cp
sys
tmp
usr -> cp
var -> cp
```

<table>
    <tr><td><code>chmod 777 tmp</code></td><td>fix tmp permissions</td></tr>
    <tr><td><code>chmod o+t tmp</code></td><td>fix tmp permissions</td></tr>
</table>

Prepare pxelinux stucture from debian-installer:
```
Debian
ldlinux.c32
libcom32.c32
libutil.c32
pxelinux.cfg
pxelinux.0
vesamenu.c32
stdmenu.cfg
splash.png
```

Setup pxelinux.cfg/default
```
DEFAULT vesamenu.c32
INCLUDE stdmenu.cfg
PROMPT 0
TIMEOUT 50

MENU TITLE  Boot Menu

LABEL Debian - NetBoot
KERNEL /Debian/vmlinuz-4.19.0-14-amd64
APPEND initrd=/Debian/initrd.img-4.19.0-14-amd64 root=/dev/nfs nfsroot=192.168.26.2:/remote_root,udp ip=dhcp rw
```

## 12. Test boot from client

Try booting client, should load our kernel.

## 13. Fix errors

Distable all interfaces in `/remote_root/etc/network/interfaces` except of `lo`.

Remove disks in `/remote_root/etc/fstab`.

## 14. Mounting additional drives

Append `/home   192.168.57.0/24(rw,async)` to `/etc/exports` on server.

Restart NFS service using `service nfs-kernel-server restart`.

Append mount `192.168.26.2:/home    /home   nfs udp,intr` to client system `/remote_root/etc/fstab`.

Check on client.

---

[< Back to menu](../README.md)

---

<p align="center">
    <b><a href="https://github.com/MGSE97" target="_blank">MGSE97</a> 🔥 2021</b>
</div>