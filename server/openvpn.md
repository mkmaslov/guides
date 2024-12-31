# Set up OpenVPN server on OpenWRT router

This guide provides instructions on configuring an OpenVPN server on a router with OpenWRT firmware.

### Contents
- [Step 1: Update firmware and software on the router](#step-1-update-firmware-and-software-on-the-router)
- [Step 2: Set up a dynamic DNS service](#step-2-set-up-a-dynamic-dns-service)
- [Step 3: Establish a public key infrastructure](#step-3-establish-a-public-key-infrastructure)

## Step 1: Update firmware and software on the router

1. [Update the OpenWRT firmware.](https://openwrt.org/docs/guide-user/installation/generic.sysupgrade) 
   
   **WARNING:** by default, this removes all installed packages and some of the configuration files. Make sure to create a backup or enable a persistent configuration before upgrading.
2. Update the package list: 
   ```
   opkg update
   ```
   View installed packages using:
   ```
   opkg list-installed
   ```
   View packages that are outdated using:
   ```
   opkg list-upgradable
   ```
3. Upgrade all installed packages:
   ```
   opkg list-upgradable | cut -f 1 -d ' ' | xargs opkg upgrade
   ```
   Router may reboot during the process.
4. Install OpenVPN packages:
   ```
   opkg install openvpn-openssl openvpn-easy-rsa luci-app-openvpn
   ```
   and dynamic DNS packages:
   ```
   opkg install ddns-scripts luci-app-ddns
   ```
   For some DDNS services, like [No-IP](https://noip.com), you may need to install helper scripts:
   ```
   opkg install ddns-scripts-noip
   ```
5. *(optional)* For SSL support in DDNS, install either:
   ```
   opkg install wget-ssl ca-certificates
   ```
   or, if you prefer certificates bundled in a single file:
   ```
   opkg install curl ca-bundle
   ```


## Step 2: Set up a dynamic DNS service

In most cases, the IP address you are receiving from your ISP is dynamic. This means it can change from time to time. To create a persistent address of your network on the Internet, you can use a [dynamic DNS service](https://openwrt.org/docs/guide-user/services/ddns/client). It periodically communicates the IP address of your network to DNS servers on the Internet. As a result, you can always locate your network via a domain name (FQDN).

In this guide, [Duck DNS](https://duckdns.org) provider is used to enable dynamic DNS. The configuration for other DDNS providers might differ. Usually, DDNS providers would have the configuration guidelines on their website.

1. Register an account with a DDNS service.
2. Open `LuCI -> Services -> Dynamic DNS`.
3. Remove all template services, if present. Press `Add new services...` and create a new DDNS service. LuCI may throw an error that the chosen service name already exists, ignore it.
4. Configure as follows:
   ```
   Enabled: <Check>
   Lookup Hostname: <hostname>.duckdns.org
   IP address version: IPv4-Address
   DDNS Service provider: duckdns.org
   Domain: <hostname>.duckdns.org
   Username: <hostname>
   Password: <Duck DNS token>
   Use HTTP Secure: <Check>
   Path to CA-Certificate: /etc/ssl/certs/ca-bundle.pem
   ```
5. To enable SSL support (HTTPS), Duck DNS requires downloading the following certificate:
   ```
   mkdir -p /etc/ssl/certs
   curl -k https://certs.secureserver.net/repository/sf_bundle-g2.crt >  /etc/ssl/certs/ca-bundle.pem
   ```
   For other DDNS providers, like No-IP, if `ca-certificates` are installed, the **Path to CA-Certificate** should be `/etc/ssl/certs`. If `ca-bundle` is installed, the path should be `/etc/ssl/certs/ca-certificates.crt`.

## Step 3: Establish a public key infrastructure

## Step 4: Create an Open VPN server configuration
