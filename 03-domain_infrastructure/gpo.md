### Zentrale Client-Konfiguration via Gruppenrichtlinien (GPOs)

Um die Sicherheit im Netzwerk der Domäne `skynet.local` zu maximieren und Compliance-Standards umzusetzen, wurden drei wesentliche, unternehmensübliche Gruppenrichtlinien implementiert.


#### 1. Kennwortrichtlinie (Password Policy)
Standardmäßig sind Windows-Kennwörter anfällig für Angriffe. Diese Richtlinie erzwingt minimale Sicherheitsstandards für alle Domänen-Benutzerkonten.

* **Pfad im Editor:** `Computerkonfiguration` ➔ `Richtlinien` ➔ `Windows-Einstellungen` ➔ `Sicherheitseinstellungen` ➔ `Kontorichtlinien` ➔ `Kennwortrichtlinie`
* **Konfiguration:**
  * **Minimale Kennwortlänge:** 8 Zeichen
  * **Kennwort muss Komplexitätsvoraussetzungen entsprechen:** Aktiviert (Groß-/Kleinschreibung, Zahlen, Sonderzeichen)
  * **Maximales Kennwortalter:** 90 Tage
* **Ziel:** Schutz der Identitäten vor Brute-Force- und Dictionary-Angriffen. 
* *Hinweis:* Muss auf die *Default Domain Policy* angewendet oder direkt mit der Domänenwurzel verknüpft werden.


#### 2. Sperrung von USB-Wechselmedien (Removable Storage Block)
Zum Schutz vor dem unbefugten Abfluss von Unternehmensdaten (Datendiebstahl) und dem Einschleusen von Schadsoftware (Malware) wird der Zugriff auf USB-Speichergeräte blockiert.

* **Pfad im Editor:** `Computerkonfiguration` ➔ `Richtlinien` ➔ `Administrative Vorlagen` ➔ `System` ➔ `Wechselmedienzugriff`
* **Konfiguration:**
  * **Wechselmedien: Alle Klassen den Zugriff verweigern:** Aktiviert
* **Ziel:** Erhöhung der Endpunkt-Sicherheit. Eingabegeräte (Maus/Tastatur) bleiben funktionsfähig, während USB-Sticks und externe Festplatten mit der Meldung "Zugriff verweigert" blockiert werden.


#### 3. Zentrale BitLocker-Verschlüsselung (BitLocker Drive Encryption)
Um Daten bei Diebstahl oder Verlust eines Endgeräts (z. B. Notebooks) vor unbefugtem physischen Auslesen zu schützen, wird die Festplattenverschlüsselung zentral gesteuert.

* **Pfad im Editor:** `Computerkonfiguration` ➔ `Richtlinien` ➔ `Administrative Vorlagen` ➔ `Windows-Komponenten` ➔ `BitLocker-Laufwerksverschlüsselung` ➔ `Betriebssystemlaufwerke`

* **Konfiguration:**

  * **Sichern von BitLocker-Wiederherstellungsinformationen in Active Directory-Domänendiensten auswählen:** **Aktiviert** (Hinterlegt die Struktur für die AD-Sicherung im Verzeichnisdienst).

  * **Festlegen, wie BitLocker-geschützte Betriebssystemlaufwerke wiederhergestellt werden können:** **Aktiviert** (Erzwingt aktiv die automatische Sicherung des 48-stelligen Wiederherstellungskennworts in den AD-DS, bevor die Verschlüsselung auf dem Client starten darf).

* **Ziel:** Lückenlose Verschlüsselung der Systemmedien bei gleichzeitiger zentraler Absicherung gegen Datenverlust. Geht ein Gerät verloren, sind die Daten geschützt. Administratoren können den Wiederherstellungsschlüssel im Ernstfall direkt im Active Directory-Computerobjekt auslesen.


#### Validierung auf dem Client
Die Zuweisung der Richtlinien 2 und 3 erfolgt dediziert über die Organisationseinheit (OU) `Clients`. Die sofortige Anwendung der GPOs auf dem Windows-Client wird über die administrative Eingabeaufforderung erzwungen:

```cmd
gpupdate /force
```

#### Aktivierung über den Windows-Assistenten

Da die GPO die Verschlüsselung nicht automatisch startet, wird der Prozess manuell am Client angestoßen:

1. **Als Admin anmelden:** Am Client (`CLIENT01`) mit administrativen Rechten anmelden.
2. **BitLocker öffnen:** In der Systemsteuerung unter *BitLocker-Laufwerksverschlüsselung* auf **„BitLocker aktivieren“** klicken.
3. **Assistenten durchlaufen:**
   * **Startmethode:** Da die VM ein (virtuelles) TPM nutzt, wird die Festplatte automatisch über das TPM geschützt. Es muss **kein** Startpasswort eingegeben oder festgelegt werden.
   * **Modus & Umfang:** Den **„Neuen Verschlüsselungsmodus“** (XTS-AES) und **„Nur verwendeten Speicherplatz verschlüsseln“** wählen.
4. **Neustart:** Den Assistenten abschließen und den PC neu starten. Der PC fährt ohne Passwortabfrage direkt bis zum Windows-Sperrbildschirm hoch, während der Schlüssel an das AD übertragen wird.


