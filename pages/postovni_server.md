pridani uzivatele - adduser user_name

# Postovni server

opet nastavit nasi dns v /ets/resolv.conf

a stejne take nastavit ip serveru v /etc/network/interfaces na .105

# Postfix
## nastaveni ip adres pro SMTP

v nasi DNS nastavime ip adresy smtp na nas postovni server /etc/bind/db.saf0068.org

    smtp    IN    A   192.168.56.105
    smtp2   IN    A   192.168.56.105

## instalace postovniho serveru Postfix 
    apt install postfix
    
    Internetovy pocitac -> mojedomena.cz

Pro pouziti utilitky netstat

    apt install net-tools
    netstat -pln #pouziti netstat utilitky, muzeme pridat "| less" pro moznost scrollovani ve vypisu
    
## Konfigurace Postfix serveru /etc/postfix/main.cf

relay - odeslani emailu na jiny smtp sever

smtpd_relay_restrictions - omezeni serveru pro relay

myhostname - nazev smtp serveru

myorigin - mailove jmeno

mydestination - seznam domen, pro ktere dany server pracuje (diva se jestli cilova domena mailu neni jedna z tehle domen, protoze tohle jsou jeho domeny a muze tak dorucovat lokalne)

mynetworks - moje site (pokud nekdo posila email z techto siti, pak muze delat relay jinak ne)

relayhost - direktiva pro pripad ze tento postovni server funguje v ramci site, ve ktere je zakazano kontaktovat postovni servery na internetu.

V tomto pripade se sem napise adresa smtp serveru, pres ktery se maji posilat relayovane emaily (kdyby bezel v ramci skolni site, musel by mit zde nastaven smtp server skoly).

Pro odzkouseni postfix serveru pouzivam utilitku telnet

    apt install telnet
    telnet localhost 25             #z vypisu netstat vime ze to jede na portu 25
    helo saf                        #vetsinou je vyzadovano pozdarveni "helo" a nejaky hostname
    mail from: saf0068@saf0068.org  #od kud posilame mail
    rcpt to: saf0068@saf0068.org    #komu posilame mail
    data                            #directiva pro pocatek psani obsahu mailu
    test1                           #obsah mailu
    .                               #ukonceni obsahu mailu
    quit                            #opustit telnet

postfix se pokusil odeslat email a protoze byl poslan na saf0068@saf0068.org tak ho najdeme ve /var/spool/mail

stejne to jde odzkouset treba na nasem virtualu, kde pres terminal pouzijeme opet utilitku telnet na adresu smtp.saf0068.org port 25

### pridani site (/etc/postfix/main.cf)

proste pripiseme do mynetwork sit, napr.: 192.168.56.0/24

je treba obnovit postfix

    postfix reload

## Soubor /etc/postfix/master.cf

konfigurace porocilejsich veci / specifiace antivirove kontroly, podepisovani serveru, ...

## Postovni aliasy /etc/aliases

muzeme pridat skupinu nebo pouzit jako presmerovani

    studenti: saf0068@saf0068.org,saf01@saf0068.org
    rockie.develop: saf0068@saf0068.org
   
po uprave souboru je potreba zavolat
    
    newaliases

mozno odzkouset poslanim mailu vsem aliasum (studentum) opet pres telnet
    
    rcpt to: studenti@saf0068.org # nebo rcpt to: rockie.develop@saf0068.org

## Kanonicke mapovani

prepsani loginu v mailu na neco jineho, typicky jmeno a prijmeni

vytvorit soubor /etc/postfix/canonical

    saf0068 rockie.develop
    
je potreba zavolat

    postmap canonical
    
upravit /etc/postfix/main.cf

    canonical_maps = hash:/etc/postfix/canonical #hash - jedna se o databazi, to jsme vytvorili tim postmapem
nezapomenout postfix reload

## Typy dorucovani postfix

Existuji 2 zpusoby jakym postfix dorucuje lokalni postu: 
MailBox - vsechny emaily uzivatelu jsou v 1 souboru (popsano to co jsme delali do ted)

### MailDir

uprava souboru /etc/postfix/main.cf

    home_mailbox = Maildir/

maily se ted budou ukladat v domovskem adresari uzivatelu /home/saf0068/Maildir/new

# Server prichozi posty

    apt install dovecot-imapd
    
## Konfigurace /etc/dovecot/conf.d/{nazev_souboru}

uprava souboru /10-auth.conf
    
    disable_plaintext_auth = no #v realnem nasazeni nemyslitelne, ale pro zakladni implementaci si to muzeme dovolit
    
soubor /10-mail.conf

    mail_location = maildir:~/Maildir/  #Postfix uklada ve formatu MailDir do domovskeho adresare do slozky Maildir - tam hledej
    
potreba resetovat sluzbu dovecot

    service dovecot restart
    
pomoci utilitky netstat zjistime ze dovecot funguje na portech 993 a 143

# Postovni klient

## uprava DNS /etc/bind/db.saf0068.org

    imap    IN  CNAME   smtp.saf0068.org.

    _imap._tcp  IN  SRV 1   2   143 imap.saf0068.org. ; _http sluzba, _tcp protokol, 1 priorita, 2 vaha  #pokud pro domenu saf0068.org chces pouzit protokol IMAP, tak se zeptej serveru imap.saf0068.org na portu 143
    _smtp._tcp  IN  SRV 1   2   25  smtp.saf0068.org.    # pokud chces pouzit server odchozi posty SMTP, pak pouzij na portu 25 smtp.saf0068.org
    
muzeme overit treba pomoci thunderbird aplikace
    
## web-based klient
upravime LAMP server

vytvorim tabulky pro uzivatele

    mysql -u root -p
    use mysql
    CREATE USER 'roundcube'@'localhost' IDENTIFIED BY 'roundcube';
    \q
    
    mysql -u root
    CREATE DATABASE roundcube;
    use roundcube
    GRANT ALL ON roundcube.* TO roundcube@localhost;
    
Ted nainstaluju roundcube

    apt install roundcube
    
### vytvoreni virtualni domeny pro roundcube /etc/apache2/sites-available

    ServerName roundcube.saf0068.org
    DocumentRoot /var/www/roundcube.saf0068.org

povoleni domeny

    a2ensite roundcube.saf0068.org

ve /var/www vytvorit symbolicky link na misto kde je roundcube umisten

    ln -s /usr/share/roundcube/ roundcube.saf0068.org
    
v DNS konfiguraci potreba pripsat CNAME na roundcube
    
    roundcube   IN  CNAME   lamp.saf0068.org.
    
 Nezapomenout restartovat bind9 servicu
 
    service bind9 restart
    
### Konfigurace roundcubu /etc/roundcube/config.inc.php

    $config['default_host'] = 'imap.saf0068.org';
    $config['smtp_server'] = 'smtp.saf0068.org';

na roundcube domene by se mel zobrazit roundcube s jiz nadefinovanym serverem a melo by byt mozne se tam prihlasit

<p align="center">
    <b><a href="https://github.com/rockieDevelop" target="_blank">rockieDevelop</a> 2021</b>
</div>
