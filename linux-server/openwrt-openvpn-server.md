# Guide to OpenVPN server configuration on an OpenWRT router

This guide provides instructions on configuring an OpenVPN server on a router with OpenWRT firmware.

### Contents
- [Update router software](#update-router-software)
- [Get a public IP address](#get-a-public-ip-address)
- [Set up a Dynamic DNS service](#set-up-a-dynamic-dns-service)
- [Create network interface and set firewall rules](#create-network-interface-and-set-firewall-rules)
- [Set up an OpenVPN server](#set-up-an-openvpn-server)


## Update router software

1. Update the local package database: `opkg update`.
2. Upgrade all installed packages using: 
    ```
    opkg list-upgradable | cut -f 1 -d ' ' | xargs opkg upgrade
    ```
    **WARNING: router/LuCI may reboot during the process.** 
    
    To inspect packages:
   - Display installed packages: `opkg list-installed`
   - Display upgradable packages: `opkg list-upgradable`


## Get a public IP address

In order for clients to be able to connect to the server via its IP address, the IP address of the server needs to be a **public** IP address. Due to continuous depletion of IPv4 addresses, some Internet Service Providers (ISP) use Carrier-Grade NAT (CG-NAT) to assign **private** IP addresses to their clients. These are local addresses inside the ISP's NAT, usually in 10.0.0.0/8 or 100.64.0.0/10 address blocks. Private IPs are hard to access from outside the CG-NAT, they are mostly used for mobile devices that normally do not need to receive inbound connections. **Most ISPs switch users to a full public IP address upon reasonable request (via Support Service).** To verify that the router is receiving a private IP address (from a CG-NAT):

1. Detect the real public IP address of a router using `curl https://ifconfig.me/ip`. [Here](https://openwrt.org/docs/guide-user/services/ddns/client#detecting_wan_ip) is a list of alternative IP-address-detection websites.
2. Trace the traffic to the detected public IP using `traceroute *.*.*.*`. **For a public IP address,** there is going to be one (default) or two (if router and modem are separate devices) hops. **For a private IP address,** there is going to be two or three hops respectively.


## Set up a Dynamic DNS service

In most cases, the IP address you are receiving from your ISP is dynamic. This means it can change from time to time. To create a persistent address of your network on the Internet, you can use a [dynamic DNS service](https://openwrt.org/docs/guide-user/services/ddns/client). It periodically communicates the current IP address of your network to DNS servers on the Internet. As a result, you can always locate your network via a domain name (FQDN).

In this guide, [Duck DNS](https://duckdns.org) provider is used to enable dynamic DNS. The configuration for other DDNS providers should be similar, yet the particular configuration details (domain, hostname, password) might differ. Usually, DDNS providers would have the configuration guidelines on their website.

1. Register an account with a DDNS provider, like [duckdns.org](https://duckdns.org).
2. Install DDNS packages: 
    ```
    opkg install ddns-scripts luci-app-ddns
    ```
    Depending on the DDNS service, you may need to install provider-specific helper scripts:
    - for [NoIP](https://www.noip.com/): `opkg install ddns-scripts-noip`.
    - for [Cloudflare](https://www.cloudflare.com/learning/dns/glossary/dynamic-dns/): `opkg install ddns-scripts-cloudflare`.
3. To enable SSL (HTTPS) support, install system CA certificates one of the following ways:
    - All certificates in a single file (`/etc/ssl/certs/ca-certificates.crt`):
        ```
        opkg install curl ca-bundle
        ```
    - Every certificate in a separate file (stored in `/etc/ssl/certs`), as required by `wget-ssl`:
        ```
        opkg install wget-ssl ca-certificates
        ```
4. Open `LuCI → Services → Dynamic DNS`.
5. Remove all template services, if present. Press `Add new services...` and create a new DDNS service (IPv4). LuCI may throw an error that the chosen name for a new service *already exists*, ignore it.
6. Edit `/etc/config/ddns` to include:
    ```
    config ddns 'global'
        option ddns_dateformat '%F %R'
        option ddns_loglines '250'
        option ddns_rundir '/var/run/ddns'
        option ddns_logdir '/var/log/ddns'
        option use_curl '1'
        option cacert '/etc/ssl/certs'

    config service 'duckdns_OWRT_BerylAX'
        option service_name 'duckdns.org'
        option use_ipv6 '0'
        option enabled '1'
        option lookup_host '<HOSTNAME>.duckdns.org'
        option domain '<HOSTNAME>.duckdns.org'
        option username '<HOSTNAME>'
        option password '<DUCK-DNS-TOKEN>'
        option use_https '1'
        option cacert '/etc/ssl/certs'
        option ip_source 'network'
        option interface 'wan'
        option use_syslog '2'
        option check_interval '5'
        option check_unit 'minutes'
        option force_interval '5'
        option force_unit 'minutes'
        option retry_unit 'seconds'
    ```
    If WAN does not have a Public IP, e.g., when the router is in modem's LAN, change the `ip_source` option:
    ```
    option ip_source 'web'
    option ip_url 'https://ifconfig.me/ip'
    ```
    More websites for IP detection [here](https://openwrt.org/docs/guide-user/services/ddns/client#detecting_wan_ip).
7. Verify that DDNS works via the website of the DDNS provider.


## Create a network interface and set firewall rules

Create a network interface in `/etc/config/network`:
```
config interface 'vpn_net'
	option proto 'none'
	option device 'tun0'
```

Add firewall rules to `/etc/config/firewall`:
```
config zone
        option name 'vpn_net'
        option input 'ACCEPT'
        option output 'ACCEPT'
        option forward 'ACCEPT'
        option masq '1'
        option masq6 '1'
        option mtu_fix '1'
        list network 'vpn_net'

config forwarding
        option src 'vpn_net'
        option dest 'wan'

config redirect
        option dest 'vpn_net'
        option target 'DNAT'
        option name 'OpenVPN-Server-Net'
        option family 'ipv4'
        list proto 'tcp'
        option src 'wan'
        option src_dport '<SOURCE-PORT>'
        option dest_port '<DEST-PORT>'
```
In this case, the router will forward the traffic from `<SOURCE-PORT>` to `<DEST-PORT>`.


## Set up an OpenVPN server

Install OpenVPN packages:
```
opkg install openvpn-openssl openvpn-easy-rsa luci-app-openvpn
```
Clean up the `/etc/easy-rsa` directory:
```
rm -rf /etc/easy-rsa/pki
```
Do **not** remove the `/etc/easy-rsa/vars` file!

Initialize `Public Key Infrastructure (PKI)`:
```
easyrsa --vars=/etc/easy-rsa/vars --pki-dir=/etc/easy-rsa/pki --use-algo=rsa --digest=sha512 --keysize=2048 init-pki
```
[Look here for additional `init-pki` parameters](https://github.com/OpenVPN/easy-rsa/blob/master/doc/EasyRSA-Advanced.md#environmental-variables-reference).

Generate Diffie-Hellman (DH) parameters on **a client computer**, since generation on the router is very slow. **Note:** keys longer than `2048 bits` cause `OpenVPN` to be slow. Generate DH parameters:
```
openssl dhparam -out <TEMPORARY-DIR>/dh.pem 2048
cat <TEMPORARY-DIR>/dh.pem
```
Move key to `/etc/easy-rsa/pki/dh.pem` on the router:
```
nvim /etc/easy-rsa/pki/dh.pem
```
Build `Certificate Authority (CA)`:
```
easyrsa --vars=/etc/easy-rsa/vars --pki-dir=/etc/easy-rsa/pki --use-algo=rsa --digest=sha512 --keysize=2048 build-ca
```
This command will prompt for **two** passphrases: a `CA key passphrase` and a `CA PEM passphrase`.

Create a server certificate:
```
easyrsa --vars=/etc/easy-rsa/vars --pki-dir=/etc/easy-rsa/pki --use-algo=rsa --digest=sha512 --keysize=2048 build-server-full server
```
This command will prompt for **two** passphrases: a `Server PEM passphrase` and a `CA PEM passphrase` (see above).

Save `Server PEM passphrase` to `/etc/easy-rsa/pki/server-PEM-passphrase.txt`:
```
nvim /etc/easy-rsa/pki/server-PEM-passphrase.txt
```

Generate a `Server TLS key`:
```
openvpn --genkey tls-crypt-v2-server /etc/easy-rsa/pki/private/tls-crypt-v2-server.key
```

Create a client certificate:
```
easyrsa --vars=/etc/easy-rsa/vars --pki-dir=/etc/easy-rsa/pki --use-algo=rsa --digest=sha512 --keysize=2048 build-client-full client
```
This command will prompt for **two** passphrases: a `Client PEM passphrase` and a `CA PEM passphrase` (see above).

Generate a `Client TLS key`:
```
openvpn --tls-crypt-v2 /etc/easy-rsa/pki/private/tls-crypt-v2-server.key --genkey tls-crypt-v2-client /etc/easy-rsa/pki/private/tls-crypt-v2-client.key
```

This is how `/etc/easy-rsa/pki` should look like:
```console
$ tree /etc/easy-rsa/pki

|-- issued
|---- client.crt
|---- server.crt
|-- private
|---- ca.key
|---- client.key
|---- server.key
|---- tls-crypt-v2-client.key
|---- tls-crypt-v2-server.key
|---- ...
|-- dh.pem
|-- ca.crt
|-- server-PEM-passphrase.txt
```

Create an `OpenVPN` config file at `/etc/config/openvpn` with the following contents:
```
config openvpn 'OpenVPN_Server'
	
	option dev 'tun'
	option proto 'tcp4-server'
	option port '<DEST-PORT>'
	option server '10.0.100.0 255.255.255.0'
	option topology 'subnet'
	option dh '/etc/easy-rsa/pki/dh.pem'
	option ca '/etc/easy-rsa/pki/ca.crt'
	option cert '/etc/easy-rsa/pki/issued/server.crt'
	option key '/etc/easy-rsa/pki/private/server.key'
	option askpass '/etc/easy-rsa/pki/server-PEM-passphrase.txt'
	option tls_crypt_v2 '/etc/easy-rsa/pki/private/tls-crypt-v2-server.key'
	option tls_version_min '1.2'
	option allow_compression 'no'
	option keepalive '10 60'
	option auth_nocache '1'
	option cipher 'AES-256-GCM'
	option script_security '2'
	option client_to_client '1'
	option max_clients '10'
	option mssfix '1400'
	option tcp_queue_limit '256'
	option socket_flags 'TCP_NODELAY'
	list push 'block-outside-dns'
	list push 'redirect-gateway def1'
	list push 'dhcp-option DNS 8.8.8.8'
	list push 'route 10.0.100.0 255.255.255.0'
	list push 'socket-flags TCP_NODELAY'
	option enabled '1'
	option persist_key '1'
	option persist_tun '1'
	option user 'nobody'
	option group 'nogroup'
	option verb '4'
	option log_append '/var/log/openvpn.log'
```

Serve this config file to clients:
```
client
dev tun
proto tcp4-client
port <SOURCE-PORT>
remote <HOSTNAME>.duckdns.org
mssfix 1400
resolv-retry infinite
register-dns
block-outside-dns
nobind
persist-key
persist-tun
auth-nocache 1
keepalive 10 60
verb 4
user nobody
group nogroup
allow-compression no
cipher AES-256-GCM
remote-cert-tls server
tls-version-min 1.2
ca [inline]
cert [inline]
key [inline]
tls-crypt-v2 [inline]
<key>

Contents of /etc/easy-rsa/pki/private/client.key

</key>
<cert>

Contents of /etc/easy-rsa/pki/issued/client.crt


</cert>
<ca>

Contents of /etc/easy-rsa/pki/ca.crt

</ca>
<tls-crypt-v2>

Contents of /etc/easy-rsa/pki/private/client.pem

</tls-crypt-v2>
```
