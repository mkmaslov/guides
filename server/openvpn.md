# Set up OpenVPN server on OpenWRT router

## Step 1: Update firmware and software of the router.

1. Update OpenWRT firmware. **WARNING: by default, this removes all installed packages and some configuration files.**
2. Update the package list: `opkg update`.
3. Installed and upgradable packages can be viewed using, respectively: `opkg list-installed` and `opkg list-upgradable`.
4. Upgrade all installed packages:<br>
   `opkg list-upgradable | cut -f 1 -d ' ' | xargs opkg upgrade`<br>
   Router may reboot during the process.
5. Install packages for OpenVPN:<br>
   `opkg install openvpn-openssl luci-app-openvpn`<br>
   and DDNS:<br>
   `opkg install ddns-scripts luci-app-ddns`<br>
   Depending on a DDNS service you may need to install helper scripts:<br>
   `opkg install ddns-scripts-noip`
6. (optional) For SSL support in DDNS, install either:
    `opkg install wget-ssl ca-certificates`<br>
    or:
    `opkg install curl ca-bundle`<br> 
    (in the latter case the certificates are bundled in one file)


## Step #: Dynamic DNS

5. Enable SSL support (HTTPS). If `ca-certificates` are installed, the **Path to CA-Certificate** should be `/etc/ssl/certs`. If `ca-bundle` is installed, this path should be `/etc/ssl/certs/ca-certificates.crt`.
