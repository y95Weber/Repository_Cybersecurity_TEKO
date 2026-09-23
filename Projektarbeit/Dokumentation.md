# Projektarbeit
## Penetration Testing mit Metasploit

**Autor:** Yves Weber
**Modul:** Cybersecurity
**Institution:** TEKO Bern
**Dozent:** Christian Locher
**Datum Abgabe:** 23.09.2026

---
## 1. Einleitung
 
### Motivation
Cyberangriffe auf Unternehmensnetzwerke nehmen stetig zu. Penetration Testing ist eine zentrale Methode, um Schwachstellen proaktiv zu identifizieren, bevor sie von Angreifern ausgenutzt werden können. Diese Arbeit befasst sich mit dem Metasploit Framework als einem der wichtigsten Werkzeuge im Bereich der Netzwerk- und Systemausnutzung (Exploitation).
 
### Zielsetzung
Ziel dieser Arbeit ist es,
- den grundsätzlichen Ablauf eines Penetrationstests darzustellen,
- die dabei eingesetzten Tools den jeweiligen Phasen zuzuordnen,
- das Metasploit Framework im Detail zu erläutern (Aufbau, Funktionsweise, Einsatzzweck),
- und die Anwendung anhand einer praktischen Laborumgebung zu demonstrieren.
### Abgrenzung
Der Fokus liegt bewusst auf Metasploit als Exploitation-Framework für netzwerkbasierte Dienste. Aus diesem Grund wurde **Metasploitable3** als Zielsystem gewählt (nicht OWASP Juice Shop, da dieses eher auf Web-App-Logikfehler ausgelegt ist, wofür Tools wie Burp Suite besser geeignet sind). Ebenfalls nicht Teil dieser Arbeit ist die Integration eines SIEM-Systems (z. B. Wazuh), um den Scope auf Metasploit fokussiert zu halten.
 
---
 
## 2. Grundlagen: Penetration Testing
 
### 2.1 Definition
Ein Penetrationstest ist ein autorisierter, simulierter Angriff auf ein IT-System mit dem Ziel, Sicherheitslücken zu identifizieren und deren Ausnutzbarkeit zu bewerten.
 
### 2.2 Phasenmodell
Ein typischer Pentest lässt sich in folgende Phasen gliedern:
 
1. **Reconnaissance (Aufklärung):** Sammeln von Informationen über das Zielsystem
2. **Scanning / Enumeration:** Identifikation offener Ports, Dienste, Versionen
3. **Exploitation:** Ausnutzung gefundener Schwachstellen zur Erlangung von Zugriff
4. **Post-Exploitation / Privilege Escalation:** Ausweitung der Rechte, Absicherung des Zugriffs
5. **Reporting:** Dokumentation der Ergebnisse und Handlungsempfehlungen
### 2.3 Tools je Phase
 
| Phase                | Tools                                               |
|----------------------|-----------------------------------------------------|
| Reconnaissance       | Nmap, OSINT                                         |
| Scanning/Enumeration | Nmap, Gobuster/Dirbuster                            |
| Exploitation         | Metasploit, Burp Suite                              |
| Post-Exploitation    | Metasploit (Meterpreter), Mimikatz, LinPEAS/WinPEAS |
| Passwort-Angriffe    | John the Ripper, Hashcat                            |
| Traffic-Analyse      | Wireshark                                           |
| WLAN                 | Aircrack-ng                                         |
 
> Hinweis: Die Wahl des Tools hängt stark vom Zieltyp ab.
 
---
 
## 3. Das Metasploit Framework
 
### 3.1 Was ist Metasploit?
Metasploit ist ein Open-Source-Framework für die Entwicklung, das Testen und die Ausführung von Exploits gegen Zielsysteme. Es wurde ursprünglich 2003 von H. D. Moore entwickelt und wird seit 2009 von Rapid7 weitergeführt.
 
Ursprünglich war Metasploit in **Perl** geschrieben, wurde 2007 mit Version 3.0 komplett in **Ruby** neu implementiert und ist bis heute die Basis des Frameworks. Seitdem hat es sich von einer reinen Exploit-Sammlung zu einem vollständigen Framework mit Datenbank-Anbindung, Meterpreter, umfangreicher Post-Exploitation mit Automatisierungsfunktionalität sowie einer kommerziellen Pro-Version entwickelt.
 
### 3.2 Einsatzzweck
- Simulation realer Angriffe im Rahmen autorisierter Pentests
- Validierung von Schwachstellen (Proof of Concept)
- Schulung und Sicherheitsforschung
- Community Edition (kostenlos, CLI-basiert) vs. Metasploit Pro (kommerziell, GUI, Automatisierung)

### 3.3 Architektur und Funktionsweise
 
#### 3.3.1 Modul-Typen
- **Exploits:** Code zur Ausnutzung einer spezifischen Schwachstelle
- **Payloads:** Code, der nach erfolgreichem Exploit auf dem Zielsystem ausgeführt wird
- **Auxiliary:** Hilfsmodule (z. B. Scanner, Fuzzer, DoS)
- **Post:** Module für die Post-Exploitation-Phase
- **Encoders:** Verschleierung von Payloads (z. B. zur AV-Umgehung)
- **NOPs:** Sind wirkungslose Füllbefehle, die vor codierten Payloads eingesetzt werden, damit der Decoder trotz leichter Abweichungen im Einstiegspunkt zuverlässig ausgeführt wird. Zudem sorgen sie für eine konsistente Payload Grösse.


#### 3.3.2 msfconsole
Zentrale Konsole zur Interaktion mit dem Framework. Anbindung an eine Datenbank (PostgreSQL) zur Verwaltung von Workspaces, Hosts und Ergebnissen.

**Die wichtigsten Befehle:**

| Befehl                    | Beschreibung                                                         |
|---------------------------|-----------------------------------------------------------------------|
| `help` / `?`              | Zeigt alle verfügbaren Befehle mit Beschreibung an                    |
| `show all`                | Listet alle verfügbaren Module auf (Exploits, Payloads, Encoders etc.)|
| `show exploits`           | Listet alle Exploit-Module auf                                        |
| `show payloads`           | Listet alle Payload-Module auf                                        |
| `show encoders`           | Listet alle Encoder-Module auf                                        |
| `show options`            | Zeigt die konfigurierbaren Optionen des aktuell geladenen Moduls an   |
| `search <keyword>`        | Sucht Module nach Name, CVE oder Stichwort                            |
| `use <module>`            | Lädt ein bestimmtes Modul in die Konsole                              |
| `set <option> <value>`    | Setzt eine Option für das aktuelle Modul                              |
| `setg <option> <value>`   | Setzt eine globale Variable (bleibt auch bei Modulwechsel erhalten)   |
| `unset <option>`          | Entfernt eine zuvor gesetzte Option                                   |
| `run` / `execute`         | Führt das aktuelle Modul aus                                          |
| `sessions`                | Listet aktive Sessions auf                                            |
| `route`                   | Leitet Traffic über eine Session (für Pivoting)                       |
| `history`                 | Zeigt den Befehlsverlauf an                                           |
| `version`                 | Zeigt Framework- und Library-Versionsnummern an                       |
 


#### 3.3.3 msfvenom
Ist Metasploits eigenständiges Kommandozeilen-Tool zur Generierung und Kodierung von Payloads außerhalb der msfconsole.
 
#### 3.3.4 Payload-Konzepte
- **Staged vs. Stageless:** gestufte Übertragung des Payloads vs. vollständiger Payload in einem Schritt
- **Bind vs. Reverse Shell:** Zielsystem öffnet Port (bind) vs. Zielsystem baut Verbindung zum Angreifer auf (reverse); Reverse Shells sind in der Praxis meist vorzuziehen (Firewall/NAT-Umgehung)
#### 3.3.5 Meterpreter
Fortgeschrittener, im RAM laufender Payload mit erweiterten Funktionen (Dateisystemzugriff, Prozessmanipulation, Pivoting, Screenshot, Keylogging etc.). Zentrales Werkzeug für die Post-Exploitation.
 
#### 3.3.6 Post-Exploitation & Privilege Escalation
- `local_exploit_suggester` (Linux) automatisiertes Vorschlagen passender lokaler Privilege-Escalation-Exploits
- `getsystem` (Windows) automatisierte Rechteausweitung auf SYSTEM-Ebene

#### 3.3.7 Typischer Modul-Workflow in msfconsole
Der Umgang mit einem Modul folgt in msfconsole immer demselben Schema:

1. **Suchen**: `search <begriff>`
2. **Laden**: `use <modulpfad>`
3. **Optionen prüfen**: `show options`
4. **Optionen setzen**: `set <OPTION> <wert>` (z. B. RHOSTS, LHOST, LPORT)
5. **Ausführen**: `run` bzw. bei Exploits zusätzlich Payload setzen (`set payload ...`)
Dieses Schema ist unabhängig vom Modultyp (Auxiliary, Exploit, Post) immer gleich und bildet die praktische Grundlage für den späteren Demo-Ablauf.

### 3.4 Von Scan zu Exploit: Schwachstellenidentifikation

#### 3.4.1 Referenztabelle: Häufige Ports und typische Schwachstellen

| Port      | Dienst    | Typische Schwachstellen                                                                 |
|-----------|-----------|-------------------------------------------------------------------------------------------|
| 21        | FTP       | Anonymous Login erlaubt, veraltete Server-Versionen mit bekannten Backdoors               |
| 22        | SSH       | Schwache/Default-Credentials, Brute-Force möglich, veraltete Versionen mit CVEs           |
| 23        | Telnet    | Unverschlüsselte Übertragung (Credentials im Klartext), Default-Zugangsdaten              |
| 25        | SMTP      | User-Enumeration via VRFY/EXPN, offen konfiguriertes Open Relay                           |
| 80 / 443  | HTTP(S)   | Veraltete Webserver-/CMS-Versionen, bekannte Applikations-CVEs, Default-Admin-Panels       |
| 139 / 445 | SMB       | Veraltete SMB-Protokollversion, Null-Session-Zugriff, fehlerhafte Authentifizierung        |
| 1433      | MSSQL     | Schwache SA-Credentials, veraltete/ungepatchte Version                                    |
| 3306      | MySQL     | Schwache Root-Credentials, veraltete Version mit bekannten CVEs                           |
| 3389      | RDP       | Schwache Credentials, veraltete Windows-Version (BlueKeep-Klasse Schwachstellen)          |
| 5985/5986 | WinRM     | Schwache/Default-Credentials, ungenügend eingeschränkter Zugriff                          |
| 8080      | HTTP-Alt (z.B. Tomcat) | Default-Credentials im Manager-Interface, unsichere Deployment-Funktion     |

> Diese Schwachstellenklassen sind exemplarisch und typisch für Testumgebungen wie
> Metasploitable3. Ob sie tatsächlich vorliegen, entscheidet immer die konkret
> erkannte Version aus dem Nmap-Scan.

#### 3.4.2 Exploit-Suche: Von der Schwachstelle zum Modul

Sobald eine mögliche Schwachstelle vermutet wird, folgt die gezielte Suche nach
einem passenden Modul:

- **In msfconsole:** `search <dienst>` oder `search cve:<jahr>-<nummer>`. Durchsucht die integrierte Modul-Datenbank nach Name, CVE oder Stichwort
- **Offline/extern:** `searchsploit <dienst> <version>`. Durchsucht die
  Exploit-DB nach passenden Public Exploits
- **Google-Suche:** Ergänzend hilfreich, z. B. `"<dienst> <version> exploit"`
  oder `"<dienst> <version> CVE"`. Liefert oft Blog-Artikel, Advisories oder
  Proof-of-Concepts, die (noch) nicht in Exploit-DB/Metasploit gelistet sind
- **Version prüfen:** Vor dem Laden mit `info <modul>` kontrollieren, ob die
  betroffene Versionsspanne mit der gescannten Version übereinstimmt
- **Ranking beachten:** msfconsole bewertet Module (`excellent`, `great`,
  `normal` etc.). Ein höheres Ranking bedeutet stabileren, zuverlässigeren
  Exploit-Erfolg bei geringerem Risiko eines Absturzes des Zieldienstes

Danach folgt der bereits beschriebene Standard-Workflow (`use` → `show options`
→ `set` → `run`).

### 3.5 Stärken und Grenzen
**Stärken:**
- Sehr grosse, gepflegte Exploit-Datenbank
- Starke Automatisierung
- Ideal für netzwerk-/dienstbasierte Schwachstellen
**Grenzen:**
- Weniger geeignet für komplexe Web-Applikationslogik (Besser Burp Suite)
- Erkennung durch moderne AV/EDR-Systeme möglich
- Exploit-Erfolg abhängig von Patch-Stand des Zielsystems
---
 
## 4. Praktischer Aufbau: Laborumgebung
 
### 4.1 Infrastruktur
- Virtualisierung mit **Vagrant** und **VirtualBox**
- Netzwerktopologie: privates/host-only Netz im Bereich `192.168.56.x`
- Zielsysteme: **Metasploitable3** (primäres Ziel)
### 4.2 Zielsystem-Beschreibung
Als Zielsystem wurde **Metasploitable3** in einer isolierten Laborumgebung verwendet. Beim Nmap-Scan wurde auf Port 21 der Dienst **ProFTPD 1.3.5** erkannt.

Für den initialen Zugriff wurde das Metasploit-Modul `exploit/unix/ftp/proftpd_modcopy_exec` eingesetzt. Dadurch konnte eine Command-Shell mit den Rechten des Benutzers `www-data` erlangt werden.

Anschliessend wurde die Session zu Meterpreter erweitert. Der `local_exploit_suggester` schlug unter anderem **CVE-2021-4034 (PwnKit)** als möglichen Privilege-Escalation-Exploit vor. Durch dessen Ausnutzung konnte eine Meterpreter-Session mit **Root-Rechten** erlangt und anschliessend auf die geschützte Flag-Datei zugegriffen werden.
 
### 4.3 Demo-Szenario (Übersicht)
Die Demo folgt einer "Capture the Flag"-Logik:
 
1. Recon (Nmap)
2. Exploit
3. Low-Privilege Shell
4. Fehlgeschlagener Flag-Zugriff (fehlende Rechte)
5. Privilege Escalation
6. Erfolgreicher Flag-Zugriff
Flag-Ablageorte: `/root/flag.txt` (Linux) bzw. `C:\Users\Administrator\Desktop\flag.txt` (Windows), jeweils mit eingeschränkten Zugriffsrechten.
 
---
 
## 5. Fazit

### 5.1 Zusammenfassung
Die Arbeit zeigt, dass ein Penetest einem klar strukturierten Ablauf folgt. Von der Aufklärung über Scanning und Exploitation bis zur Rechteausweitung und dass für jede Phase spezialisierte Tools existieren. Am Beispiel von Metasploit wurde deutlich, wie ein modulares Framework diesen gesamten Prozess unterstützt. Die praktische Umsetzung im Labor (Vagrant/VirtualBox, Metasploitable3) bestätigte, dass Metasploit vor allem bei klassischen Netzwerk- und Dienst-Schwachstellen seine Stärken ausspielt. 
### 5.2 Reflexion
**Was hat gut funktioniert?**
- Der CTF-artige Aufbau der Demo (Recon → Exploit → gescheiterter Flag-Zugriff → Privilege Escalation → Erfolg) hat eine klare, nachvollziehbare Story ergeben, ohne dass ein kompliziertes Szenario nötig war.
- Die Beschränkung auf Metasploit-native Methoden bei der Rechteausweitung hat die Demo konsistent gehalten und unnötige Tool-Wechsel vermieden.
- Die Entscheidung für Metasploitable3 als Zielsystem statt Juice Shop hat den Fokus klar auf netzwerkbasierte Schwachstellen gelenkt, für die Metasploit ausgelegt ist.


**Welche Herausforderungen gab es?**
- Das Aufsetzen der Laborumgebung mit Vagrant/VirtualBox war anfangs mit kleineren Problemen verbunden, u.a. IP-Konflikten im `192.168.56.x`-Netz zwischen mehreren VMs.
- Der bewusste Verzicht auf eine SIEM-Integration (Wazuh) bedeutete, dass die Detektions-Perspektive komplett ausserhalb des Scops blieb. Für eine vollständigere Betrachtung wäre das ein sinnvoller nächster Schritt.
- Abhängig vom Wissensstand: Der Umgang mit `msfconsole`, Payload-Typen (staged/stageless) und Meterpreter erforderte etwas Einarbeitungszeit, bevor sich die einzelnen Konzepte sauber ineinanderfügten.

### 5.3 Ausblick
- Mögliche Erweiterung um SIEM-Integration (z. B. Wazuh) für Detektions-Perspektive
- Weitere Zielsysteme / komplexere Szenarien
---
 
## 6. Anhang

Der Anhang ist wie folgt gegliedert:

Bilder --> [Bilder](Anhang/Bilder)
