# Installation and config

## Install

```bash
sudo apt update
sudo apt install ufw
```

## Enabling

To turn UFW on with the default set of rules:

```bash
sudo ufw enable
```

To check status:

```bash
sudo ufw status verbose
```

The output should be like this:

```bash
youruser@yourcomputer:~$ sudo ufw status verbose
[sudo] password for youruser:
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing)
New profiles: skip
youruser@yourcomputer:~$
```

Note that by default, deny is being applied to incoming. There are exceptions, which can be found in the output of this command:

```bash
sudo ufw show raw
```

You can also read the rules files in /etc/ufw (the files whose names end with .rules).

Doc at: [UFW - Community Help Wiki](https://help.ubuntu.com/community/UFW)

## Allow and Deny (specific rules)

### Allow

Syntax should be like this:

```bash
sudo ufw allow <port>/<optional: protocol>
```

Example: To allow incoming tcp and udp packet on port 53

```bash
sudo ufw allow 53
```

Example: To allow incoming tcp packets on port 53

```bash
sudo ufw allow 53/tcp
```

### Deny

Here is the syntax:

```bash
sudo ufw deny <port>/<optional: protocol>
```

Example: To deny tcp and udp packets on port 53

```bash
sudo ufw deny 53
```

Example: To deny incoming udp packets on port 53

```bash
sudo ufw deny 53/udp
```

## Delete existing rule

To delete a rule, simply prefix the original rule with delete. For example, if the original rule was:

```bash
sudo ufw deny 80/tcp
```

Use this to delete it:

```bash
sudo ufw delete deny 80/tcp
```

## Services

You can also allow or deny by service name since ufw reads from /etc/services To see get a list of services:

```bash
less /etc/services
```

### Allow by Service Name

```bash
sudo ufw allow <service name>
```

Example: To allow ssh by name

```bash
sudo ufw allow ssh
```

### Deny by service name

```bash
sudo ufw deny <service name>
```

Example: To deny ssh by name

```bash
sudo ufw deny ssh
```

## Status

Checking the status of ufw will tell you if ufw is enabled or disabled and also list the current ufw rules that are applied to your iptables.

```bash
sudo ufw status

Firewall loaded

To                         Action  From
--                         ------  ----
22:tcp                     DENY    192.168.0.1
22:udp                     DENY    192.168.0.1
22:tcp                     DENY    192.168.0.7
22:udp                     DENY    192.168.0.7
22:tcp                     ALLOW   192.168.0.0/24
22:udp                     ALLOW   192.168.0.0/24
```

## Logging

To enable logging, use:

```bash
sudo ufw logging on
```

To disable logging, use:

```bash
sudo ufw logging off
```
