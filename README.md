# Redline

A Wi-Fi speed test that will eventually look and sound like a supercar dashboard.

This is **v1**: a bare-minimum speed tester in a single HTML file. No styling, no frameworks, no installs.

## What v1 does

- Measures **ping** (median of 5 tiny requests)
- Measures **download speed** in Mbps, updating live while the test runs

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
- **Download:** request 25 MB and read it chunk by chunk with `response.body.getReader()`, recalculating the speed after each chunk.

## Known limitations

- Uses a single connection, so very fast connections may read low.
- Speed is averaged from the start of the download, so it reacts slowly to changes.
- Downloads a fixed 25 MB instead of running for a set time.
- Download only, no upload test.

## Roadmap

The idea: instead of a plain speedometer, your internet speed drives a car's rev counter. Faster connections go up through more gears, with real engine sounds and gear shifts.

- [x] **v1:** Basic ping and download test
- [ ] **v2:** More accurate test: timed run, parallel connections, smoothed live speed
- [ ] **v3:** Rev counter gauge drawn on a canvas, with a needle that follows the speed
- [ ] **v4:** Gearbox: speed ranges mapped to gears 1 to 6, with rev drops on each shift
- [ ] **v5:** Engine sound that revs with the needle
- [ ] **v6:** Real recorded engine clips for each gear, upshifts and downshifts
- [ ] **v7:** Full supercar-style dashboard design with startup sequence, animations, results and run history

## Built with

Plain HTML and JavaScript.

## License

MIT
