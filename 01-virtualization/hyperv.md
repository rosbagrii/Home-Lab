### 💻 Hyper-V: Unternehmensumgebung (Büro-Simulation)

Dieses Dokument beschreibt die Konfiguration und Rolle der Hyper-V-Umgebung. Im Gesamtszenario simuliert der Hyper-V-Host die physischen Arbeitsplätze und die Infrastruktur der Unternehmenszentrale (Büro-Umgebung). 

### 🏗️ Architektur & VM-Rollen

Auf diesem Host laufen die virtuellen Maschinen, die typische Endgeräte und Netzwerkknoten eines Firmenstandorts abbilden: 

* **Linux-Router & DHCP-Relay:** Steuert den Netzwerkverkehr der simulierten Büro-Clients und leitet DHCP-Anfragen segmentübergreifend weiter.
* **Domain Client:** Ein emulierter Mitarbeiter-Arbeitsplatz (Windows/Linux), der als Mitglied fest in die Active-Directory-Domäne integriert ist.
* **Proxmox VE (Nested Virtualization):** Eine virtuelle Instanz von Proxmox, die innerhalb von Hyper-V läuft. Sie simuliert den logisch getrennten, physischen Serverraum des Unternehmens.

### 📊 Ressourceneinteilung (Sizing)
*Hinweis: Die VMs nutzen dynamische VHDX-Festplatten im isolierten "Skynet-Netz".*

| VM / Rolle | Betriebssystem | vCPU | RAM | Speicher |

| **Client** (Mitarbeiter-PC) | Windows 11 | 2 | 8 GB | 60 GB |
| **DHCP-Relay** (Linux-Router) | Linux (Ubuntu/Debian) | 1 | 2 GB | 10 GB |
| **Proxmox** (Serverraum-Host) | Proxmox VE | 4 | 16 GB | 60 GB |

### ⚙️ Technische Kernkonfigurationen

### 1. Nested Virtualization (Proxmox-Unterstützung)

Damit Proxmox als VM innerhalb von Hyper-V eigene VMs starten kann, wurde die geschachtelte Virtualisierung (Nested Virtualization) für diese spezifische VM per PowerShell aktiviert: 

Set-VMProcessor -VMName "Proxmox" -ExposeVirtualizationExtensions $true

### 2. Netzwerk-Anbindung

* **Interner/Externer Switch:** Die gesamte Umgebung ist über einen internen Hyper-V-Switch namens Skynet-Netz komplett von der Außenwelt und dem Heimnetzwerk isoliert.
* **MAC-Address Spoofing:** In den Einstellungen der Proxmox-VM unter Netzwerkkarte -> Erweiterte Features aktiviert. Ohne diese Option blockiert Hyper-V den Datenverkehr (insbesondere DHCP-Anfragen) der inneren (verschachtelten) VMs, da diese mit eigenen, für Hyper-V "fremden" MAC-Adressen kommunizieren.