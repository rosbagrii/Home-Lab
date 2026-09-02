### 🛡️ Firewall-Konfiguration (UFW auf dem Router)

Um die Netzwerke sauber zu isolieren, wird auf dem Linux-Router die Firewall `ufw` nach dem Prinzip **"Default Deny"** eingerichtet. Jeglicher Datenverkehr zwischen den VLANs wird blockiert, außer er wird explizit erlaubt.


#### 1. Standard-Verhalten auf Blockieren setzen
```bash
# Standardmäßig allen eingehenden und Weiterleitungs-Verkehr verbieten
sudo ufw default deny incoming
sudo ufw default deny forward
sudo ufw default allow outgoing
```

#### 2. DHCP-Relay Ports freigeben
Damit die Clients IPs vom DC beziehen können, müssen die DHCP-Ports (67/68) auf den Router-Schnittstellen offen sein.
```bash
# DHCP-Anfragen an den Router erlauben
sudo ufw allow in on eth0.20 to any port 67 proto udp
```

#### 3. Weiterleitungs-Regeln (Routing-Filter)
UFW filtert den Verkehr *zwischen* den Schnittstellen über die Datei `/etc/ufw/before.rules`. 

Am Ende der Datei `/etc/ufw/before.rules` (vor dem finalen `COMMIT`) werden folgende Regeln eingefügt:

```text
# --- Skynet VLAN Routing Regeln ---
-A ufw-before-forward -i eth0.10 -j ACCEPT
-A ufw-before-forward -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A ufw-before-forward -i eth0.20 -o eth0.10 -p udp --dport 67 -d 10.0.10.2 -j ACCEPT
-A ufw-before-forward -i eth0.20 -o eth0.30 -p tcp --dport 80 -j ACCEPT
-A ufw-before-forward -i eth0.20 -o eth0.30 -p tcp --dport 443 -j ACCEPT
-A ufw-before-forward -i eth0.30 -o eth0.40 -p tcp --dport 3306 -j ACCEPT
```
*(Bedeutung: VLAN 10 darf alles | Erlaubt Antworten auf bestehende Verbindungen | Weiterleitung von DHCP-Anfragen | Clients dürfen auf Webserver HTTP/S | Webserver darf auf MySQL-Datenbank)*

#### 4. Firewall aktivieren
```bash
sudo ufw enable
```