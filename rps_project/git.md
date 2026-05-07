# Install on Ubuntu

It is easiest to install Git on Linux with your distribution's package manager.

```bash
sudo apt install git
```

For Ubuntu, this PPA provides the latest stable upstream Git version

```bash
sudo add-apt-repository ppa:git-core/ppa
sudo apt update; apt install git
```

Doc at : [Git Installation Docs](https://git-scm.com/install/linux)

## Initial setup

### Identity

Setting user name and email address

```bash
git config --global user.name "John Doe"
git config --global user.email "johndoe@example.com"
```

### Default branch name

To set main as the default branch name do:

```bash
git config --global init.defaultBranch main
```

### Checking settings

Use `git config --list` to list all the settings
