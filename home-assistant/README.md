# Home Assistant - iSaver pool-pomp besturing (RS485)

Deze map bevat de HA-kant voor het aansturen van de iSaver+ via de
RS485-config (`esphome-web-3e8cfc-rs485.yaml` in de hoofdmap).

## Bestanden
- `scripts.yaml`     - presets (Nacht/Dag/Backwash/Uit) + Max-2-uur
- `dashboard.yaml`   - Lovelace-kaart (gauge, schuif, presetknoppen, status)
- `automations.yaml` - voorbeeld dagschema

## Installeren
1. Flash eerst `esphome-web-3e8cfc-rs485.yaml` en voeg het device toe in HA.
2. Controleer de entity_ids (Instellingen > Apparaten > Entiteiten):
   - `number.waveshare_relay_rs485_pomp_rpm_rs485`
   - `sensor.waveshare_relay_rs485_huidige_rpm_rs485`
   - `sensor.waveshare_relay_rs485_modbus_status`
   Wijken ze af? Pas ze dan aan in de YAML-bestanden.
3. Voeg de scripts toe, daarna de dashboard-kaart, en optioneel de automatisering.

## Belangrijk
- RS485 werkt alleen als de iSaver via het FRONTPANEEL is aangezet.
- Alleen rpm lezen/schrijven werkt; vermogen/status/foutcodes komen NIET
  over RS485 (wel een Modbus online/offline-status).
- Gebruik RS485 OF de relais-backup, niet door elkaar.
