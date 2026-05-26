## Deploy a server on Hetzner

1. Install `hcloud` - command line interface for Hetzner Cloud: `sudo pacman -S hcloud`
2. Create context and enter API token: `hcloud context create MAIN`
3. Create a server: 
```
hcloud server create --name test --type cx23 --image debian-13 --location nbg1 --without-ipv6 --ssh-key <SSH key name> --user-data-from-file <path to cloud-init.yaml>
```
4. To remove a server: `hcloud server delete test`