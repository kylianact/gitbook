---
description: Configuration d'un serveur Proxy avec squid.
---

# Proxy

Installation :&#x20;

<mark style="color:green;">`sudo apt update && sudo apt install squid -y`</mark>

Configuration :&#x20;

<mark style="color:green;">`sudo nano /etc/squid/squid.conf`</mark>

```
# Autoriser l'accès aux clients locaux
acl localnet src 192.168.0.0/16   # Adapter selon le réseau de l'IUT
acl localhost src 127.0.0.1

http_access allow localnet
http_access allow localhost

# Interdire tout le reste
http_access deny all

# Rediriger tout le trafic via le cache de l'IUT
cache_peer cache.univ-pau.fr parent 3128 0 no-query default

# Forcer Squid à toujours utiliser ce cache
never_direct allow all

# Définir le port d'écoute local
http_port 3128

# Activer le cache DNS interne
dns_v4_first on

# Optimisation des performances du cache
cache_mem 256 MB
maximum_object_size 50 MB
cache_dir ufs /var/spool/squid 1000 16 256

```

* <mark style="color:green;">**`cache_peer cache.univ-pau.fr parent 3128 0 no-query default`**</mark>\
  → Définit <mark style="color:green;">`cache.univ-pau.fr`</mark> comme **proxy parent** sur le port <mark style="color:green;">`3128`</mark>.
* <mark style="color:green;">**`never_direct allow all`**</mark>\
  → Interdit aux clients d’accéder directement à Internet (**oblige le passage par&#x20;**<mark style="color:green;">**`cache.univ-pau.fr`**</mark>).
* <mark style="color:green;">**`acl localnet src 192.168.0.0/16`**</mark>\
  → Définit la plage d'adresses IP autorisée (à adapter selon le réseau de l'IUT).
* <mark style="color:green;">**`http_access allow localnet`**</mark>\
  → Autorise les machines locales à utiliser le proxy.
* <mark style="color:green;">**`http_access deny all`**</mark>\
  → Bloque tout le trafic qui ne correspond pas aux règles autorisées.



<mark style="color:green;">sudo systemctl restart squid</mark>

<mark style="color:green;">sudo systemctl status squid</mark>
