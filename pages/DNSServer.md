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

---

[< Back to menu](../README.md)

---

<p align="center">
    <b><a href="https://github.com/MGSE97" target="_blank">MGSE97</a> 🔥 2021</b>
</div>