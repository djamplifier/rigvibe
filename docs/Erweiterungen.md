# Erweiterungen für RigVibe entwickeln

RigVibe lässt sich mit **Erweiterungen** (Plugins) ausbauen – mit eigenen Fenstern, Registern und Menüpunkten in der Oberfläche und eigenen Funktionen im Server. Wer lieber von außen steuert, nutzt die **API** (`/api/docs` im laufenden Programm) oder den **MCP-Server** für KI-Assistenten (siehe Anleitung, Kapitel „Zugang, API und KI-Assistenten“).

Ein vollständiges Beispiel liegt bei: **F8 → Erweiterungen → Beispiel „Stationsanzeige“ installieren**, dann einschalten. Den Quelltext findest du im Datenordner unter `plugins/beispiel/` (bzw. im Repository unter `beispiele/plugins/beispiel/`).

## Aufbau

Eine Erweiterung ist ein Ordner im Datenordner unter `plugins/<id>/` (Datenordner: Windows `%APPDATA%\RigVibe`, macOS `~/Library/Application Support/RigVibe`, Linux `~/.local/share/wellenwerk`). Zum Weitergeben den Ordner als ZIP packen; installiert wird über **F8 → Erweiterungen → ZIP installieren**.

```
plugins/meine-erweiterung/
  manifest.json     Pflicht
  plugin.js         optional: Oberflächen-Teil
  plugin.py         optional: Server-Teil
  daten/            wird angelegt: Ablage für den Server-Teil (nicht über das Netz abrufbar)
```

### manifest.json

```json
{
  "id": "meine-erweiterung",
  "name": "Meine Erweiterung",
  "version": "1.0",
  "author": "DL0ABC",
  "description": "Was sie tut – erscheint in den Programmoptionen.",
  "frontend": "plugin.js",
  "backend": "plugin.py",
  "settings": [
    { "key": "server", "label": "Server-Adresse", "type": "text", "default": "" },
    { "key": "kennwort", "label": "Kennwort", "type": "password" },
    { "key": "intervall", "label": "Intervall (s)", "type": "number", "default": 30 },
    { "key": "aktiv", "label": "automatisch senden", "type": "bool", "default": false },
    { "key": "band", "label": "Band", "type": "select", "options": ["20m", "40m"], "default": "20m" }
  ]
}
```

`id`: Kleinbuchstaben, Ziffern, `-` und `_`. Die **Einstellungen** erzeugt RigVibe selbst als Formular in den Programmoptionen; Werte vom Typ `password` werden nie wieder an die Oberfläche geschickt.

## Oberflächen-Teil (JavaScript)

```js
RigVibe.register("meine-erweiterung", {
  start(ctx) {
    ctx.addPanel("log", "liste", {               // Fenster im Register Logbuch
      title: "Meine Liste",
      create(root) {
        const draw = () => root.textContent = `${ctx.station().callsign} – ${ctx.settings.band}`;
        ctx.on("station", draw);                  // Stationsdaten geändert (Profil, Rufzeichen, Locator …)
        ctx.on("plugin-settings", draw);          // eigene Einstellungen geändert
        draw();
      },
    });
    ctx.addMenuItem("Meine Aktion", () => ctx.toast("Hallo"));
  },
  stop() { /* aufräumen */ },
});
```

| `ctx.…` | Bedeutung |
|---|---|
| `station()` | Stationsdaten: `callsign`, `home_callsign`, `operator`, `locator`, `lat`, `lon`, `tx_pwr`, `profile`, `my_pota_ref`, `my_sota_ref`, `my_wwff_ref`, `winlink_call`, `lang` (Sprache der Station) |
| `lang`, `locale()`, `t(text)` | Sprache der Oberfläche in diesem Browser (`de`, `en`), passendes Gebietsschema für Datum/Zahlen, Übersetzung eines deutschen Textes |
| `settings` | eigene Einstellungen (aktuelle Werte) |
| `rig()` | Zustand des aktuellen Funkgeräts (`freq` in Hz, `mode`, `ptt`, `meters` …) |
| `on(ereignis, fn)` | Ereignisse: `rig`, `station`, `spot`, `qso`, `decode`, `cluster`, `rotor`, `winlink`, `radios`, `spectrum`, `settings`, `plugin-settings`, `tx_beginn`, `tx_ende`, `tx_unterlauf` sowie eigene Meldungen des Server-Teils (über ihren `type`). Wird beim Ausschalten automatisch abgemeldet. |
| `api(pfad, optionen)` / `post(pfad, daten)` | REST-API. Pfad ohne führenden `/` = eigene Routen des Server-Teils (`/api/plugins/<id>/…`) |
| `setFrequency(hz)`, `setMode(mode)` | Funkgerät abstimmen |
| `logQso({call, band, mode, …})` | QSO eintragen (ADIF-Feldnamen) |
| `addPanel(register, schlüssel, {title, create})` | Fenster in einem Register: `rig`, `log`, `dm`, `wl`, `sat`, `rot`, `map` |
| `addApp(schlüssel, {title, icon, panels})` | eigenes Hauptregister (wird beim nächsten Laden angezeigt) |
| `addMenuItem(text, fn, register?)` | Punkt im Menü „Erweiterungen“ |
| `storage.get/set` | kleine Ablage im Browser |
| `toast`, `esc`, `icon`, `dialog` | Hilfsfunktionen der Oberfläche |
| `tx`, `audio`, `modes()`, `selectMode()`, `wege()`, `apiVersion`, `caps` | eigene Betriebsarten: senden ohne Taste, Ton, Wahl am Gerät (siehe [Eigene Betriebsarten](#eigene-betriebsarten)) |

### Mehrsprachigkeit

Die Oberfläche übersetzt alle sichtbaren Texte selbst anhand von `static/i18n/<sprache>.json`. Eigene Texte einer Erweiterung schreibst du deutsch und lieferst die Übersetzungen mit:

```js
RigVibe.addTranslations("en", { "Meine Liste": "My list", "{} QSOs heute": "{} QSOs today" });
```

`{}` steht für einen beliebigen Teil (Zahl, Rufzeichen …). Bereiche, die nicht übersetzt werden sollen (empfangene Texte, Daten), bekommen `translate="no"`. Weitere Sprachen für das ganze Programm: eine Datei `static/i18n/<sprache>.json` nach dem Muster von `en.json` anlegen und in `static/js/i18n.js` bei `LANGS`/`LOCALES` eintragen.

## Server-Teil (Python)

```python
def setup(ww):
    @ww.router.get("/status")                 # erreichbar als GET /api/plugins/<id>/status
    async def status():
        return {"rufzeichen": ww.station()["callsign"], "einstellungen": ww.settings()}

    async def neues_qso(msg):                 # jede Meldung an die Oberfläche kommt auch hier an
        q = msg["qso"]
        ww.log.info("neues QSO mit %s", q["call"])
        await ww.broadcast({"type": "mein_ereignis", "call": q["call"]})

    ww.on("qso", neues_qso)
```

| `ww.…` | Bedeutung |
|---|---|
| `router` | FastAPI-`APIRouter` unter `/api/plugins/<id>` (erscheint in `/api/docs`) |
| `on(typ, fn)` | Meldungen abonnieren (`rig`, `qso`, `spot`, `station_identity`, `winlink`, `decode`, `radios` … oder `*`) |
| `station()`, `settings()`, `program_settings()` | Stationsdaten, eigene Einstellungen, Programmoptionen (Kennwörter maskiert) |
| `broadcast(meldung)` | Meldung an alle Fenster (kommt dort als `ctx.on(type, …)` an) |
| `rig`, `radios` | aktuelles Funkgerät (`await ww.rig.set_freq(hz)` …) bzw. Geräteverwaltung |
| `log_qso(rec)` | QSO eintragen (mit Land, Callbook, Weiterleitung, Uploads) |
| `register_receiver(art, klasse, name)` | eigene Empfänger-Art (Unterklasse von `wellenwerk.sdr.base.SdrRig`) |
| `add_frame_decoder(kanal, fabrik, name)` | Rahmen eines Digitalfunk-Kanals entgegennehmen und Ton liefern (siehe [Rahmen-Schnittstelle](#rahmen-schnittstelle)) |
| `add_frame_encoder(kanal, fabrik, name)` | Senderichtung: Ton entgegennehmen und Rahmen eines Kanals liefern (siehe [Rahmen-Schnittstelle](#rahmen-schnittstelle)) |
| `add_mode(id, spec)`, `add_detector(id, fn)` | eigene Betriebsart mit Senden und Empfangen anmelden (siehe [Eigene Betriebsarten](#eigene-betriebsarten)) |
| `tx_start(…)`, `tx_stop()`, `audio(…)`, `heard(…)`, `wege()`, `api_version`, `caps` | Senden ohne Taste, Ton, Anzeige, Gerätewege, Versionsprüfung (siehe [Eigene Betriebsarten](#eigene-betriebsarten)) |
| `data_dir`, `log` | eigener Datenordner, Protokoll |

Der Server-Teil wird beim Einschalten geladen und beim Ausschalten sofort abgemeldet (Ereignisse und API-Routen); eine Funktion `teardown()` im Modul wird dann aufgerufen, falls vorhanden.

## Rahmen-Schnittstelle

Über die Rahmen-Schnittstelle bekommt eine Erweiterung die Nutzrahmen eines Digitalfunk-Kanals, die RigVibe beim eigenen Dekodieren herausliest, und gibt Ton zurück. RigVibe gibt diesen Ton im Empfänger aus, solange der Nutzer ihn in der Live-Karte des Dekoders einschaltet.

```python
class MeinDekoder:
    def decode(self, bits):             # ein Rahmen: Liste von 0/1 in Sendereihenfolge
        ...
        return werte                    # 160 Zahlen (20 ms bei 8 kHz) um -1..1, oder [] für Stille

def setup(ww):
    ww.add_frame_decoder("dmr", lambda: MeinDekoder(), "Mein Dekoder · DMR")
```

RigVibe ruft die Fabrik (`lambda: …`) für jede neue Aussendung auf – der Dekoder darf also Zustand von Rahmen zu Rahmen mitführen – und danach `decode()` für jeden Rahmen, in einem eigenen Faden. Den Pegel gleicht RigVibe an und rechnet auf seinen Tonweg um.

| Kanal | Rahmen (Bits in Sendereihenfolge) | Takt |
|---|---|---|
| `dmr` | 72 Bit: ein Drittel der 108 Nutzdibits eines Bursts mit Sprach-Sync bzw. EMB (Dibits 0–35, 36–71, 72–107; je Dibit erst das höherwertige Bit). Die Nutzdibits sind die 54 vor und die 54 nach dem mittleren Feld. Bei mehreren Zeitschlitzen folgt RigVibe dem, der zuerst sendet. | 3 Rahmen je 60 ms |
| `dstar` | 72 Bit: die ersten 72 Bit eines Rahmens (vor den 24 Bit Slow Data), ohne Synchronwort | 1 Rahmen je 20 ms |
| `ysf` | 104 Bit: ein VCH-Feld im V/D-Modus 2, unverändert (verschachtelt und geweisst) | 5 Rahmen je 100 ms |
| `m17` | 128 Bit: Nutzteil eines Stream-Rahmens (nach der Rahmennummer). RigVibe gibt diesen Kanal selbst aus (Codec2); eine Erweiterung kann ihn übernehmen. | 1 Rahmen je 40 ms |

Je Kanal gilt die zuletzt geladene Erweiterung. Wirft `decode()` einen Fehler, bleibt die Aussendung stumm, der Empfang läuft weiter; der Fehler steht im Protokoll. Ein lauffähiges Beispiel liegt bei: **F8 → Erweiterungen → Beispiel „Rahmen-Schnittstelle: Testton“ installieren** – es gibt je Rahmen einen Ton aus.

**Senderichtung.** Umgekehrt nimmt ein Kodierer Ton entgegen und liefert Rahmen im selben Format wie oben:

```python
class MeinKodierer:
    def encode(self, werte):            # 160 Zahlen (20 ms bei 8 kHz, bei m17 320 = 40 ms) um -1..1
        ...
        return bits                     # ein Rahmen: Liste von 0/1 in Sendereihenfolge (Länge wie in der Tabelle)

def setup(ww):
    ww.add_frame_encoder("m17", lambda: MeinKodierer(), "Mein Kodierer · M17")
```

Die Fabrik wird je Aussendung aufgerufen, `encode()` je Rahmen; am Ende folgt ein Rahmen Stille, damit der Kodierer leer wird. Liefert `encode()` eine falsche Bitzahl oder wirft einen Fehler, wird nicht gesendet. RigVibe nutzt die Senderichtung beim Senden von M17 (eingebaut: Codec2). Welche Kanäle gerade eine Erweiterung haben, zeigt `GET /api/voiceplugins`. Die Einstellungen der Station (z. B. `dmr_id`) liest eine Erweiterung mit `ww.station()`, eigene Werte legt sie als `settings` im Manifest an.

Bibliotheken (z. B. eine DLL) legt die Erweiterung in ihren eigenen Ordner und lädt sie selbst (`ctypes`, Pfad über `Path(__file__).parent`). Unter macOS tragen aus dem Netz geladene Dateien eine Quarantäne-Markierung; ggf. selbst entfernen (`xattr -d com.apple.quarantine <datei>`).

## Eigene Betriebsarten

Eine Erweiterung kann eine Betriebsart **vollständig selbst mitbringen**: Kodierung, Rahmenaufbau, Modulation, Demodulation und Dekodierung. RigVibe muss den Inhalt der Signale nicht kennen. Es stellt nur bereit, was es ohnehin verwaltet: Geräte mit ihren Sende- und Empfangswegen, PTT und Ablauf, Mikrofon, Tonausgabe, Bedienung und Anzeige. Abtastwerte bzw. Bits gehen **unverändert** durch – einzige Ausnahmen sind die je Gerät eingemessene Hub-/Polaritätskorrektur (ein fester Faktor) und, wo ein Gerät eine andere Rate braucht, ein phasenlinearer FIR; beides steht in `info["weg"]`.

Vollständiges Beispiel zum Abschauen: **F8 → Erweiterungen → Beispiel „Eigene Betriebsart: FSK-Text“ installieren** (Quelltext `beispiele/plugins/fsk-text/`). Es sendet und empfängt kurze Texte mit 1200 bit/s FSK, meldet Empfangenes im Display, gibt einen Quittungston aus, bringt eine Signalerkennung mit und hat einen Oberflächen-Teil, der ohne Taste sendet.

### 1. Anmelden

```python
def setup(ww):
    if ww.api_version < 2 or "add_mode" not in ww.caps:      # ältere RigVibe-Version: sauber melden
        ww.log.warning("RigVibe zu alt für eigene Betriebsarten")
        return
    ww.add_mode("meine-art", {
        "name": "Meine Betriebsart",
        "senden": ["basisband"],            # Sendeformen, die die Erweiterung liefern kann: basisband, bitstrom, iq
        "empfangen": ["diskriminator"],     # Empfangsformen, die sie braucht: diskriminator, iq
        "bandbreite_hz": 12500,             # belegte Kanalbreite (Bandprüfung, Kanalfilter des Empfangs)
        "hub_hz": 3000,                     # Nennhub: ±1,0 im Basisband = ±hub_hz
        "felder": [                         # Eingabefelder für die Aussendung (Typen wie bei "settings")
            {"key": "ziel", "label": "Ziel", "type": "text", "default": ""},
        ],
        "mikrofon": {"rate": 8000, "block": 160},   # gewünschtes Mikrofonformat oder None
        "bitraten": [4800],                 # nur bei "bitstrom": gewünschte Bitraten (erste passende gilt)
        "max_s": 180,                       # Höchstdauer je Aussendung (vom Nutzer änderbar)
        "aufbereitung": False,              # Mikrofon standardmäßig unbearbeitet (Nutzer kann umschalten)
        "vox": False,                       # VOX in dieser Betriebsart nur, wenn ausdrücklich erlaubt
        "adif": {"mode": "DYNAMIC"},        # Betriebsart fürs Logbuch, wenn ein Gehörter übernommen wird
        "sender": lambda info: MeinSender(info),
        "empfaenger": lambda info: MeinEmpfaenger(info),
    })
```

RigVibe behandelt die Betriebsart **wie eine eigene**: Sie steht in allen Betriebsartenlisten (Mode-Taste und Menü am Funkgerät, „Bänder & Betriebsarten“, Kontextmenü, Digitalbetrieb mit Auswahl und Menü *Betriebsart*, Handy-Ansicht) und heißt intern `ext:<id>` – so lässt sie sich auch in Favoriten und Speicherkanälen ablegen und über die API wählen (`POST /api/rig/mode {"mode": "ext:<id>"}`). Beim Wählen stellt RigVibe das Gerät auf FM (Diskriminator bzw. Frequenzmodulation) und schaltet in den Weg der Erweiterung; jede andere Betriebsart wählt sie wieder ab. Das Fenster **Erweiterungs-Betriebsart** zeigt Sendetaste, Eingabefelder und empfangene Stationen. Die Werte der Felder merkt sich RigVibe je Betriebsart, auf Wunsch („je Kanal“) je Frequenz, also je Speicherkanal bzw. Relais. Die Wahl je Gerät bleibt über einen Neustart erhalten.

### 2. Senden: Ablauf

Für jede Aussendung ruft RigVibe `sender(info)` auf. `info` enthält `station` (wie `ww.station()`), `felder`, `weg` (Beschreibung des Sendewegs, Abschnitt 4), `form` (die gewählte Sendeform), `frequenz`, `ablage` (Relaisablage bzw. Split in Hz), `geraet`, `betriebsart`, `trocken` und `quelle`. Das zurückgegebene Objekt hat diese Methoden; RigVibe ruft sie **in einem eigenen Faden** auf, nie in der asyncio-Schleife:

| Methode | Wann | Rückgabe |
|---|---|---|
| `start()` | einmal zu Beginn, noch vor der PTT | erster Teil des Signals (Vorlauf, Kopf) |
| `mikrofon(block)` | je Mikrofonblock im angemeldeten Format (Gleitkomma, mono) | Signal für diesen Abschnitt (darf leer sein) |
| `fuellen(n)` | wenn der Ausgabepuffer weniger als den Vorrat enthält | bis zu `n` weitere Werte; **`None` = nichts mehr, Aussendung zu Ende** |
| `stop()` | Sendetaste losgelassen bzw. `ctx.tx.stop()` / `ww.tx_stop()` | Abschluss des Signals |
| `abbruch(grund)` | `zeit` (Höchstdauer), `geraet` (Gerät/Tonausgang weg), `notaus`, `ptt` (PTT von außen beendet), `fehler` | – |
| `zeit(t)` | optional, vor jedem `fuellen()`/`mikrofon()` | – (`t` = geschätzte Unix-Zeit, zu der der nächste gelieferte Wert gesendet wird) |

Zugesagt ist:

- **Vorrat statt Stille:** RigVibe füllt vor der PTT einen Vorrat (Standard 200 ms, je Gerät einstellbar) und fordert über `fuellen(n)` nach. Läuft der Puffer trotzdem leer, kommt das Ereignis `tx_unterlauf` (Server: `ww.on("tx_unterlauf", …)`, Oberfläche: `ctx.on("tx_unterlauf", …)`), und es steht im Protokoll der Erweiterung.
- **PTT bis zum letzten Wert:** Nach `stop()` (oder `None` aus `fuellen`) gibt RigVibe alles Gelieferte vollständig aus – einschließlich der Puffer im Gerätetreiber – und löst erst dann die PTT. Danach kommt `tx_ende` mit `ok`, `grund`, `dauer`, `werte` (Anzahl gesendeter Werte), `unterlaeufe`, beim Trockenlauf `datei`, beim Mitschnitt `mitschnitt`.
- **Vorlaufzeit je Gerät:** Zeit zwischen PTT und erstem Wert (Einschwingen des Senders), steht in `info["weg"]["vorlauf_ms"]`. Beim SDR läuft sie als unmodulierter Träger.
- **Ohne Taste senden:** Oberfläche `await ctx.tx.start(felder, {betriebsart, quelle, trocken, rueckspielen, mitschnitt})` / `ctx.tx.stop()`, Server `await ww.tx_start(felder, radio=…, quelle=…)` / `await ww.tx_stop()` – mit denselben Sperren wie die Sendetaste.
- **Eine PTT für alles:** Ist die Betriebsart an einem Gerät gewählt, führen Sendetaste, Leertaste, MIDI-Pult, Handy-Ansicht, Fernbedienung über die API und der Sprachspeicher alle zu diesem Ablauf.

### 3. Sendeformen

**a) Basisband** – Gleitkommawerte, `±1,0 = ±hub_hz`, Abtastrate `weg["rate"]`. Keine Sendeton-Kette (Equalizer, Kompressor, Begrenzer, Rauschsperre, VOX), keine Pegelregelung. Über der Soundkarte spielt die Bedienoberfläche die Werte mit der Rate ihres Ausgangs aus (meist 48 kHz); eigene SDRs bekommen 48 kHz, die RigVibe mit einem phasenlinearen FIR auf die Geräterate bringt und ohne Preemphasis und ohne Hochpass frequenzmoduliert (gleichanteilfähig).

**b) Bitstrom** – Bits (0/1) in Sendereihenfolge für Geräte mit eigener FSK-Einheit; die Betriebsart nennt ihre Bitraten unter `"bitraten"` (RigVibe nimmt die erste, die das Gerät kann, und meldet sie als `weg["bitrate"]`). `fuellen(n)` zählt dann in Bits. Vorhanden beim **Quansheng UV-K1/K5 ab Firmware-Stand 14** – ohne Soundkarte, nur über USB:
- 1 200, 2 400, 4 800 oder 6 000 bit/s, fortlaufend ohne Längengrenze (gemessen: 48 000 Bit am Stück bei 4 800 bit/s, 13 Bitfehler bei kleinster Leistung, kein Versatz, kein Unterlauf), RigVibe lädt nach freiem Platz im Puffer des Geräts nach.
- Die FSK-Einheit setzt **8 Bit Präambel** (10101010) davor; das lässt sich nicht abschalten. Die **ersten 16 Bit** der Erweiterung dienen ihr als Sync-Wort und gehen unverändert hinaus – danach folgt der Strom Bit für Bit. Keine Prüfsumme, keine Weißung. `weg["bitstrom"]` beschreibt das.
- Ende genau nach dem letzten Bit, auf 8 Bit gerastert (Rest mit 0 aufgefüllt).
- Hub aus `hub_hz` der Betriebsart (gemessen bei 4 800 bit/s etwa 450 Hz … 2,4 kHz), Bit 1 = höhere Frequenz.

**c) IQ** – komplexe Werte bei `weg["rate"]` relativ zur Sendefrequenz, Betrag ≤ 1 (Betrag 1 = Leistung wie bei FM). Nur eigene SDRs.

Werte über ±1 gibt RigVibe unverändert weiter und vermerkt es einmal im Protokoll. **Basisband direkt über USB** (ohne Soundkarte) bietet der Quansheng nicht: Der Funkchip ändert den Hub nur über den analogen Mikrofonweg, die Tongeneratoren (Sinus fester Frequenz), die FSK-Einheit (zwei Stufen) oder durch Umstimmen des Synthesizers (20–40 ms Einschwingen), und vom Prozessor führt keine Leitung zum Mikrofoneingang.

### 4. Beschreibung des Sendewegs (`weg`)

```python
{"geraet": "r3", "name": "K1", "art": "soundcard",      # soundcard | sdr | empfaenger | trocken
 "formen": ["basisband", "bitstrom"], "rate": 48000, "bitraten": [1200, 2400, 4800, 6000],
 "untere_grenze_hz": 65, "obere_grenze_hz": 3000, "gleichanteil": False,
 "hub_kalibriert": True, "polaritaet": -1, "vorlauf_ms": 150, "latenz_ms": 235,
 "umgerechnet": "",                                      # bei SDRs: welcher FIR auf welche Rate
 "frequenzgang": "…", "vorentzerrer_fir": [ … ],         # gemessene Wege (Quansheng): Kehrwert des Frequenzgangs, 48 kHz
 "hinweise": ["…Masseschleifen…"], "firmware": 14, "gesperrt": False, "flach": True,
 "bitstrom": {"vorlauf_bits": 8, "raster_bits": 8, "max_bps": 6000, "hinweis": "…"}}   # Bitstrom-Weg (Quansheng ab Stand 14)
```

Damit kann die Erweiterung selbst vorentzerren, wenn der Weg tiefe oder hohe Frequenzen dämpft – das Beispiel FSK-Text faltet sein Signal mit `vorentzerrer_fir`, wenn es da ist. `GET /api/extmodes/wege` bzw. `ww.wege()` liefert die Wege aller Geräte, `ww.radios` die Geräte selbst (mit Firmware-Stand).

### 5. Mikrofon und andere Tonquellen

Bei `"mikrofon": {"rate", "block"}` kommt der Mikrofonton der Bedienoberfläche als Blöcke genau in diesem Format (mono, Gleitkomma) zu `mikrofon(block)` – wahlweise **unbearbeitet** (Standard) oder mit RigVibes Sendeton-Aufbereitung (Schalter „Aufbereitung“). VOX tastet in dieser Betriebsart nicht, außer `"vox": True`. Statt des Mikrofons geht als Quelle auch ein **Sprachspeicher** (`quelle="sprachspeicher:3"`) oder eine **Tondatei** (`quelle="datei:/pfad/ruf.wav"`, 16 Bit); am Ende der Quelle folgt automatisch `stop()`. `quelle="keine"` sendet ohne Ton (Daten).

### 6. Empfangen

Ist die Betriebsart an einem Gerät gewählt, ruft RigVibe `empfaenger(info)` auf (`info` mit `weg`, `form`, `station`, `felder`, `frequenz`, `geraet`). Das Objekt bekommt, ebenfalls in einem eigenen Faden:

| Methode | Inhalt |
|---|---|
| `werte(block, t)` | Rohsignal des Demodulators: FM-Diskriminator ungefiltert, ohne Deemphasis, mit Gleichanteil, soweit das Gerät ihn liefert; `t` = Unix-Zeit des ersten Werts |
| `iq(block, t)` | eigene SDRs, wenn `"iq"` unter `empfangen` steht: komplexe Werte des Kanals (48 kHz) |
| `luecke(t, dauer)` | verlorene Werte (USB-Ring voll, Block zu spät, eigene Aussendung) |
| `pegel(dbm)` | Signalstärke, etwa zweimal je Sekunde |
| `weg(weg)` | Empfangsweg hat sich geändert (z. B. Takt des Quansheng nachgeführt → neue Rate) |
| `pause()` / `weiter()` | eigene Aussendung beginnt bzw. ist zu Ende (Halbduplex) |
| `ende()` | Betriebsart abgewählt, Erweiterung aus, Gerät weg |

Empfangswege (`info["weg"]`: `rate`, `einheit`, `untere_grenze_hz`, `obere_grenze_hz`, `gleichanteil`, `ausgleich`, `ppm`, `takt_ausgeglichen`, `quelle`):

| Gerät | Form | Rate | Einheit |
|---|---|---|---|
| eigene SDRs (HackRF, Pluto, RTL-SDR, Hermes Lite 2 …) | diskriminator, iq | 48 kHz, eigener Kanal ±Bandbreite/2 | Hz |
| Quansheng UV-K1/K5 mit RigVibe-Firmware ab Stand 12 | diskriminator | 38,4 kHz (Takt ausgeglichen) | roh (12-Bit-Wandler, Mitte abgezogen) |
| Quansheng ältere Stände, OpenWebRX, KiwiSDR | diskriminator | 9,6 bzw. 12 kHz (Ton des Empfängers) | roh |
| Funkgerät an der Soundkarte | diskriminator | 24 kHz aus dem Eingang des Digitalbetriebs | roh |

Beim Quansheng läuft der rohe Weg auch dann, wenn RigVibes eigene Dekoder aus sind.

**Rückwege:**

- **Ton:** `ww.audio(werte, rate)` bzw. `ctx.audio(werte, rate)` – geht in den normalen Empfangston (Lautstärke, Rekorder, Fernbetrieb, Handy); Regler „Ton“ je Betriebsart. Bei Funkgeräten ohne Ton im Dienst spielt das Fenster der Betriebsart den Ton.
- **Anzeige:** `ww.heard({"rufzeichen", "name", "ziel", "text", "felder": {…}, "lat", "lon"})` – erscheint wie bei RigVibes eigenen Dekodern unter der Frequenz, in „zuletzt gehört“ (von dort ins Logbuch, Betriebsart aus `adif`), im Fenster der Betriebsart und als Meldung `decode` mit `decoder: "ext:<betriebsart>"`, `kind: "ext"`. `ww.heard_end()` beendet die Zeile.
- **Signalerkennung:** `ww.add_detector("meine-art", fn)` – `fn(werte, rate)` bekommt die letzten 1,5 s Diskriminator und liefert 0..1 (kurze Aussendungen: besten Abschnitt werten, siehe Beispiel). Ab 0,6 ordnen der Klick in den Wasserfall, der Suchlauf und die Digitalfunk-Suche im Digitalbetrieb das Signal der Erweiterung zu und schalten in ihren Weg – wie bei RigVibes eigenen Dekodern, mit Live-Anzeige in der Karte der Signalerkennung. **Ohne eigene Erkennung** probiert die Signalerkennung die Betriebsart beim Durchprobieren der Dekoder mit: Sie wählt sie am Gerät und wartet, ob `ww.heard()` innerhalb gut 2 s Aussendung etwas meldet.
- `ww.heard()` und `ww.audio()` ohne Geräteangabe gelten dem Gerät, dessen Empfänger sie aufruft (aus `werte()`/`iq()` heraus); sonst `radio=` angeben.

### 7. Gerätewege, Vorlauf und Einmessen

**Gerät …** im Fenster der Betriebsart: Vorlaufzeit, Vorrat, Latenz, Polarität, Tonwert je kHz Hub, Sendesperre. **Einmessen** (Soundkarten-Weg): sendet 2 s Prüfton (Spitze = voller Hub) und misst mit einem eigenen SDR auf derselben Frequenz Hub und Polarität; danach gilt `weg["hub_kalibriert"] = True`. Der Sendepegel des Geräts ist Teil der Messung – danach nicht mehr verstellen. Eigene SDRs sind ab Werk genau.

Ausgabe über die Soundkarte: Ein offenes RigVibe-Fenster ist der **Bedienplatz** – für jedes verbundene Gerät mit gewählter Erweiterungs-Betriebsart ein eigener Kanal mit dem Tonausgang (Soundkarte) und Sendepegel dieses Geräts, so dass z. B. ein Funkgerät an der Soundkarte und ein SDR als Messempfänger gleichzeitig laufen. Die Werte gehen ohne Bearbeitung hinaus, PTT über den Weg des Geräts (CAT, RTS/DTR, USB), zeitlich auf den ersten Wert abgestimmt (Vorlaufzeit).

**Quansheng UV-K1/K5 über die USB-Soundkarte am Mikrofon:** Ab Firmware-Stand 13 schaltet RigVibe für jede Aussendung einer Erweiterung den **flachen Sende-Tonweg** ein (`weg["flach"]`, je Gerät abschaltbar): Die Firmware setzt nach dem Einrichten des Senders Preemphasis, Hoch-/Tiefpass, Gleichanteilfilter, Kompander und Mikrofon-Regelung aus, hält das für die ganze Aussendung und stellt danach alles zurück; Hub und Mikrofonverstärkung lassen sich fest vorgeben (`k1_hub`, `k1_mikro`). Gemessen mit HackRF: Was übrig bleibt, ist die analoge Beschaltung – +9 dB um 100–150 Hz, unter 60 Hz steil abfallend, −3 dB bei 3 kHz, linear bis Vollaussteuerung, Polarität umgekehrt. Das steht als `frequenzgang`, `untere_grenze_hz`/`obere_grenze_hz` und `vorentzerrer_fir` in `weg`, dazu der Hinweis auf Masseschleifen. Die PTT erneuert RigVibe während langer Aussendungen selbst.

### 8. Sperren und Sicherheit

Für Aussendungen von Erweiterungen gilt dasselbe wie für RigVibes eigene Betriebsarten: eingetragenes Rufzeichen, Prüfung gegen die Amateurfunkbänder (Frequenz ± Bandbreite/2, inkl. Ablage), Sendesperre je Gerät, Höchstdauer je Aussendung, **Not-Aus** (Knopf im Fenster bzw. `POST /api/extmodes/notaus`; `Esc` beendet laufende Aussendungen) und der PTT-Schutz (ohne verbundene Bedienoberfläche wird nach 3 s abgeschaltet). Solange gesendet wird, zeigt jedes Fenster oben **„sendet (Erweiterung …)“**; ein Klick beendet.

### 9. Entwicklungs- und Testhilfen

- **Trockenlauf:** ohne PTT; das Signal geht nach dem Sendeweg (Kalibrierfaktor eingerechnet) als `.wav`/`.npy` in den Ordner `daten/` der Erweiterung und auf Wunsch („zurückspielen“) direkt in den eigenen Empfänger der Erweiterung – senden und dekodieren lassen sich so ohne Funkgerät prüfen.
- **Mitschnitt:** genau die Werte, die an das Gerät gingen (beim SDR die 48-kHz-Stufe vor dem Interpolator), mit geschätzter Sendezeit je Block (`mitschnitt-….npy` + `.json`).
- **Protokoll:** Unterläufe, Lücken, Zeitüberschreitungen und Fehler der Erweiterung – Knopf **Protokoll** bzw. `GET /api/pluginlog/<id>`.

### 10. Versionierung und Schnittstellen

`ww.api_version` (jetzt `2`) und `ww.caps` (Menge der vorhandenen Funktionen, z. B. `tx.basisband`, `tx.iq`, `rx.iq`, `tx.trockenlauf`, `heard`, `audio`) – in der Oberfläche `ctx.apiVersion` und `ctx.caps`. Eine Erweiterung prüft damit, was die laufende RigVibe-Version kann, und meldet sauber, was fehlt.

| Server `ww.…` | Oberfläche `ctx.…` | Bedeutung |
|---|---|---|
| `add_mode(id, spec)`, `add_detector(id, fn)` | – | Betriebsart, Signalerkennung anmelden |
| `tx_start(…)`, `tx_stop()` | `tx.start(felder, opts)`, `tx.stop()`, `tx.state()`, `tx.notaus()` | senden ohne Taste |
| `audio(werte, rate)` | `audio(werte, rate)` | Ton in den Empfang |
| `heard(d)`, `heard_end()` | – | gehörte Station melden |
| `wege()` | `wege()` | Sende-/Empfangswege aller Geräte |
| – | `modes()`, `activeMode()`, `selectMode(id)` | Betriebsarten der Erweiterungen, Wahl am Gerät |

HTTP (für Fremdprogramme, `/api/docs`): `GET /api/extmodes`, `GET /api/extmodes/wege`, `POST /api/extmodes/select`, `POST /api/extmodes/tx`, `POST /api/extmodes/tx/stop`, `POST /api/extmodes/notaus`, `POST /api/extmodes/kalibrieren`, `PUT /api/extmodes/<id>/felder|optionen`, `PUT /api/extmodes/geraet/<gerät>`, `GET /api/pluginlog/<id>`.

**Grenzen:** Basisband über USB gibt es beim Quansheng nicht (siehe 3). Betriebsarten mit Mikrofon und Soundkarten-Wege brauchen ein offenes RigVibe-Fenster (Browser oder Programm); die Handy-Ansicht wählt und tastet, liefert aber kein Mikrofon für Erweiterungen. Eigene SDRs und der Bitstrom des Quansheng brauchen kein Fenster als Tonausgang.

## Sicherheit

Erweiterungen sind nach der Installation **aus**. Eingeschaltet haben sie dieselben Rechte wie RigVibe – sie können das Funkgerät steuern (auch senden), das Logbuch ändern und Dateien lesen. Nur Erweiterungen aus vertrauenswürdiger Quelle einschalten.
