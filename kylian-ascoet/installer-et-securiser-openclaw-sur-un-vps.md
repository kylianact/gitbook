# 5. Verrouiller la configuration

Ce projet installe OpenClaw à la main sur un VPS Ubuntu 24.04, sans l'option en un clic de l'hébergeur. Les agents tournent en arrière-plan dans une sandbox, la page web n'est jamais exposée sur Internet, et Claude peut exploiter le serveur par SSH sans être root.

Le projet a été réalisé avec l'aide de Claude, de la conception aux tests.

### Résultat

| Élément    | Choix                                                      |
| ---------- | ---------------------------------------------------------- |
| Système    | Ubuntu 24.04 sur un VPS                                    |
| OpenClaw   | Installé à la main, version épinglée                       |
| Exposition | Seul SSH est joignable depuis Internet, par clé uniquement |
| Page web   | Accessible uniquement par tunnel SSH                       |
| Agents     | Docker rootless, sans réseau, racine en lecture seule      |
| Postes     | Mac et PC Windows                                          |
| Assistance | Claude par SSH, avec un sudo limité                        |

### Architecture

```
Mac ou PC Windows
    |
    | SSH, clé uniquement, tunnel limité à la passerelle
    v
VPS Ubuntu
    |
    +-- Compte admin : administration, sudo avec mot de passe
    +-- Compte claude-ops : exploitation d'OpenClaw, jamais root
    +-- Compte openclaw : passerelle sur 127.0.0.1:18789
            |
            +-- Docker rootless
                    |
                    +-- Sandbox des agents : sans réseau
```

| Compte       | Rôle                               | Accès                               |
| ------------ | ---------------------------------- | ----------------------------------- |
| `admin`      | Administration du serveur          | SSH par clé, sudo avec mot de passe |
| `claude-ops` | Exploitation d'OpenClaw par Claude | SSH par clé, sudo limité            |
| `openclaw`   | Passerelle et Docker rootless      | Aucune connexion SSH                |
| `oc-pkg`     | Installation des paquets npm       | Aucune connexion SSH, aucun shell   |

### 1. Préparer le serveur

1. Créer un snapshot chez l'hébergeur.
2. Faire un inventaire en lecture seule : système, comptes, ports, pare-feu et Docker.
3. Créer des clés SSH dédiées sur chaque poste, protégées par une phrase de passe.
4. Tester les scripts sur un banc local avant de toucher au VPS.

### 2. Durcir le système

La configuration SSH refuse root et les mots de passe. Le compte d'administration ne peut ouvrir qu'un seul tunnel, vers la passerelle.

```
PermitRootLogin no
PasswordAuthentication no
AllowTcpForwarding no
AllowUsers admin claude-ops

Match User admin
    AllowTcpForwarding local
    PermitOpen 127.0.0.1:18789
```

> Avant de recharger SSH, garde une session ouverte et prévois un retour arrière.

Le pare-feu local ferme tout sauf SSH, et fail2ban bannit les adresses qui insistent.

```bash
ufw default deny incoming
ufw allow 22/tcp
ufw enable
```

### 3. Installer OpenClaw

La passerelle tourne dans un service systemd isolé. Voici un extrait, pas un fichier complet :

```ini
[Service]
User=openclaw
ExecStart=/opt/openclaw/current/bin/openclaw gateway --port 18789
ProtectSystem=strict
NoNewPrivileges=yes
CapabilityBoundingSet=
MemoryMax=70%
```

Le score d'exposition systemd du service est tombé à 1,6. Le code appartient à root, et npm tourne sous un compte sans shell.

### 4. Enfermer les agents

La configuration part d'un principe simple : tout est fermé, puis seul l'utile est ouvert.

```javascript
{
  gateway: { mode: "local", bind: "loopback", port: 18789, auth: { mode: "token" } },
  agents: { defaults: { sandbox: { mode: "all", backend: "docker", docker: { network: "none", readOnlyRoot: true, capDrop: ["ALL"], memory: "1g", cpus: 1 } } } },
  tools: { elevated: { enabled: false }, fs: { workspaceOnly: true }, exec: { host: "sandbox" } },
}
```

Pour vérifier la sandbox, demande à l'agent de lancer une commande réseau : elle ne doit résoudre aucun nom.

### 5. Verrouiller la configuration

Une fois la configuration validée, la passerelle lit une copie protégée, immuable, posée par root. Ni un agent ni la page web ne peuvent la changer : toute modification demande le mot de passe sudo. L'ajout de skills et de plugins est bloqué par une politique d'installation.
