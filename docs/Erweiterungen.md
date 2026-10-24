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
| `on(ereignis, fn)` | Ereignisse: `rig`, `station`, `spot`, `qso`, `decode`, `cluster`, `rotor`, `winlink`, `radios`, `spectrum`, `settings`, `plugin-settings` sowie eigene Meldungen des Server-Teils (über ihren `type`). Wird beim Ausschalten automatisch abgemeldet. |
| `api(pfad, optionen)` / `post(pfad, daten)` | REST-API. Pfad ohne führenden `/` = eigene Routen des Server-Teils (`/api/plugins/<id>/…`) |
| `setFrequency(hz)`, `setMode(mode)` | Funkgerät abstimmen |
| `logQso({call, band, mode, …})` | QSO eintragen (ADIF-Feldnamen) |
| `addPanel(register, schlüssel, {title, create})` | Fenster in einem Register: `rig`, `log`, `dm`, `wl`, `sat`, `rot`, `map` |
| `addApp(schlüssel, {title, icon, panels})` | eigenes Hauptregister (wird beim nächsten Laden angezeigt) |
| `addMenuItem(text, fn, register?)` | Punkt im Menü „Erweiterungen“ |
| `storage.get/set` | kleine Ablage im Browser |
| `toast`, `esc`, `icon`, `dialog` | Hilfsfunktionen der Oberfläche |

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

Bibliotheken (z. B. eine DLL) legt die Erweiterung in ihren eigenen Ordner und lädt sie selbst (`ctypes`, Pfad über `Path(__file__).parent`). Unter macOS tragen aus dem Netz geladene Dateien eine Quarantäne-Markierung; ggf. selbst entfernen (`xattr -d com.apple.quarantine <datei>`).

## Sicherheit

Erweiterungen sind nach der Installation **aus**. Eingeschaltet haben sie dieselben Rechte wie RigVibe – sie können das Funkgerät steuern (auch senden), das Logbuch ändern und Dateien lesen. Nur Erweiterungen aus vertrauenswürdiger Quelle einschalten.
