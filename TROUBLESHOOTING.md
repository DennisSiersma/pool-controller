# Troubleshooting-log

Onderzoek naar het crashen van de pool-controller (ESP32-S3) na enkele uren.
Datum: 30 mei 2026.

## Symptoom

Device draaide een tijd stabiel (uren tot ~een dag) en ging daarna in een
reboot-loop: verbinding viel elke ~15 s weg en kwam terug.

## Drie aangepakte oorzaken

### 1. Geheugen / BLE
De oude config draaide `arduino`-framework met tegelijk `web_server`,
`esp32_improv` (BLE-stack), WiFi, API en `neopixelbus`. Die mix is een
geheugen-slokop; "werkt uren, crasht dan" past bij heap-fragmentatie / OOM.

**Fix:** `esp32_improv` (BLE) verwijderd — BLE was alleen nog voor de
zwembadsensoren en wordt anders opgelost. `improv_serial` (serieel, geen BLE)
behouden. Resultaat: ~261 KB heap free na boot, ~212 KB grootste blok.

### 2. Verouderde firmware
Device draaide ESPHome **2024.8.0**, terwijl het dashboard op **2026.5.1** zat.
Sindsdien zijn meerdere geheugenlekken gefixt.

**Fix:** opnieuw geflasht → firmware nu 2026.5.1.

### 3. Zwak WiFi-signaal (hoofdoorzaak van OTA-fouten)
De ESP hing op **-85 / -79 dBm** aan de verre AP **U7 Pro Max**. Reden: de
dichtstbijzijnde AP (**U7 In-Wall**, ~8 m van het zwembad) stond **offline
sinds 26 mei**. Daardoor moest de ESP uitwijken naar een verder AP.

Symptomen hiervan:
- OTA-uploads faalden halverwege ("Connection reset by peer", Errno 104).
- Zeer lage datarate (1 Mbps), wankele verbinding.

**Fix:**
1. U7 In-Wall herstart → weer online (kreeg meteen firmware 8.6.11).
2. ESP een force-reconnect gegeven (ESP's roamen niet uit zichzelf).
3. Resultaat: ESP nu op de **U7 In-Wall, kanaal 6, -55 dBm** (SNR ~40 dB).

## Diagnostiek toegevoegd

In de config zitten nu sensoren om dit te blijven volgen:
`Heap Free`, `Heap Max Block`, `Loop Time`, `Uptime`, `WiFi Signal` en
`Reset Reason`. Een Reset Reason van "software via esp_restart" = normale
herstart (flash/OTA); een panic/watchdog/brownout wijst op een echte crash.

Er draait een geplande controle (elke 2 uur) die uptime, heap, signaal en
reset-reason checkt en waarschuwt bij een reboot, wegzakkend geheugen of
offlinestatus.

## Open punten

- Statisch IP `192.168.1.47` wordt (nog) niet toegepast; device pakt via DHCP
  192.168.1.238. `use_address` staat daarom tijdelijk op .238 voor OTA.
- WiFi-stabiliteit blijven monitoren nu de In-Wall weer draait.
- Overweeg `framework: esp-idf` als er ooit toch nog geheugendruk optreedt.
