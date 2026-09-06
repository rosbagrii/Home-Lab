### 🔮 Proxmox VE: Virtueller Serverraum

Dieses Dokument beschreibt die Konfiguration und Rolle der Proxmox-Umgebung. Innerhalb des Gesamtszenarios wird diese Instanz als *Nested VM* unter Hyper-V betrieben und simuliert den logisch sowie physisch getrennten **Serverraum des Unternehmens**. 

### 🏗️ Architektur & Server-Rollen

Im virtuellen Serverraum laufen die zentralen Enterprise-Dienste und Server-Infrastrukturen der simulierten Domäne: 

* **Domain Controller (DC):** Das Herzstück der Identitätsverwaltung. Stellt die zentralen Kerndienste **Active Directory (AD)**, **DNS** und **DHCP** für das gesamte Unternehmensnetzwerk bereit.
* **Webserver:** Hostet die internen Web-Projekte und modularen Applikationen der simulierten Firmenumgebung.
* **MySQL-Datenbank:** Bietet die relationale Datenhaltung und stellt der Web-Applikation (Webserver) die benötigten Daten bereit.

### 📊 Ressourceneinteilung (Sizing)

| Server / VM | Betriebssystem | vCPU | RAM | Speicher |
| :--- | :--- | :---: | :---: | :---: |
| **Domain Controller (DC)** | Windows Server 2025 | 2 | 8 GB | 60 GB |
| **Webserver** | Linux (Ubuntu/Debian) | 1 | 1 GB | 15 GB |
| **MySQL-Database** | Linux (Ubuntu/Debian) | 1 | 1 GB | 15 GB |

### ⚙️ Netzwerk-Integration (vmbr0)

Die virtuelle Bridge von Proxmox greift direkt auf die durch Hyper-V bereitgestellte Netzwerkschnittstelle im **"Skynet-Netz"** zu. Dank des aktivierten MAC-Spoofings auf der übergeordneten Hyper-V-Ebene können alle hier dokumentierten Server problemlos eigene IP-Adressen beziehen und untereinander sowie mit den Büro-Clients kommunizieren.

### ⚠️ Lessons Learned & Stolpersteine

### Windows Server Installation & VirtIO-Treiber

Bei der Bereitstellung des Windows Domain Controllers erkennt das Windows-Setup standardmäßig die schnellen, virtualisierten Proxmox-Festplatten (SCSI) und Netzwerkkarten nicht. 

* **Lösung:** Einbinden der aktuellen virtio-win.iso als zweites CD-Laufwerk in den VM-Hardware-Einstellungen. Während der Windows-Installation wurden die Treiber manuell nachgeladen.