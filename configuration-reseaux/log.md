---
description: Mise en place d'une redirection des logs des VMs vers le serveur Rsyslog.
---

# Log

**A executer sur toutes les machines de l'infrastructure:**&#x20;

**Bien verifier qu'il a le droit d'etre executer :**&#x20;

<mark style="color:green;">`chmod +x <nom_du_fichier>.sh`</mark>

```bash
#!/bin/bash
# Script pour rediriger les logs vers le serveur Syslog 192.168.60.9

# Ajouter la configuration de redirection dans le fichier /etc/rsyslog.conf
echo "*.* @192.168.60.9:514" | sudo tee -a /etc/rsyslog.conf > /dev/null

# Redémarrer le service rsyslog pour appliquer les changements
sudo systemctl restart rsyslog

echo "Les logs sont maintenant redirigés vers le serveur Syslog à 192.168.60.9."

```
