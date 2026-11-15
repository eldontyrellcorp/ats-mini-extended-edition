# ATS Mini firmware – extended edition

Firmware for the SI4732 (ESP32-S3) Mini/Pocket Receiver, with decoders, propagation tools, weather and a multilingual interface added.

## Credits

This firmware is based on the original ATS Mini firmware created by **Max Arnold**: <https://github.com/esp32-si4732/ats-mini>

All the credit for the receiver firmware itself (tuning, display, menus, memories, Wi-Fi/Bluetooth, web interface, online updates and the rest of the base features) goes to Max Arnold and the contributors of that project. This edition adds the features listed below on top of it. It is released under the same MIT license.

## What is new in version 1.02

### Added

* **Speaker On/Off** in the short press menu of the decoders (FT8, FT4, CW, RTTY, PSK31, SSTV, WEFAX) and of the live waterfall: decode silently with the ADC input on IO11. The speaker comes back when leaving the decoder, and the setting is remembered.
* **FT8 and FT4 tuning step** of 100 Hz, 500 Hz or 1 kHz, selected from the short press menu.
* **ETM sensitivity** (Low, Normal, High) in the short press menu of the ETM screen: High catches weaker stations. The band is scanned again when it changes, and the setting is remembered.
* **REST API** on the web server, available whenever Wi-Fi is on: read the status, control the receiver and download the saved SSTV and WEFAX pictures. See [REST API](#rest-api) below.

## What is new in version 1.01

### Added

* **Five-page interface** (Radio, Signal, Decode, Tools, Setup): press and rotate the knob to flip pages, click to open an item. It replaces the old main menu.
* **ETM (Easy Tuning Mode):** scan the current band once, then step through the stations found with the encoder. The list is kept per band, separately from the memories.
* **Live audio waterfall** for any band and mode, while the receiver keeps playing.
* **Sleep Clock:** a big clock with the date, the home city temperature, the frequency and the battery on the sleeping display.
* **Twelve new color themes.**
* **Experimental LW band** (150 - 300 kHz).
* **Browser remote control page** over Bluetooth LE (`remote/index.html`): tuning, band, mode, volume, live status and screenshots.
* **FAV22** (3881 and 6825 kHz) in the CW frequency list.
* **Saved pictures:** keep up to five SSTV and WEFAX pictures and view them later with Pictures on the Tools page.
* **WEFAX lines setting:** show all lines, 1 of 2 or 1 of 6, so charts are no longer stretched vertically.

### Changed

* **FT8 and FT4** start decoding on the tuned frequency; the standard frequency is available from the short press menu.
* The About screen credits this edition and the original firmware, and the first start shows a short welcome screen.

### Fixed

* Wrong colors in SSTV pictures, the FT8/FT4 waterfall, WEFAX images and the propagation map.

## Features added in this edition

### Decoders (Decode menu)

* **FT8** and **FT4** with a waterfall, UTC slot timing, decoded messages with CQ calls highlighted, and a list of heard stations with country and distance.
* **Automatic band hopping** for FT8 and FT4 across the amateur bands that the propagation estimate says are likely to be open.
* **CW** with presets.
* **RTTY** with presets for amateur and weather-service frequencies (DWD), several baud rate and shift modes, and automatic tone finding.
* **PSK31** (BPSK31) with automatic carrier tracking, calling frequencies and a squelch.
* **SSTV** image reception: Martin, Scottie, Robot and black-and-white modes, with a remembered mode.
* **WEFAX** weather fax reception.
* **Audio input choice:** an INMP441 I2S microphone, or the receiver audio read directly from the ADC on IO11 (V4A/V4B) for a cleaner signal that does not depend on the speaker volume.
* **Speaker On/Off** while decoding with the ADC input, from the short press menu of each decoder and of the live waterfall.
* **Waterfall colors** (seven presets) and a **noise filter** that keeps the background dark so that signals stand out, with a live preview.

### Misc menu

* **Weather:** current conditions, the next 24 hours and a 10-day forecast for your home city. Pick the country and city from a list, or type any other city with a virtual keyboard.
* **Propagation:** estimated band conditions from the sun position and solar data, with a grey-line map.
* **Sunrise, sunset and grey line** for your location.
* **Best band now:** the bands most likely to be open at this moment.
* **On air now:** the broadcast stations from the EiBi schedule that are on the air right now in the current band.
* **Spectrum and waterfall** for every band the receiver covers (VHF, MW, SW, amateur bands).
* **Space-weather alert** for solar flares and geomagnetic storms.

### Interface

* **Multilingual menus:** English, French, Dutch and German, chosen in Settings.
* A regrouped main menu with a DSP submenu, menu boxes that adapt to the longest entry, hint bars on the new screens and progress indication for downloads.
* The new screens follow the selected color theme.

## REST API

When Wi-Fi is in the AP Only, AP+Connect or Connect mode, the web server of the receiver answers a small REST API on port 80. It uses the same optional login as the web pages (HTTP basic authentication) and has no encryption, so keep it on a trusted network. Answers are in JSON.

| Request                            | Parameters               | Description                                                                 |
|------------------------------------|--------------------------|-----------------------------------------------------------------------------|
| `GET /api/status`                  |                          | Frequency (Hz), band, mode, step, bandwidth, AGC, volume, mute, RSSI, SNR, battery voltage, firmware version |
| `GET /api/bands`                   |                          | The bands with their index, name, mode and frequency limits (Hz)            |
| `POST /api/frequency`              | `value`, `delta` (Hz)    | Tune within the current band                                                |
| `POST /api/band`                   | `name`, `value`, `delta` | Select a band by name (`40M`), by index, or the next/previous one           |
| `POST /api/mode`                   | `name`, `delta`          | `AM`, `LSB` or `USB` (the mode of the FM band cannot be changed)            |
| `POST /api/step`                   | `name`, `delta`          | Tuning step as shown on the receiver (`5k`)                                 |
| `POST /api/bandwidth`              | `name`, `delta`          | Bandwidth as shown on the receiver (`3.0k`)                                 |
| `POST /api/agc`                    | `value`, `delta`         | AGC/attenuation value                                                       |
| `POST /api/volume`                 | `value`, `delta`         | Volume 0 ... 63                                                             |
| `POST /api/mute`                   | `value`                  | `1` to mute, `0` to unmute                                                  |
| `GET /api/pictures`                |                          | The saved SSTV and WEFAX pictures: slot, label, date, size, download URL    |
| `GET /api/pictures/N.bmp`          |                          | The picture of slot N as a BMP file                                         |
| `DELETE /api/pictures/N`           |                          | Delete the picture of slot N                                                |

Each `POST` request takes one parameter, in the query string or as form data: `value` sets a number, `name` selects by name, `delta` moves up or down. It answers with the new status, or with an `error` message.

```
curl http://atsmini.local/api/status
curl -X POST -d name=40M http://atsmini.local/api/band
curl -X POST -d value=7074000 http://atsmini.local/api/frequency
curl -X POST -d delta=-1 http://atsmini.local/api/volume
curl -o picture.bmp http://atsmini.local/api/pictures/0.bmp
```

Notes:

* A frequency outside the current band is refused: select the band first.
* While a decoder or another full screen tool (Scan, Spectrum, Pictures...) is open, the status and the pictures stay available but the `POST` requests answer `503 Receiver is busy`.
* The decoders using the ADC input need Wi-Fi off, so the API is not reachable meanwhile: save the picture, then turn Wi-Fi on to download it.

## Documentation

The full user manual is in [`docs/source/manual.md`](docs/source/manual.md). Each change is listed in the [`changelog/`](changelog) folder.

## Building

The firmware is an Arduino sketch in the `ats-mini/` folder. Build it from the repository root with:

```
arduino-cli compile --warnings all ats-mini
```

See [`docs/source/development.md`](docs/source/development.md) for the details.

## License

MIT, see [`LICENSE`](LICENSE). Copyright of the original work belongs to its authors.
