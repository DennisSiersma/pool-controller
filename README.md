# Pool Controller

ESPHome-config voor een zwembadpomp-controller op een **Waveshare ESP32-S3-Relay-6CH**.

## Hardware

- Board: Waveshare ESP32-S3-Relay-6CH (ESP32-S3, 6 relaiskanalen, RS485, buzzer, RGB-LED, 4 knoppen met LEDs)
- Aansturing: pompsnelheden (laag/mid/hoog) via relais, flow-sensor, knoppen met status-LEDs

### Pinout

| Functie | GPIO |
|---|---|
| BOOT | GP0 |
| Buzzer | GP21 |
| RGB LED | GP38 |
| Relais CH1..CH6 | GP1, GP2, GP41, GP42, GP45, GP46 |
| RS485 TX / RX | GP17 / GP18 |
| Knoppen 1..4 | GP5, GP4, GP40, GP39 |
| Knop-LEDs 1..4 | GP13, GP14, GP15, GP16 |
| Flow-sensor | GP12 |
| I2C SDA / SCL | GP8 / GP10 |

## Crash-onderzoek (historie)

**Symptoom (2025):** device draaide een tijd stabiel (uren tot ~een dag) en ging daarna in een reboot-loop (verbinding viel elke ~15 s weg en terug).

**Bevindingen:**

- Geen logische fout in de YAML: pinindeling klopt, relais/knop-interlocks zijn correct, geen blocking lambda's of hoogfrequente intervals.
- Verdachte resource-mix: `arduino`-framework + `web_server` + `esp32_improv` (BLE-stack) + WiFi + API + `neopixelbus` tegelijk. Past bij heap-fragmentatie / out-of-memory na verloop van tijd.
- Firmware op device was verouderd (2024.8.0) t.o.v. dashboard (2026.5.1).
- Na het verwijderen van BLE (v2): stabiel. Diagnose-sensoren (juni 2026): heap 259 kB vrij, geen crash-patroon.

## Versies (zie git-historie)

1. **v1 — originele config.** Werkende basis, maar crashte na enkele uren. BLE (`esp32_improv`) actief voor het uitlezen van zwembadsensoren.
2. **v2 — diagnoseversie (`esphome-web-3e8cfc-rs485.yaml`, nu actief op het device).**
   - `esp32_improv` / BLE verwijderd (grootste onnodige RAM-verbruiker). `improv_serial` behouden (serieel, kost vrijwel niks).
   - Diagnostiek toegevoegd: `Heap Free`, `Heap Max Block`, `Loop Time`, `Uptime`, `WiFi Signal` en `Reset Reason`.
   - RS485-aansturing van de iSaver+ (traploos rpm), relais als backup.
3. **v3 — ESP-IDF + Bluetooth proxy (`esphome-web-3e8cfc-rs485-ble.yaml`, NOG NIET GEFLASHT).**
   - `framework: arduino` → `esp-idf` (sinds ESPHome 2026.1 de standaard; zuiniger met RAM en de aanbevolen basis voor BLE).
   - `neopixelbus` (Arduino-only) → `esp32_rmt_led_strip`.
   - `esp32_ble_tracker` + `bluetooth_proxy` (active, 1 connection slot, conservatieve scan-window) om de **Blue Riiot Blue Connect Plus** via BLE in Home Assistant te krijgen.
   - Bewaking: HA-automation "Poolcontroller: stabiliteitsbewaking" pusht bij elke herstart (Reset Reason) en bij heap < 100 kB. Onstabiel? → v2 terugflashen.

## Flashplan v3

1. OTA via ESPHome add-on naar 192.168.1.238 (eerste IDF-build compileert 10-15 min).
2. Pomp blijft fysiek draaien tijdens flash; alleen besturing is even weg. Worst case bij mislukte OTA over de framework-wissel: USB-herflash.
3. Na flash checken: `Modbus status` = Online, RPM klopt, RGB-led-kleuren goed (anders `rgb_order: RGB`).
4. Blue Connect verschijnt daarna als ontdekt BLE-apparaat in HA; uitlezen via de BlueConnect HACS-component.

## Secrets

Kopieer `secrets.yaml.example` naar `secrets.yaml` en vul je eigen waarden in. `secrets.yaml` staat in `.gitignore` en wordt nooit gecommit.
