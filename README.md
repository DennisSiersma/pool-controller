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

## Crash-onderzoek

**Symptoom:** device draait een tijd stabiel (uren tot ~een dag) en gaat daarna in een reboot-loop (verbinding valt elke ~15 s weg en terug).

**Bevindingen:**

- Geen logische fout in de YAML: pinindeling klopt, relais/knop-interlocks zijn correct, geen blocking lambda's of hoogfrequente intervals.
- Verdachte resource-mix: `arduino`-framework + `web_server` + `esp32_improv` (BLE-stack) + WiFi + API + `neopixelbus` tegelijk. Past bij heap-fragmentatie / out-of-memory na verloop van tijd.
- Firmware op device was verouderd (2024.8.0) t.o.v. dashboard (2026.5.1).

## Versies (zie git-historie)

1. **Commit 1 — originele config.** Werkende basis, maar crasht na enkele uren. BLE (`esp32_improv`) actief voor het uitlezen van zwembadsensoren.
2. **Commit 2 — diagnoseversie (huidige).** 
   - `esp32_improv` / BLE verwijderd (grootste onnodige RAM-verbruiker). `improv_serial` behouden (serieel, kost vrijwel niks).
   - Diagnostiek toegevoegd: `Heap Free`, `Heap Max Block`, `Loop Time`, `Uptime`, `WiFi Signal` en `Reset Reason`.
   - Functioneel verder identiek.

## Zo sluit je de oorzaak uit

1. Flash de diagnoseversie en werk meteen de firmware bij naar de actuele ESPHome-versie.
2. Volg over de uren `Heap Free` en `Heap Max Block` in Home Assistant.
   - Dalen die gestaag tot vlak voor een crash → bevestigd geheugenprobleem (geen logica-bug).
3. Kijk na een crash naar `Reset Reason` — die toont waarom de ESP herstartte (panic, watchdog, brownout, ...).
4. Blijft het crashen, dan is `web_server` de volgende kandidaat om uit te zetten. Overweeg ook `framework: esp-idf` voor lager RAM-gebruik.

## Secrets

Kopieer `secrets.yaml.example` naar `secrets.yaml` en vul je eigen waarden in. `secrets.yaml` staat in `.gitignore` en wordt nooit gecommit.
