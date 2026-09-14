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
 
1. **Reconnaissance (Aufklärung)** – Sammeln von Informationen über das Zielsystem
2. **Scanning / Enumeration** – Identifikation offener Ports, Dienste, Versionen
3. **Exploitation** – Ausnutzung gefundener Schwachstellen zur Erlangung von Zugriff
4. **Post-Exploitation / Privilege Escalation** – Ausweitung der Rechte, Absicherung des Zugriffs
5. **Reporting** – Dokumentation der Ergebnisse und Handlungsempfehlungen
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
 
*(→ hier ggf. kurze Zeitleiste/Geschichte ergänzen)*
 
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
- **NOPs:** Padding zur Stabilisierung von Exploits
#### 3.3.2 msfconsole
Zentrale Konsole zur Interaktion mit dem Framework. Wichtige Befehle: `search`, `use`, `set`, `show options`, `exploit/run`. Anbindung an eine Datenbank (PostgreSQL) zur Verwaltung von Workspaces, Hosts und Ergebnissen.
 
#### 3.3.3 Payload-Konzepte
- **Staged vs. Stageless:** gestufte Übertragung des Payloads vs. vollständiger Payload in einem Schritt
- **Bind vs. Reverse Shell:** Zielsystem öffnet Port (bind) vs. Zielsystem baut Verbindung zum Angreifer auf (reverse); Reverse Shells sind in der Praxis meist vorzuziehen (Firewall/NAT-Umgehung)
#### 3.3.4 Meterpreter
Fortgeschrittener, im RAM laufender Payload mit erweiterten Funktionen (Dateisystemzugriff, Prozessmanipulation, Pivoting, Screenshot, Keylogging etc.). Zentrales Werkzeug für die Post-Exploitation.
 
#### 3.3.5 Post-Exploitation & Privilege Escalation
- `local_exploit_suggester` (Linux) – automatisiertes Vorschlagen passender lokaler Privilege-Escalation-Exploits
- `getsystem` (Windows) – automatisierte Rechteausweitung auf SYSTEM-Ebene
### 3.4 Stärken und Grenzen
**Stärken:**
- Sehr grosse, gepflegte Exploit-Datenbank
- Starke Automatisierung (Post-Exploitation, Privilege Escalation)
- Ideal für netzwerk-/dienstbasierte Schwachstellen
**Grenzen:**
- Weniger geeignet für komplexe Web-Applikationslogik (dort eher Burp Suite)
- Erkennung durch moderne AV/EDR-Systeme möglich
- Exploit-Erfolg abhängig von Patch-Stand des Zielsystems
---
 
## 4. Praktischer Aufbau: Laborumgebung
 
### 4.1 Infrastruktur
- Virtualisierung mit **Vagrant** und **VirtualBox**
- Netzwerktopologie: privates/host-only Netz im Bereich `192.168.56.x`
- Zielsysteme: **Metasploitable3** (primäres Ziel)
### 4.2 Zielsystem-Beschreibung
Kurze Beschreibung der verwundbaren Dienste auf Metasploitable3, die im Rahmen der Demo ausgenutzt werden.
 
### 4.3 Demo-Szenario (Übersicht)
Die Demo folgt einer "Capture the Flag"-Logik:
 
1. Recon (Nmap)
2. Exploit
3. Low-Privilege Shell
4. Fehlgeschlagener Flag-Zugriff (fehlende Rechte)
5. Privilege Escalation
6. Erfolgreicher Flag-Zugriff
Flag-Ablageorte: `/root/flag.txt` (Linux) bzw. `C:\Users\Administrator\Desktop\flag.txt` (Windows), jeweils mit eingeschränkten Zugriffsrechten.
 
*(Detaillierter Ablauf inkl. Screenshots gehört primär in die Präsentation / den Anhang, hier reicht die Kurzskizze.)*
 
---
 
## 5. Fazit
 
### 5.1 Zusammenfassung
*(kurze Zusammenfassung der wichtigsten Erkenntnisse – nach Fertigstellung des praktischen Teils ergänzen)*
 
### 5.2 Reflexion
- Was hat gut funktioniert?
- Welche Herausforderungen gab es (z. B. Lab-Setup, IP-Konflikte im Vagrantfile)?
### 5.3 Ausblick
- Mögliche Erweiterung um SIEM-Integration (z. B. Wazuh) für Detektions-Perspektive
- Weitere Zielsysteme / komplexere Szenarien
---
 
## 6. Anhang

Der Anhang ist wie folgt gegliedert:

Bilder --> Projektarbeit/Anhang/Bilder
Konfig-Files --> /Projektarbeit/Anhang/Konfigurationen
