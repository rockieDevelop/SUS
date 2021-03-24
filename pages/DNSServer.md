[< Back to menu](../README.md)

---

# DNS Server

## 1. Install Packages

<table>
    <tr><td><code>apt install bind9 dnsutils udns-utils</code></td><td>install Bind9 DNS Server and utils</td></tr>
    <tr><td><code>nslookup domain [dns]</code></td><td>lookup domain ip on dns server</td></tr>
</table>

## 2. Configure DNS Server

<table>
    <tr><td><code>/etc/bind/</code></td><td>DNS Server configuration folder</td></tr>
    <tr><td><code>/etc/bind/zones.rfc1918</code></td><td>blocked ip ranges</td></tr>
    <tr><td><code>/etc/bind/named.conf[.*]</code></td><td>zones configurations</td></tr>
    <tr><td><code>/etc/bind/db.*</code></td><td>zone configuration</td></tr>
</table>

### ve skole musi byt nastavena vychozi dns na skolni dns - soubor named.conf.options

    //zde nastavit skolni dns
    forwarders{
        158.196.0.53;
    };

### definice zonoveho souboru /etc/bind - nazev souboru by mel byt ve tvaru db.example.cz
    
    $TTL 3h 			   ;doba expirace všech záznamů
    @       IN      SOA     ns1.example.cz. spravce.example.cz. (  ; autoritativní DNS server + email správce bez @
                         2018040302 ; seriové číslo, často ve formě data
                         4h         ; jak často si stahuje data sekundární server
                         2h         ; za jak dlouho se má sek.server pokusit stáhnout data při neúspěchu
                         2w         ; kdy platnost dat v sek.serveru vyprší
                        1h )       ; jak dlouho si mají data pamatovat cache servery
    ;
    @       IN      NS      ns1.example.cz. ; autoritativní servery pro doménu, musí mít i A záznamy
    @       IN      NS      ns2.example.cz. ; autoritativní servery pro doménu, musí mít i A záznamy


    example.cz.    IN      MX      10      smtp.example.cz.  ; primární emailový server
    example.cz.    IN      MX      20      smtp2.example.cz. ; sekundární emailový server
    example.cz.    IN      A       192.168.56.101          ; primární záznamy
    ns1            IN      A       192.168.56.101
    ns2            IN      A       192.168.56.102
    smtp           IN      A       192.168.56.101
    smtp2          IN      A       192.168.56.102
    www            IN      CNAME   example.cz.
    wiki	        IN      CNAME   example.cz.
    test	        IN      CNAME   example.cz.
    subdomena1     IN      A       192.168.56.101
    subdomena2     IN      CNAME   example.cz.
    www1	        IN      A       192.168.56.101
    www2           IN      A       192.168.56.101

    _http._tcp     IN      SRV     1 2 80  www1 ; _http sluzba, _tcp protokol, 1 priorita, 2 váha
                   IN      SRV     5 3 80  www2

    _http._tcp.www IN      SRV     1 2 80  www1.example.cz. ; _http sluzba, _tcp protokol, 1 priorita, 2 váha
                   IN      SRV     5 3 80  www2.example.cz.

    *._tcp         IN      SRV     0 0 0   .  ;ostatní služby nejsou podporovány

### odkaz na zonovy soubor v named.conf.local
    zone "example.cz" {
           type master;    ;jedná se o primární server pro danou doménu
           file "/etc/bind/db.example.cz";
    };
 
### overeni funkcnosti dns
je treba restartovat service bind9

    service bind9 restart
    service bind9 status

    nslookup example.cz 127.0.0.1
<table>
<tr><td>server 192.168.56.101 </td><td>nastavi ktery serevr pouziju</td></tr>
<tr><td>set type=MX </td><td> specifikuju vypsani MX zaznamu</td></tr>
</table>

je mozno pouzit i dig

    dig example.cz @127.0.0.1
    dig _http._tcp.example.cz SRV @127.0.0.1

## Sekundarni server
### pridani access listu a povoleni transferu na hlavnim serveru (uprava souboru, ktery uz jsme jednou upravovali) - named.conf.local

    acl "example.cz" {
        192.168.56.101;
    };

    zone "example.cz" {
           type master;
           file "/etc/bind/db.example.cz";
           allow-transfer { "example.cz"; };
    };

### na sekundarnim serveru - named.conf.local
    masters example.cz-master { 192.168.56.101; };

    zone example.cz {
        type slave;
        file "/var/cache/bind/db.example.cz";
        masters { example.cz-master; };
    };

## Reverzni zaznamy
vytvorit soubor /etc/bind/db.192.168.56

    $TTL    86400
    56.168.192.in-addr.arpa.       IN      SOA     ns1.example.cz admin.example.cz. (
                               20120820 ; serial
                               4h       ; slave refresh
                               2h       ; slave retry interval
                               2w       ; slave data expiration
                               1h )     ; maximum caching time when lookups fail
    ;
    56.168.192.in-addr.arpa.       IN      NS      ns1.example.cz.
    56.168.192.in-addr.arpa.       IN      NS      ns2.example.cz. 

    10.56.168.192.in-addr.arpa.    IN      PTR     webserver.example.org.
    1.56.168.192.in-addr.arpa.     IN      PTR     router.example.org.

v named.conf.local uvezt odkaz na soubor

    zone "56.168.192.in-addr.arpa" {
           type master;
           file "/etc/bind/db.192.168.56";
    };

---

[< Back to menu](../README.md)

---

<p align="center">
    <b><a href="https://github.com/MGSE97" target="_blank">MGSE97</a> 🔥 2021</b>
</div>
<p align="center">
    <b>updated by <a href="https://github.com/rockieDevelop" target="_blank">rockieDevelop</a> 2021</b>
</div>
