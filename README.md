# RigVibe

*(bis Version 1.4.3 „Wellenwerk“ – Einstellungen, Logbuch und Anordnung übernimmt RigVibe beim ersten Start automatisch)*

Stationssteuerung, Logbuch und Digitalbetrieb für Funkamateure – ein Desktop-Programm für Windows, macOS und Linux.

## Download (Version 1.4.6)
| System | Datei |
|---|---|
| Windows 10/11 | [RigVibe-1.4.6-Windows-Setup.exe](https://github.com/djamplifier/rigvibe/releases/download/v1.4.6/RigVibe-1.4.6-Windows-Setup.exe) |
| Mac mit Apple-Chip (M1–M4) | [RigVibe-1.4.6-Mac-AppleSilicon.dmg](https://github.com/djamplifier/rigvibe/releases/download/v1.4.6/RigVibe-1.4.6-Mac-AppleSilicon.dmg) |
| Mac mit Intel-Chip | [RigVibe-1.4.6-Mac-Intel.dmg](https://github.com/djamplifier/rigvibe/releases/download/v1.4.6/RigVibe-1.4.6-Mac-Intel.dmg) |
| Linux (alle Distributionen) | [RigVibe-1.4.6-Linux.AppImage](https://github.com/djamplifier/rigvibe/releases/download/v1.4.6/RigVibe-1.4.6-Linux.AppImage) |
| Debian/Ubuntu | [rigvibe_1.4.6_amd64.deb](https://github.com/djamplifier/rigvibe/releases/download/v1.4.6/rigvibe_1.4.6_amd64.deb) |
| Linux-Server (Dienst, z. B. für einen Heimserver) | [RigVibe-1.4.6-Linux-Server.tar.gz](https://github.com/djamplifier/rigvibe/releases/download/v1.4.6/RigVibe-1.4.6-Linux-Server.tar.gz) |

Alle Versionen und Neuerungen: [Releases](https://github.com/djamplifier/rigvibe/releases). Installierte Programme melden neue Versionen selbst.

## Funktionen
- **Gerätesteuerung** (Hamlib, über 300 Funkgeräte, mehrere gleichzeitig): große Frequenzanzeige, VFO A/B, Split, RIT/XIT,
  alle Regler und Schalter des Geräts, Abstimmskalen je Band mit Feinskala, Schnellspeicher, Favoriten, Speicherkanäle,
  Scanner, Makros, Sprachspeicher, Band-Durchlauf, Panadapter (Geräte-Scope oder IQ-Soundkarte), WinKeyer,
  Freigabe des Geräts für andere Programme (Hamlib NET rigctl oder CAT-Emulation Kenwood TS-2000), Schnappschüsse,
  Synchronisierung zweier Geräte, Fernbetrieb-Audio (WebRTC) mit PTT, TCI-Schnittstelle für Log-/Contest-Programme, Layout frei anpassbar
- **Contest**: Super Check Partial, Doppel-Prüfung und neue Multiplikatoren schon beim Tippen, Wertungen (CQ WW, WPX, IARU, UKW-km, DOK, eigene), laufender Punktestand mit Rate, F-Tasten und „Enter sendet Text“, Cabrillo mit Punktzahl; **„Was fehlt mir?“** ordnet Cluster- und FT8-Stationen danach, was sie dem Log bringen; **Diagnose „Warum dekodiert nichts?“**; Speicherkanäle und Relais als **CHIRP-Datei**
- **Club-Anzeigetafel**: Rangliste der Club-Mitglieder, gemeinsame Ziele, Weltkarte der Club-Länder und Live-Ticker – im eigenen Netz oder übers Internet, ein RigVibe ist Gastgeber; nur Zahlen werden übertragen, keine Logbücher; Vollbild für den Beamer
- **Logbuch**: mehrere Logbücher, Rufzeichen-Info mit Callbook (HamQTH/QRZ.com) und gearbeitet-Matrix, DX-Cluster mit
  Alarmen, FT8-Alert und Antworten über WSJT-X, Bandkarte, Graulinie, Sonnendaten, Kalender (Contests, DXpeditionen),
  QSL (Papier, LoTW, eQSL), Uploads zu QRZ.com/eQSL/LoTW/Club Log, Diplome, Etiketten, Cabrillo, ADIF,
  POTA/SOTA/WWFF mit Stationsprofilen, Filter-Bedingungen und gespeicherte Ansichten, QSO-Austausch mit N1MM, VarAC, JTAlert, GridTracker
- **Digitalbetrieb**: Wasserfall, PSK/QPSK, RTTY, CW, MFSK, DominoEX, THOR, Olivia, Contestia, Throb, MT63, Hellschreiben,
  SSTV (18 Bildmodi mit Vorlagen und Galerie), FreeDV (RADE V1/V2, 700D/700E/1600, mit FreeDV Reporter), Makros, RigVibe Sweeper (bis 16 Signale gleichzeitig),
  Chat (Tastatur-QSO) über ARDOP, VARA, Packet und Telnet
- **Signalerkennung per Klick**: Klick auf ein Signal im Wasserfall zeigt die Betriebsart (z. B. RTTY mit Baud/Shift, FT8, PSK31,
  SSTV mit Bildmodus, FreeDV RADE/700D/700E/1600, CW mit Tempo, Olivia, Sprache, AM) – SSTV und FreeDV schon nach 4 Sekunden – und öffnet auf Wunsch gleich den passenden Dekoder
- **Empfänger ohne Funkgerät**: KiwiSDR, OpenWebRX, öffentliche Empfängerlisten, Wasserfall 2D/3D mit DX-Cluster- und POTA/SOTA/WWFF-Spots (Rufzeichen mit Flagge, Klick stimmt ab)
- **Eigene SDRs**: RTL-SDR (auch V4), HackRF One und ADALM-Pluto direkt (Pluto auch im Netzwerk, mit automatischer Suche), Hermes Lite 2 im Netz; **Senden** mit HackRF, Pluto und Hermes Lite 2
- **Handfunkgeräte Quansheng UV-K1 / UV-K5 V3** über USB-C: mit der mitgelieferten RigVibe-Firmware (Aufspielen per Assistent, ohne Zusatzprogramm) steuert RigVibe das Gerät komplett – Frequenz, Betriebsart, Leistung, Senden über USB, Rauschsperre, Bandübersicht, fester Tonpegel für die Soundkarte
- **QO-100**: Pluto mit LNB im Vollduplex, Uplink automatisch, LNB-Drift über die Bake ausgeglichen, Transponder-Bandplan, Satelliten-QSO im Logbuch (nur in den Amateurfunkbändern)
- **Handy-Ansicht** (/m, als App installierbar): Gerät mit PTT, Wasserfall, Meshtastic, Winlink, APRS, Logbuch – von unterwegs über **Tailscale**
- **APRS und Packet Radio** mit eigenem Modem: APRS-Karte im Umkreis (auch APRS-IS), Nachrichten mit Quittung, Bake mit eigenem Text, DOK und QRV-Frequenz, Packet-Chat (AX.25)
- **Funkruf (POCSAG/DAPNET):** Rufe empfangen (512/1200/2400 Bd), über das DAPNET oder selbst per SDR/Funkgerät senden, RigVibe als DAPNET-Sender (Node) mit Zeitschlitzen
- **Meshtastic**: eigenen Knoten im Netzwerk, per USB oder Bluetooth bedienen (Nachrichten, Knoten, Karte); **Winlink-Brücke** in eine Meshtastic-Gruppe für unterwegs
- **Sendeton**: Mikrofon-Aufbereitung mit Rauschsperre, Hoch-/Tiefpass, 10-Band-Equalizer, Kompressor, Sprachprozessor, Begrenzer, Abhören und VOX – für jeden Sendeweg; **Sprachspeicher** mit automatisch wiederholtem CQ-Ruf (auch am Handy)
- **Messinstrumente** (S-Meter, Leistung, SWR, ALC) als Zeigerinstrument – klassisch beleuchtet, modern, LED oder Kreuzzeiger; **Digitalfunk im Display** (Rufzeichen, Name, Sprechgruppe bei DMR, D-STAR, C4FM, M17, NXDN, P25); Spektrum flüssig mit 30 Bildern/s (wahlweise 60), mit Fläche, Glättung und Spitzenwert; Wasserfall auf der Grafikkarte
- **MIDI-Pulte und DJ-Controller**: Jogwheel als VFO, Fader für Pegel, Tasten für PTT, Band und Betriebsart – Belegung per Assistent, LEDs zeigen den Zustand, kein Treiber nötig
- **Packet-Monitor**: zeigt jeden AX.25-Rahmen wie ein TNC (auch Knoten- und Mailbox-Verkehr anderer Stationen, NET/ROM-Knotenlisten lesbar); der Demodulator repariert ein bis zwei Bitfehler wie Direwolf; Packet-Frequenzen je Band nach Bandplan als Auswahlliste
- **Rekorder** in der Gerätesteuerung: empfangenen Ton aufnehmen (SDR, WebSDR, OpenWebRX direkt, Funkgerät über die Soundkarte), mit Frequenz/Betriebsart speichern, anhören oder auf Sendung abspielen („so kommst du hier an“)
- **Baken-Monitor**: die 18 NCDXF/IARU-Baken live, Signal-Rausch-Abstand und hörbare Leistungsstufe je Band – Ausbreitung auf einen Blick
- **QSL-Karten**: selbst gestalten (Bild, Texte, QSO-Feld, QR-Code mit den QSO-Daten), per E-Mail mit Bild und ADIF senden oder drucken; eingegangene Karten mit QR-Code per Kamera oder Foto bestätigen
- **Logbuch-Werkzeuge**: EDI-Export für UKW-Contests, Papierlog schnell nachtragen, Hot-Ordner für ADIF-Import, Abgleich zweier RigVibe-Installationen (z. B. Shack-PC und Laptop)
- **DX-Cluster**: Ausschlussliste, „nur DX“ und Alarm, wenn ein Spot im Contest ein neuer Multiplikator wäre
- **SSTV-Einstieg mitten im Bild**: erkennt den Modus am Zeilentakt, auch ohne VIS-Kennung
- **Für deutsche Stationen**: Rufzeichenliste der Bundesnetzagentur als Callbook, Ausbildungsfunkbetrieb mit Nachweis (§ 12 AFuV), Prüfungstrainer mit allen Fragen der BNetzA (Klassen N/E/A), Amateurfunk-Lexikon
- **Rauschfilter**: Störimpulsaustaster, Rauschminderung spektral und per KI (RNNoise), Auto-Notch, Notch per Rechtsklick – für jede Tonquelle; im Digitalbetrieb schaltet RigVibe NB/NR/ANF in Funkgerät und Rauschfilter automatisch aus und bei Sprache wieder ein
- **Winlink**: E-Mail über Funk – Postfach, Verfassen mit Anhängen, Austausch über Telnet, ARDOP, VARA (TCP-Schnittstelle) und Packet (AGW/Direwolf)
- **Satelliten** (Doppler- und Rotor-Nachführung, Zeitplan, gemeinsame Sichtbarkeit), **Rotor** (Anschlag, Versatz, Überlauf), **Karten**
- **Erweiterungen** (Plugins), Oberfläche Deutsch/Englisch, automatische Aktualisierung, Rückmeldung an die Entwickler aus dem Programm
- Andockbare Fenster (lösen, auf zweiten Bildschirm, Layouts speichern), Rechtsklick-Menüs überall

**[➜ Ausführliche Anleitung mit Bildern](docs/Anleitung.md)** · **[English manual](docs/Manual.md)**

## Installation (eigenes Programm, kein Browser)
RigVibe ist ein Desktop-Programm mit eigenem Fenster; der Dienst und Hamlib (Funkgeräte-/Rotorsteuerung) sind eingebaut.
- **Windows:** `RigVibe-x.y.z-Windows-Setup.exe` ausführen. Beim ersten Start meldet Windows evtl. „Unbekannter Herausgeber“ → „Weitere Informationen“ → „Trotzdem ausführen“. USB-Treiber des Funkgeräts (z. B. Silicon Labs CP210x) installieren.
- **macOS:** `RigVibe-x.y.z-Mac-AppleSilicon.dmg` (Apple-Chip) bzw. `-Mac-Intel.dmg` öffnen, RigVibe in „Programme“ ziehen. Beim ersten Start Rechtsklick → „Öffnen“ (nicht notarisiert). Mikrofon-Zugriff erlauben (für Digitalbetrieb/SSTV).
- **Linux:** `.deb` installieren oder die `.AppImage` ausführbar machen und starten; Benutzer in die Gruppe `dialout` aufnehmen.

Danach: Gerätesteuerung → Verbinden → Hersteller/Modell, COM-Port, Baudrate (oder „Baudrate suchen“), bei Icom die CI-V-Adresse.
Daten liegen im Benutzerordner (Windows `%APPDATA%\RigVibe`, macOS `~/Library/Application Support/RigVibe`).


## Fehler melden und Wünsche

Fehler, Fragen und Ideen bitte als [Issue](https://github.com/djamplifier/rigvibe/issues) – RigVibe kann Programmfehler auf Wunsch auch selbst melden (Hilfe → Fehlerprotokoll).

## Lizenz

RigVibe ist ab Version 1.4.6 kostenlos nutzbar, aber nicht quelloffen – es gelten die [Lizenzbedingungen](LICENSE). Versionen bis 1.4.5 bleiben unter der GPL 3.
Für die Steuerung von Sendern und Rotoren und die Einhaltung der Amateurfunk-Vorschriften ist die bedienende Person verantwortlich.
Mitgelieferte Fremdkomponenten (u. a. Hamlib, LGPL 2.1) und ihre Lizenzen: [LIZENZEN.md](LIZENZEN.md).
RigVibe ist ein eigenständiges Programm und steht in keiner Verbindung zu Herstellern anderer Funk-Software oder von Funkgeräten.


Der Quelltext der bis Version 1.4.5 unter der GPL 3 veröffentlichten Fassungen liegt unter [Releases → GPL-Quelltexte](https://github.com/djamplifier/rigvibe/releases/tag/gpl-quelltexte).
