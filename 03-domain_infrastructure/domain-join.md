#### Domänenaufnahme & Client-Verwaltung
Ein repräsentativer Client aus VLAN 20 wurde erfolgreich in die Domäne integriert, um die zentrale Verwaltung zu validieren.


* **Voraussetzung:** Der Client bezieht seine Netzwerkkonfiguration (IP, Gateway, DNS auf `10.0.10.2`) automatisch über den DHCP-Bereich von VLAN 20.

* **Durchführung (Domänenbeitritt):** In den Windows-Systemeinstellungen wurde die Arbeitsgruppe verlassen und der Beitritt zur Domäne `skynet.local` unter Angabe von Domänen-Admin-Anmeldedaten durchgeführt.

* **Durchführung (Benutzerverwaltung & Anmeldung):** 
  * Auf dem Domänencontroller wurde über die Konsole *Active Directory-Benutzer und -Computer* (`dsa.msc`) ein **neuer Domänen-Benutzer** (z. B. im Container `Users` oder einer dedizierten Organisationseinheit) mit individuellen Anmeldedaten und Kennwort angelegt.
  * Nach dem fälligen Neustart des Client-PCs erfolgte die **Erst-Anmeldung des Benutzers** am System unter Verwendung des neuen Domänen-Kontos (`benutzername@skynet.local`).
  
* **Ergebnis:** Das Computerkonto wird nun im Active Directory im Standard-Container `Computers` gelistet, während das Benutzerkonto zentral im AD verwaltet wird. Der Client ist damit vollständig in die Sicherheits- und Berechtigungsstruktur der Domäne integriert.
