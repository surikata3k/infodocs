# System

## Services

List all services:

```bash
systemctl list-unit-files --type service -all
```

## Firewall: iptables rules

```bash
iptables -L --line-numbers
```

## Show the IP before login

Adds the IP address to the login screen message:

```bash
echo "IP: \4" >> /etc/issue
```

## Keep running when the laptop lid is closed

```ini title="/etc/systemd/logind.conf"
HandleLidSwitch=ignore
```

## GRUB

```ini title="/etc/default/grub"
GRUB_TIMEOUT=0
GRUB_CMDLINE_LINUX="consoleblank=60"
```

Apply the changes:

```bash
update-grub
```

!!! info
    `GRUB_TIMEOUT=0` skips the boot menu. `consoleblank=60` turns off the console screen after 60 seconds.

## Text editor from the terminal

Opens the graphical editor (Linux Mint):

```bash
xed
```
