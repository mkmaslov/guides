## Guide to Secure Shell (ssh)

### Contents

- [Installation](#installation)
- [Configuration](#configuration)
- [Key generation](#key-generation)
- [Connection to a git instance](#connection-to-a-git-instance)
- [Connection to a server](#connection-to-a-server)

### Installation 

On most Linux distributions, OpenSSH implementation of the Secure Shell comes pre-installed. Whether the SSH tools are present in the system or not can be verified by running: `ssh -v`, which should return anything but error.


### Configuration

Create a folder to store SSH keys and the SSH configuration file:
```sh
mkdir "${HOME}/.ssh" && touch "${HOME}/.ssh/config" 
```


The configuration file should list all the hosts (URLs) and paths to the respective key files. For instance, entries for [github.com](https://github.com) and [gitlab.com](https://gitlab.com) would look like this:
```console
$ cat "${HOME}/.ssh/config"

# GitHub
Host github.com
  PreferredAuthentications publickey
  IdentityFile ${HOME}/.ssh/github_key
# GitLab
Host gitlab.com
  PreferredAuthentications publickey
  IdentityFile ${HOME}/.ssh/gitlab_key
```

### Key generation

To generate the SSH key pair, run:
```sh
ssh-keygen -t <keytype> -C "<hostname>" -f "${HOME}/.ssh/<keyname>"
```
- `-t` specifies the key type. Use `ED25519`, unless suggested otherwise.
- `-C` creates a comment for the key. It is useful for distinguishing key files from one another. Just use the `<hostname>` of a server, unless using several key pairs with the same server.

The above command creates a pair of key files: a private (named `<keyname>`) and a public one (named `<keyname>.pub`). **Make sure you don't share any of your private keys with anyone!**

### Connection to a git instance

[Generate the SSH key pair](#key-generation) and [create the respective entry in the SSH configuration file](#configuration). Note that GitLab specifically [suggests](https://docs.gitlab.com/ee/user/ssh.html#ed25519-ssh-keys) using `ED25519` SSH key type (set by the `-t` option).

Print out the newly generated public key and manually copy-paste it into your [GitHub settings](https://github.com/settings/keys) (Settings → SSH and GPG keys) or [GitLab settings](https://gitlab.com/-/user_settings/ssh_keys) (User Settings → SSH keys):
```console
$ cat "${HOME}/.ssh/github_key.pub"

ssh-ed25519 AANAC5NDaC17ZDI114E5AACAI******************hah9WoDAf1HLRLiH6odOAAlUK GitHub
```

Verify that your SSH connection to GitHub or GitLab works:
```console
$ ssh -T git@github.com

Hi <GitHub_username>! You've successfully authenticated, but GitHub does not provide shell access.


$ ssh -T git@gitlab.com

Welcome to GitLab, @<GitLab_username>!
```
Note that git username (`git`) and hostname (`gitlab.com`) may be different, if you are using a self-hosted GitLab instance.

**Finally**, you can use the SSH to clone a repository:
```sh
git clone git@github.com:<GitHub_username>/<repo_name>.git
```

### Connection to a server

[Generate the SSH key pair](#key-generation) and [create the respective entry in the SSH configuration file](#configuration).<br>
Authorize the public key on server using: 
```sh
ssh-copy-id -i ~/.ssh/<keyname>.pub <username>@<hostname>
```
