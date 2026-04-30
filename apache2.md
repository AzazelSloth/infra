# Personnalization

For better securty, edit `apache2` configuration as follows:

```bash
nano /etc/apache2/conf/httpd.conf
```

Then look for and edit following lines:

```apache
# The Listen port can be updated using 'Tweak Settings' -> 'System',
# However, if you have any Apache Reserved IPs, then this Tweak setting will
# be ignored. Instead, each IP on your system (excluding Apache Reserved IPs)
# will be listed here.
Listen 0.0.0.0:[http_port]
Listen [::]:[http_port]

# The Listen port can be updated using 'Tweak Settings' -> 'System',
# However, if you have any Apache Reserved IPs, then this Tweak setting will
# be ignored. Instead, each IP on your system (excluding Apache Reserved IPs)
# will be listed here.
Listen 0.0.0.0:[https_port]
Listen [::]:[https_port]
```

Some issues were encontered while doing the configuration.
Here is the new one `/etc/apache2/sites-available/rps.conf`

```conf
<VirtualHost *:80>
    ServerName appli.laroche360.ca
    ServerAlias automation.laroche360.ca

    RewriteEngine On
    RewriteCond %{REQUEST_URI} !^/\.well-known/acme-challenge/
    RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]

    ErrorLog ${APACHE_LOG_DIR}/appli.laroche360.ca-error.log
    CustomLog ${APACHE_LOG_DIR}/appli.laroche360.ca-access.log combined
</VirtualHost>

<VirtualHost *:443>
    ServerName appli.laroche360.ca
    ServerAlias automation.laroche360.ca

    SSLEngine on
    SSLProtocol TLSv1.2 TLSv1.3
    SSLCipherSuite HIGH:!aNULL:!MD5
    SSLHonorCipherOrder on
    SSLCompression off
    SSLUseStapling off

    SSLCertificateFile /etc/letsencrypt/live/appli.laroche360.ca/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/appli.laroche360.ca/privkey.pem

    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-XSS-Protection "1; mode=block"
    Header always set Referrer-Policy "strict-origin-when-cross-origin"
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"

    ProxyPreserveHost On
    ProxyRequests Off
    ProxyVia Off

    ProxyPass /.well-known/acme-challenge/ !
    ProxyPass / http://127.0.0.1:8786/
    ProxyPassReverse / http://127.0.0.1:8786/

    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-For "%{REMOTE_ADDR}s"
    RequestHeader set X-Real-IP "%{REMOTE_ADDR}s"
    RequestHeader set X-Forwarded-Host "%{HTTP_HOST}s"
    RequestHeader set X-Forwarded-Port "%{SERVER_PORT}s"

    <Proxy *>
        Require all granted
    </Proxy>

    ErrorLog ${APACHE_LOG_DIR}/appli.laroche360.ca-ssl-error.log
    CustomLog ${APACHE_LOG_DIR}/appli.laroche360.ca-ssl-access.log combined
</VirtualHost>
```

Then enable the new configuration and restart apache2:

```bash
# Check apache syntax
sudo apache2ctl configtest
# Check website activation
sudo a2ensite rps.conf
sudo apache2ctl -S
# Check necessary modules
sudo apache2ctl -M | grep -E 'ssl|proxy|proxy_http|rewrite|headers'
# Restart apache2
sudo systemctl restart apache2
```
