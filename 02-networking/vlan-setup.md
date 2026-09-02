### 🌐 Netzwerksegmentierung & VLAN-Infrastruktur

Dieses Dokument beschreibt die logische Aufteilung des isolierten "Skynet-Netzes" in verschiedene Subnetze und VLANs. Durch diese Mikrosegmentierung wird sichergestellt, dass die einzelnen Systemrollen (Arbeitsplätze, Server, Datenbanken) strikt voneinander getrennt sind.


### 🗺️ IP- und VLAN-Adressplanung

Als Basis wird der private IP-Bereich `10.0.0.0/16` verwendet. Jedes Segment besitzt einen eigenen IP-Bereich und eine zugewiesene VLAN-ID. Das Standard-Gateway für alle Segmente ist der Linux-Router auf der IP-Adresse `.1`.


| Segment / Rolle | VLAN-ID | Subnetz | Gateway-IP (Router) | Statische IP / Host |
| :--- | :---: | :--- | :---: | :--- |
| **Domain Controller (DC)** | **10** | `10.0.10.0/24` | `10.0.10.1` | `10.0.10.2` |
| **Domain Client** | **20** | `10.0.20.0/24` | `10.0.20.1` | `10.0.20.2` (via DHCP) |
| **Webserver** | **30** | `10.0.30.0/24` | `10.0.30.1` | `10.0.30.2` |
| **MySQL-Datenbank** | **40** | `10.0.40.0/24` | `10.0.40.1` | `10.0.40.2` |


*Hinweis: Der Hyper-V-Host sowie die Proxmox-Zentralinstanz selbst besitzen im "Skynet-Netz" kein explizites VLAN-Tagging auf Interface-Ebene, da das Routing und die Segmentierung rein über den virtuellen Linux-Router gesteuert werden.*