---
description: Dans cette partie nous verrons la configuration de l'infrastructure de
coverY: 0
layout:
  cover:
    visible: true
    size: hero
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Configuration Réseaux

## SAE R401

### Description de la SAE

Cette SAE consiste à concevoir et configurer un réseau d'entreprise sécurisé, en utilisant du matériel réel ou virtuel, sans recourir à Packet Tracer. Il s'agit de monter une infrastructure réseau avec plusieurs services (WWW, DNS, TFTP, DHCP, NTP, SYSLOG) et de segmenter le réseau en VLANs (Commercial, Technique, Administration). Les équipements utilisés incluent des switches, des routeurs et des machines virtuelles pour gérer les services.\


### Objectif de la SAE

L'objectif est de créer un réseau fonctionnel et sécurisé, en respectant les spécifications techniques et en mettant en place des règles de filtrage pour assurer la sécurité entre les VLANs et vers l'extérieur. Le projet permettra de démontrer les compétences en configuration réseau, gestion des services et sécurisation des communications au sein d'une entreprise.

### Réalisation

#### Segmentations Réseaux

Nous avons décidé de segmenter notre réseau, en deux parties distinctes physique avec deux firewall, donc avec une zone "non sécurisée" avec les serveurs (Web, Dns, Proxy, reverse Proxy et réseau invités) et une zone sécurisée qui contient notre Intranet. Pour L'intranet, nous avons choisi de différencier nos réseaux avec des réseaux différents et chacun se trouve dans un Vlan (sauf commercial et technicien qui peuvent être branchés sur le Firewall de droite en Vswitch).

#### Répartition des tâches

* **Kylian** : Proxy, Reverse Proxy, DNS, Web, règles de filtrage, configuration SSH, scripts d'automatisation, configuration du switch (VLANs), TFTP.
* **Sacha** : NTP, Rsyslog, DHCP, configuration du switch (VLANs), DHCP Relay, configuration SSH, scripts d'automatisation, règles de filtrage.

### Services

#### DNS

Le DNS (192.168.50.2) utilise bind9 et est utilisé pour toute l'infrastructure du réseau et permet de rediriger toutes les requêtes DNS (53) du réseau vers le DNS de l'IUT en 194.167.156.13. Cela permet un contrôle des requêtes DNS et de voir ce qui est transmis vers le DNS externe.

#### Proxy

Le Proxy utilise squid et permet de filtrer certains sites avec l'utilisation d'un fichier .txt où l'on peut répertorier tous les sites indésirables que l'on veut. Il peut aussi stocker le cache des sites fréquemment utilisé auparavant, permet de récupérer le cache transmis par le proxy de l'IUT (en 194.167.156.15).

#### Web

Le serveur Web utilise Apache2 et est relié au DNS avec un Reverse-Proxy intégré à celui-ci pour normalement permettre la sortie vers l'extérieur de l'infrastructure. Cependant, il y a un problème, car le nom de domaine qu'on utilise pour la page web est déjà utilisé depuis Internet. Il se peut que le problème soit de ne pas pouvoir donner de nom de domaine et donc utiliser son adresse IP directement…

#### DHCP

Le DHCP se trouve sur une machine Ubuntu (Raspberry), dans le Vlan serveur, nous avons utilisé un serveur DHCP, avec isc-dhcp-server. Celui-ci permet la gestion et l'attribution de l'adressage ip dans l'intranet (Technicien, Commercial, Vlan Admin…). Pour distribuer l'adressage ip dans l'intranet nous avons besoin d'un serveur dhcp relay (FW droite) qui permet la bonne distribution dans tous les réseaux. Le serveur DHCP envoie la configuration réseaux (Adresse IP, Masque réseau, Passerelle) ainsi que la configuration ntp et la configuration DNS.

#### NTP

Le serveur NTP se trouve sur la même machine que le serveur DHCP, le serveur NTP permet d'avoir une seule et même horloge pour notre réseau, ce qui permet de mettre à jour nos machines, ainsi qu'à la simplification pour les log si toutes nos machines sont réglées sur la même horloge. Le serveur NTP est créé avec le service NTPD et est diffusé à l'aide du serveur DHCP, et pour les machines qui n'arrivent pas à prendre la configuration NTP, on force la synchronisation à l'aide d'un script ntp.sh qui utilise le service ntpdate.

#### Rsyslog

Le Serveur Rsyslog se trouve sur une machine Ubuntu qui est positionnée dans le Vlan serveur, celui-ci rassemble tous les log de toutes les machines sur le réseau. Chacune des machines à un fichier personnel avec le nom de son Hostname afin de pouvoir analyser les logs plus rapidement. Les logs se trouvent dans le répertoire /var/log/remote/.

#### SSH

Nous pouvons nous connecter depuis l'ordinateur admin à n'importe quelle machine de notre réseau via SSH. Pour mettre en place rapidement et efficacement tous nos services nous avons utilisé des scripts que nous exécutions via SSH sur les machines de notre réseau.

#### Zyxel

Le Zyxel utilise le DHCP Relay qui utilise la plage d'adresse IP du serveur DHCP du Pfsense. Il récupère la configuration du serveur DHCP et donc laisse passer les différentes configurations automatiques faites par le Pfsense tel que le DNS de l'infra, le NTP, .... De plus, le Zyxel donne une connexion internet tout en passant par le Firewall DMZ.

### Problèmes Rencontrés et Résolutions

#### - Problème avec "apt update"

Lors de la configuration du proxy pour accéder au cache de l'IUT, une anomalie empêchait le bon fonctionnement des mises à jour via apt update. Le problème venait du fait que le proxy ne redirigeait pas exclusivement les requêtes vers le serveur de cache (cache.univ-pau.fr). Au lieu de cela, il tentait également de récupérer des paquets depuis d'autres serveurs, ce qui entraînait des échecs de téléchargement et empêchait la mise à jour correcte des paquets. Le Pfsense bloqué les sortie provenant de la DMZ.

#### Résolution "apt update"&#x20;

Pour corriger ce problème, il a été nécessaire de configurer le proxy de manière à forcer le passage unique par le serveur "cache.univ-pau.fr" (en 194.167.156.15). Cela a été réalisé en ajustant les paramètres du proxy dans le fichier "/etc/squid/squid.conf" afin qu'il ne cherche plus à contacter d'autres serveurs externes pour récupérer les paquets. Après cette correction, apt update a pu s'exécuter correctement, garantissant ainsi un accès fiable aux mises à jour des paquets via le cache de l'université.

#### - Problème NTP et Syslog

Le problème est survenu lorsque les équipements de la DMZ ne pouvaient pas envoyer leurs requêtes NTP et Syslog vers le VLAN Serveur. Le serveur NTP et Syslog étant situés dans le VLAN Serveur, les équipements de la DMZ ne parvenaient pas à les atteindre en raison de l'absence d'une route adéquate sur le PfSense de gauche. Cette absence de route empêchait le trafic NTP et Syslog de transiter correctement entre la DMZ et le VLAN Serveur.

#### Résolution NTP et Syslog

Pour résoudre ce problème, j'ai ajouté la route manquante sur le PfSense de gauche pour permettre au trafic NTP et Syslog provenant de la DMZ de rejoindre le VLAN Serveur. J'ai aussi vérifié les règles de pare-feu pour m'assurer que ces services étaient autorisés à passer entre la DMZ et le VLAN Serveur. Une fois la route correctement configurée, les équipements de la DMZ ont pu envoyer leurs requêtes NTP et Syslog vers les serveurs situés dans le VLAN Serveur.

#### - Problème de ports physiques

Nous avons rencontré un problème initialement lié au manque de ports physiques disponibles sur la machine pour connecter les VLANs Technicien et Commercial. Ce manque de ports à empêché la configuration correcte des VLANs.

#### Résolution des port physiques

Pour résoudre ce problème, nous avons créé des VSwitchs (commutateurs virtuels) sur le Pfsense de droite, permettant de simuler des interfaces réseau pour les VLANs Technicien et Commercial. Cela a permis de continuer la configuration sans nécessiter de ports physiques supplémentaires.

#### - Problème de connexion réseau

L'un de nos PC avait des problèmes de connexion avec le réseau de l'IUT et l'hyperviseur. Il ne parvenait pas à booter correctement, et après vérification, le port réseau était défectueux.

#### Résolution de connexion réseau

Après avoir contacté un administrateur réseau, nous avons constaté que le port était cassé. Ce problème nous a fait perdre beaucoup de temps sur le projet. En attendant une solution, nous avons limité le passage des gens près de la fenêtre, car ils marchaient sur le câble et coupaient la connexion.

#### - Problème de route externe

Les VMs ne pouvaient pas accéder à Internet malgré des règles de filtrage ouvertes. Le routeur externe redirigeait mal les requêtes, créant une boucle et bloquant la connexion. &#x20;

#### Résolution de route externe

En débranchant temporairement le routeur externe, la connexion a été rétablie via pfSense. Le problème venait probablement d'une mauvaise configuration des routes de sortie, nécessitant une correction de la table de routage. &#x20;

#### - Problème d'accès serveur web

Le serveur web était accessible uniquement depuis la DMZ et sans proxy. Avec le proxy activé, une erreur empêchait son fonctionnement, probablement due à un mauvais acheminement DNS et une mauvaise configuration du proxy. &#x20;

#### Résolution d'accès serveur web

Nous avons vérifié les enregistrements DNS et corrigé la configuration du proxy pour qu'il reconnaisse le serveur web. Après ces ajustements, l'accès via le proxy a été rétabli.

### Recherches non abouties

Nous avons exploré plusieurs pistes pour améliorer notre infrastructure, mais certaines recherches n'ont pas abouti. Nous avons tenté de mettre en place un reverse proxy avec BIND9 et Apache, ainsi qu'un serveur DNS avec Unbound, mais ces solutions se sont révélées trop complexes ou inadaptées à notre architecture. De même, nous avons étudié l'utilisation d'un routeur et du port forwarding sur PfSense sans parvenir à une configuration fonctionnelle. Côté supervision, nous avons envisagé LogAnalyzer, Grafana avec ElasticSearch, Loki et Graylog, mais le manque de temps nous a empêchés de finaliser leur intégration. Enfin, nous avons cherché à configurer le NAT sur le Zyxel et Chrony pour la synchronisation NTP, sans succès.

### Organisation

Nous avons principalement travaillé séparément sur le déploiement des services. Kylian s'occupait de l'extranet avec le pfSense de gauche, tandis que Sacha gérait l'intranet avec le pfSense de droite. Cependant, en début de projet, nous avons collaboré sur les branchements et la configuration des switches. Une fois les services déployés, nous avons finalisé ensemble les règles de filtrage pour terminer le projet.

### Réflexions sur la SAE

Cette SAE nous a permis d'approfondir notre compréhension des règles de filtrage, du fonctionnement d'un proxy et d'un reverse proxy, ainsi que du DHCP Relay et des serveurs NTP. C'était un projet concret qui nous a aidés à mieux saisir non seulement ce que nous faisions, mais aussi pourquoi nous le faisions. Nous avons dû effectuer des recherches autonomes sur la mise en place et le fonctionnement de divers services.L'architecture réseau a évolué au fil du projet en fonction des contraintes techniques et de nos choix. Certains services ont été abandonnés, tandis que d'autres ont nécessité des ajustements pour assurer une meilleure communication entre les VLANs. Cette adaptation nous a permis de mieux appréhender le déploiement d'une infrastructure réseau.
