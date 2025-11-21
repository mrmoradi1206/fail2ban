# Fail2ban + Telegram + GeoIP + Banned IP List

This setup integrates **Fail2ban** with **Telegram bot alerts**, including:

- New banned IP
- GeoIP (country of attacker)
- All currently banned IPs in the jail

⚠️ Note: The action calls `fail2ban-client` from inside Fail2ban.  
This gives rich info, but on very busy systems it **may be heavier**.

---

## Files

### `jail.local`

Main Fail2ban configuration enabling the `sshd` jail and connecting it to the custom Telegram action:

```ini
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






