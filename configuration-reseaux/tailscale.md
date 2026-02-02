# Tailscale

## Installation de Tailscale depuis homebrew

```shellscript
brew install --formula tailscale
sudo brew services start tailscale
sudo tailscale up
sudo tailscale status
```

## Installation de Tailscale depuis LXC Debian12

Depuis le shell proxmox :&#x20;

```shellscript
### Dans le fichier extension au LXC utilisé :
nano /etc/pve/lxc/1XX.conf

### En bas de la page :
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

```shellscript
### Dans le LXC faire ces deux commandes shell :
curl -fsSL https://tailscale.com/install.sh | sh
systemctl start tailscale
```

