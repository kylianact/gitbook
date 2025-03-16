---
description: >-
  Ce script utilise la librairie scapy, elle permet de scanner l'OS que la
  machine cible utilise (Linux, Windows, Cisco Router,...)
---

# Scan de l'OS pour PC

```python
from scapy.all import *

def identify_os(ip):
    # Send ICMP echo request
    ping = IP(dst=ip)/ICMP()
    response, un_response = sr(ping, timeout=2, verbose=False)
    if response:
        rep = response[0][1]
        print(rep.show())
        ttl = rep.ttl
        window_size = rep.window if hasattr(response, 'window') else None
        
        # Basic OS fingerprinting based on TTL and window size
        if ttl == 64:
            if window_size == 5840:
                return "Linux"
            elif window_size == 65535:
                return "FreeBSD"
        elif ttl == 128:
            if window_size == 8192:
                return "Windows"
        elif ttl == 255:
            return "Cisco Router"
        
        return "Unknown OS"
    else:
        return "No response"

if __name__ == "__main__":
    ip_address = input("Enter the IP address to scan: ")
    os = identify_os(ip_address)
    print(f"The operating system is likely: {os}")
```

Bien sur, les analyseurs d'OS de nos jours peuvent et doivent être bien plus pousser que cela mais voici déjà le principe de base de comment reconnaitre les signatures d'un OS en particulier.
