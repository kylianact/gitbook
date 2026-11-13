---
description: >-
  Mise en place d'un script pour forcer le NTP vers le serveur NTP sur chacune
  des machines utilisant ce script.
---

# NTP

**Bien verifier qu'il a le droit d'etre executer :**&#x20;

<mark style="color:green;">`chmod +x <nom_du_fichier>.sh`</mark>

```bash
#### Script pour forcer le NTP / installer avant NTPDATE

#!/bin/bash

# Vérifier si le script est exécuté avec les privilèges root
if [[ $EUID -ne 0 ]]; then
   echo "Ce script doit être exécuté en tant que root. Utilise sudo." 
   exit 1
fi

# Serveur NTP
NTP_SERVER="192.168.60.15"

# Forcer la synchronisation avec le serveur NTP
echo "Synchronisation de l'heure avec le serveur NTP $NTP_SERVER..."
ntpdate -u $NTP_SERVER

# Vérifier si la commande ntpdate a réussi
if [[ $? -eq 0 ]]; then
    echo "Synchronisation réussie avec $NTP_SERVER."
else
    echo "Erreur lors de la synchronisation avec $NTP_SERVER."
fi
```
