# Modularbeit II – Detection Lab
## Blue Team / SOC Monitoring mit OPNsense und Wazuh

**Autor:** Yves
**Modul:** Modularbeit II
**Institution:** TEKO Bern
**Dozent:** Christian Locher
**Datum Abgabe:** 01.09.2026

---

## 1. Einleitung

### 1.1 Ausgangslage
Im Rahmen dieser Modularbeit wurde ein Detection Lab aufgebaut, das die zentralen Konzepte des Blue-Team- und SOC-Monitorings praktisch demonstriert. Der Fokus liegt auf den drei Bereichen **Hardening**, **Firewall** und **Monitoring** in einer virtualisierten Laborumgebung.

### 1.2 Bezug zum Modul
Im Modul Cybersecurity haben wir uns intensiv mit den Themen Firewall, Hardening und Monitoring auseinandergesetzt. Ich selbst hatte jedoch bis dahin noch nie praktisch eine Firewall installiert und konfiguriert und verfügte nur über geringe Monitoring-Kenntnisse. Dieses Setup erschien mir daher als ideales Projekt, um diese Technologien praxisnah zu erlernen und zu vertiefen. Zusätzlich bietet mir das aufgebaute Lab eine solide Grundlage für die Projektarbeit, welche am Ende dieses Moduls präsentiert wird.

### 1.3 Zielsetzung
Ziel dieser Arbeit ist der Aufbau einer funktionsfähigen Detection-Lab-Umgebung, in der:
- ein zentrales Firewall-System (OPNsense) das Netzwerk segmentiert und absichert,
- eine SIEM/XDR-Lösung (Wazuh) sicherheitsrelevante Ereignisse überwacht und protokolliert,
- eine bewusst verwundbare Webapplikation (OWASP Juice Shop) als Monitoring-Ziel dient.

### 1.4 Abgrenzung
Die aktive Angriffssimulation (z.B. mittels OWASP Top 10 gegen Juice Shop) ist **nicht Teil dieser Arbeit** und explizit für eine spätere, separate Modularbeit vorgesehen. Diese Arbeit fokussiert sich auf den Aufbau der defensiven Infrastruktur (Hardening, Firewalling, Monitoring-Grundlagen).

---

## 2. Architekturüberblick

### 2.1 Netzwerkdiagramm
[Diagramm einfügen: OPNsense mit WAN/LAN10/LAN20, Wazuh-VM, Juice-Shop-VM]

### 2.2 Komponentenübersicht

| Komponente | Rolle | IP-Adresse |
|---|---|---|
| OPNsense | Firewall / zentrales Routing | WAN: em0 (NAT) |
| Wazuh Manager | SIEM / XDR Plattform | 192.168.10.10 (LAN10) |
| OWASP Juice Shop | Monitoriertes Zielsystem | 192.168.20.20 (LAN20) |

### 2.3 Netzwerksegmentierung

| Interface | Name | Netz | Zweck |
|---|---|---|---|
| em0 | WAN | VirtualBox NAT | Internetanbindung |
| em1 | LAN10_Admin_Wazuh | 192.168.10.0/24 | Administration & SIEM |
| em2 | LAN20_JuiceShop | 192.168.20.0/24 | Monitoriertes Zielsystem |

### 2.4 Design-Entscheidungen
- **Separate virtuelle NICs statt VLANs**: Priorisierung von Einfachheit und Nachvollziehbarkeit gegenüber VLAN-Tagging, da für den Laborumfang keine zusätzliche Komplexität nötig ist.
- **Host-Zugriff via Host-only-Adapter**: Der Host-PC greift direkt auf das Wazuh-Dashboard zu, ohne über die Firewall geroutet zu werden (reiner Verwaltungszugriff).

---

## 3. Technische Umsetzung

### 3.1 OPNsense Installation & Interface-Konfiguration
Die OPNsense-VM wurde in VirtualBox mit drei Netzwerkadaptern eingerichtet: einer Netzwerkbrücke für WAN (em0) sowie zwei Host-only-Adaptern für LAN10 (em1) und LAN20 (em2). Nach dem Standard-Installationsvorgang von OPNsense erfolgte die Interface-Zuweisung über die Konsole (*Assign Interfaces*), wobei em0 als WAN, em1 als LAN10_Admin_Wazuh und em2 als LAN20_JuiceShop benannt wurden.

Im Anschluss wurden die statischen IPv4-Adressen der internen Interfaces gesetzt:
- LAN10_Admin_Wazuh (em1): 192.168.10.1/24
- LAN20_JuiceShop (em2): 192.168.20.1/24
- WAN_Internet_Simulation (em0): DHCP-Bezug aus dem Host-Netzwerk (192.168.1.x)

Die Konfiguration der Interfaces sowie die weiteren Grundeinstellungen (Hostname, DNS-Server) erfolgten anschliessend über die WebGUI, erreichbar via `https://192.168.10.1`. Für die DNS-Auflösung wurde OPNsense so konfiguriert, dass sowohl es selbst als auch die dahinterliegenden VMs öffentliche DNS-Server nutzen konnten, nachdem die zu Beginn erwähnten Probleme mit veralteten Einträgen behoben waren (siehe Kapitel 4).

### 3.2 Wazuh Manager Installation (Debian 13 "Trixie")
Bei der Installation musste das Installationsskript `wazuh-install.sh` angepasst werden, da die Abhängigkeit `software-properties-common` in Debian 13 "Trixie" nicht mehr existiert. Die Zeile wurde mittels `nano` entfernt, danach verlief die Installation erfolgreich.

```bash
# Beispiel: Bearbeitung des Installationsskripts
# Zeile mit "software-properties-common" entfernen/anpassen
sudo nano wazuh-install.sh

sudo bash wazuh-install.sh -a
```

### 3.3 Wazuh Agent Deployment auf Juice Shop
Auf der Juice-Shop-VM (192.168.20.20) wurde das Wazuh-Agent-Paket installiert und dabei die Adresse des Wazuh Managers (192.168.10.10) als Zielserver hinterlegt. Nach der Installation wurde der Agent-Dienst gestartet und für den automatischen Start beim Booten aktiviert.

Die Registrierung des Agents am Manager erfolgte automatisch über den in der Installation hinterlegten Enrollment-Mechanismus (Kommunikation über Port 1515), die eigentliche Ereignis- und Log-Übertragung läuft anschliessend über Port 1514. Damit diese Kommunikation stattfinden konnte, war die entsprechende Firewall-Freigabe auf LAN20 (siehe Kapitel 3.4) Voraussetzung.

Der erfolgreiche Abschluss des Deployments wurde im Wazuh-Dashboard unter *Agents* überprüft, wo der registrierte Agent mit dem Status **"Active"** angezeigt wird [Screenshot einfügen].

### 3.4 Firewall-Regelwerk

Das Regelwerk umfasst 14 Regeln über die Interfaces LAN10, LAN20 und WAN und folgt dem Prinzip der geringsten Rechte (Least Privilege).

**Kernprinzipien:**
- **First-Match-Wins**: OPNsense wertet Regeln in der definierten Reihenfolge aus (`quick`-Keyword) – spezifische Ausnahmeregeln müssen vor generellen Block-Regeln stehen.
- **Agent-initiierte Verbindungen**: Wazuh-Agenten bauen die Verbindung immer aktiv zum Manager auf. Die entsprechende Firewall-Ausnahme (Port 1514–1515) muss daher auf dem Segment des Agents (LAN20) definiert werden, nicht auf dem des Managers.
- **Internet-Traffic**: Als Ziel für internetgebundenen Traffic muss "any" verwendet werden, nicht der WAN-Alias, da dieser nur das lokale WAN-Subnetz abdeckt.
- **Aliases statt Rohdaten**: Wo möglich wurden Host-Aliases (z.B. `Wazuh_Manager_Host`) anstelle einzelner /32-IP-Einträge verwendet, um die Wartbarkeit zu erhöhen.

**Vollständiges Regelwerk (14 Regeln)**

Die Regeln sind je Interface in der Reihenfolge aufgeführt, in der sie in OPNsense ausgewertet werden (first-match-wins).

**Interface: LAN10_Admin_Wazuh**

| # | Aktion | Protokoll | Quelle | Ziel | Zielport | Beschreibung |
|---|---|---|---|---|---|---|
| 1 | Pass | TCP | LAN10_Admin_Wazuh network | This Firewall | https | Allow LAN10 to Firewall GUI (443) |
| 2 | Pass | TCP/UDP | LAN10_Admin_Wazuh network | This Firewall | domain | Allow LAN10 DNS (53) |
| 3 | Pass | TCP | LAN10_Admin_Wazuh network | * | Web_Ports | Allow LAN10 to Internet (80/443) |
| 4 | Pass | UDP | LAN10_Admin_Wazuh network | WAN_Internet_Simulation network | ntp | Allow LAN10 NTP (123) |
| 5 | Pass | ICMP | LAN10_Admin_Wazuh network | * | * | Allow LAN10 Ping (Testing) |
| 6 | Block | * | LAN10_Admin_Wazuh network | LAN20_JuiceShop network | * | Block remaining LAN10 to LAN20 |

**Interface: LAN20_JuiceShop**

| # | Aktion | Protokoll | Quelle | Ziel | Zielport | Beschreibung |
|---|---|---|---|---|---|---|
| 7 | Pass | TCP | LAN20_JuiceShop network | LAN10_Host_192_168_10_10 | 1514–1515 | Allow LAN20 Wazuh Agent to Manager (1514/1515) |
| 8 | Block | * | LAN20_JuiceShop network | LAN10_Admin_Wazuh network | * | Block remaining LAN20 to LAN10 |
| 9 | Pass | TCP/UDP | LAN20_JuiceShop network | This Firewall | domain | Allow LAN20 DNS (53) |
| 10 | Pass | TCP | LAN20_JuiceShop network | * | Web_Ports | Allow LAN20 to Internet (80/443) |
| 11 | Pass | UDP | LAN20_JuiceShop network | WAN_Internet_Simulation network | ntp | Allow LAN20 NTP (123) |
| 12 | Pass | ICMP | LAN20_JuiceShop network | * | * | Allow LAN20 Ping (Testing) |
| 13 | Block | * | LAN20_JuiceShop network | * | * | Block remaining LAN20 |

**Interface: WAN_Internet_Simulation**

| # | Aktion | Protokoll | Quelle | Ziel | Zielport | Beschreibung |
|---|---|---|---|---|---|---|
| 14 | Block | * | WAN_Internet_Simulation network | * | * | Block WAN to alles (kein eingehender Traffic) |

**Anmerkungen zur Reihenfolge:**
- Auf LAN10 steht die Ausnahme für Firewall-GUI-Zugriff (#1) ganz oben, die Block-Regel Richtung LAN20 (#6) folgt erst nach den generellen Allow-Regeln, da Letztere ohnehin nicht auf das LAN20-Netz zielen.
- Auf LAN20 steht die spezifische Wazuh-Agent-Ausnahme (#7) zwingend **vor** der generellen Block-Regel Richtung LAN10 (#8) – ansonsten würde die Agent-Kommunikation blockiert.
- Die abschliessenden "Block remaining"-Regeln (#13, #14) fungieren als explizite Catch-all-Regeln und dokumentieren das Least-Privilege-Prinzip, auch wenn OPNsense implizit ohnehin alles blockt, was nicht explizit erlaubt ist.

### 3.5 Validierung / Tests
- Wazuh-Agent zeigt Status "Active" im Dashboard [Screenshot]
- Konnektivitätstests zwischen den Segmenten (erlaubte/blockierte Verbindungen) [ggf. Nachweise]

---

## 4. Herausforderungen & Lösungen

| Herausforderung | Ursache | Lösung |
|---|---|---|
| Instabiles Routing | NAT-Adapter und segmentierte Adapter gleichzeitig aktiv | Entfernen der NAT-Adapter, sobald reguläres Routing funktionsfähig war |
| Fehlerhafte DNS-Auflösung | Veraltete Einträge in `/etc/resolv.conf` mit falschem Gateway | DNS explizit in der Interface-Konfiguration gesetzt |
| Kein Internetzugriff trotz Regel | Firewall-Regel fälschlich auf WAN-Subnetz-Alias statt "any" beschränkt | Zielbereich der Regel auf "any" angepasst |
| Installationsskript schlägt fehl | `software-properties-common` in Debian Trixie nicht mehr vorhanden | Skript manuell angepasst vor Ausführung |

---

### 4.1 Bekannte Limitation: Host-Zugriff via VirtualBox Host-only Networking

VirtualBox Host-only Networking gewährt dem Host systembedingt direkten, ungefilterten Zugriff auf alle Host-only-Segmente – dies stellt in einer echten Produktivumgebung eine Umgehung der Firewall dar und müsste dort z. B. durch reines Internal Network + dediziertes Management-Interface vermieden werden.

---

## 5. Erkenntnisse / Lessons Learned

- Firewall-Regeln für Agent-basierte Systeme müssen aus Sicht der Verbindungsrichtung gedacht werden, nicht aus Sicht der "Wichtigkeit" des Systems.
- Regelreihenfolge ist in OPNsense entscheidend – Ausnahmen immer vor generellen Blockregeln.
- Kompatibilitätsprüfungen bei neuen OS-Versionen (z.B. Debian Trixie) sind vor der Installation von Drittsoftware essenziell.
- Aliases erhöhen die Lesbarkeit und Wartbarkeit von Firewall-Regelwerken erheblich.

---

## 6. Ausblick

Das aufgebaute Detection Lab bildet die Grundlage für eine geplante, separate Folgearbeit im Bereich Angriffssimulation. Dabei sollen gezielt Schwachstellen gemäss **OWASP Top 10** gegen die Juice-Shop-Instanz ausgenutzt und die Erkennung dieser Angriffe über Wazuh demonstriert werden (u.a. mittels File Integrity Monitoring, Security Configuration Assessment, Threat Hunting und Vulnerability Detection).

**Wichtig für die geplante Angriffssimulation:** Das bestehende Firewall-Regelwerk soll dabei **nicht gelockert** werden. OWASP-Top-10-Angriffe (z.B. SQL Injection, XSS, Broken Access Control) finden auf Applikationsebene (Layer 7) statt und werden von einer klassischen Netzwerk-Firewall (Layer 3/4) ohnehin nicht abgefangen, solange der reguläre Web-Traffic zu Juice Shop erlaubt ist. Das Regelwerk bleibt somit nach dem Least-Privilege-Prinzip bestehen; benötigt wird lediglich eine zusätzliche, dokumentierte Regel, die einer dedizierten Angreifer-VM (z.B. Kali Linux, eigenes Segment) gezielten Zugriff auf den Juice-Shop-Webport erlaubt – analog zu einem realen Nutzerzugriff.

Diese Trennung zeigt exemplarisch das Prinzip des **Defense-in-Depth**: Die Firewall schützt die Netzwerkebene, während Wazuh als Monitoring-Layer applikationsseitige Angriffe sichtbar macht, die auf Netzwerkebene nicht erkennbar sind.

---

## 7. Anhang
Im Ordner Modularbeit_2/Anhang - sind alle Screenshots, Tabellen und Konfigurationsdateien zu finden

### 7.1 Vollständige Firewall-Regeltabelle
[Alle 14 Regeln einfügen]

### 7.2 Screenshots
- Wazuh Dashboard – Agent Status
- OPNsense Firewall-Regeln
- [weitere]

### 7.3 Konfigurationsauszüge
[relevante Config-Dateien, z.B. Interface-Konfiguration, DNS-Einstellungen]