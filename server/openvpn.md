# Set up OpenVPN server on OpenWRT router

This guide provides instructions on configuring an OpenVPN server on a router with OpenWRT firmware.

**Pre-requisites:**
- a router with OpenWRT firmware installed
- LuCI and UCI user interfaces enabled (should be the default)
- SSH connection to the router set up

## Step 1: Update firmware and software on the router

1. [Update the OpenWRT firmware.](https://openwrt.org/docs/guide-user/installation/generic.sysupgrade) **WARNING:** by default, this removes all installed packages and some of the configuration files. Make sure to create a backup or enable a persistent configuration before upgrading.
2. Update the package list: 
   ```
   opkg update
   ```
3. View installed packages using:
   ```
   opkg list-installed
   ```
   View packages that are outdated using:
   ```
   opkg list-upgradable
   ```
4. Upgrade all installed packages:
   ```
   opkg list-upgradable | cut -f 1 -d ' ' | xargs opkg upgrade
   ```
   The router may reboot during the process.
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


## Step 2: Set up a dynamic DNS service

In most cases, the IP address you are receiving from your ISP is dynamic. This means it can change from time to time. To create a persistent address of your network on the Internet, you can use a dynamic DNS service. It periodically communicates the IP address of your network to DNS servers on the Internet. As a result, you can always locate your network via a domain name (FQDN).

In this guide, [Duck Dns](https://duckdns.org) service is used to enable DDNS. The configuration for other DDNS services might differ. You can usually find the instructions

1. Register an account with a DDNS service.
2. Open `LuCI -> Services -> Dynamic DNS`.



5. Enable SSL support (HTTPS). If `ca-certificates` are installed, the **Path to CA-Certificate** should be `/etc/ssl/certs`. If `ca-bundle` is installed, this path should be `/etc/ssl/certs/ca-certificates.crt`.
