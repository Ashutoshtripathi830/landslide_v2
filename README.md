# YAKSHA Landslide Monitor — Merged Final Package (11)

This is file 9 (current system) merged with file 10 (upgraded admin/history
build). File 10's firmware and dashboard are a strict superset of file 9's —
every feature from file 9 (tilt/vibration/displacement tiles, phone siren,
custom alert rule builder, offline SD-served map) is already present in file
10, plus the new SD-history endpoint, device health stats, trend charts, and
prediction readout. So the merge = take file 10's upgraded firmware +
dashboard, and carry forward the file-9 assets that file 10 doesn't replace
(Leaflet library, tile downloader script).

## What's in this package
```
receiver_history_admin.ino   -> flash this to the receiver (was 9_receiver_history_admin.ino)
index.html                   -> upload this (was 10_index.html)
leaflet.js, leaflet.css      -> Leaflet map library, bundled locally (unchanged since file 9)
download_tiles.py            -> run once on your PC to prepare offline map tiles (unchanged since file 9)
```

## Still needed from earlier packages (not part of file 9 or file 10)
If you have these from an even earlier package, reuse them as-is — nothing
in this merge changes them:
- `manifest.json`
- `service-worker.js`
- `icon-192.png`, `icon-512.png`

## 1. Flash the firmware
Flash `receiver_history_admin.ino`. It includes everything from the older
CORS build (tilt threshold endpoint, offline map tile serving, SD logging,
CORS headers, relay/siren, LED priority, geofence) **plus**:
- `GET /history?n=<rows>` — tails the SD card's CSV log as compact JSON, used
  by the dashboard's "Full Log (SD)" chart mode. Bounded read, so it can't
  run the board out of RAM.
- `/data` now also reports `freeHeap`, `wifiRSSI`, and `uptimeSec` for the
  Admin View's device health table.

## 2. Prepare the offline map (optional, but recommended)
See `download_tiles.py` — run once on your PC with internet:
- Layer 1 (Assam-wide overview, zoom 6–12) is already configured.
- Layer 2 (your deployment site, zoom 13–19, true street level) still has a
  placeholder box — edit `SITE_MIN_LAT`/`MAX_LAT`/`MIN_LON`/`MAX_LON` to your
  real site's coordinates once you have them.
- Copy the resulting `tiles/` folder onto the SD card root.

**You don't have to do this before using the app.** The dashboard's map now
tries the receiver's SD card first, and if a given tile isn't there yet,
falls back to the live OpenStreetMap server automatically — as long as
whatever phone/laptop is viewing the dashboard has its own internet
connection at that moment. The receiver itself still never needs internet.
Once you do prepare and copy the SD tiles, the map switches to using those
first (zero internet needed at the site) without any other change.

## 3. Host the app files
Upload `index.html` together with `manifest.json`, `service-worker.js`, the
icons, and `leaflet.js`/`leaflet.css` to your static host.

## 4. First run
Open the hosted URL → tap ⚙ → enter `landslide.local` or the receiver's IP →
Save → Add to Home Screen.

## New: Set Home Location Directly (for a portable/carried unit)
Since this unit is compact and can be carried between sites, you can now
type in the exact home/site coordinates from the Admin View instead of
waiting for a GPS fix or editing any files:
- **Set Home Location** — type latitude and longitude, tap the button. Takes
  effect immediately, and is saved to the receiver's flash so it survives a
  reboot. All displacement/geofence alerts now measure drift from this point.
- **Use Next GPS Fix Instead** — clears the manual override and goes back to
  the original behavior (the next GPS fix becomes the new home point).
- The Admin View also now shows the current home coordinates and whether
  they came from GPS or were typed in manually.
- New firmware endpoints: `POST /sethome` (`lat`, `lng`) and `POST /resethome`.

## Feature summary (everything is now in one build)
- Live Temperature/Humidity/Soil + Tilt/Vibration/Displacement tiles with
  page-configurable thresholds.
- Phone-based siren tones, tap-to-acknowledge.
- Custom "if this + this -> do this" alert rule builder (conditions, AND/OR,
  tone/notify/vibrate/webhook actions).
- Fully offline map served off the receiver's own SD card.
- **New:** Admin View trend charts (Risk, Tilt, Vibration, Soil, Displacement,
  Temperature, Humidity) in Live (session) or Full Log (SD) mode, with a
  threshold line and a simple linear trend/"reaches threshold in ~X hours"
  prediction readout. Not a validated forecasting model — an early heads-up
  alongside the real sensor alerts.
- **New:** Device health table — uptime, free heap, WiFi RSSI, LoRa packet
  error rate.
- **New:** Session stats table (min/avg/max) for key metrics since page load.
- **New:** Export shown chart data as CSV, no server round-trip.

## Notes
- Everything is local-network or on-phone; nothing calls the public internet
  except an optional webhook you configure yourself in a Custom Alert Rule.
- Full Log chart mode needs the SD card present and logging — same
  requirement as the log download feature.
