---
description: >-
  Ce script utilise python avec scapy comme librairie et sert à analyser le
  réseau souhaiter pour permettre de faire ressortir les machines actives de ce
  réseau en particulier.
---

# Mapping d'adresse IP

Ce programme doit obligatoirement être exécuté en tant qu'administrateur car il envoie des paquets BRUT directement vers le réseau et donc il doit y avoir les droits d'administrateur pour le faire.

```python
from scapy.all import *
import os

# Désactiver les avertissements de Scapy
#conf.verb = 0

# Ajouter une route statique pour éviter les avertissements de diffusion
#conf.route.add(net="192.168.1.0/24", gw="192.168.1.1")

def scan_reseaux(prefix_network):
     # Vérification des privilèges
    if os.geteuid() != 0:    # sur windows -> os.getpid()
        print("Ce script doit être exécuté en tant qu'administrateur (sudo)")
        #sys.exit(1)
    
    print(f"Scan du réseau {prefix_network}.0/24")
    machines_actives = 0

    for i in range(1, 255):
        IP_src = "192.168.1.54"
        IP_dst = f"{prefix_network}.{i}"
        ping = IP(src=IP_src, dst=IP_dst) / ICMP()
        #ping.show()
        try:
            # Utilisation de sr() au lieu de srp() car pas besoin de couche Ethernet
            rep, non_rep = sr(ping, timeout=1, verbose=False)
            #print(f"paquet envoyé a l'adresse : {IP_dst}")
            #print(rep)
            if rep:
                machines_actives += 1
                print(f"[+] Machine active : {IP_dst}")
        except Exception as e:
            print(f"Erreur lors du scan de {IP_dst}: {e}")

    print(f"\nScan terminé. {machines_actives} machines actives trouvées.")
    
if __name__ == "__main__":
    scan_reseaux("192.168.1")


```
