[< Back to menu](../README.md)

---

# Building kernel

## 1. Get kernel from web [kernel.org](https://kernel.org)

<table>
    <tr><td><code>wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.11.tar.xz</code></td></tr>
</table>

## 2. Unpack kernel

<table>
    <tr><td><code>xz -d linux-5.11.tar.xz</code></td><td>decompress archive</td></tr>
    <tr><td><code>tar -xf linux-5.11.tar</code></td><td>split file to directory</td></tr>
    <tr><td><code>ln -s /usr/src/linux-5.11 linux</code></td><td>make symbolic link for build</td></tr>
    <tr><td><code>cd /usr/src/linux</code></td><td>go to build dir</td></tr>
</table>

## 3. Create configuration

<table>
    <tr><td><code>make localmodconfig</code></td><td>use current config</td></tr>
    <tr><td><code>make menuconfig</code></td><td>show menu to configure (don't forget to save)</td></tr>
</table>

## 4. Build kernel

<table>
    <tr><td><code>make -j 36</code></td><td>build using 36 parallel tasks (+- 2 * Logical processors)</td></tr>
    <tr><td><code>make -j 36 modules</code></td><td>build drivers & addons</td></tr>
</table>

## 5. Install kernel

<table>
    <tr><td><code>make -j 36 modules_install</code></td><td>/lib/modules/</td></tr>
    <tr><td><code>make -j 36 install</code></td><td>/boot/</td></tr>
    <tr><td><code>reboot</code></td><td>finaly reboot system and boot to your new kernel</td></tr>
    <tr><td><code>cat /proc/version</code></td><td>check current kernel version</td></tr>
</table>

---

[< Back to menu](../README.md)

---

<p align="center">
    <b><a href="https://github.com/MGSE97" target="_blank">MGSE97</a> 🔥 2021</b>
</div>