# Pulse

Sur le Proxmox, creer un CT Templates > Debian-12-standard.

Une fois fait, vous pouvez créer le LXC Pulse.

```shellscript
apt-get update
apt-get install curl
curl -fsSL https://raw.githubusercontent.com/rcourtman/Pulse/main/install.sh | bash
systemctl status pulse
```

Puis ensuite pour y avoir acces depuis l'exterieur, suivre les infos données dans l'onglet Tailscale pour l'installer
