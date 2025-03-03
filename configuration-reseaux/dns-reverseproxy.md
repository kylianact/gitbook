---
description: Configuration d'un serveur DNS ReverseProxy avec Bind9.
---

# DNS/ReverseProxy

**Configuration Serveur WEB apache2 en 192.168.50.3/29 :**&#x20;

<mark style="color:green;">`sudo apt-get install apache2 -y`</mark>

<mark style="color:green;">`sudo systemctl enable apache2`</mark>

<mark style="color:green;">`sudo systemctl start apache2`</mark>

_<mark style="color:purple;">allez sur Firefox et tapez :</mark> <mark style="color:purple;"></mark><mark style="color:purple;">**http://127.0.0.1/**</mark>_



**Configuration du dns/proxy :**&#x20;

<mark style="color:green;">`sudo apt update && sudo apt install bind9 -y`</mark>

<mark style="color:green;">`sudo systemctl start bind9`</mark>

<mark style="color:green;">`sudo systemctl status bind9`</mark>

<mark style="color:green;">`sudo nano /etc/bind/named.conf.options`</mark>

```clike
options {
    directory "/var/cache/bind";
    
    # Autoriser le DNS Récursif uniquement pour ton Réseau
        allow-recursion { 
                192.168.1.0/29;
                192.168.50.0/29;
                192.168.10.0/31; 
                127.0.0.1; };
    
    # Forwarding des requêtes vers des DNS publics sécurisés
        forwarders {
                194.167.156.13;  # DNS IUT
        };

    # Active le cache DNS
    dnssec-validation auto;
    
    # Écoute sur toutes les interfaces
    listen-on { any; };
};
```

<mark style="color:green;">`sudo systemctl restart bind9`</mark>

<mark style="color:green;">`sudo systemctl status bind9`</mark>

<mark style="color:purple;">**test:**</mark>

* <mark style="color:blue;">**Depuis la machine DNS :**</mark>&#x20;

<mark style="color:green;">`dig univ-pau.fr @127.0.0.1`</mark>

* <mark style="color:blue;">**Depuis une des machines clientes du  DNS :**</mark>&#x20;

<mark style="color:green;">`nslookup univ-pau.fr 192.168.50.2`</mark>

\
\
Configuration du REVERSE Proxy :&#x20;

_<mark style="color:red;">**ATTENTION ce nom de domaine est deja pris et donc ne fonctionne pas.**</mark>_&#x20;

_<mark style="color:red;">**Meme avec un autre nom de domaine la configuration ne fonctionne pas**</mark>_&#x20;

<mark style="color:green;">`sudo nano /etc/bind/db.kylian.com`</mark>

```
$TTL 86400
@   IN  SOA  ns1.kylian.com. root.kylian.com. (
     20240220  ; Serial (mets à jour à chaque modif)
     3600  ; Refresh
     1800  ; Retry
     604800 ; Expire
     86400 )   ; Negative Cache TTL


; Déclaration du serveur DNS
@   IN  NS ns1.kylian.com.
ns1 IN  A 192.168.50.2  ; IP de ton serveur DNS


; Entrées pour ton site web
@   IN  A 192.168.50.3   ; Le domaine principal pointe vers le serveur web
www IN  A 192.168.50.3   ; www.kylian.com pointe aussi vers le serveur web
```

<mark style="color:purple;">**Test de syntaxe:**</mark>&#x20;

<mark style="color:green;">`sudo named-checkzone kylian.com /etc/bind/db.kylian.com`</mark>

\
Si tout est bon :&#x20;

<mark style="color:green;">`sudo systemctl restart bind9`</mark>

\
<mark style="color:purple;">**Test :**</mark>&#x20;

* <mark style="color:blue;">Depuis la machine DNS :</mark>&#x20;

<mark style="color:green;">`dig kylian.com @127.0.0.1`</mark>

<mark style="color:green;">`dig www.kylian.com @127.0.0.1`</mark>

* <mark style="color:blue;">Depuis un client :</mark>&#x20;

<mark style="color:green;">`nslookup kylian.com 192.168.50.2`</mark>

<mark style="color:green;">`nslookup www.kylian.com 192.168.50.2`</mark>
