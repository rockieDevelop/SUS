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

