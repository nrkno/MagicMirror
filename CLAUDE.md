# MagicMirror — Oslo Edition

Custom MagicMirror setup for a Raspberry Pi behind a physical mirror. Oslo-focused: local time, Ruter public transport departures, Norwegian holidays, and weather.

Upstream: [MagicMirrorOrg/MagicMirror](https://github.com/MagicMirrorOrg/MagicMirror). Fork lives at `nrkno/MagicMirror`.

## Project goals

- Display Ruter departures for a specific stop (coordinates-based)
- Oslo clock, Norwegian holidays calendar, later: Yr weather
- Keep the UI minimal — no clutter, dark mirror aesthetic

## Running the app

### First time setup

```sh
npm run install-mm
```

Use `install-mm` instead of plain `npm install` — it omits dev dependencies and avoids engine version conflicts with ESLint plugins that require a newer Node patch than what's installed.

### On macOS (local dev)

```sh
npm run server
```

Then open http://localhost:8080 in any browser. No Electron needed.

If port 8080 is in use, find and kill the occupying process first:
```sh
lsof -ti :8080 | xargs kill
npm run server
```

To open a native Electron window instead of the browser:
```sh
./node_modules/.bin/electron js/electron.js
```

### On the Pi (with a real screen)

```sh
npm start
```

This runs `start:wayland`. Requires a display attached to the Pi.

### On the Pi (headless, for testing)

`npm start` fails without a Wayland compositor. Use Xvfb:
```sh
xvfb-run -a env DISPLAY=:99 ELECTRON_OZONE_PLATFORM_HINT=x11 ./node_modules/.bin/electron . --no-sandbox
```
Requires `apt install xvfb`.

## Config

Config lives at `config/config.js`. Create it from the sample if it doesn't exist:
```
cp config/config.js.sample config/config.js
```

Key settings for this setup:
- Timezone: `Europe/Oslo`
- Language: `nb` (Norwegian Bokmål) or `en`
- Norwegian holiday calendar:
  `https://calendar.google.com/calendar/ical/nb.norwegian%23holiday%40group.v.calendar.google.com/public/basic.ics`

## Custom modules

Custom modules go in `modules/`. Each module is a directory:
```
modules/
  MMM-RuterDepartures/
    MMM-RuterDepartures.js   # module definition (browser-side)
    node_helper.js           # server-side helper (optional, for API calls)
    MMM-RuterDepartures.css  # styles
```

### Ruter departures module (in progress)

Target stop: coordinates `59.932267, 10.724083` (configurable in `config/config.js`).

Ruter's public API endpoint for stop discovery and departures:
- Stop by coordinates: `https://api.ruter.no/v2/travel/getStopsByCoordinates/?X=<lat>&Y=<lon>&proposals=1`
- Departures: `https://api.ruter.no/v2/travel/getdepartures/<stopId>`

The module should fetch from `node_helper.js` (server side) and pass results to the frontend module for rendering — do not embed the Ruter monitor website (`mon.ruter.no`) in an iframe.

## Node / npm

Node 24 via nvm is required. The system Node on Linux was too old — install nvm and `nvm use 24` before running `npm install`.

## Repo hygiene

- `config/config.js` is safe to commit as long as it contains no secrets (API keys, tokens). The current Oslo config has none, so commit it freely.
- If you add a private API key later, use `config/config.js.template` instead — put `${MY_SECRET}` placeholders in it and MagicMirror will compile it to `config.js` at startup using environment variables. Commit the template, not the compiled file.
- `modules/` third-party modules should each be gitignored or tracked separately
- Keep `node_modules/` out of commits (already in `.gitignore`)
