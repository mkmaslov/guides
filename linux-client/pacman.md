## Guide to Arch Linux package manager (pacman)

### Basic commands

- Install a package: `sudo pacman -S <package>`
- Remove package and its dependencies: `sudo pacman -Rns <package>`
- Update package database: `sudo pacman -Sy`
- Upgrade outdated packages: `sudo pacman -Syu`
- Clean up cache: `sudo pacman -Sc`
- Remove all orphaned packages: `pacman -Qdtq | sudo pacman -Rns -`

### Install outdated versions of packages

The official `pacman` repositories only retain the *latest* versions of packages. However, Arch maintains a full snapshot archive of the entire package repository. One can use it to manually download and install older packages.
1. Go to: [https://archive.archlinux.org/packages/](https://archive.archlinux.org/packages/)
2. Browse or search for the package you want.
3. Download the desired version (a `.pkg.tar.zst` file).
4. Install it manually:
    ```sh
    sudo pacman -U /path/to/package.pkg.tar.zst
    ```