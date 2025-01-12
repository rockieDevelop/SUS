# Priprava virtualu

VBOXNet1 (druha sit) - sit bez DHCP

## Boot server

nastavit 2 sitovky - 1. klasicky NAT, 2. sit pouze s hostem nastavit na tu nasi sit bez DHCP (VBOXNet1 - v mem pripade sit 192.168.64.0/24)

# Konfigurace Boot serveru

## Konfigurace rozhrani enp0s8 - /etc/network/interfaces

Nakonfigurovat rozhrani enp0s8 (aby dostal ip do startu)
/etc/network/interfaces

    allow-hotplug enp0s8
    iface enp0s8 inet static
      address 192.168.64.2/24
      
nahodit interface - ifup enp0s8

## DHCP server

    apt install isc-dhcp-server

### Konfigurace /etc/default/isc-dhcp-server (uvadim na kterem rozhrani ma nas DHCP server fungovat)

    INTERFACESv4="enp0s8"
    
### Konfigurace /etc/dhcp/dhcpd.conf

    option domain-name "mojedomena.cz";
    option domain-name-servers 8.8.8.8; # pokud jsem na skolni siti musim nastavit skolni DNS 158.196.0.53
    
    default-lease-time 600; # na jak dlouho ma PC zapujcenou IP adresu v sekundach
    
    #authoritative # aby fungoval IP HELPER (pokud ho pouzivam) - kdyz mam vic subnetu, tak abych nemusel v kazdem nastavovat interface pro DHCP
    
    subnet 192.168.64.0 netmask 255.255.255.0 {
      range 192.168.64.100 192.168.64.200;      #range, ve kterem bude prirazovat
      option broadcast-address 192.168.64.255;  #broadcast
      option routers 192.168.64.2;              #router - adresa naseho serveru
    }

spusteni serveru

    service isc-dhcp-server start
    
!overit zda DHCP server funguje pomoci Testovaciho PC!
    
## Testovaci PC

Zmenit nastaveni sitovych karet - nechat jen 1 a tu pripojit na sit pouze s hostem - na tu nasi sit bez DHCP (VBOXNet1)
Po zapnuti by mel mit prirazenou adresu od naseho DHCP serveru (ip add: enp0s3 bude mit adresu z naseho rozsahu)

## NAT - Preklad sitovych adres
Pouzivam IPTABLES(linuxovsky firewall), tabulku NAT, Postrouting - paket projde routovacima tabulkama a kdyz vychazi rozhranim enp0s3 (nas interface do site NAT VirtualBoxu) tak proved MASQUERADE (pouzije prvni adresu daneho rozhrani)
    
    iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

Povoleni preposilani paketu mezi rozhranimi

    echo 1 > /proc/sys/net/ipv4/ip_forward

Po tomto kroku by melo Testovaci PC byt schopno pingat i adresy mimo sit, jako treba skolni DNS 158.196.0.53

### Nakonfigurovani predchozich kroku hned po startu

Povoleni preposilani paketu mezi rozhranimi hend po startu /etc/sysctl.conf

    net.ipv4.ip_forward=1

Spusteni po startu IPTABLES

    apt install iptables-persistent

do /etc/iptables/rules.v4 nam zapsal pravidla pro ipv4

## TFTP server

    apt install tftpd-hpa
    
Konfigurace v souboru /etc/default/tftpd-ha - Neni potreba nic menit, ale je potreba vedet (nebo si vypsat tu konfiguraci), ze soubory se ukladaji do /srv/tftp

### Muzeme otestovat:

v /srv/tftp vytvorime nejaky soubor (echo Neco > neco)

Na testovacim PC:
    
    apt install tftp-hpa
    
    tftp 192.168.64.2
    binary
    get neco
    quit
    
Mel by se nam na Testovaci PC stahnout soubor neco

## Bootovani ze site

Potreba upravit konfiguraci DHCP serveru /etc/dhcp/dhcpd.conf

    subnet 192.168.64.0 netmask 255.255.255.0 {
      range 192.168.64.100 192.168.64.200;      #range, ve kterem bude prirazovat
      option broadcast-address 192.168.64.255;  #broadcast
      option routers 192.168.64.2;              #router - adresa naseho serveru
      next-server 192.168.64.2;                 #klientovi, ktery bootuje, rekne klientovi na jake adrese si ma pozadat o soubor, ze ktereho bude bootovat
      filename "pxelinux.0";                    #nazev bootovaciho souboru
    }

Po kazde zmene v DHCP serveru je dobre zkontrolovat zda je konfigurace spravna

    dhcpd -t

Restart DHCP

    service isc-dhcp-server restart
    
### pxelinux
Stahnnu a rozbalim pxelinux

    apt install wget
    wget http://ftp.debian.org/debian/dists/stretch/main/installer-amd64/current/images/netboot/netboot.tar.gz
    gzip -d netboot.tar.gz
    tar -xf netboot.tar
    
## PC bez disku - PC pro netboot

Vytvorim nove PC bez virtualniho disku

Nastaveni site - 1 sitova karta s VBOXNet1 (sit pouze s hostem, bez DHCP)
Nastaveni systemu - poradi bootovani dat jenom sit

Pokud nebude fungovat - vyzkouset restartovani PC, restartovat servicu isc-dhcp-server (dhcpd -t)

## NFS
Chci namapovat root file system na system sdileni souboru po siti (NFS)

    apt install nfs-kernel-server

Konfigurace v /etc/exports

    /remote_root     192.168.64.0/24(rw,async,no_root_squash) # korenovy adresar toho pocitace ktery bootuje ze site - pro pocitace s IP adresou 192.168.64
    
Spustit servicu (a vytvorit korenovy adresar)
    
    mkdir /remote_root
    service nfs-kernel-server restart
    
### Testovani funkcnosti:

V /remote_root vytvorim soubor nnn (echo test_nfs > nnn)

Na Testovacim PC:

    apt install nfs-common
    mount 192.168.64.2:/remote_root /mnt/
    
Ve slozce /mnt by ted mel byt nas soubor nnn (mame tam namapovany /remote_root, takze by to melo fungovat interaktivne)

## Kopirovani Root File Systemu

Kopirujeme z / do /remote_boot (cp - zkopirovat, nic - staci vytvorit novou slozku)

    bin -> cp
    boot -> cp
    dev
    etc -> cp
    home
    lib -> cp
    lib32 -> cp
    lib64 -> cp
    libx32 -> cp
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
    
v /remote_root zmenit prava slozce tmp

    chmod 777 tmp/
    chmod o+t tmp/

### Konfigurace boot menu

    cd /srv/tftp/

Vytvorim si backup slozku do ktere presunu vsechno v te slozce

Ted budeme zpatky kopirovat z /srv/tftp/bac do /srv/tftp

    /srv/tftp/bac/debian-installer/amd64/pxelinux.cfg     - z tehle zkopirovane slozky nasledne smazu soubor default
    /srv/tftp/bac/debian-installer/amd64/pxelinux.0
    
    /srv/tftp/bac/debian-installer/amd64/boot-screens/ldlinux.c32
    /srv/tftp/bac/debian-installer/amd64/boot-screens/libcom32.c32
    /srv/tftp/bac/debian-installer/amd64/boot-screens/libutil.c32
    /srv/tftp/bac/debian-installer/amd64/boot-screens/vesamenu.c32
    
Zkopiruju jeste 
    
    /srv/tftp/bac/debian-installer/amd64/boot-screens/syslinux.cfg     do     /srv/tftp/pxelinux.cfg

A prejmenuju na default

    mv syslinux.cfg default
    
Soubor default ted prepisu na (cely obsah smazat a nahodit tohle):
    
    DEFAULT vesamenu.c32
    PROMPT 0

    MENU TITLE  Boot Menu

    LABEL Debian - NetBoot
    KERNEL /Debian/vmlinuz-4.19.0-14-amd64      #od kud si ma stahnout kernel (z nasi slozky /srv/tftp/Debian) - jaky kernel si ma vzit z TFTP serveru
    APPEND initrd=/Debian/initrd.img-4.19.0-14-amd64 root=/dev/nfs nfsroot=192.168.64.2:/remote_root ip=dhcp rw   #parametry ktere dame kernelu - initrd si veme ze serveru na dane ceste, root file system z NFS, namountuje root file system ze serveru z /remote_root, ip adresu ziska z DHCP serveru
    
Vytvorit slozku Debian v /srv/tftp a do tehle slozky prenest kernel

    /boot/vmlinuz-4.19.0-14-amd64     zkopiruju do    /srv/tftp/Debian
    /boot/initrd.img-4.19.0-14-amd64     zkopiruju do    /srv/tftp/Debian
    
Ted uz by mel vzdaleny pocitac bootovat nas kernel (ale jeste to uplne nepujde, ale bootovani by se melo spustit)

### Uprava Root File Systemu vzdaleneho pocitace

Soubor /remote_root/etc/network/interfaces zakomentovat:
    
    #allow-hotplug enp0s3 
    #iface enp0s3 inet dhcp 
    
    #allow-hotplug enp0s8  
    #iface enp0s8 inet static
    #  address 192.168.64.2/24
    
Soubor /remote_root/etc/fstab zakomentovat: (nebude si mountovat root file system z disku, takze zadne UUID)
    
    #UUID=316ad501-828f-438f-af3f-36a8a0165f34 /               ext4    errors=remount-ro 0       1
    #UUID=b95c8317-4bbc-4353-8d67-85243a0ee2a0 none            swap    sw              0       0
    
Jeste by se mely odstranit ze vzdaleneho PC konfigurace DHCP, TFTP, NFS ... (apt-get remove)

Vypis mountu NFS (abych videl Root File System)
    
    mount | grep nfs

### Uprava konfigurace komunikace z TCP na UDP protokol

Upravit soubor /srv/tftp/pxelinux.cfg/default

    APPEND initrd=/Debian/initrd.img-4.19.0-14-amd64 root=/dev/nfs nfsroot=192.168.64.2:/remote_root,udp ip=dhcp rw
    
### Mountovani dalsich slozek (treba home)

Pridat do /etc/exports
    
    /home 192.168.64.0/24(rw,async)

Restartovat NFS servicu
    
    service nfs-kernel-server restart

Pridat mount klientovi - do /remote_root/etc/fstab

    192.168.64.2:/home /home nfs udp,intr

Overit muzem opet pomoci vypsani mountu:

    mount | grep nfs

<p align="center">
    <b><a href="https://github.com/rockieDevelop" target="_blank">rockieDevelop</a> 2021</b>
</div>
