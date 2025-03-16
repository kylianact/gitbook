---
description: >-
  Ce script utilise la librairie scapy, elle permet de scanner tout les ports
  non éphémère d'une machine pour voir s'il utilise un protocole en particulier
  (TCP, UDP, ICMP).
---

# Scan des ports

```python
from scapy.all import *
import time

def scan_port(ip, port):
    paquets = IP(dst=ip) / TCP(dport=port)
    #paquets.show()

    # Visualise les paquets TCP emis et recus :   
    rep, un_rep = sr(paquets, timeout=1, verbose=False)
    if rep:
        for sent, received in rep:
            if IP in received:
                # Analyse des informations IP
                ip_src = received[IP].src
                ip_dst = received[IP].dst
                    
                # Analyse du protocole
                protocol = "Inconnu"
                port_info = ""
                    
                if TCP in received:
                    protocol = "TCP"
                    port_info = f"Port src: {received[TCP].sport}, Port dst: {received[TCP].dport}"
                elif UDP in received:
                    protocol = "UDP"
                    port_info = f"Port src: {received[UDP].sport}, Port dst: {received[UDP].dport}"
                elif ICMP in received:
                    protocol = "ICMP"
                
                # Affichage formaté
                print(f"{'='*60}")
                print(f"Temps: {time.strftime('%H:%M:%S')}")
                print(f"Protocole: {protocol}")
                print(f"Source: {ip_src} -> Destination: {ip_dst}")
                if port_info:
                    print(f"Ports: {port_info}")
                print(f"Taille du paquet: {len(received)} bytes")
                print(f"{'='*60}\n")

def scan_ports(ip_visé):
    print(f"Scan des ports de {ip_visé}")
    for port in range(1, 1024):
        scan_port(ip_visé, port)
    print("scan terminé !!!")

if __name__ == "__main__":
    scan_ports("192.168.1.1")  # Utilisez l'adresse IP cible ici

```
