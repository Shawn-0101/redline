# Redline

A Wi-Fi speed test that looks and sounds like a supercar dashboard.

This is **v7**: the full supercar-style dashboard. Single HTML file with the audio built in.

## What it does

- Measures **ping** and runs a **10 second download test** over 4 parallel connections
- **Rev counter** with a 6-speed gearbox, driven by your connection speed
- **Real engine recordings** for the starter, idle, every gear, and downshifts back to 2nd
- **Startup sequence:** light sweep, scale draws in, needle sweep and shift-light ripple
- **Shift lights**, rev limiter strobe, and a red zone that glows as you approach it
- **Screen shake** on every shift, plus an engine buzz that grows with RPM
- **Peak and average** dials, a **results strip**, and **run history** saved in your browser
- **Demo run** that always goes through all 6 gears
- Works on phones and respects reduced motion settings

## Run it

1. Clone the repo
   ```bash
   git clone https://github.com/Shawn-0101/redline.git
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
- **Gauge:** drawn on a `<canvas>` every frame with `requestAnimationFrame`.
- **Gearbox:** each gear covers a speed range (1st 0–10, 2nd 10–30, 3rd 30–80, 4th 80–200, 5th 200–500, 6th 500+ Mbps). Where you are in that range sets the RPM.
- **Needle:** a spring. It's pulled toward the target RPM and slowed by damping, so it overshoots slightly and settles like a real needle.
- **Engine:** each gear plays its own recorded clip. The needle follows an RPM curve measured from each clip's pitch, so the visuals match the audio.
- **Design:** night-time performance cluster. Charcoal glass with a carbon hood and aluminium trim, Geist and Geist Mono fonts, one red accent.
- **Animations:** CSS keyframes for the startup and page entrance, a spring on the cluster for screen shake, and count-up numbers for results.
- **History:** the last 8 runs are stored in `localStorage`.
- **Fallback:** if Cloudflare can't be reached, it says so and runs a demo instead.

## Known limitations

- Download only, no upload test.
- The HTML file is about 3.5 MB because the audio is built in.
- A full run with downshifts takes about 30 seconds.
- Ad blockers or strict networks can block Cloudflare, which switches it to demo mode.

## Roadmap

The idea: instead of a plain speedometer, your internet speed drives a car's rev counter. Faster connections go up through more gears, with real engine sounds and gear shifts.

- [x] **v1:** Basic ping and download test
- [x] **v2:** More accurate test: timed run, parallel connections, smoothed live speed
- [x] **v3:** Rev counter gauge drawn on a canvas, with a needle that follows the speed
- [x] **v4:** Gearbox: speed ranges mapped to gears 1 to 6, with rev drops on each shift
- [x] **v5:** Engine sound that revs with the needle
- [x] **v6:** Real recorded engine clips for each gear, upshifts and downshifts
- [x] **v7:** Full supercar-style dashboard design with startup sequence, animations, results and run history

## Built with

Plain HTML and JavaScript, Canvas and the Web Audio API. No frameworks, no build step.

## License

MIT
