## Guide to GNU Privacy Guard (GPG)

[GnuPG or GPG](https://wiki.gnupg.org/GnuPG) is a modular crypto engine that allows user to encrypt and sign data and communications. It is used, for instance, to sign `git` commits. Below is an introductory guide on how to use GPG.

Generate a GPG key pair (=a private key and a public key):
```sh
gpg --full-generate-key
```
You will be prompted to enter your name, email and *private key password*. Store passwords securely!

To list public GPG keys that are already registered in your system, use respectively:
```console
$ gpg --list-keys

...
pub   ed25519 YYYY-MM-DD [SC] [expires: YYYY-MM-DD]
      C785719C3D1F622492D00C3068F0A5FEFFB4392C      <-- this is <FULL-KEY-ID>
uid           [ultimate] Name Surname <email@email.com>
sub   cv25519 YYYY-MM-DD [E] [expires: YYYY-MM-DD]
...
```
Analogously, use the following command to list private keys:
```console
$ gpg --list-secret-keys --keyid-format=long

...
sec   ed25519/68F0A5FEFFB4392C YYYY-MM-DD [SC] [expires: YYYY-MM-DD]
      C785719C3D1F622492D00C3068F0A5FEFFB4392C
uid                 [ultimate] Name Surname <email@email.com>
ssb   cv25519/F32335DE2AAA88B7 YYYY-MM-DD [E] [expires: YYYY-MM-DD]
...

```
Here, `<KEY-ID>=68F0A5FEFFB4392C` (listed under `sec`) and `<SUBKEY-ID>=F32335DE2AAA88B7` (listed under `ssb`). The subkeys are a good way of signing something without the risk of exposing the `<FULL-KEY-ID>`.

A GPG key pair can be transferred to a different physical machine. Also, GPG public key can be published on the Internet to verify GPG signatures, i.e., on [GitHub](https://github.com) or [GitLab](https://gitlab.com). Both of these operations require exporting public (and/or private) key as an ASCII file. This can be done by following commands (the latter one requires *private key password*):
```sh
gpg --export --armor <FULL-KEY-ID> > public-key.asc
gpg --export-secret-keys --armor <FULL-KEY-ID> > private-key.asc
```
Without the `--armor` option, `gpg --export` outputs binary data in **base16** format, which cannot directly be displayed as text. `gpg --export --armor` outputs **base64** encoded data, alongside a plaintext header and footer.

To import the key on another machine:
```sh
gpg --import public-key.asc
gpg --import private-key.asc
```
Then, set the trust level to `ultimate`:
```console
$ gpg --edit-key <FULL-KEY-ID>

gpg> trust
gpg> 5
gpg> save
```

To remove a key, **first delete the secret key**, then the public key:
```sh
gpg --delete-secret-keys <FULL-KEY-ID>
gpg --delete-keys <FULL-KEY-ID>
```