# YAKSHA Landslide Monitor — PWA

Installable app version of the receiver dashboard, matching the original firmware
page's look. Live data, Tilt/Vibration/Displacement tiles with page-configurable
thresholds, phone-based siren tones, a fully custom "if this + this -> do this"
alert rule builder, and now a **completely offline map** served off the receiver's
own SD card.

## Files
```
1_index.html                        -> rename to index.html
2_manifest.json                     -> rename to manifest.json
3_service-worker.js                 -> rename to service-worker.js
4_icon-192.png, 5_icon-512.png
leaflet/                            -> Leaflet map library, bundled locally (no CDN, no internet needed)
6_receiver_alert_relay_soil_CORS.ino -> flash this to the receiver (adds CORS + tilt/tile endpoints)
8_download_tiles.py                 -> run ONCE on your PC (with internet) to prepare offline map tiles
```

## 1. Flash the firmware
Flash `6_receiver_alert_relay_soil_CORS.ino`. On top of the CORS fix from before,
this version adds:
- `tiltThresholdDeg` as a runtime-settable value (was hardcoded at 8°) with a new
  `/settilt` endpoint, saved to flash so it survives reboot.
- `tiltThreshold` field in the `/data` JSON.
- `/tiles/<z>/<x>/<y>.png` endpoint that serves map tiles straight off the SD card
  for the offline map (see step 2).

## 2. Prepare the offline map (one-time, needs internet — do this at home/office)
1. `pip install requests`
2. Open `8_download_tiles.py`, edit the `MIN_LAT`/`MAX_LAT`/`MIN_LON`/`MAX_LON` box
   to cover your deployment site (get corner coordinates from Google Maps —
   right-click a point, the numbers shown are lat, lon).
3. Run `python download_tiles.py`. It creates a `tiles/` folder shaped like
   `tiles/<z>/<x>/<y>.png` — the standard XYZ tile layout the receiver expects.
4. Copy the whole `tiles` folder onto the **root** of the SD card (so the card has
   `/tiles/13/...`, `/tiles/14/...` etc.), then put the card back in the receiver.

After this, the dashboard's map works with **zero internet at the site** — tiles
come from the receiver over your local WiFi, same as the sensor data.

## 3. Host the app files
Rename off the numeric prefixes (`1_index.html` → `index.html`, etc.) and upload
`index.html`, `manifest.json`, `service-worker.js`, `icon-192.png`, `icon-512.png`,
and the whole `leaflet/` folder together to any static host (GitHub Pages,
Netlify, etc. — same as before). The map library itself is now bundled locally
too, so once the app is installed, nothing about it needs the public internet —
only your phone's local WiFi connection to the receiver.

## 4. First run
Same as before: open the hosted URL → tap ⚙ → enter `landslide.local` or the
receiver's IP → Save → Add to Home Screen.

## New: Tilt / Vibration / Displacement tiles
Below the Temperature/Humidity/Soil row, three more tiles:
- **Tilt** — "Excessive" or "Nil". Tap its ⚙ to set the alert threshold in degrees
  (this actually rewrites the receiver's setting via `/settilt` — needs the new
  firmware flashed).
- **Vibration** — "Yes" or "Nil".
- **Displacement** — meters of GPS drift from the home point, or "Nil". Tap its ⚙
  to set the radius (same value as "Geofence Radius" in Admin View).

## New: Phone Siren
Real audio, played from your phone (you have no physical buzzer). Tap
"Phone Siren: ON/OFF" to mute/unmute — it's on by default and unlocks itself the
first time you tap anywhere on the page. Three tones matching alert severity;
tap "Acknowledge Alert" to silence the current one without dismissing the alert
itself.

## New: Custom Alert Rules
Tap "Custom Alert Rules" to open the builder. Each rule is:
- One or more **conditions** (metric + operator + value) — vibration, tilt
  degrees, displacement meters, soil %, temperature, humidity, risk score, or any
  of the boolean alert flags.
- Combined with **AND** (all must be true) or **OR** (any one).
- One or more **actions**: play a phone tone (choose frequency/beep count), show
  a phone notification, vibrate, send a webhook (paste any Telegram bot API URL,
  Discord webhook, IFTTT Maker webhook, etc. — the rule fires a POST with the
  rule name + live data as JSON), or just log silently.

Example: "Vibration Alert = true AND Tilt > 5 AND Displacement > 3" → tone +
notification + webhook. Everything runs on your phone against the live feed, so
you can add/edit/delete rules anytime without re-flashing anything. Rules and
their activity log are saved in your browser (per device/browser you install the
app on).

Tap "🔔 Enable Phone Notifications" once to allow the notify action to actually
show a system notification (browser permission requirement).

## Notes
- Everything (data, map tiles, rule engine) is local-network / on-phone. Nothing
  here calls out to the internet except an optional webhook you configure
  yourself, and even that's your choice per-rule.
- If "Cannot reach ___" shows up, check the address in ⚙ settings and that your
  phone is on the same WiFi as the receiver (or its AP if still in setup mode).
