# ATS Mini firmware – extended edition

Firmware for the SI4732 (ESP32-S3) Mini/Pocket Receiver, with digital mode decoders, Internet radio, MQTT, propagation tools, weather and a multilingual interface added.

Latest version: **1.10**. Download it from the [Releases](https://github.com/eldontyrellcorp/ats-mini-extended-edition/releases) page.

## Credits

This firmware is based on the original ATS Mini firmware created by **Max Arnold**: <https://github.com/esp32-si4732/ats-mini>

All the credit for the receiver firmware itself (tuning, display, menus, memories, Wi-Fi/Bluetooth, web interface, online updates and the rest of the base features) goes to Max Arnold and the contributors of that project. This edition adds the features listed below on top of it. It is released under the same MIT license.

## Contents

* [Installing](#installing)
* [Features](#features)
* [Internet radio: PCM5102 DAC or PDM output](#internet-radio-pcm5102-dac-or-pdm-output)
* [Microphone input for the decoders](#microphone-input-for-the-decoders)
* [Memories](#memories)
* [MQTT](#mqtt)
* [REST API](#rest-api)
* [Version history](#version-history)

## Installing

### Which file: OPI or QSPI

Each release has two variants, one per type of PSRAM on the ESP32-S3 module of the receiver:

| File | Receiver |
|------|----------|
| `ats-mini-extended-edition-X.YY-OPI.zip`  | ESP32-S3 with 8 MB **octal** PSRAM (OPI) |
| `ats-mini-extended-edition-X.YY-QSPI.zip` | ESP32-S3 with 2 MB **quad** PSRAM (QSPI) |

To find out which one you have, look at **Settings → About** on the receiver: with the right firmware, the PSRAM size is not zero. Over USB, `esptool flash-id` shows `Embedded PSRAM 8MB` (OPI) or `Embedded PSRAM 2MB` (QSPI).

### First installation (USB)

Each zip holds:

| File | Flash address | Use |
|------|---------------|-----|
| `ats-mini.ino.merged.bin`     | `0x0`     | Everything in one file. Erases the settings. |
| `ats-mini.ino.bootloader.bin` | `0x0`     | Bootloader |
| `ats-mini.ino.partitions.bin` | `0x8000`  | Partition table |
| `boot_app0.bin`               | `0xe000`  | Starts the newly flashed application |
| `ats-mini.ino.bin`            | `0x10000` | The application |

Flash either the merged file at `0x0`, or the four separate files at their addresses, for example with the [ESP web flasher](https://espressif.github.io/esptool-js/) (Chrome or Edge) or with esptool:

```
esptool --chip esp32s3 --port /dev/ttyACM0 write-flash 0x0 ats-mini.ino.merged.bin
```

Hold the BOOT button while connecting the USB cable if the receiver is not detected.

### Updates over Wi-Fi (from 1.05)

Once 1.05 or newer is installed, connect the receiver to the Internet over Wi-Fi (with a correct clock), then either:

* on the receiver, **Settings → Update FW → Check** then **Update**, or
* in the web interface, **Update → Check for updates**, then **Update**.

The right file for OPI or QSPI is chosen automatically (`-OPI-ota.bin` or `-QSPI-ota.bin` of the latest release). Settings, memories, SWL log and favorites are kept. **Manual upload** on the web Update page installs a downloaded `-ota.bin` file (or the `ats-mini.ino.bin` of a zip) without Internet access.

From 1.04 or older, upload `ats-mini.ino.bin` with **Manual upload** on the web Update page once, or flash by USB.

## Features

### Interface

* **Five pages:** RADIO, SIGNAL, DECODE, TOOLS and SETUP. Press and rotate the knob to flip pages, rotate to choose a tile, click to open it, hold (0.3 s) to go back to the RADIO page.
* **Quick menu:** a click on the RADIO page opens it: favorites, volume, band, mode, step, bandwidth, AGC, squelch, SoftMute, AVC, frequency.
* **Quick favorites:** five buttons, each a frequency or an Internet radio station, recalled with one click.
* **Memories screen:** a double-click on the RADIO page lists the stored memories like the ETM screen, with their names and frequencies, saves the current frequency with an automatic name, and scans the memories. See [Memories](#memories).
* **ETM (Easy Tuning Mode):** scans the band once and steps through the stations found, with a Low, Normal or High sensitivity.
* **Languages:** English, French, Dutch and German.
* **Themes:** twelve additional color themes, followed by all the new screens.
* **Sleep clock:** a big clock with date, home city temperature, frequency and battery on the sleeping display.
* **LW band** (experimental, 150 - 300 kHz).

### Decoders (DECODE page)

* **FT8** and **FT4**: waterfall, UTC slot timing, decoded messages with CQ highlighted (7 lines in a small font, or 4 in a larger one), list of the stations heard with country and distance, scrolling, 1.2k ... 4.0k bandwidth, 100 Hz ... 1 kHz tuning step, automatic band hopping across the open bands, on any frequency (also CB).
* **CW** with presets, **RTTY** (amateur and DWD weather presets, baud rate and shift, automatic tones, tone hold), **PSK31** (carrier tracking, squelch).
* **SSTV** (Martin, Scottie, Robot, PD, B/W; automatic from the VIS header, or a chosen mode with **Start now**) and **WEFAX**, with up to five saved pictures.
* **Beacons:** the 18 NCDXF/IARU beacons on 14.100 ... 28.200 MHz.
* **Input:** the receiver audio on IO11 (ADC, default, with Wi-Fi off) or an INMP441 I2S microphone (**Input: Mic**, works with Wi-Fi on).
* In every decoder: input level meter, speaker on/off, knob as volume control, seven waterfall color presets and a noise filter.

### Tools (TOOLS and SIGNAL pages)

* **Weather** for your home location anywhere in the world (city search worldwide, coordinates, or a country list; also on the web Config page) (now, 24 hours, 10 days), **Propagation** with a grey-line map, **Best band now**, **On air now** (EiBi), **Space-weather alert**.
* **Spectrum**, **waterfall** and **live waterfall** on every band.
* **Internet radio** through a PCM5102 DAC or the PDM output. See [Internet radio](#internet-radio-pcm5102-dac-or-pdm-output).
* **SWL log:** the FT8/FT4 stations heard, and the CW/RTTY stations whose callsign is decoded twice, kept as ADIF with a signal report; browse it on the receiver or on the web, download it, or upload it to eQSL.cc.
* **Bricks** game.

### Network

* **Firmware update over Wi-Fi** from the releases of this repository.
* **MQTT:** state, RDS, decoded messages and SWL log published to a broker, and remote control. See [MQTT](#mqtt).
* **REST API** on the web server. See [REST API](#rest-api).
* **Web pages:** memories (with CSV import), Config (with MQTT and My stations), Wi-Fi, SWL log, Update.
* **Wi-Fi power** setting (2 ... 19.5 dBm) and a Bluetooth LE remote control page.

## Internet radio: PCM5102 DAC or PDM output

The Internet radio (TOOLS page) plays stations from the [radio-browser.info](https://www.radio-browser.info) directory (top stations, your country, genres, search), up to 32 favorites and up to 20 stream URLs of your own (**My stations** on the web Config page). MP3, AAC, FLAC, Opus and Vorbis are supported. It needs a Wi-Fi connection to the Internet and a correct clock. The receiver itself is muted while the Internet radio plays: the sound comes out of the DAC or of the PDM output only, not out of the receiver's speaker.

The SI4732 receiver has no digital audio input, so the stream needs its own output. There are two ways:

| | PCM5102 DAC | PDM output |
|---|---|---|
| Parts | A PCM5102 (PCM5102A) I2S DAC module | Two resistors and three capacitors |
| Pins | IO12, IO13, IO14 (+ IO44 for mute) | IO43 only |
| Sound | Stereo, line level, very clean | Mono, line level after the filter, some hiss |
| Setting | **PCM5102 DAC: On** | **IO43 Pin: PDM out** |

Both can be used together: the Internet radio then plays on both outputs. The pins below are spare pads of the receiver board, or pins of the ESP32-S3 module:

| GPIO | Board | Used for |
|------|-------|----------|
| IO12 | spare pad | DAC BCK / microphone SCK |
| IO13 | spare pad | DAC DIN / microphone WS (DAC LRCK with **IO43 Pin: DAC DIN**) |
| IO14 | spare pad | DAC LRCK / microphone SD |
| IO43 | TXD0, pin 37 of the ESP32-S3 module | PDM output, or the moved microphone SD or DAC DIN (see below) |
| IO44 | RXD0, pin 36 of the ESP32-S3 module | DAC XSMT (mute): high while a station plays, low otherwise |

IO43 and IO44 are the UART0 pins of the module; the firmware uses the USB port for the serial console, so they are free. All the other pins are taken (IO39-42 and IO45-48 are the display data bus): do not use them.

### With a PCM5102 DAC

Wiring of a common PCM5102A module (purple "GY-PCM5102" boards; check the markings of yours):

| PCM5102 module | Receiver |
|----------------|----------|
| VIN (or VCC)   | 3.3 V of the receiver board (most modules also take 5 V) |
| GND            | GND |
| BCK            | IO12 |
| DIN            | IO13 |
| LCK (LRCK)     | IO14 |
| SCK            | GND (the DAC makes its own clock) |
| XMT (XSMT)     | IO44 (or 3.3 V to leave it always on) |
| FMT, FLT, DMP  | GND (I2S format, normal filter, no de-emphasis) |

Many modules have solder bridges on the back (H1L FLT, H2L DEMP, H3L XSMT, H4L FMT) that set these pins: bridge FLT, DEMP and FMT to L. For XSMT, leave its bridge open when you wire it to IO44, or bridge it to H to leave the DAC always on. Driving XSMT from IO44 avoids a click when stations start and stop.

Plug headphones, an active speaker or an amplifier into the 3.5 mm jack of the module (line level). Keep the wires short, and route them away from the antenna input: the I2S clocks are stopped and the pins held low when the Internet radio is closed, to avoid birdies on HF.

On the receiver: **SETUP → PCM5102 DAC: On**. The Internet radio tile is grayed out until the DAC or the PDM output is turned on.

### With the PDM output (no DAC)

With **SETUP → IO43 Pin: PDM out**, the Internet radio sends its audio (mono, 48 kHz) as a PDM bit stream on IO43: a 3.3 V digital signal whose density of ones follows the audio. A low-pass filter turns it into line level audio:

```
IO43 ──[1 kΩ]──┬──[1 kΩ]──┬──||──── amplifier input (line level)
               │          │  1 µF
             10 nF      10 nF
               │          │
GND ───────────┴──────────┴───────── amplifier GND
```

* Two RC stages of 1 kΩ and 10 nF (cut-off near 16 kHz) remove the PDM clock; the 1 µF capacitor removes the DC level (1.65 V).
* Connect the output to a line input: an active speaker, a small amplifier module (PAM8403, LM386...) or headphones through an amplifier. Do not connect a speaker directly to IO43.
* If it is too loud for your amplifier, add a 10 kΩ potentiometer or a divider after the capacitor.
* The volume is set with the knob on the Internet radio screen.

The PDM output needs IO43 only; its clock is not output. With **PDM out**, the microphone and the DAC use IO12-14 as with **IO43 Pin: Unused**.

### Using the microphone and the DAC together

The INMP441 microphone and the PCM5102 DAC use the same three pads. To wire both, **SETUP → IO43 Pin** moves one data line to IO43:

| IO43 Pin | Microphone (SCK, WS, SD) | PCM5102 DAC (BCK, LRCK, DIN) |
|----------|--------------------------|------------------------------|
| Unused   | IO12, IO13, IO14 | IO12, IO14, IO13 (not with the microphone) |
| Mic SD   | IO12, IO13, **IO43** | IO12, IO14, IO13 |
| DAC DIN  | IO12, IO13, IO14 | IO12, **IO13**, **IO43** |
| PDM out  | IO12, IO13, IO14 | IO12, IO14, IO13; the PDM audio is on IO43 |

The pins are shared: the microphone and the DAC work one at a time (decoders or Internet radio), never together.

## Microphone input for the decoders

The decoders read the receiver audio from the ADC on IO11 by default (V4A/V4B boards). The ADC cannot be read while Wi-Fi is on: with this input, turn Wi-Fi off before starting a decoder (the decoder shows "Wi-Fi must be off" otherwise). To decode with Wi-Fi on (for MQTT or the web pages), use an INMP441 I2S microphone placed close to the speaker:

| INMP441 | Receiver |
|---------|----------|
| VDD | 3.3 V |
| GND | GND |
| L/R | GND |
| SCK | IO12 |
| WS  | IO13 |
| SD  | IO14 (or IO43 with **IO43 Pin: Mic SD**) |

Then choose **Input: Mic** in the Decode menu and set the volume so the speaker is clearly audible.

## Memories

99 memory slots, with names of up to 9 characters.

* **Open:** double-click on the RADIO page (or the **Memories** tile of the SIGNAL page). The screen lists the stored memories on the left with their name and frequency (empty slots skipped) and shows the tuned station, its slot, band, mode and signal on the right.
* **Recall:** rotate to tune from memory to memory; click to leave the screen on the chosen one.
* **Save:** the frequency you were on is the **+ New** entry: hold and choose **Save** to store it in the first free slot (a click on it leaves without saving). The name comes from the RDS station name on FM, or from the EiBi schedule or a known frequency.
* **Menu:** hold the knob (0.3 s): save, scan, delete, **Back to start** (leave on the frequency you came from). A deleted memory becomes **+ New**, so it can be saved back.
* **Scan:** double-click on the memories screen: 5 s on each memory, or, with the squelch set, as long as a signal opens it (moving on 3 s after it goes away). Click to keep the station, rotate to stop.
* **Web Memory page:** edit, reorder, export and import. **Import** takes a JSON export, or a CSV list of stations that fills the empty slots:

```
frequency,mode,name
7205,AM,BBC
98.5,FM,Radio 1
14074,USB,FT8 20m
```

Columns are found from a header (`Frequency`, `Mode`, `Name`; `MHz` or `kHz` in the header sets the unit), or in any order without a header. Without a unit, numbers below 150 are MHz, below 200000 kHz, above Hz. Separators can be commas, semicolons, tabs or spaces. The band is chosen from the frequency. Press **Save** after the import.

## MQTT

Set it up in the **MQTT** section of the web **Config** page: turn it on, enter the broker (`mqtt://host:1883`, or `mqtts://host:8883` for TLS), the login if needed and the base topic (`ats-mini/<id>` by default). The receiver connects whenever Wi-Fi is connected to an access point.

Published under the base topic (JSON unless stated):

| Topic | Content |
|-------|---------|
| `online` | `online` / `offline` (retained, last will) |
| `state` | `frequency` (Hz), `band`, `mode`, `step`, `bandwidth`, `agc`, `volume`, `muted`, `rssi`, `snr`, `battery`, `version`, `screen` (retained, on changes and every minute) |
| `rds` | FM `ps`, `rt`, `info`, `pi`, frequency (retained) |
| `ft8`, `ft4` | Each decoded message: `time`, `dial`, `audio`, `snr`, `text` |
| `cw`, `rtty` | Decoded `text`, a line at a time, with the `frequency` |
| `swl` | Each new SWL log entry: `call`, `band`, `mode`, `frequency`, `report`, `grid` |
| `iradio` | Internet radio `state`, `name`, `url`, `title`, `bitrate` (retained) |
| `result` | The answer to each command |

With **Remote Control** on, commands are taken on `<base>/cmd/<command>`: `frequency`, `band`, `mode`, `step`, `bandwidth`, `agc`, `volume`, `mute`, `memory` (1 ... 99), `favorite` (1 ... 5), `decode` (`ft8`, `ft4`, `cw`, `rtty`, `psk31`, `sstv`, `wefax`, `stop`), `iradio` (a URL, `{"url":"...","name":"..."}` or `stop`) and `state`. The payload is a value, `+N`/`-N`, a name, or JSON. Retained commands are ignored.

```
mosquitto_sub -h broker.local -t 'ats-mini/#' -v
mosquitto_pub -h broker.local -t ats-mini/a1b2c3/cmd/band -m 20M
mosquitto_pub -h broker.local -t ats-mini/a1b2c3/cmd/decode -m ft8
```

With the ADC input the decoders need Wi-Fi off, so `decode` is refused: use **Input: Mic**. Anyone who can publish on the broker controls the receiver: protect the broker with a password, or leave **Remote Control** off.

### Home Assistant

Install the **Mosquitto broker** add-on and the **MQTT** integration, point the receiver at `mqtt://<Home Assistant IP>:1883` with a Home Assistant user, and watch the topics in **Settings → Devices & services → MQTT → Configure → Listen to a topic** (`ats-mini/#`). Then add sensors in `configuration.yaml` (replace the base topic):

```yaml
mqtt:
  binary_sensor:
    - name: "ATS Mini online"
      state_topic: "ats-mini/a1b2c3/online"
      payload_on: "online"
      payload_off: "offline"
      device_class: connectivity
  sensor:
    - name: "ATS Mini frequency"
      state_topic: "ats-mini/a1b2c3/state"
      value_template: "{{ (value_json.frequency / 1000) | round(2) }}"
      unit_of_measurement: "kHz"
    - name: "ATS Mini RDS station"
      state_topic: "ats-mini/a1b2c3/rds"
      value_template: "{{ value_json.ps | default('') }}"
    - name: "ATS Mini last station heard"
      state_topic: "ats-mini/a1b2c3/swl"
      value_template: "{{ value_json.call }}"
      json_attributes_topic: "ats-mini/a1b2c3/swl"
```

Commands are sent with the `mqtt.publish` action, for example topic `ats-mini/a1b2c3/cmd/memory`, payload `3`.

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

## Version history

### Version 1.10

#### Added

* **SSTV: Start now** in the hold menu: receives right away in the selected mode (or the mode of the last image), without waiting for the VIS header or sync pulses, and locks on the line sync as soon as it finds one. A selected mode also starts by itself more easily (3 regular sync pulses instead of 4, with a looser timing).
* **Home location anywhere in the world:** **Search any city...** (worldwide) and **Enter coordinates...** (latitude and longitude, with an optional name) in the Weather location menu, and a **Home Location** section with a worldwide search on the web Config page. The country of the chosen city becomes **My country** of the Internet radio.
* **Memories screen: Back to start** in the menu, to leave on the frequency the screen was opened on.

#### Changed

* **Memories screen:** each memory shows its name and its frequency. A click on **+ New** leaves without saving; save with hold, then **Save**.
* **FT8/FT4:** the decoded messages use a smaller font, 7 lines instead of 4; **Text: large** in the menu brings back the larger font.

### Version 1.09

#### Added

* **Memories screen**, laid out like ETM: double-click on the RADIO page to see the stored memories (empty slots skipped) and tune from one to the next with the knob. The frequency you were on appears as **+ New**: one click saves it to the first free slot, named from the RDS station name or the EiBi schedule. Hold for a menu (save, scan, delete).
* **Memory scan**: double-click on the memories screen to scan the memories, 5 s on each, or as long as the squelch is open when it is set.
* **CSV import** on the web Memory page: a list of stations (frequency, mode, name) fills the empty slots, the band is chosen from the frequency.

#### Changed

* A click on the RADIO page opens the quick menu after a short wait (0.35 s), to tell it from a double-click.

### Version 1.08

#### Fixed

* Setting **IO43 Pin** to **PDM out** froze the display (since 1.06): the PDM output used IO46, a data line of the display. It now needs IO43 only.

### Version 1.07

#### Fixed

* Opening the **Config** page of the web interface restarted the receiver (since 1.06).

### Version 1.06

#### Added

* **MQTT:** set up in the new MQTT section of the web Config page (broker, login, base topic). The receiver publishes its state, the FM RDS data, the decoded FT8/FT4 messages, the CW/RTTY text, the SWL log entries and the Internet radio state, and with **Remote Control** on it takes commands on `<base>/cmd/...`: frequency, band, mode, step, bandwidth, AGC, volume, mute, memories, quick favorites, decoders (`decode ft8` ... `decode stop`) and Internet radio streams. The decoders can run with Wi-Fi on with the microphone input only. See [MQTT](#mqtt) below.
* **PDM output for the Internet radio:** with **IO43 Pin** set to **PDM out**, the Internet radio plays without a PCM5102 DAC, as a PDM signal on IO43 to filter with an RC low-pass filter (for example two stages of 1 kΩ and 10 nF, then 1 µF to the amplifier). With the DAC on as well, both outputs play.

#### Changed

* **Update FW** is back in the Settings menu (Check / Update over Wi-Fi).

### Version 1.05

#### Changed

* **Firmware update over Wi-Fi:** **Settings → Update FW** and the web **Update** page now check and download the releases of this firmware (ats-mini-extended-edition), with the right file for OPI or QSPI PSRAM picked automatically. The installed version is shown on the web page. Settings, memories, SWL log and favorites are kept. To go from 1.04 or older to 1.05, upload `ats-mini.ino.bin` (from the zip for your receiver) with **Manual upload** on the web Update page, or flash by USB; later versions can then be installed over Wi-Fi.

### Version 1.04

#### Added

* **SWL log** (Tools page and Misc menu): the FT8/FT4 stations heard, and the CW/RTTY stations whose callsign is decoded twice after DE or before K, are kept as an ADIF file with an approximate signal report, to browse on the receiver (grouped by day, with a band, mode, upload and grid filter, and fast scrolling) or on the new SWL Log web page, download, or upload to an eQSL.cc SWL account.
* **Microphone input again:** the INMP441 I2S microphone can be chosen with **Input: Mic** in the Decode menu, besides the IO11 line-in (ADC, default).
* **IO43 Pin** setting: move the microphone SD line, or the PCM5102 DAC DIN line (with its LRCK on IO13), to IO43 to wire both the microphone and the DAC.
* **Quick favorites:** five buttons opened from the quick menu of the RADIO page (click, then click on Favorites), each holding a receiver frequency or an Internet radio station, tuned or played with one click.
* **Internet radio, My stations:** up to 20 stream URLs of your own, entered with their names on the web Config page.
* **Volume from the decoders:** **Knob: volume** in the short press menu of the decoders and of the live waterfall turns the encoder into a volume control, **Knob: tune** back to tuning.
* **Input level meter** in the title bar of the decoders and of the live waterfall: the live level of the ADC or microphone input, red when it clips.

#### Changed

* **FT8/FT4** decode on any frequency, for example 27.265 MHz in the CB band, not only near the standard amateur frequencies.

### Version 1.03

#### Added

* **Internet radio** (Tools page and Misc menu): stations from the [radio-browser.info](https://www.radio-browser.info) directory (top stations, your country, genres, search by name) and up to 32 favorites, played through a PCM5102 DAC wired to IO12 (BCK), IO13 (DIN) and IO14 (LRCK), with GPIO44 driving its XSMT (mute) pin. Turn on **PCM5102 DAC** in Settings to enable it; otherwise the entry is grayed out.
* **Beacons** (Decode menu and Tools page): monitor the 18 NCDXF/IARU International Beacon Project beacons on 14.100 ... 28.200 MHz and count the dashes heard (100 W, 10 W, 1 W, 100 mW), on one band or following one beacon across the five bands.
* **Bricks**, a brick-breaking game played with the encoder (Tools page and Misc menu), with levels and a saved best score.
* **Wi-Fi Power** setting (2 ... 19.5 dBm) to lower the Wi-Fi transmit power and keep the voltage regulator cooler.
* **FT8/FT4:** scrollable list of the decoded messages and a choice of receiver bandwidth (1.2k, 2.2k, 3.0k, 4.0k) in the short press menu.
* **RTTY:** **Tones: hold** option to stay on the decoded signal instead of switching to a stronger one.

#### Changed

* The audio decoders use the IO11 line-in (ADC) only; the INMP441 I2S microphone input was removed, its pins are used by the PCM5102 DAC.
* The Tools page continues on a second page when there are more tiles than fit on the screen.
* The Speaker option was removed from the FT8/FT4 menu; the speaker stays on in these decoders.

#### Fixed

* The I2S clocks of the DAC are stopped when leaving the Internet radio, and its pins are held low otherwise, to avoid birdies on HF.

### Version 1.02

#### Added

* **Speaker On/Off** in the short press menu of the decoders (FT8, FT4, CW, RTTY, PSK31, SSTV, WEFAX) and of the live waterfall: decode silently with the ADC input on IO11. The speaker comes back when leaving the decoder, and the setting is remembered.
* **FT8 and FT4 tuning step** of 100 Hz, 500 Hz or 1 kHz, selected from the short press menu.
* **ETM sensitivity** (Low, Normal, High) in the short press menu of the ETM screen: High catches weaker stations. The band is scanned again when it changes, and the setting is remembered.
* **REST API** on the web server, available whenever Wi-Fi is on: read the status, control the receiver and download the saved SSTV and WEFAX pictures. See [REST API](#rest-api) below.

### Version 1.01

#### Added

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

#### Changed

* **FT8 and FT4** start decoding on the tuned frequency; the standard frequency is available from the short press menu.
* The About screen credits this edition and the original firmware, and the first start shows a short welcome screen.

#### Fixed

* Wrong colors in SSTV pictures, the FT8/FT4 waterfall, WEFAX images and the propagation map.

### Version 1.00

The first release of this edition: FT8 and FT4 (with band hopping), CW, RTTY, PSK31, SSTV and WEFAX decoders, with the INMP441 microphone or the IO11 ADC as input; weather, propagation, grey line, best band, On air now (EiBi), spectrum and waterfall, space-weather alert; English, French, Dutch and German menus.


## License

MIT. Copyright of the original work belongs to its authors.
