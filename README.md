# Fail2ban + Telegram + GeoIP + Banned IP List

This setup integrates **Fail2ban** with a **Telegram bot** and sends
alerts when an IP is banned or unbanned.

Each alert includes: - 🚫 New banned IP\
- 🌍 GeoIP country info\
- 📋 All currently banned IPs

------------------------------------------------------------------------

## jail.local

``` ini
[DEFAULT]
bantime  = 10m
findtime = 10m
maxretry = 5

bantime.increment = true
bantime.factor    = 2
bantime.maxtime   = 1d

[sshd]
enabled  = true
port     = ssh
logpath  = /var/log/auth.log
backend  = systemd

action = %(action_)s
         telegram-notify
```

------------------------------------------------------------------------

## action.d/telegram-notify.conf

``` ini
[Definition]

actionstart =
actionstop  =
actioncheck =

actionban = GEO=$(/usr/bin/geoiplookup <ip> 2>/dev/null | head -n1 | sed 's/.*: //'); BANNED=$(/usr/bin/fail2ban-client status <name> 2>/dev/null | sed -n 's/.*Banned IP list: *//p'); /usr/bin/curl -s -X POST "https://api.telegram.org/botYOUR_BOT_TOKEN/sendMessage" -d chat_id="YOUR_CHAT_ID" -d text="[Fail2Ban] 🚫 Jail <name> banned IP <ip> (GeoIP: $GEO) for <bantime> seconds (failures: <failures>). All banned IPs: $BANNED"

actionunban = /usr/bin/curl -s -X POST "https://api.telegram.org/botYOUR_BOT_TOKEN/sendMessage" -d chat_id="YOUR_CHAT_ID" -d text="[Fail2Ban] ✅ Jail <name> unbanned IP <ip>"

[Init]
```

------------------------------------------------------------------------

## How to Use

1.  Install packages:

``` bash
sudo apt install -y fail2ban curl geoip-bin geoip-database
```

2.  Copy files:

``` bash
sudo cp jail.local /etc/fail2ban/jail.local
sudo cp action.d/telegram-notify.conf /etc/fail2ban/action.d/telegram-notify.conf
```

3.  Edit your bot token & chat ID in:

```{=html}
<!-- -->
```
    /etc/fail2ban/action.d/telegram-notify.conf

4.  Reload Fail2ban:

``` bash
sudo fail2ban-client reload
sudo systemctl restart fail2ban
```

5.  Test:

``` bash
sudo fail2ban-client set sshd banip 1.2.3.4
```
