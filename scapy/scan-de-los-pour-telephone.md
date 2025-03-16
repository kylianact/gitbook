# Scan de l'OS pour téléphone

```python
from scapy.all import *
import re

def detect_devices():
    def dhcp_packet_callback(packet):
        if DHCP in packet and packet[DHCP].options[0][1] == 1:  # DHCP Discover
            mac_address = packet[Ether].src
            vendor = get_vendor_by_mac(mac_address)
            print(f"Detected device: MAC={mac_address}, Vendor={vendor}")

    def http_packet_callback(packet):
        if packet.haslayer(HTTP):
            http_layer = packet.getlayer(HTTP)
            if http_layer.fields.get('User-Agent'):
                user_agent = http_layer.fields['User-Agent'].decode('utf-8')
                os = identify_os_by_user_agent(user_agent)
                print(f"Detected device: User-Agent={user_agent}, OS={os}")

    def get_vendor_by_mac(mac):
        mac_prefix = mac[:8].upper()
        if mac_prefix.startswith("00:1A:11"):
            return "Apple"
        elif mac_prefix.startswith("00:1B:22"):
            return "Samsung"
        else:
            return "Unknown"
        
        # Cette fonction fait un mapping d'@ MAC avec le prefix vendor
        # Pour plus de simplicité, nous utilisons un exemple codé en dur directement
        # Nous pouvons rajouter une BDD de prefix de vendor pour permettre d'etre plus precis dans le scan, 
        # L'ajout d'une BDD fait pousser l'entrainemet bien plus loin.


    def identify_os_by_user_agent(user_agent):
        if "iPhone" in user_agent or "iPad" in user_agent:
            return "iOS"
        elif "Android" in user_agent:
            return "Android"
        else:
            return "Unknown"

    print("Listening for DHCP and HTTP packets...")
    sniff(filter="udp and (port 67 or 68) or tcp port 80", prn=lambda x: dhcp_packet_callback(x) if DHCP in x else http_packet_callback(x), store=0)

if __name__ == "__main__":
    detect_devices()


```
