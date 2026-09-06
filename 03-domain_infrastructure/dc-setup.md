# 📂 Bereitstellung Domain Controller (Active Directory, DNS, DHCP & GPOs)

Dieses Dokument beschreibt die Installation, Konfiguration und Bereitstellung eines zentralen Windows Domain Controllers (DC) innerhalb der isolierten "Skynet"-Netzwerkinfrastruktur. Die Instanz läuft virtuell auf einer verschachtelten (Nested) Virtualisierungsebene.


### 🗺️ Netzwerk-Eckdaten (VLAN 10)

Der Domain Controller befindet sich im dedizierten Management- und Server-VLAN (VLAN 10) und besitzt eine statische IP-Konfiguration.

* **IP-Adresse:** `10.0.10.2`
* **Subnetzmaske:** `255.255.255.0`
* **Standard-Gateway:** `10.0.10.1` (Virtueller Linux-Router)
* **DNS-Server (Loopback):** `127.0.0.1` (bzw. `10.0.10.2`)


### ⚙️ Rollen- und Dienstekonfiguration

#### 1. Active Directory Domain Services (AD DS)
* **Domänenname (FQDN):** `skynet.local`
* **Funktionsebene:** Windows Server 2025
* Die Installation erfolgte über den Server-Manager via *Rollen und Features hinzufügen* und anschließender Heraufstufung zum Domänencontroller (Neuer Gesamtstruktur-Stamm).

#### 2. DNS-Server (Domain Name System)
* Die DNS-Rolle wurde automatisch während der AD DS-Heraufstufung installiert und im Active Directory integriert.
* **Forward-Lookupzone:** Verwaltet die Namensauflösung für alle internen Hosts der Domäne.
* **Reverse-Lookupzone:** Manuell angelegt für das Subnetz `10.0.10.0/24`, um IP-Adressen wieder in Hostnamen aufzulösen.

#### 3. DHCP-Server (Dynamic Host Configuration Protocol)
Der DC übernimmt die zentrale Adressvergabe für unmanaged Endgeräte im Netzwerk. Auf dem Server wurden folgende **DHCP-Bereiche (Scopes)** definiert:

* **Bereich VLAN 20 (Clients):**
  * Start-IP: `10.0.20.10` | End-IP: `10.0.20.100`
  * Bereichsoption **003 (Router/Gateway):** `10.0.20.1` *(Manuell angepasst auf das VLAN 20 Gateway des Linux-Routers!)*
  * Bereichsoption **006 (DNS-Server):** `10.0.10.2` (Verweis auf den DC)

*Die Autorisierung des DHCP-Servers erfolgte im Active Directory, um den Dienst im Netzwerk freizuschalten.*
