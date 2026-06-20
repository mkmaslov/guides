# OpenWRT router configuration

### Fix browser certificate warning

By default, browsers will complain about LuCI's self-signed HTTPS certificate. The solution to this problem is to create your own certificate and configure both OpenWRT and the browser to use it.

#### Create local certificate authority, create `uhttpd` certificate and sign it

Use this script:
```
curl -O https://raw.githubusercontent.com/mkmaslov/automate/refs/heads/main/dist/openwrt/create-uhttpd-cert.sh
chmod +x create-uhttpd-cert.sh
```
Usage instructions:
```
Usage: create-uhttpd-cert.sh -t SSH_TARGET [-h ROUTER_HOST] [-a ROUTER_IP] [-d DIR_CERT]

Examples:
  create-uhttpd-cert.sh -t root@192.168.10.1
  create-uhttpd-cert.sh -t root@192.168.10.1 -h router.lan -a 192.168.10.1 -d ./OPENWRT-CERT

Defaults:
  ROUTER_HOST=router.lan
  ROUTER_IP=192.168.10.1
  DIR_CERT=./OPENWRT-CERT
```

#### Import CA certificate into browser

- For Firefox, navigate to `Settings → Privacy & Security → Certificates → Manage Certificates → Authorities → Import`
- Choose `${DIR-CERT}/local-CA.crt`
- Choose `Trust this CA to identify websites`

**WARNING!** Do not import this file as an authority: `${DIR-CERT}/uhttpd.crt`. This file is the LuCI server certificate, not the CA certificate.
