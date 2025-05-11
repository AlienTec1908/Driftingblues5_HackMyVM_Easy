# Driftingblues5 - HackMyVM (Easy)

![Driftingblues5.png](Driftingblues5.png)

## Übersicht

*   **VM:** Driftingblues5
*   **Plattform:** [HackMyVM](https://hackmyvm.eu/machines/machine.php?vm=Driftingblues5)
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 22. November 2022
*   **Original-Writeup:** https://alientec1908.github.io/Driftingblues5_HackMyVM_Easy/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser "Easy"-Challenge war es, Root-Zugriff auf der Maschine "Driftingblues5" zu erlangen. Die Enumeration deckte einen Webserver (Apache) mit einer WordPress-Installation auf. Nach der Identifizierung von WordPress-Benutzern via `wpscan` und dem (vermutlich durch Brute-Force oder externen Hinweis erlangten) Admin-Passwort für `gill` wurde Zugriff auf das WordPress-Backend erlangt. In der Medienbibliothek wurde ein Bild (`dblogo.png`) gefunden, dessen Exif-Daten das SSH-Passwort (`59583hello`) für den Benutzer `gill` enthielten. Als `gill` wurde eine KeePass-Datenbankdatei (`keyfile.kdbx`) und die User-Flag gefunden. Das Master-Passwort der KeePass-Datei (`porsiempre`) wurde mit `keepass2john` und `john` geknackt. Parallel dazu wurde ein Mechanismus im Verzeichnis `/keyfolder` entdeckt: Durch das Erstellen spezifischer (oder beliebiger) Dateien in diesem Verzeichnis erschien eine Datei `rootcreds.txt`, die das Root-Passwort (`___drifting___`) enthielt. Mit diesem Passwort wurde via `su root` Root-Zugriff erlangt.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   `nikto`
*   `wpscan`
*   `wget`
*   `exiftool`
*   `ssh`
*   `python3 http.server`
*   `keepass2john`
*   `john` (John the Ripper)
*   KeePass Client (zur Anzeige der KDBX-Datei)
*   `su`
*   Standard Linux-Befehle (`ls`, `cat`, `mv`, `cd`, `touch`, `id`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Driftingblues5" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web Enumeration (WordPress):**
    *   IP-Findung mittels `arp-scan` (Ziel: `192.168.2.120`, Hostname `driftingblues`).
    *   `nmap`-Scan identifizierte SSH (22/tcp) und Apache (80/tcp) mit einer WordPress-Seite ("diary").
    *   `gobuster` und `nikto` bestätigten die WordPress-Struktur.
    *   `wpscan --enumerate u` identifizierte mehrere WordPress-Benutzernamen, darunter `gill`.

2.  **Information Disclosure (WordPress) & Initial Access (als `gill`):**
    *   (Annahme: Durch einen nicht gezeigten Brute-Force-Angriff oder externen Hinweis wurden die WordPress-Admin-Credentials `gill:interchangeable` erlangt.)
    *   Login in das WordPress-Backend (`/wp-admin/`).
    *   In der Medienbibliothek wurde das Bild `dblogo.png` gefunden und heruntergeladen.
    *   `exiftool dblogo.png` enthüllte in den Metadaten (Text Layer Text) das SSH-Passwort: `59583hello`.
    *   Ein SSH-Login als `gill` mit dem Passwort `59583hello` war erfolgreich.

3.  **KeePass Cracking & Root Credential Disclosure:**
    *   Als `gill` wurde im Home-Verzeichnis die User-Flag und die Datei `keyfile.kdbx` (eine KeePass-Datenbank) gefunden.
    *   `keyfile.kdbx` wurde mittels eines Python-HTTP-Servers und `wget` auf die Angreifer-Maschine übertragen.
    *   Mit `keepass2john keyfile.kdbx > hash` wurde der Hash des Master-Passworts extrahiert.
    *   `john --wordlist=/usr/share/wordlists/rockyou.txt hash` knackte das Master-Passwort zu `porsiempre`.
    *   (Parallel oder alternativ) Der Benutzer `gill` untersuchte das Verzeichnis `/keyfolder`. Durch das Erstellen mehrerer Dateien (`touch [dateiname]`) in diesem Verzeichnis erschien eine Datei `rootcreds.txt`.
    *   `cat rootcreds.txt` enthüllte das Root-Passwort `___drifting___` (und `imjustdrifting31`).

4.  **Privilege Escalation (zu `root`):**
    *   Mit dem Befehl `su root` und dem aus `rootcreds.txt` stammenden Passwort `___drifting___` wurde erfolgreich Root-Zugriff erlangt.
    *   Die Root-Flag wurde aus `/root/root.txt` gelesen.

## Wichtige Schwachstellen und Konzepte

*   **WordPress Enumeration:** Identifizierung von Benutzern über `wpscan`.
*   **Information Disclosure in Bild-Metadaten (Exif):** Ein SSH-Passwort wurde in den Exif-Daten eines Bildes gefunden.
*   **Unsichere Speicherung von KeePass-Datenbanken:** Die `keyfile.kdbx` war für den Benutzer `gill` zugänglich.
*   **Schwaches KeePass Master-Passwort:** Das Master-Passwort `porsiempre` konnte leicht geknackt werden.
*   **Versteckter Mechanismus / Information Disclosure (`/keyfolder`):** Ein ungewöhnlicher Mechanismus, der durch Dateierstellung Root-Credentials preisgab.
*   **Schwache Root-Passwörter:** Das Passwort `___drifting___` war über den `/keyfolder`-Mechanismus zugänglich.

## Flags

*   **User Flag (`/home/gill/user.txt`):** `F83FC7429857283616AE62F8B64143E6`
*   **Root Flag (`/root/root.txt`):** `9EFF53317826250071574B4D4EE56840`

## Tags

`HackMyVM`, `Driftingblues5`, `Easy`, `WordPress`, `Exiftool`, `Information Disclosure`, `SSH`, `KeePass`, `keepass2john`, `John the Ripper`, `Privilege Escalation`, `File System Trigger`, `Linux`
