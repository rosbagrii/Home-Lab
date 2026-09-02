### ⚙️ Zentraler DHCP- und Routing-Fluss

Dieser Abschnitt beschreibt das Routing im Netzwerk. Ein zentraler Windows Domain Controller stellt die DHCP-Dienste bereit, während der Linux-Router über VLAN-Grenzen hinweg als Vermittler (DHCP-Relay) agiert, um Broadcasts gerichtet weiterzuleiten.


### DHCP-Relay (Linux-Router)

Da DHCP-Anfragen (Broadcasts) standardmäßig nicht über VLAN-Grenzen hinweg transportiert werden, fungiert der Linux-Router auf Hyper-V-Ebene als **DHCP-Relay (Transmitter)**. 

* Er fängt die DHCP-Broadcasts aus den Netzen (z. B. von VLAN 20 für den Client) ab.
* Er leitet diese als Unicast-Paket gezielt an den Domain Controller unter `10.0.10.2` weiter.
* Der DC weist dem Gerät daraufhin eine passende IP-Adresse aus dem korrekten Subnetz zu.


### Einrichtung des Linux-Routers

#### 1. IP-Konfiguration & VLANs (Netplan)

Die statischen IP-Adressen und VLANs sind in der Netplan-Konfigurationsdatei (z. B. `/etc/netplan/00-installer-config.yaml`) definiert:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
  vlans:
    eth0.10:
      id: 10
      link: eth0
      addresses:
        - 10.0.10.1/24
      nameservers:
        addresses:
          - 10.0.10.2
      optional: true
    eth0.20:
      id: 20
      link: eth0
      addresses:
        - 10.0.20.1/24
      optional: true
    eth0.30:
      id: 30
      link: eth0
      addresses:
        - 10.0.30.1/24
      optional: true
    eth0.40:
      id: 40
      link: eth0
      addresses:
        - 10.0.40.1/24
      optional: true
```

Konfiguration anwenden:
```bash
sudo netplan apply
```


#### 2. IP-Forwarding (Routing aktivieren)

Aktiviert die permanente Paketweiterleitung zwischen den VLAN-Schnittstellen.

1. Zeile in Datei `/etc/sysctl.conf` aktivieren/hinzufügen:
   ```ini
   net.ipv4.ip_forward=1
   ```
2. Konfiguration sofort ohne Neustart laden:
   ```bash
   sudo sysctl -p
   ```


#### 3. DHCP-Relay installieren & konfigurieren

1. Relay-Dienst installieren:
   ```bash
   sudo apt update && sudo apt install isc-dhcp-relay -y
   ```
2. Konfiguration in Datei `/etc/default/isc-dhcp-relay` anpassen:
   ```ini
   SERVERS="10.0.10.2"
   INTERFACES="eth0.10 eth0.20 eth0.30 eth0.40"
   OPTIONS=""
   ```
3. Dienst starten und für den automatischen Systemstart aktivieren:
   ```bash
   sudo systemctl restart isc-dhcp-relay
   sudo systemctl enable isc-dhcp-relay
   ```
