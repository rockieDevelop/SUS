# Postovni server

opet nastavit nasi dns v /ets/resolv.conf

a stejne take nastavit ip serveru v /etc/network/interfaces na .105

## nastaveni ip adres pro SMTP

v nasi DNS nastavime ip adresy smtp na nas postovni server

    smtp    IN    A   192.168.56.105
    smtp2   IN    A   192.168.56.105

## instalace postovniho serveru Postfix 
    apt install postfix

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


<p align="center">
    <b><a href="https://github.com/rockieDevelop" target="_blank">rockieDevelop</a> 2021</b>
</div>
