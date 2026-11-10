# ATS Mini firmware – extended edition

Firmware for the SI4732 (ESP32-S3) Mini/Pocket Receiver, with decoders, propagation tools, weather and a multilingual interface added.

## Credits

This firmware is based on the original ATS Mini firmware created by **Max Arnold**: <https://github.com/esp32-si4732/ats-mini>

All the credit for the receiver firmware itself (tuning, display, menus, memories, Wi-Fi/Bluetooth, web interface, online updates and the rest of the base features) goes to Max Arnold and the contributors of that project. This edition adds the features listed below on top of it. It is released under the same MIT license.

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
