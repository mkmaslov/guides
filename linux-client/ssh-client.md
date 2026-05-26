## Guide to Secure Shell (ssh)


### Contents

- [Installation](#installation)
- [Configuration](#configuration)
- [Key generation](#key-generation)
- [Connection to a git instance](#connection-to-a-git-instance)
- [Connection to a server](#connection-to-a-server)


### Installation 

On most Linux distributions, Mac OS and Windows, OpenSSH implementation of the Secure Shell comes pre-installed. Whether the SSH tools are present in the system or not can be verified by running: `ssh -V`, which should return the SSH version.


### Configuration

A good practice is to keep the directory that contains SSH files (`~/.ssh` on Linux) organized. For instance, one could use the following structure:
```console
$ tree "${HOME}/.ssh"

|-- keys
|---- <key_name_1>
|---- <key_name_1>.pub
|---- <key_name_2>
|---- <key_name_2>.pub
|---- ...
|-- config
|-- known_hosts
```

To begin, create a folder to store SSH keys and the SSH configuration file:
```sh
mkdir -p "${HOME}/.ssh/keys" && touch "${HOME}/.ssh/config" 
```

The configuration file should list all the hostnames (URLs or IPs) together with paths to the respective key files and the relevant settings. Here is an example of an `~/.ssh/config`:
```sh
# Global settings (for all hosts)
Host *
  # Disable all authentication methods, except public key authentication
  PasswordAuthentication no
  ChallengeResponseAuthentication no
  PubkeyAuthentication yes
  PreferredAuthentications publickey
  # Disable data compression
  Compression no
  # Add keys to a keyring to avoid re-typing SSH key passphrases
  AddKeysToAgent yes
  # Forward the keyring, when performing a proxy jump
  ForwardAgent yes
  # Use only local known_hosts file
  UserKnownHostsFile ~/.ssh/known_hosts
  GlobalKnownHostsFile /dev/null
  # Ignore Post-Quantum Cryptography warning
  WarnWeakCrypto no-pq-kex

# git instances
# (git over SSH uses "git" as a username, not the GitHub/GitLab username,
#  with the exception of specifically configured self-hosted GitLab instances)

# GitHub
Host GitHub
  Hostname github.com
  User git
  IdentityFile ${HOME}/.ssh/keys/GitHub
  HostKeyAlias GitHub
  HostKeyAlgorithms ssh-ed25519

# GitHub_alt
# (use this for a second/different GitHub account)
Host GitHub_alt
  Hostname github.com
  User git
  IdentityFile ${HOME}/.ssh/keys/GitHub_alt
  HostKeyAlias GitHub_alt
  HostKeyAlgorithms ssh-ed25519

# GitLab
Host GitLab
  Hostname gitlab.com
  User git
  IdentityFile ${HOME}/.ssh/keys/GitLab
  HostKeyAlias GitLab
  HostKeyAlgorithms ssh-ed25519

# Servers

# OpenWRT
Host OpenWRT
  Hostname 192.168.1.1
  Port 1234
  User root
  IdentityFile ${HOME}/.ssh/keys/OpenWRT
  HostKeyAlias OpenWRT
  HostKeyAlgorithms ssh-ed25519

# Server
Host Server
  Hostname <DOMAIN-NAME>
  User <USER-NAME>
  Port <PORT>
  IdentityFile ${HOME}/.ssh/keys/Server
  HostKeyAlias Server
  HostKeyAlgorithms ssh-ed25519
  # If ssh-ed25519 is not available:
  # HostKeyAlgorithms rsa-sha2-512

# Server (GVFS-specific)
# (gvfs does not recognize hostname aliases, use full <DOMAIN-NAME>)
Host <DOMAIN-NAME>
  User <USER-NAME>
  Port <PORT>
  IdentityFile ${HOME}/.ssh/keys/Server
  HostKeyAlias Server
  HostKeyAlgorithms rsa-sha2-512

# Server's LAN node
# (some server nodes are unaccessible from the Internet, so a proxy jump
#  through frontnode (Server) to LAN node (node* - wildcard alias) is required)
Host node*
  Port <PORT>
  User <USER-NAME>
  ProxyCommand ssh -W %h:%p Server  
  HostKeyAlias Server
```


### Key generation

To generate an SSH key pair, run:
```sh
ssh-keygen -t <KEY-TYPE> -a  1000 -C "<HOST>" -f "${HOME}/.ssh/keys/<KEY-NAME>"
```
- `-t` specifies the key type. Use [elliptic curve](https://en.wikipedia.org/wiki/Curve25519), i.e. `<KEY-TYPE> = ED25519`, unless requested otherwise. Note that GitLab specifically [suggests](https://docs.gitlab.com/ee/user/ssh.html#ed25519-ssh-keys) using `ED25519` SSH key type.
- `-a` specifies the number of key generations. A higher setting makes keys harder to brute-force.
- `-C` creates a comment for the key. This is useful for distinguishing key files from one another. Just use the ID of a connection from `~/.ssh/config`, i.e. `<HOST>=GitHub`. Name key files accordingly, i.e. `<KEY-NAME>=GitHub`.


The above command will ask for an SSH key password. Optimal length is **64 characters**. Longer passwords, i.e. 128 characters, may cause significant delay during an SSH login. `ssh-keygen` creates a pair of key files: a private key (named `<KEY-NAME>`) and a public one (named `<KEY-NAME>.pub`). **Make sure NOT to share any of the private keys with anyone!**

To avoid entering the SSH key password at every login, one could use `ssh-agent` to store it in the system's keyring, i.e. GNOME Keyring on Arch Linux, using:
```sh
ssh-add "${HOME}/.ssh/keys/<KEY-NAME>"
```

### Connection to a git instance

[Generate the SSH key pair](#key-generation) and [create the respective entry in the SSH configuration file](#configuration).

Print out the newly generated public key and manually copy-paste it into your [GitHub settings](https://github.com/settings/keys) (Settings → SSH and GPG keys) or [GitLab settings](https://gitlab.com/-/user_settings/ssh_keys) (User Settings → SSH keys):
```console
$ cat "${HOME}/.ssh/keys/GitHub.pub"

ssh-ed25519 AANAC5NDaC17ZDI114E5A**************************hah9WoDAf1HLRLiH6odOAAlUK GitHub
```

Verify that the SSH connection to GitHub or GitLab works:
```console
$ ssh -T GitHub

Hi <GITHUB-USERNAME>! You've successfully authenticated, but GitHub does not provide shell access.


$ ssh -T GitLab

Welcome to GitLab, @<GITLAB-USERNAME>!
```

**Finally**, one can use git over SSH to clone a repository:
```sh
git clone GitHub:<GITHUB-USERNAME>/<REPOSITORY-NAME>.git
```

### Connection to a server

[Generate the SSH key pair](#key-generation) and [create the respective entry in the SSH configuration file](#configuration).<br>

Authorize the public key on server using: 
```sh
ssh-copy-id -i ~/.ssh/keys/Server.pub <USER-NAME>@<DOMAIN-NAME>:<PORT>
```
Then, connect to the server using the ID from `~/.ssh/config`:
```sh
ssh Server
```
