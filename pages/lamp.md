# LAMP - Linux, Apache, MariaDB/MySQL, PHP

### nastavime ip nasi dns /etc/resolv.conf
    nameserver 192.168.56.101

### upravime interfacy /etc/network/interfaces
    allow-hotplug enp0s3
    iface enp0s3 inet dhcp
    
    allow-hotplug enp0s8
    iface enp0s8 inet static
            address 192.168.56.104
            netmask 255.255.255.0

# Apache
## instalace apache
    apt update
    apt install apache2
    apt install libapache2-mod-php
    service apache2 restart

stranky se nachazeji v /var/www/html


## konfigurace apache
se nachazi v /etc/apache2
<table>
  <tr><td>apache2.conf</td><td>nejdulezitejsi soubor, jsou v nem odkazy na dalsi kousky konfiguraci</td></tr>
  <tr><td>envvars</td><td>promenne, ktere pouziva apache</td></tr>
  <tr><td>ports.conf</td><td>na jakych portech apache posloucha</td></tr>
  <tr><td>slozky -available</td><td>vsechny dostupne</td></tr>
  <tr><td>slozky -enabled</td><td>vsechny povolene (symblicke linky na polozky v -available)</td></tr>
</table>

## povoleni modulu/stranky/konfigu

### vysvetleni prikazu
a2enmod

a2 - nazev utilitky

en - enable / dis - disable

mod - modul / site - stranka / conf - konfigurace

    a2enmod userdir
    a2dismod userdir //takhle by to vypadalo pro zruseni
    
pro aktivaci nove konfigurace je treba restartovat apache

    systemctl restart apache2

# Virtualni web
muzu se ptat 1 IP adresy a ta mi bude podle GET pozadavku vracet ruzne stranky

## uprava DNS
na dns serveru v zonovem souboru pridam A zaznam na Lamp

    lamp    IN     A     192.168.56.104

a CNAME zaznamy jako www,wiki,mail necham odkazovat na lamp

    www      IN     CNAME    lamp.saf0068.org.
    wiki     IN     CNAME    lamp.saf0068.org.
    mail     IN     CNAME    lamp.saf0068.org.

vsechny tyto domeny pote budou odkazovat na lamp a vsechny je bude obhospodarovat 1 webovy server

musime na webovem serveru vytvorit tyto virtualni domeny

nezapomenout zmenit seriove cislo!

pro provedeni zmen, opet restartovat service bind9

    service bind9 restart
    
## lamp vytvareni domen
je zvykem vytvorit slozku se stejnym jmenem jako je domena v /var/www

    cd /var/www/
    mkdir www.saf0068.org
    
ve slozce vytvorime jednoduchy `index.html` soubor

    echo AHOJ > index.html

### vytvoreni konfiguracnich souboru pro virtualni domeny

    cd /etc/apache2/sites-available/

jednoduse zkopiruju konfiguracni soubor ktery uz existuje
    
    cp 000-default.conf www.saf0068.org.conf

zmeny v konfiguracnim souboru:
    
    ServerName www.saf0068.org
    
    ServerAdmin webmaster@saf0068.org //email na spravce webu
    DocumentRoot /var/www/www.saf0068.org //misto kde se nachazi web

ted chceme zakazat defaultni stranky a povolit ty nase vytvorene

    a2dissite 000-default
    a2ensite www.saf0068.org


# MariaDB
## instalace
    apt install default-mysql-server
    
## vytvoreni uzivatele
    mysql -u root -p
    MariaDB [(none)]> use mysql
    MariaDB [mysql]> CREATE USER 'wiki'@'localhost' IDENTIFIED BY 'wiki';
    MariaDB [mysql]> \q
    
## vytvoreni databaze
    # mysql -u root
    MariaDB [(none)]> CREATE DATABASE wiki;
    MariaDB [(none)]> use wiki
    MariaDB [wiki]> GRANT ALL ON wiki.* TO wiki@localhost;
    Query OK, 0 rows affected (0.01 sec)
    MariaDB [wiki]>\q
    
## zprovozneni MediaWiki
stahnu MediaWiki

    wget https://releases.wikimedia.org/mediawiki/1.35/mediawiki-1.35.1.zip
    unzip mediawiki.zip

nakopiruju do /var/www

smazu wiki.saf0068.org

vytvorim symbolicky link na MediaWiki pojmenovany wiki.saf0068.org

    ln -s mediawiki-1.35.1 wiki.saf0068.org
    
MediaWiki potrebuje doinstalovat balicky mbstring, xml a k php jeste pridat podporu mysql

    apt install php-mbstring
    apt install php-xml
    apt install php-mysql
    
    service apache2 restart
    
Po nastaveni MediaWiki stahnout LocalSettings.php do /var/www/wiki.saf0068.org

    scp LocalSettings.php saf0068@www.saf0068.org

Soubor se stahne na Lamp server do home slozky saf0068
    


<p align="center">
    <b><a href="https://github.com/rockieDevelop" target="_blank">rockieDevelop</a> 2021</b>
</div>
