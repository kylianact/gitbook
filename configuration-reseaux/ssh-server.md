---
description: >-
  Configuration du ssh coté server automatisé avec utilisation forcé d'une clé
  publique.
---

# ssh-server

Avant l'execution de ce code en mode **root**, il faut faire un : &#x20;

<mark style="color:green;">`ssh-keygen -t rsa -b 4096`</mark>

Copier la clé public lorsque le code est en execution, il vous la demandera.&#x20;

<mark style="color:green;">`cat /home/"user"/.ssh/id_rsa.pub`</mark>

Bien verifier qu'il a le droit d'etre executer :&#x20;

<mark style="color:green;">`chmod +x <nom_du_fichier>.sh`</mark>

{% code lineNumbers="true" %}
```bash
#!/bin/bash

# Variables
USER_ADMIN="supervisor"  # Remplacez par l'utilisateur supervisor
SSH_DIR="/home/$USER_ADMIN/.ssh"
AUTHORIZED_KEYS="$SSH_DIR/authorized_keys"
SSH_CONFIG="/etc/ssh/sshd_config"

# Mettre à jour les paquets et installer SSH
echo "[+] Installation de OpenSSH Server..."
apt update -y && apt install -y openssh-server

# Vérifier si l'utilisateur admin existe, sinon le créer
if id "$USER_ADMIN" &>/dev/null; then
    echo "[+] L'utilisateur $USER_ADMIN existe déjà."
else
    echo "[+] Création de l'utilisateur $USER_ADMIN..."
    useradd -m -s /bin/bash "$USER_ADMIN"
    passwd -d "$USER_ADMIN"  # Supprime le mot de passe pour forcer l'utilisation de SSH
fi

# Création du dossier SSH et ajout de la clé publique
echo "[+] Configuration de l'authentification par clé publique..."
mkdir -p "$SSH_DIR"
chmod 700 "$SSH_DIR"
touch "$AUTHORIZED_KEYS"
chmod 600 "$AUTHORIZED_KEYS"

echo "[+] Veuillez copier la clé publique dans $AUTHORIZED_KEYS"
read -r -p "Collez la clé publique ici : " KEY
echo "$KEY" >> "$AUTHORIZED_KEYS"

chown -R "$USER_ADMIN":"$USER_ADMIN" "$SSH_DIR"

# Configurer SSHD
echo "[+] Configuration du serveur SSH..."
cp "$SSH_CONFIG" "${SSH_CONFIG}.bak"  # Sauvegarde de l'ancien fichier

cat <<EOF > "$SSH_CONFIG"
# Configuration SSH Sécurisée
Port 22
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
AllowUsers $USER_ADMIN
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding no
EOF

# Redémarrer SSH
echo "[+] Redémarrage du service SSH..."
systemctl restart ssh

echo "[✓] Configuration terminée ! Vous pouvez maintenant vous connecter avec :"
echo "ssh $USER_ADMIN@<IP_DU_SERVEUR>"
```
{% endcode %}

Sur le PC admin pour eviter d'utiliser la passphrase a chaque connexion, vous pouvez stocker la clé temporairement en memoire avec ces 2 commandes :&#x20;

<mark style="color:green;">`eval "$(ssh-agent -s)"`</mark>\ <mark style="color:green;">`ssh-add ~/.ssh/id_rsa`</mark>
