# Redline

A Wi-Fi speed test that will eventually look and sound like a supercar dashboard.

This is **v2**: a basic speed tester in a single HTML file. No styling, no frameworks, no installs.

## What it does

- Measures **ping** (median of 5 tiny requests)
- Runs a **10 second download test** over 4 parallel connections
- Shows **live speed** (last 0.7s) and the **average** for the whole test

## Run it

1. Clone the repo
   ```bash
   git clone https://github.com/<your-username>/redline.git
   cd redline
   ```
2. Open `index.html` in your browser (double-click it), then press **Start**.

If your browser blocks the test when opened as a file, serve it locally instead:

```bash
python -m http.server 8000
```

Then go to `http://localhost:8000`.

## How it works

A browser can't measure your connection directly, so it downloads data and times it:

```
speed (Mbps) = bytes downloaded × 8 ÷ seconds ÷ 1,000,000
```

- **Server:** Cloudflare's public endpoint `speed.cloudflare.com/__down?bytes=N` returns N bytes of data.
- **Ping:** request 0 bytes and time the round trip with `performance.now()`.
- **Download:** 4 downloads run at once and keep restarting for 10 seconds, counting every byte that arrives.
- **Live speed:** every 120ms, compare the byte count now with 0.7s ago.
- **Stopping:** an `AbortController` cancels all downloads at once when time is up.

## Known limitations

- Download only, no upload test.
- Results depend on Cloudflare's nearest server, so they won't match every speed test site.

## Roadmap

The idea: instead of a plain speedometer, your internet speed drives a car's rev counter. Faster connections go up through more gears, with real engine sounds and gear shifts.

- [x] **v1:** Basic ping and download test
- [x] **v2:** More accurate test: timed run, parallel connections, smoothed live speed
- [ ] **v3:** Rev counter gauge drawn on a canvas, with a needle that follows the speed
- [ ] **v4:** Gearbox: speed ranges mapped to gears 1 to 6, with rev drops on each shift
- [ ] **v5:** Engine sound that revs with the needle
- [ ] **v6:** Real recorded engine clips for each gear, upshifts and downshifts
- [ ] **v7:** Full supercar-style dashboard design with startup sequence, animations, results and run history

## Built with

Plain HTML and JavaScript.

## License

MIT
