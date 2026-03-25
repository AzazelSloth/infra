# Installation

## Install snapd

Follow instructions on snapcraft's site: [Install the daemon - Snap documentation](https://snapcraft.io/docs/installing-snapd/)

## Remove certbot-auto and any Certbot OS packages

```bash
sudo apt-get remove certbot
```

## Install Certbot

Run this command on the command line on the machine to install Certbot.

```bash
sudo snap install --classic certbot
```

## Prepare the Certbot command

Execute the following instruction on the command line on the machine to ensure that the `certbot` command can be run.

```bash
sudo ln -s /snap/bin/certbot /usr/local/bin/certbot
```

## Choose how to run Certbot

### Get and install your certificates

Run this command to get a certificate and have Certbot edit your nginx configuration automatically to serve it, turning on HTTPS access in a single step.

```bash
sudo certbot --nginx
```

### Or, just get a certifcate

If you're feeling more conservative and would like to make the changes to your nginx configuration by hand, run this command.

```bash
sudo certbot certonly --nginx
```

## Test automatic renewal

The Certbot packages on your system come with a cron job or systemd timer that will renew your certificates automatically before they expire. You will not need to run Certbot again, unless you change your configuration. You can test automatic renewal for your certificates by running this command:

```bash
sudo certbot renew --dry-run
```

## Confirm that Certbot worked

To confirm that your site is set up properly, visit [Your website URL](https://yourwebsite.com/) in your browser and look for the lock icon in the URL bar.

Docs at: [Certbots Intsructions | Certbot](https://certbot.eff.org/instructions?ws=nginx&os=snap)
