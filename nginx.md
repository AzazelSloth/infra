# Ubuntu installation

## Initialize

Install the prerequisites:

```bash
sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
```

Import an official nginx signing key so apt could verify the packages authenticity. Fetch the key:

```bash
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
```

Verify that the downloaded file contains the proper key:

```bash
gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
```

The output should contain the full fingerprint `573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62` as follows:

```bash
pub   rsa2048 2011-08-19 [SC] [expires: 2027-05-24]
      573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
uid                      nginx signing key <signing-key@nginx.com>
```

Note that the output can contain other keys used to sign the packages.

## Repository setup

To set up the apt repository for stable nginx packages, run the following command:

```bash
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
https://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

If you would like to use mainline nginx packages, run the following command instead:

```bash
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
https://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

Set up repository pinning to prefer our packages over distribution-provided ones:

```bash
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx
```

## Installation

To install nginx, run the following commands:

```bash
sudo apt update
sudo apt install nginx
```

Doc at: [Install Nginx on Ubuntu](https://nginx.org/en/linux_packages.html#Ubuntu)

---

# Config

Edit `apache-proxy.conf` file by running command:

```bash
nano /etc/nginx/conf.d/apache-proxy.conf
```

Then copy the script bellow:

```nginx
# -------------------------------
# HTTP → HTTPS redirection
# -------------------------------
server {
    listen 80;
    # Allow all requests
    server_name _;

    # Permanent HTTPS redirection
    return 301 https://$host$request_uri;
}

# -------------------------------
# Optionnal for internal HTTP (proxy to Apache on [PORT])
# -------------------------------
# If you also want to serve HTTP directly via proxy (optional)
# Normally, the above redirection is sufficient, so this block can be ignored
server {
    listen 127.0.0.1:8080;
    server_name localhost;

    location / {
        proxy_pass http://127.0.0.1:8080;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}

# -------------------------------
# Important Notes
# -------------------------------
# 1. No 'listen 443 ssl' block: HTTPS is handled by Apache
# 2. server_name _ ; means that Nginx accepts all domains
# 3. X-Forwarded-* headers are important so that Apache knows the real domain and IP address
# 4. HTTP → HTTPS redirection avoids modifying Apache's existing SSL configuration
```
