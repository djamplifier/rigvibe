# Lizenzen

## RigVibe

Copyright (C) 2026 Marcus Adena, DM5MN. Alle Rechte vorbehalten.

Ab Version 1.4.6 ist RigVibe kostenlos nutzbar, aber nicht mehr quelloffen – es gelten die Bedingungen in der Datei
[`LICENSE`](LICENSE). Versionen bis einschließlich 1.4.5 wurden unter der GNU General Public License 3 veröffentlicht und bleiben unter der GPL.

**Haftung:** Für die Einhaltung der Amateurfunk-Vorschriften (Frequenzen, Leistung, Betriebsarten) ist die bedienende Funkamateurin
bzw. der bedienende Funkamateur verantwortlich – das gilt insbesondere für die Steuerung von Funkgeräten, Sendern und Rotoren. Einzelheiten in [`LICENSE`](LICENSE).

RigVibe ist ein eigenständiges Programm und steht in keiner Verbindung zu Herstellern anderer Funk-Software oder von Funkgeräten.
Genannte Produkt- und Firmennamen (z. B. von Funkgeräten oder Online-Diensten) dienen nur der Beschreibung der Kompatibilität
und sind Marken der jeweiligen Inhaber.

**Programme unter GPL, die RigVibe als eigenständige Programme mitliefert** (Hamlib-Programme, `rtl_tcp` mit librtlsdr, `rtl_433`,
unter Windows `wdi-simple.exe`): Sie laufen als eigene Prozesse und werden unverändert weitergegeben. Den Quelltext der jeweils
mitgelieferten Fassung gibt es bei den unten genannten Projekten; auf Anfrage (E-Mail an den Rechteinhaber) stellen wir ihn
mindestens drei Jahre lang auch selbst bereit.

## Mitgelieferte und verwendete Fremdkomponenten

| Komponente | Verwendung | Lizenz |
|---|---|---|
| [Hamlib](https://github.com/Hamlib/Hamlib) (`rigctld`, `rotctld`, `rigctl`, `rotctl` und Bibliotheken) | Funkgeräte- und Rotorsteuerung, unverändert als eigene Programme mitgeliefert | GNU LGPL 2.1 (Programme: GPL 2) – Text in [`licenses/LGPL-2.1.txt`](licenses/LGPL-2.1.txt). Quelltext der mitgelieferten Versionen: <https://github.com/Hamlib/Hamlib/releases> (Windows und Linux: Version 4.6.5; macOS: Homebrew-Version zum Zeitpunkt des Baus). Auf Anfrage stellen wir den Quelltext der jeweils mitgelieferten Version bereit. |
| Protokollbeschreibungen „The IZ8BLY MFSK Varicode“ (Nino Porcino IZ8BLY, Murray Greenman ZL1BPU) und „DominoEX Varicode Table“ (Murray Greenman ZL1BPU) | `static/js/dsp2-tables.js`: Zeichenreihenfolge der Varicodes für MFSK, THOR und DominoEX; die Codewörter selbst erzeugt RigVibe nach den Regeln der Protokolle | Protokollspezifikationen der Autoren |
| [codec2](https://github.com/drowe67/codec2) (David Rowe VK5DGR u. a.) | FreeDV (digitale Sprache): Bibliothek `libcodec2`, unverändert mitgeliefert und zur Laufzeit geladen | GNU LGPL 2.1 – Text in [`licenses/LGPL-2.1.txt`](licenses/LGPL-2.1.txt); Quelltext: <https://github.com/drowe67/codec2> (Windows: MSYS2-Paket, macOS: Homebrew, Linux: Ubuntu-Paket der jeweiligen Version) |
| LZHUF-Kompression (Haruhiko Okumura, Haruyasu Yoshizaki, Kenji Rikitake) | `wellenwerk/winlink/lzhuf.py`: eigene Python-Umsetzung des Verfahrens für das Winlink-B2-Format | „Use, distribute, and modify this program freely“ (Freigabe der Autoren, siehe Kopf der Datei) |
| [wl2k-go](https://github.com/la5nta/wl2k-go) (Martin Hebnes Pedersen, LA5NTA) | Referenz für das B2F-Protokoll, die Winlink-Passwortabfrage (Salzwert aus paclink-unix) und die Modem-Schnittstellen; Prüfdaten für die LZHUF-Umsetzung. Kein Code übernommen, eigene Umsetzung in Python | MIT |
| [Electron](https://www.electronjs.org/) | Programmfenster | MIT; Chromium-Lizenzen liegen dem Installationspaket bei (`LICENSES.chromium.html`) |
| [Python](https://www.python.org/) mit [PyInstaller](https://pyinstaller.org/) | Laufzeit des Dienstes | PSF-Lizenz; PyInstaller-Bootloader: GPL 2 mit Ausnahme für gebündelte Programme |
| [FastAPI](https://fastapi.tiangolo.com/), Starlette, Pydantic, AnyIO, h11 | Webdienst | MIT bzw. BSD-3-Clause |
| [Uvicorn](https://www.uvicorn.org/), websockets, httptools, uvloop, watchfiles | Webserver | BSD-3-Clause bzw. MIT/Apache-2.0 |
| [python-multipart](https://github.com/Kludex/python-multipart) | Datei-Uploads | Apache-2.0 |
| [pySerial](https://github.com/pyserial/pyserial) | serielle Schnittstellen | BSD-3-Clause |
| [pypdf](https://github.com/py-pdf/pypdf) 5.4.0 | Rufzeichenliste der Bundesnetzagentur lesen (nur deutsche Stationen) | BSD-3-Clause |
| [dockview](https://github.com/mathuo/dockview) 8.4.1 | andockbare Fenster | MIT |
| [Leaflet](https://leafletjs.com/) 1.9.4 | Karten | BSD-2-Clause |
| [D3](https://d3js.org/) 7.9.0 | (Satelliten-Überflug, Hilfsfunktionen) | ISC |
| Erdbilder `static/vendor/earth/` | 3D-Erde (Graulinie, Karten, Peilkarte, Satelliten) | gemeinfrei – NASA Earth Observatory: „Blue Marble: Next Generation“ (Reto Stöckli, NASA GSFC, Dezember 2004, mit Relief und Meeresboden) und „Earth at Night 2016 / Black Marble“ (NASA Earth Observatory, Joshua Stevens, Suomi NPP VIIRS). Auf 4096×2048 bzw. 3600×1800 Pixel verkleinert. |
| [satellite.js](https://github.com/shashwatak/satellite-js) 5.0.0 | Satellitenbahnen | MIT |
| [RNNoise](https://github.com/xiph/rnnoise) (Xiph.Org, Jean-Marc Valin), als WebAssembly von [@shiguredo/rnnoise-wasm](https://github.com/shiguredo/rnnoise-wasm) 2025.1.5 (`static/vendor/rnnoise/`) | KI-Rauschminderung im Fenster „Rauschfilter“, wird nur bei Bedarf geladen | BSD-3-Clause (RNNoise, `licenses/rnnoise-BSD-3-Clause.txt`), Apache-2.0 (WebAssembly-Fassung, `static/vendor/rnnoise/LICENSE`) |
| [rtl_tcp und librtlsdr](https://github.com/rtlsdrblog/rtl-sdr-blog) (Osmocom, Fassung RTL-SDR Blog V1.4.0) | RTL-SDR am USB: RigVibe startet das mitgelieferte Programm `rtl_tcp` als eigenen Prozess und liest dessen IQ-Daten | GPL-2.0-or-later (`licenses/librtlsdr-GPL-2.0.txt`), unverändert |
| [libhackrf](https://github.com/greatscottgadgets/hackrf) (Great Scott Gadgets) | HackRF One direkt per USB, Empfang und Senden | BSD-3-Clause (`licenses/libhackrf-BSD-3-Clause.txt`) |
| [libiio](https://github.com/analogdevicesinc/libiio) 0.26 (Analog Devices), unter Windows mit libxml2 und der Microsoft-Laufzeit | ADALM-Pluto, Empfang und Senden | LGPL-2.1-or-later (`licenses/libiio-LGPL-2.1.txt`), libxml2: MIT |
| [RADE](https://github.com/freedv/rade_c) (Radio Autoencoder, David Rowe VK5DGR, Peter B Marks u. a.) mit dem FARGAN-Vokoder aus [Opus](https://opus-codec.org) (Xiph.Org, Mozilla, Amazon u. a.), Bibliothek `librade` | FreeDV RADE V1/V2 | BSD-2-Clause (RADE), BSD-3-Clause (Opus) |
| [dxcc-json](https://github.com/k0swe/dxcc-json) (k0swe), Datei `static/dxccflags.json` | Landesflagge bzw. Landeskürzel je DXCC-Gebiet für die Spots im Wasserfall | Apache-2.0 |
| [rtl_433](https://github.com/merbanan/rtl_433) 25.02 (Benjamin Larsson u. a.), Programm `rtl_433` | Funksensoren mit eigenen SDRs entschlüsseln | GPL-2.0-or-later |
| [libwdi](https://github.com/pbatard/libwdi) 1.5.1 (Pete Batard), Hilfsprogramm `wdi-simple.exe` (nur Windows) mit den WinUSB-Installationsdateien des Microsoft WDK 8 (Redistributable) | WinUSB-Treiber für RTL-SDR/HackRF einrichten, ohne Zadig | GPL-3.0 (libwdi), Microsoft WDK Redistributable License (WinUSB/WDF-Installer) |
| [libusb](https://libusb.info) 1.0, unter Windows zusätzlich pthreads-win32 und die Microsoft-Laufzeit msvcr100 (aus dem RTL-SDR-Blog-Paket) | USB-Zugriff der SDR-Bibliotheken, Erkennung der RTL-SDR-Sticks | LGPL-2.1-or-later (`licenses/LGPL-2.1.txt`), msvcr100: Microsoft Visual C++ Redistributable |
| [bleak](https://github.com/hbldh/bleak) 3.0.2 mit pyobjc (macOS), pywinrt (Windows), dbus-fast (Linux) | Bluetooth zu Meshtastic-Knoten | MIT |
| Firmware [F4HWN Edition Fusion](https://github.com/armel/uv-k1-k5v3-firmware-custom) 6.3.0 für Quansheng UV-K1 / UV-K5 V3 (Armel F4HWN, muzkr, aufbauend auf DualTachyon, Egzumer u. a.), Dateien `packaging/k1-firmware/f4hwn.fusion.bin` (unverändert, offizielle Veröffentlichung) und `f4hwn.rigvibe.bin` (von uns geändert: Steuerung durch RigVibe, Änderungen vollständig in `packaging/k1-firmware/0001-rigvibe-cat.patch`, Bau mit `build.sh`) | Firmware-Assistent: wird auf das Funkgerät geschrieben, läuft nicht auf dem Rechner | Apache-2.0 – Text in [`licenses/F4HWN-Apache-2.0.txt`](licenses/F4HWN-Apache-2.0.txt), Hinweise der Autoren in [`licenses/F4HWN-NOTICE.txt`](licenses/F4HWN-NOTICE.txt) |
| [serialtool](https://github.com/muzkr) (muzkr) | Referenz für das Bootloader-Protokoll von UV-K1 / UV-K5 V3 (`wellenwerk/qsflash.py`, eigene Umsetzung; Verschleierungstabellen übernommen) | MIT |
| [certifi](https://github.com/certifi/python-certifi) (Mozilla-Zertifikatsliste) | HTTPS-Prüfung in der gepackten Fassung (Karten, Satellitendaten) | MPL-2.0 |
| [qrcode-generator](https://github.com/kazuhikoarase/qrcode-generator) 1.4.4 (Kazuhiko Arase), `static/vendor/qrcode.js` | QR-Code der Handy-Adresse und auf QSL-Karten | MIT |
| [jsQR](https://github.com/cozmo/jsQR) 1.4.0 (Cosmo Wolfe), `static/vendor/jsqr.js` | QR-Codes auf QSL-Karten mit Kamera oder aus einem Foto lesen, wird nur bei Bedarf geladen | Apache-2.0 |
| [topojson-client](https://github.com/topojson/topojson-client) 3.1.0, [world-atlas](https://github.com/topojson/world-atlas) 2.0.2 | Ländergrenzen (Natural Earth, gemeinfrei) | ISC |
| [marked](https://github.com/markedjs/marked) 12.0.2 | Anzeige der Anleitung | MIT |
| Schrift [Share Tech Mono](https://fonts.google.com/specimen/Share+Tech+Mono) | Frequenz- und Uhranzeigen | SIL Open Font License 1.1 – Copyright (c) 2012, Carrois Type Design, Ralph du Carrois, with Reserved Font Name „Share“. Text in [`licenses/OFL-1.1.txt`](licenses/OFL-1.1.txt) |

## Daten und Dienste zur Laufzeit

RigVibe lädt bei Bedarf Daten von öffentlichen Quellen; es gelten deren Bedingungen:
Länderdatei [AD1C / country-files.com](https://www.country-files.com/),
Sonnendaten [NOAA SWPC](https://www.swpc.noaa.gov/),
Kurzwellen-Sendeplan [EiBi](http://www.eibispace.de/),
Satellitenbahnen [CelesTrak](https://celestrak.org/), Sender [SatNOGS](https://db.satnogs.org/),
für deutsche Stationen im Prüfungstrainer die „Prüfungsfragen zum Erwerb von Amateurfunkprüfungsbescheinigungen, Bundesnetzagentur, 3. Auflage, März 2024, (www.bundesnetzagentur.de/amateurfunk), Datenlizenz Deutschland – Namensnennung – Version 2.0 (www.govdata.de/dl-de/by-2-0)“ (bei Bedarf direkt bei der BNetzA geladen; zum Üben werden die Antworten gemischt angezeigt, Formeln ohne LaTeX dargestellt), für deutsche Stationen die [Rufzeichenliste der Bundesnetzagentur](https://www.bundesnetzagentur.de/) (Verzeichnis der zugeteilten deutschen Amateurfunkrufzeichen, wird direkt bei der BNetzA geladen und nur im eigenen Datenordner gehalten),
Contest-Kalender [WA7BNM](https://www.contestcalendar.com/), DXpeditionen [NG3K ADXO](https://www.ng3k.com/Misc/adxo.html),
Kartenkacheln beim Hineinzoomen [OpenStreetMap](https://www.openstreetmap.org/copyright) (© OpenStreetMap-Mitwirkende, Daten unter ODbL; über den RigVibe-Dienst mit Zwischenspeicher abgerufen, gemäß der [Tile Usage Policy](https://operations.osmfoundation.org/policies/tiles/)),
APRS-IS (nur lesen, auf Wunsch),
sowie die vom Nutzer eingerichteten Dienste (HamQTH, QRZ.com, eQSL, LoTW, Club Log, DX-Cluster).

## ft8_lib (native/ftx, static/vendor/ftx.js)

FT8/FT4-Dekoder von Karlis Goba, https://github.com/kgoba/ft8_lib – MIT-Lizenz (native/ftx/ft8_lib/LICENSE). In RigVibe als WebAssembly übersetzt (native/ftx/build.sh), unverändert bis auf die Lautstärke der Protokollmeldungen.

## Sprachmodul (optional, wird nicht mitgeliefert)

- **mbelib** (https://github.com/szechyjs/mbelib, Copyright (C) 2010 mbelib Author, ISC-Lizenz): weder enthalten noch von RigVibe geladen. Wer die Sprache hören will, übersetzt mbelib selbst und legt die Bibliothek in den Ordner `module` des Datenordners; RigVibe bindet sie dann ein (wellenwerk/dvvoice.py). AMBE ist patentrechtlich geschützt; die Nutzung verantwortet der Nutzer.
- **Verschachtelungstabellen für DMR- und D-STAR-Sprachrahmen** in wellenwerk/dvvoice.py aus DSD (https://github.com/szechyjs/dsd, Copyright (C) 2010 DSD Author, ISC-Lizenz):

  > Permission to use, copy, modify, and/or distribute this software for any purpose with or without fee is hereby granted, provided that the above copyright notice and this permission notice appear in all copies.

- M17 (Faltungscode, Punktierung, Verwürfelung, LICH, CRC) ist nach der offenen M17-Spezifikation selbst umgesetzt; libm17 (GPL-2) wurde nur zum Gegenprüfen verwendet, nicht übernommen.

## Frequenz- und Relaisdaten (zur Laufzeit abgerufen, nicht im Programm enthalten)

- Rundfunk-Sendeplan: EiBi (eibispace.de, Eike Bierwirth) – frei verwendbare Sendeliste, mit Quellenangabe.
- Relais: DARC-Relaisliste von DL3EL (relais.dl3el.de, Umkreissuche als CSV), EchoLink-Linkliste (echolink.org/links.jsp, Umkreissuche), BrandMeister-Geräteliste (api.brandmeister.network, DMR-Relais), hearham.com (offene Relais-Datenbank).
- Namen zu Digitalfunk-Kennungen: RadioID.net (DMR- und NXDN-IDs mit Rufzeichen, Vorname, Ort), BrandMeister-Talkgroupliste – nur lokal nachgeschlagen.
- Typische Frequenzen (static/freqs.json): zusammengestellt aus dem IARU-Region-1-Bandplan (Aktivitätszentren), den Standardfrequenzen von WSJT-X sowie öffentlich bekannten Wetter-, Zeit-, CB-, PMR446- und Freenet-Frequenzen.

## Signal-Datenbank (static/sigdb.json)

Auszug aus dem Signal Identification Wiki (https://www.sigidwiki.com): Signalnamen, Frequenzangaben, Betriebsart, Modulation, Bandbreite und ACF von rund 550 Signalseiten (Stand 2026-10-16). Übernommen wurden nur diese Sachangaben, keine Beschreibungstexte, Bilder oder Tonbeispiele; jeder Eintrag verlinkt auf die Originalseite.

## Postleitzahl-Mittelpunkte (wellenwerk/notfunk/plz_de.json.gz)

Mittelpunkte der deutschen Postleitzahlen, berechnet aus den Postleitzahl-Daten von [GeoNames](https://www.geonames.org) (Stand 2026-10-22), lizenziert unter [Creative Commons Namensnennung 4.0](https://creativecommons.org/licenses/by/4.0/). Verändert: je Postleitzahl nur der gemittelte Ort (Breite/Länge auf drei Nachkommastellen). Verwendung: Entfernungsschätzung im Pager-Verzeichnis des Notfunks.
