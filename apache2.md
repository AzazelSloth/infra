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
