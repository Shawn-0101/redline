# Redline: Project Overview

## The idea

Redline is a Wi-Fi speed test that works like a supercar dashboard. Instead of a plain speedometer, your internet speed drives a rev counter: faster connections climb through more gears. Real recorded engine sounds play for every gear, with gear shifts, downshifts and screen shake.

The point is fun, not lab-grade accuracy. The test is real, but the dashboard and the sound are the main feature.

## How it plays out

1. **Engine Start**: the starter fires, a light sweeps across the glass, the scale draws itself in, the needle sweeps to 9,000 RPM and back, and the shift lights ripple. Then the engine settles into idle.
2. **Ping**: 5 tiny requests are timed and the median is shown.
3. **Launch**: the download test starts and the car pulls away in 1st.
4. **Climbing gears**: each gear plays its own recorded clip. When the speed passes the top of a gear's range, it shifts up. The dash jolts, the gear number slides up and the revs drop.
5. **Holding**: if the connection maxes out mid-gear, the engine holds near the top of that gear on a looped clip.
6. **Downshifts**: when the test ends, it plays the recorded downshifts from the top gear reached down to 2nd, then settles to idle and switches off.
7. **Results**: the peak and average gauges fill, the results strip counts up, and the run is saved to history.

## Features

- Live download speed, average, peak, ping and data used
- 6-speed gearbox driven by connection speed
- Real engine audio: starter, idle, 6 upshift clips, 4 downshift clips and a settle-to-2nd clip
- Needle that follows the actual pitch of the recordings
- Spring-physics needle with overshoot
- Shift lights, rev limiter strobe, and a red zone that glows harder as you approach it
- Screen shake on shifts: hardest in low gears, softer in high gears
- Engine buzz vibration that grows with RPM
- Demo mode that always goes through all 6 gears
- Results strip and run history (last 8 runs, saved in the browser)
- Responsive layout down to phone width
- Respects the system's reduced-motion setting

## Gearbox

| Gear | Speed range |
|---|---|
| 1st | 0–10 Mbps |
| 2nd | 10–30 Mbps |
| 3rd | 30–80 Mbps |
| 4th | 80–200 Mbps |
| 5th | 200–500 Mbps |
| 6th | 500+ Mbps |

- **Target gear**: worked out from the live smoothed speed during the test, then from the final average after it.
- **Upshifting**: the engine only upshifts at the end of the current gear's clip, so the sound always finishes cleanly. If the target gear is higher, it moves straight into the next clip.
- **Holding a gear**: if the target gear isn't higher, it crossfades into a cruise loop. While cruising, an increase in speed triggers an immediate upshift.
- **In 6th**: once the test is done and 6th has played for 2.5s, it goes to the downshifts.

## Speed test

- **Server**: Cloudflare's public endpoint `https://speed.cloudflare.com/__down?bytes=N` returns N bytes.
- **Fallback**: if Cloudflare can't be reached, it says so on the dash and runs a demo instead.
- **Ping**: request 0 bytes 5 times and take the median.
- **Download**: 4 parallel streams for 10 seconds. Each one keeps re-requesting 25 MB and counting bytes as chunks arrive (`response.body.getReader()`).
- **Live speed**: every 120ms, compare the byte count with 0.7s ago.
- **Stopping**: an `AbortController` cancels all streams when time is up.
- **Average**: the mean of the live speed samples after the first second.

```
Mbps = bytes × 8 ÷ seconds ÷ 1,000,000
```

## Sound engine (Web Audio API)

All clips are decoded into `AudioBuffer`s on page load, so START fires instantly.

| Clip | Use |
|---|---|
| `start` | Starter sound, one-shot |
| `idle` | Seamless idle loop, crossfaded |
| `g1`–`g6` | One pull per gear, played from the start on each upshift |
| `cruise` | Loop cut from the 6th gear clip, pitched to match the top of the current gear |
| `d6`–`d3` | Downshift clips (6→5, 5→4, 4→3, 3→2) |
| `dend` | Settles in 2nd, then crossfades to idle |

- **Scheduling**: downshift clips are queued back to back on the audio clock (`source.start(when)`), so there are no gaps.
- **Gains**: each clip has its own `GainNode`, used for crossfades. A master output gain handles mute.
- **Autoplay**: browsers block audio until the user clicks, so the `AudioContext` is resumed on START.

### How the clips were prepared

The source clips came from recordings of an R8 V10. They were processed offline with ffmpeg and Python (numpy and scipy):

- **Loops** (idle and cruise): cut and given a crossfade at the loop point so they repeat with no click.
- **Format**: exported as 32 kHz mono WAV. MP3 adds silence padding that breaks seamless loops.
- **Loudness**: all clips share one gain, so their relative loudness matches the recordings.
- **RPM curves**: the pitch of each clip was tracked every 0.1s (FFT harmonic sum, octave errors filtered) and converted to RPM with `rpm = pitch(Hz) × 62`. These curves are stored in a `META` object, and the needle follows them, so it matches what you hear.
- **Downshift points**: the gear-change moment in each downshift clip was found by detecting the rev-match blip (a pitch jump of more than 12%), and stored as `shiftAt`.
- **Embedding**: all clips are base64 data URIs inside the HTML, about 3.5 MB.

## Needle and visuals

- **Needle physics**: a spring with damping.
  ```
  vel += ((target - rpm) * k - vel * c) * dt
  rpm += vel * dt
  ```
  It uses k=220 and c=26 while driving, and k=70 and c=15 during the startup sweep. `dt` keeps it frame-rate independent.
- **Tacho**: drawn on a `<canvas>` every frame with `requestAnimationFrame`. The scale covers 0–9,000 RPM over a 270° sweep, with the redline at 7,500.
  - A lit arc trails the needle.
  - Numerals brighten as the needle passes them.
  - The red zone glow scales with RPM.
- **Shift lights**: 7 LEDs light from 6,200 to 7,600 RPM, then strobe above 7,750 (the limiter).
- **Screen shake**: a separate spring on the cluster's transform.
  - Upshifts kick it backwards. The force starts at 3.5 on launch and then shrinks gear by gear, so higher gears kick less.
  - Downshifts give a lighter kick forwards.
  - Random sub-pixel buzz scales with RPM.

## Design

- **Look**: a night-time performance cluster. A carbon-weave hood sits over charcoal glass with a brushed-aluminium trim, and the reflection on the glass follows the pointer.
- **Palette**: off-black and neutral greys, with one accent: performance red `#e0312b`, used for the redline, the needle, the start button and active states.
- **Type**: Geist for UI text and Geist Mono for numbers, using tabular figures so digits don't jitter.
- **Layout**: the cluster fills the first screen.
  - Left: a list of download, ping, server and data used.
  - Centre: the tacho, with Mbps and the gear in the middle.
  - Right: peak and average dials.
  - Below the cluster: Engine Start/Stop, Demo run, then the results strip and run history.
- **Motion**:
  - Entrance rise when the page loads.
  - Boot sweep and LED ripple on each run.
  - Gear digit slides up or down on shifts.
  - Breathing ring on the start button, which also pulls slightly towards the cursor.
  - Results count up and the dials fill.
  - A slow scan line drifts across the glass.
  - Everything is cut down for users with reduced motion turned on.
- **Copyright**: there are no Audi or R8 logos and no copied UI. It's an original design inspired by supercar digital clusters.

## Code structure

It's a single `index.html` with no frameworks and no build step.

| Section | Contents |
|---|---|
| `<style>` | Design tokens on `:root`, layout, animations, responsive breakpoints |
| Markup | Top bar, cluster (strip / panels / tacho / dials / footer), controls, results, history |
| `SND`, `META` | Embedded audio and RPM curves |
| `engine` object | `init`, `start`, `pull`, `toCruise`, `toDown`, `toIdle`, `stop`, and `tick()`, which returns the target RPM each frame |
| Network | `pickSource`, `measurePing`, `measure`, `simSpeed` (demo) |
| Run flow | `start()`: boot, ping, launch, measure, wait for the engine to finish, show results |
| Frame loop | `update()` (speed, needle, gear, LEDs, shake) and `draw()` (tacho) |
| History | `localStorage` key `redline-history`, wrapped in try/catch |

### Key constants

```js
MAX_RPM = 9000, REDLINE = 7500, IDLE = 950
GEARS = [0, 10, 30, 80, 200, 500, Infinity]
TEST_MS = 10000          // download test length
META.K = 62              // rpm per Hz of engine pitch
```

## Demo mode

- **Speed target**: picked at random between 620 and 770 Mbps, so it always reaches 6th.
- **Readout**: the fake speed is tied to the engine. Within each gear it rises through that gear's band as the clip plays, then falls steadily during the downshifts and reaches 0 at idle.
- **When it ends**: it doesn't run on a timer. It ends once 6th gear has been reached.

## Running it

- **Easiest**: open `index.html` directly.
- **If the browser blocks the test** when opened as a file, serve it locally:
  ```bash
  python -m http.server 8000
  ```
- **Hosting**: works on GitHub Pages or any static host.
- **Network blocks**: ad blockers or strict networks can block Cloudflare. If that happens it switches to demo mode.

## Known limitations

- Download only, no upload test.
- Accuracy depends on Cloudflare's nearest server and browser overhead.
- Fast connections need the parallel streams to saturate the link. Very fast links (1 Gbps or more) may still read low.
- Gear clips are short, about 1.9s each, so on a fast connection the engine reaches its target gear a few seconds after the speed does.
- A full run with downshifts lasts about 30 seconds.
- The HTML file is about 3.5 MB because the audio is embedded.

## Future ideas

- Upload test with its own moment (reverse gear, or a launch-control style countdown)
- Drive modes (Comfort / Dynamic / Performance) that recolour the cluster and change sound volume and shift aggressiveness
- Dedicated startup and rev-limiter recordings
- Shareable result cards
- PWA install with offline demo mode
- Optional split into `styles.css`, `js/` modules and a `sounds/` folder for easier maintenance
