# Developing extensions for RigVibe

*Deutsch: [Erweiterungen.md](Erweiterungen.md)*

RigVibe can be extended with **extensions** (plugins) – with their own windows, tabs and menu items in the interface and their own functions in the server. If you prefer to control it from outside, use the **API** (`/api/docs` in the running program) or the **MCP server** for AI assistants (see the manual, chapter “Access, API and AI assistants”).

A complete example is included: **F8 → Extensions → install example “Stationsanzeige”**, then switch it on. The source code is in the data folder under `plugins/beispiel/` (or in the repository under `beispiele/plugins/beispiel/`).

## Structure

An extension is a folder in the data folder under `plugins/<id>/` (data folder: Windows `%APPDATA%\RigVibe`, macOS `~/Library/Application Support/RigVibe`, Linux `~/.local/share/wellenwerk`). To pass it on, zip the folder; it is installed via **F8 → Extensions → Install ZIP**.

```
plugins/my-extension/
  manifest.json     required
  plugin.js         optional: interface part
  plugin.py         optional: server part
  daten/            created automatically: storage for the server part (not reachable over the network)
```

### manifest.json

```json
{
  "id": "my-extension",
  "name": "My extension",
  "version": "1.0",
  "author": "N0CALL",
  "description": "What it does – shown in the program options.",
  "frontend": "plugin.js",
  "backend": "plugin.py",
  "settings": [
    { "key": "server", "label": "Server address", "type": "text", "default": "" },
    { "key": "password", "label": "Password", "type": "password" },
    { "key": "interval", "label": "Interval (s)", "type": "number", "default": 30 },
    { "key": "active", "label": "send automatically", "type": "bool", "default": false },
    { "key": "band", "label": "Band", "type": "select", "options": ["20m", "40m"], "default": "20m" }
  ]
}
```

`id`: lower-case letters, digits, `-` and `_`. RigVibe builds the **settings** form in the program options itself; values of type `password` are never sent back to the interface.

## Interface part (JavaScript)

```js
RigVibe.register("my-extension", {
  start(ctx) {
    ctx.addPanel("log", "list", {                 // window in the Logbook tab
      title: "My list",
      create(root) {
        const draw = () => root.textContent = `${ctx.station().callsign} – ${ctx.settings.band}`;
        ctx.on("station", draw);                  // station data changed (profile, callsign, locator …)
        ctx.on("plugin-settings", draw);          // own settings changed
        draw();
      },
    });
    ctx.addMenuItem("My action", () => ctx.toast("Hello"));
  },
  stop() { /* clean up */ },
});
```

| `ctx.…` | Meaning |
|---|---|
| `station()` | station data: `callsign`, `home_callsign`, `operator`, `locator`, `lat`, `lon`, `tx_pwr`, `profile`, `my_pota_ref`, `my_sota_ref`, `my_wwff_ref`, `winlink_call`, `lang` (language of the station) |
| `lang`, `locale()`, `t(text)` | interface language in this browser (`de`, `en`), matching locale for dates/numbers, translation of a German text |
| `settings` | own settings (current values) |
| `rig()` | state of the current radio (`freq` in Hz, `mode`, `ptt`, `meters` …) |
| `on(event, fn)` | events: `rig`, `station`, `spot`, `qso`, `decode`, `cluster`, `rotor`, `winlink`, `radios`, `spectrum`, `settings`, `plugin-settings`, `tx_beginn`, `tx_ende`, `tx_unterlauf` and the server part's own messages (by their `type`). Unsubscribed automatically on switch-off. |
| `api(path, options)` / `post(path, data)` | REST API. A path without leading `/` = the server part's own routes (`/api/plugins/<id>/…`) |
| `setFrequency(hz)`, `setMode(mode)` | tune the radio |
| `logQso({call, band, mode, …})` | log a QSO (ADIF field names) |
| `addPanel(tab, key, {title, create})` | window in a tab: `rig`, `log`, `dm`, `wl`, `sat`, `rot`, `map` |
| `addApp(key, {title, icon, panels})` | own main tab (shown on the next load) |
| `addMenuItem(text, fn, tab?)` | item in the “Extensions” menu |
| `storage.get/set` | small storage in the browser |
| `toast`, `esc`, `icon`, `dialog` | interface helpers |
| `tx`, `audio`, `modes()`, `selectMode()`, `wege()`, `apiVersion`, `caps` | custom operating modes: transmit without button, audio, selection on the radio (see [Custom operating modes](#custom-operating-modes)) |

### Languages

The interface translates all visible texts itself using `static/i18n/<language>.json` (German text → translation). Texts of an extension can be written in English if it only targets English users, or in German with translations supplied:

```js
RigVibe.addTranslations("en", { "Meine Liste": "My list", "{} QSOs heute": "{} QSOs today" });
```

`{}` stands for any part (number, callsign …). Areas that must not be translated (received text, data) get `translate="no"`. More languages for the whole program: create `static/i18n/<language>.json` following `en.json` and add it to `LANGS`/`LOCALES` in `static/js/i18n.js`.

## Server part (Python)

```python
def setup(ww):
    @ww.router.get("/status")                 # reachable as GET /api/plugins/<id>/status
    async def status():
        return {"callsign": ww.station()["callsign"], "settings": ww.settings()}

    async def new_qso(msg):                   # every message to the interface also arrives here
        q = msg["qso"]
        ww.log.info("new QSO with %s", q["call"])
        await ww.broadcast({"type": "my_event", "call": q["call"]})

    ww.on("qso", new_qso)
```

| `ww.…` | Meaning |
|---|---|
| `router` | FastAPI `APIRouter` under `/api/plugins/<id>` (appears in `/api/docs`) |
| `on(type, fn)` | subscribe to messages (`rig`, `qso`, `spot`, `station_identity`, `winlink`, `decode`, `radios` … or `*`) |
| `station()`, `settings()`, `program_settings()` | station data (including `lang`), own settings, program options (passwords masked) |
| `broadcast(message)` | message to all windows (arrives there as `ctx.on(type, …)`) |
| `rig`, `radios` | current radio (`await ww.rig.set_freq(hz)` …) and radio management |
| `log_qso(rec)` | log a QSO (with country, callbook, forwarding, uploads) |
| `register_receiver(kind, cls, name)` | own receiver type (subclass of `wellenwerk.sdr.base.SdrRig`) |
| `add_frame_decoder(channel, factory, name)` | receive the frames of a digital radio channel and return audio (see [Frame interface](#frame-interface)) |
| `add_frame_encoder(channel, factory, name)` | transmit direction: take audio and return frames of a channel (see [Frame interface](#frame-interface)) |
| `add_mode(id, spec)`, `add_detector(id, fn)` | register a custom operating mode with transmit and receive (see [Custom operating modes](#custom-operating-modes)) |
| `tx_start(…)`, `tx_stop()`, `audio(…)`, `heard(…)`, `wege()`, `api_version`, `caps` | transmit without button, audio, display, radio paths, version check (see [Custom operating modes](#custom-operating-modes)) |
| `data_dir`, `log` | own data folder, log |

The server part is loaded on switch-on and unregistered immediately on switch-off (events and API routes); a function `teardown()` in the module is then called if present.

## Frame interface

Through the frame interface an extension receives the payload frames of a digital radio channel that RigVibe reads out during its own decoding, and returns audio. RigVibe plays this audio in the receiver while the user switches it on in the decoder's live card.

```python
class MyDecoder:
    def decode(self, bits):             # one frame: list of 0/1 in transmission order
        ...
        return values                   # 160 numbers (20 ms at 8 kHz) around -1..1, or [] for silence

def setup(ww):
    ww.add_frame_decoder("dmr", lambda: MyDecoder(), "My decoder · DMR")
```

RigVibe calls the factory (`lambda: …`) for every new transmission – so the decoder may keep state from frame to frame – and then `decode()` for every frame, in a separate thread. RigVibe adjusts the level and converts to its audio path.

| Channel | Frame (bits in transmission order) | Rate |
|---|---|---|
| `dmr` | 72 bits: one third of the 108 payload dibits of a burst with voice sync or EMB (dibits 0–35, 36–71, 72–107; most significant bit of each dibit first). The payload dibits are the 54 before and the 54 after the centre field. With several time slots RigVibe follows the one that transmits first. | 3 frames per 60 ms |
| `dstar` | 72 bits: the first 72 bits of a frame (before the 24 bits of slow data), without sync word | 1 frame per 20 ms |
| `ysf` | 104 bits: one VCH field in V/D mode 2, unchanged (interleaved and whitened) | 5 frames per 100 ms |
| `m17` | 128 bits: payload of a stream frame (after the frame number). RigVibe plays this channel itself (Codec2); an extension can take it over. | 1 frame per 40 ms |

Per channel the most recently loaded extension applies. If `decode()` raises an error, that transmission stays silent and reception continues; the error is logged. A working example is included: **F8 → Extensions → install example “Rahmen-Schnittstelle: Testton”** – it outputs a tone per frame.

**Transmit direction.** Conversely, an encoder takes audio and returns frames in the same format as above:

```python
class MyEncoder:
    def encode(self, values):           # 160 numbers (20 ms at 8 kHz, for m17 320 = 40 ms) around -1..1
        ...
        return bits                     # one frame: list of 0/1 in transmission order (length as in the table)

def setup(ww):
    ww.add_frame_encoder("m17", lambda: MyEncoder(), "My encoder · M17")
```

The factory is called per transmission, `encode()` per frame; a final frame of silence flushes the encoder. If `encode()` returns the wrong number of bits or raises an error, nothing is transmitted. RigVibe uses the transmit direction when sending M17 (built in: Codec2). `GET /api/voiceplugins` shows which channels currently have an extension. An extension reads the station settings (e.g. `dmr_id`) with `ww.station()` and declares its own values as `settings` in the manifest.

Libraries (e.g. a DLL) go into the extension's own folder and are loaded by the extension itself (`ctypes`, path via `Path(__file__).parent`). On macOS, files downloaded from the internet carry a quarantine flag; remove it yourself if needed (`xattr -d com.apple.quarantine <file>`).

## Custom operating modes

An extension can bring a complete operating mode of its own: coding, framing, modulation, demodulation and decoding. RigVibe does not need to know what the signals contain. It only provides what it manages anyway: radios with their transmit and receive paths, PTT and sequencing, microphone, audio output, controls and display. Samples or bits pass through **unchanged** – the only exceptions are the per-radio calibrated deviation/polarity correction (one fixed factor) and, where a radio needs another rate, a linear-phase FIR; both are stated in `info["weg"]`.

Complete example to learn from: **F8 → Extensions → install example “Eigene Betriebsart: FSK-Text”** (source `beispiele/plugins/fsk-text/`). It transmits and receives short texts with 1200 bit/s FSK, shows received stations in the display, plays an acknowledgement tone, brings its own signal detector and has an interface part that transmits without the PTT button.

The interface uses German names (keys, methods, events) like the rest of the RigVibe API; the tables translate them.

### 1. Registering

```python
def setup(ww):
    if ww.api_version < 2 or "add_mode" not in ww.caps:      # older RigVibe: report cleanly
        ww.log.warning("RigVibe too old for custom operating modes")
        return
    ww.add_mode("my-mode", {
        "name": "My mode",
        "senden": ["basisband"],            # transmit forms the extension can deliver: basisband, bitstrom, iq
        "empfangen": ["diskriminator"],     # receive forms it needs: diskriminator, iq
        "bandbreite_hz": 12500,             # occupied channel width (band check, receive channel filter)
        "hub_hz": 3000,                     # nominal deviation: ±1.0 baseband = ±hub_hz
        "felder": [                         # input fields for a transmission (types as in "settings")
            {"key": "ziel", "label": "Destination", "type": "text", "default": ""},
        ],
        "mikrofon": {"rate": 8000, "block": 160},   # wanted microphone format, or None
        "bitraten": [4800],                 # "bitstrom" only: wanted bit rates (first match applies)
        "max_s": 180,                       # maximum length of one transmission (user can change it)
        "aufbereitung": False,              # microphone unprocessed by default (user can switch)
        "vox": False,                       # VOX only if explicitly allowed
        "adif": {"mode": "DYNAMIC"},        # logbook mode when a heard station is taken over
        "sender": lambda info: MySender(info),
        "empfaenger": lambda info: MyReceiver(info),
    })
```

RigVibe treats the mode **like one of its own**: it is listed in all mode lists (mode button and menu on the radio, “Bands & modes”, context menu, Digital modes selection and menu *Mode*, phone view) and is called `ext:<id>` internally – so it can also be stored in favourites and memory channels and chosen via the API (`POST /api/rig/mode {"mode": "ext:<id>"}`). When chosen, RigVibe sets the radio to FM (discriminator or frequency modulation) and switches to the extension's path; any other mode deselects it again. The window **Extension operating mode** shows the transmit button, input fields and received stations. RigVibe remembers the field values per mode and, if wanted (“per channel”), per frequency, i.e. per memory channel or repeater. The selection per radio survives a restart.

### 2. Transmitting: sequence

For every transmission RigVibe calls `sender(info)`. `info` contains `station` (like `ww.station()`), `felder` (field values), `weg` (description of the transmit path, section 4), `form` (chosen transmit form), `frequenz`, `ablage` (repeater offset or split in Hz), `geraet` (radio), `betriebsart` (mode), `trocken` (dry run) and `quelle` (source). The returned object has these methods; RigVibe calls them **in a thread of their own**, never in the asyncio loop:

| Method | When | Returns |
|---|---|---|
| `start()` | once at the beginning, before PTT | first part of the signal (preamble, header) |
| `mikrofon(block)` | for every microphone block in the registered format (float, mono) | signal for this period (may be empty) |
| `fuellen(n)` (fill) | when the output buffer holds less than the reserve | up to `n` further values; **`None` = nothing more, transmission ends** |
| `stop()` | PTT released or `ctx.tx.stop()` / `ww.tx_stop()` | end of the signal |
| `abbruch(grund)` (abort) | `zeit` (maximum length), `geraet` (radio/audio output gone), `notaus` (emergency stop), `ptt` (PTT ended from outside), `fehler` (error) | – |
| `zeit(t)` | optional, before every `fuellen()`/`mikrofon()` | – (`t` = estimated Unix time at which the next delivered value is transmitted) |

Guaranteed:

- **Reserve instead of silence:** before PTT RigVibe fills a reserve (default 200 ms, adjustable per radio) and requests more via `fuellen(n)`. If the buffer still runs empty, the event `tx_unterlauf` (underrun) is sent (server `ww.on("tx_unterlauf", …)`, interface `ctx.on("tx_unterlauf", …)`) and logged.
- **PTT until the last value:** after `stop()` (or `None` from `fuellen`) RigVibe outputs everything delivered – including buffers inside the device driver – and only then releases PTT. Then `tx_ende` follows with `ok`, `grund` (reason), `dauer` (duration), `werte` (values sent), `unterlaeufe` (underruns), for a dry run `datei` (file), for a recording `mitschnitt`.
- **Lead time per radio:** time between PTT and first value (transmitter settling), in `info["weg"]["vorlauf_ms"]`. On an SDR it runs as an unmodulated carrier.
- **Transmit without the button:** interface `await ctx.tx.start(fields, {betriebsart, quelle, trocken, rueckspielen, mitschnitt})` / `ctx.tx.stop()`, server `await ww.tx_start(felder, radio=…, quelle=…)` / `await ww.tx_stop()` – with the same locks as the PTT button.
- **One PTT for everything:** while the mode is selected on a radio, the transmit button, space bar, MIDI controller, phone view, remote control via the API and the voice keyer all lead to this sequence.

### 3. Transmit forms

**a) Baseband (`basisband`)** – floats, `±1.0 = ±hub_hz`, sample rate `weg["rate"]`. No transmit audio chain (equalizer, compressor, limiter, noise gate, VOX), no level control. Via a sound card the interface plays the values at the rate of its output (usually 48 kHz); own SDRs get 48 kHz, which RigVibe brings to the device rate with a linear-phase FIR and frequency-modulates without pre-emphasis and without high-pass (DC capable).

**b) Bit stream (`bitstrom`)** – bits (0/1) in transmission order for radios with their own FSK unit; the mode lists its bit rates under `"bitraten"` (RigVibe takes the first one the radio supports and reports it as `weg["bitrate"]`). `fuellen(n)` then counts bits. Available on the **Quansheng UV-K1/K5 from firmware version 14** – no sound card, USB only:
- 1200, 2400, 4800 or 6000 bit/s, continuous without length limit (measured: 48,000 bits in one go at 4800 bit/s, 13 bit errors at lowest power, no slip, no underrun); RigVibe refills according to the free space in the radio's buffer.
- The FSK unit puts **8 bits of preamble** (10101010) in front; this cannot be switched off. The extension's **first 16 bits** serve as its sync word and go out unchanged – the stream follows bit by bit. No checksum, no whitening. `weg["bitstrom"]` describes this.
- Ends exactly after the last bit, in steps of 8 bits (rest padded with 0).
- Deviation from the mode's `hub_hz` (measured at 4800 bit/s roughly 450 Hz … 2.4 kHz), bit 1 = higher frequency.

**c) IQ (`iq`)** – complex values at `weg["rate"]` relative to the transmit frequency, magnitude ≤ 1 (magnitude 1 = same power as FM). Own SDRs only.

Values beyond ±1 are passed on unchanged and noted once in the log. **Baseband directly over USB** (without a sound card) is not available on the Quansheng: the radio chip changes the deviation only via the analogue microphone path, the tone generators (sine of fixed frequency), the FSK unit (two levels) or by retuning the synthesizer (20–40 ms settling), and there is no connection from the processor to the microphone input.

### 4. Transmit path description (`weg`)

```python
{"geraet": "r3", "name": "K1", "art": "soundcard",      # soundcard | sdr | empfaenger (receiver) | trocken (dry)
 "formen": ["basisband", "bitstrom"], "rate": 48000, "bitraten": [1200, 2400, 4800, 6000],
 "untere_grenze_hz": 65, "obere_grenze_hz": 3000, "gleichanteil": False,
 "hub_kalibriert": True, "polaritaet": -1, "vorlauf_ms": 150, "latenz_ms": 235,
 "umgerechnet": "",                                      # SDRs: which FIR to which rate
 "frequenzgang": "…", "vorentzerrer_fir": [ … ],         # measured paths (Quansheng): inverse response, 48 kHz
 "hinweise": ["…ground loops…"], "firmware": 14, "gesperrt": False, "flach": True,
 "bitstrom": {"vorlauf_bits": 8, "raster_bits": 8, "max_bps": 6000, "hinweis": "…"}}   # bit stream path (Quansheng from version 14)
```

With this the extension can pre-equalize itself when the path attenuates low or high frequencies – the FSK-Text example convolves its signal with `vorentzerrer_fir` when present. `GET /api/extmodes/wege` or `ww.wege()` returns the paths of all radios, `ww.radios` the radios themselves (with firmware version).

### 5. Microphone and other audio sources

With `"mikrofon": {"rate", "block"}` the microphone audio of the interface arrives at `mikrofon(block)` in exactly this format (mono, float) – **unprocessed** (default) or with RigVibe's transmit audio processing (switch “Processing”). VOX does not key in this mode unless `"vox": True`. Instead of the microphone a **voice keyer slot** (`quelle="sprachspeicher:3"`) or an **audio file** (`quelle="datei:/path/call.wav"`, 16 bit) can be the source; at its end `stop()` follows automatically. `quelle="keine"` transmits without audio (data).

### 6. Receiving

When the mode is selected on a radio, RigVibe calls `empfaenger(info)` (`info` with `weg`, `form`, `station`, `felder`, `frequenz`, `geraet`). The object receives, also in a thread of its own:

| Method | Content |
|---|---|
| `werte(block, t)` | raw demodulator signal: FM discriminator unfiltered, without de-emphasis, with DC as far as the radio delivers it; `t` = Unix time of the first value |
| `iq(block, t)` | own SDRs, if `"iq"` is listed under `empfangen`: complex values of the channel (48 kHz) |
| `luecke(t, dauer)` (gap) | lost values (USB ring full, block late, own transmission) |
| `pegel(dbm)` | signal strength, about twice per second |
| `weg(weg)` | receive path changed (e.g. Quansheng clock tracked → new rate) |
| `pause()` / `weiter()` | own transmission starts / has ended (half duplex) |
| `ende()` | mode deselected, extension off, radio gone |

Receive paths (`info["weg"]`: `rate`, `einheit` (unit), `untere_grenze_hz`, `obere_grenze_hz`, `gleichanteil`, `ausgleich` (compensation), `ppm`, `takt_ausgeglichen` (clock corrected), `quelle`):

| Radio | Form | Rate | Unit |
|---|---|---|---|
| own SDRs (HackRF, Pluto, RTL-SDR, Hermes Lite 2 …) | diskriminator, iq | 48 kHz, own channel ±bandwidth/2 | Hz |
| Quansheng UV-K1/K5 with RigVibe firmware from version 12 | diskriminator | 38.4 kHz (clock corrected) | raw (12-bit converter, mid-scale removed) |
| Quansheng older versions, OpenWebRX, KiwiSDR | diskriminator | 9.6 or 12 kHz (receiver audio) | raw |
| radio on a sound card | diskriminator | 24 kHz from the Digital modes input | raw |

On the Quansheng the raw path also runs when RigVibe's own decoders are off.

**Return paths:**

- **Audio:** `ww.audio(values, rate)` or `ctx.audio(values, rate)` – goes into the normal receive audio (volume, recorder, remote audio, phone); “Audio” slider per mode. For radios without audio in the server the mode's window plays it.
- **Display:** `ww.heard({"rufzeichen" (call), "name", "ziel" (destination), "text", "felder": {…}, "lat", "lon"})` – appears like RigVibe's own decoders under the frequency, in “recently heard” (from there into the logbook, mode from `adif`), in the mode's window and as a `decode` message with `decoder: "ext:<mode>"`, `kind: "ext"`. `ww.heard_end()` ends the line.
- **Signal detection:** `ww.add_detector("my-mode", fn)` – `fn(values, rate)` gets the last 1.5 s of discriminator and returns 0..1 (short transmissions: rate the best section, see the example). From 0.6 a click into the waterfall, the scanner and the digital voice search in Digital modes assign the signal to the extension and switch to its path – like RigVibe's own decoders, with a live view in the signal identification card. **Without a detector** signal identification tries the mode while probing the decoders: it selects it on the radio and waits whether `ww.heard()` reports something within about 2 s of transmission.
- `ww.heard()` and `ww.audio()` without a radio apply to the radio whose receiver calls them (from within `werte()`/`iq()`); otherwise pass `radio=`.

### 7. Radio paths, lead time and calibration

**Radio …** in the mode's window: lead time, reserve, latency, polarity, audio value per kHz deviation, transmit lock. **Calibrate** (sound card path): transmits a 2-s test tone (peak = full deviation) and measures deviation and polarity with an own SDR on the same frequency; afterwards `weg["hub_kalibriert"] = True`. The radio's transmit level is part of the measurement – do not change it afterwards. Own SDRs are exact out of the box.

Sound card output: an open RigVibe window is the **operating position** – for every connected radio with an extension mode selected there is a channel of its own with that radio's audio output (sound card) and transmit level, so e.g. a radio on a sound card and an SDR as measuring receiver run at the same time. The values go out without processing, PTT via the radio's path (CAT, RTS/DTR, USB), timed to the first value (lead time).

**Quansheng UV-K1/K5 via the USB sound card into the microphone input:** from firmware version 13 RigVibe switches on the **flat transmit audio path** for every extension transmission (`weg["flach"]`, can be switched off per radio): after setting up the transmitter the firmware turns off pre-emphasis, high/low-pass, DC filter, compander and microphone AGC, holds this for the whole transmission and restores everything afterwards; deviation and microphone gain can be fixed (`k1_hub`, `k1_mikro`). Measured with a HackRF: what remains is the analogue circuitry – +9 dB around 100–150 Hz, steep roll-off below 60 Hz, −3 dB at 3 kHz, linear up to full scale, inverted polarity. This is in `weg` as `frequenzgang`, `untere_grenze_hz`/`obere_grenze_hz` and `vorentzerrer_fir`, together with a ground-loop hint. RigVibe renews the PTT itself during long transmissions.

### 8. Locks and safety

Transmissions of extensions follow the same rules as RigVibe's own modes: own callsign entered, check against the amateur bands (frequency ± bandwidth/2, including offset), transmit lock per radio, maximum length per transmission, **emergency stop** (button in the window or `POST /api/extmodes/notaus`; `Esc` ends running transmissions) and the PTT guard (switched off after 3 s without a connected interface). While transmitting, every window shows **“sendet (Erweiterung …)”** (transmitting) at the top; a click ends it.

### 9. Development and test aids

- **Dry run (Trockenlauf):** no PTT; the signal goes, after the transmit path (calibration factor included), as `.wav`/`.npy` into the extension's `daten/` folder and, if wanted (“play back”), straight into the extension's own receiver – transmitting and decoding can be checked without a radio.
- **Recording (Mitschnitt):** exactly the values that went to the radio (for SDRs the 48-kHz stage before the interpolator), with the estimated transmit time per block (`mitschnitt-….npy` + `.json`).
- **Log:** underruns, gaps, timeouts and errors of the extension – button **Log** or `GET /api/pluginlog/<id>`.

### 10. Versioning and interfaces

`ww.api_version` (now `2`) and `ww.caps` (set of available functions, e.g. `tx.basisband`, `tx.iq`, `rx.iq`, `tx.trockenlauf`, `heard`, `audio`) – in the interface `ctx.apiVersion` and `ctx.caps`. An extension checks with them what the running RigVibe version can do and reports cleanly what is missing.

| Server `ww.…` | Interface `ctx.…` | Meaning |
|---|---|---|
| `add_mode(id, spec)`, `add_detector(id, fn)` | – | register mode, signal detector |
| `tx_start(…)`, `tx_stop()` | `tx.start(fields, opts)`, `tx.stop()`, `tx.state()`, `tx.notaus()` | transmit without the button |
| `audio(values, rate)` | `audio(values, rate)` | audio into receive |
| `heard(d)`, `heard_end()` | – | report a heard station |
| `wege()` | `wege()` | transmit/receive paths of all radios |
| – | `modes()`, `activeMode()`, `selectMode(id)` | extension modes, selection on the radio |

HTTP (for other programs, `/api/docs`): `GET /api/extmodes`, `GET /api/extmodes/wege`, `POST /api/extmodes/select`, `POST /api/extmodes/tx`, `POST /api/extmodes/tx/stop`, `POST /api/extmodes/notaus`, `POST /api/extmodes/kalibrieren`, `PUT /api/extmodes/<id>/felder|optionen`, `PUT /api/extmodes/geraet/<radio>`, `GET /api/pluginlog/<id>`.

**Limits:** baseband over USB does not exist on the Quansheng (see 3). Modes with microphone and sound card paths need an open RigVibe window (browser or program); the phone view selects and keys, but does not supply a microphone for extensions. Own SDRs and the Quansheng's bit stream need no window as audio output.

## Security

Extensions are **off** after installation. Once switched on they have the same rights as RigVibe – they can control the radio (including transmitting), change the logbook and read files. Only switch on extensions from a trusted source.
