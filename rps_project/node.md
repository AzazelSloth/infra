# Installation

## Official docs

### Docker installation

For official steps (docs) using docker:

```bash
# Docker has specific installation instructions for each operating system.
# Please refer to the official documentation at https://docker.com/get-started/

# Pull the Node.js Docker image:
docker pull node:24-alpine

# Create a Node.js container and start a Shell session:
docker run -it --rm --entrypoint sh node:24-alpine

# Verify the Node.js version:
node -v # Should print "v24.14.1".

# Verify npm version:
npm -v # Should print "11.11.0".
```

### NVM Instrallation

Getting Node.js® for linux using `nvm` with `npm`

```bash
# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash

# in lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 24

# Verify the Node.js version:
node -v # Should print "v24.14.1".

# Verify npm version:
npm -v # Should print "11.11.0".

```

## APT Installation

Using APT:

```bash
sudo apt install nodejs npm
```
