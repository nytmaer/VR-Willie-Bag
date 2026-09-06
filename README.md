# VR-Willie-Bag

A number bag for the Cus D'Amato punch numbering system. Single file, no build step, no dependencies beyond a CDN copy of three.js.

Cus called numbers; the fighter threw punches. This does the same thing. One number lights up on the bag, you throw that punch. Throw the wrong one and the round ends. There is no timer — the whole point is learning the combinations slowly, in order, until they stop being numbers.

Not a rhythm game. There is nothing to keep up with.

## The numbers

| # | Punch |
|---|-------|
| 1 | Left hook to the head |
| 2 | Right hand to the head |
| 3 | Left uppercut |
| 4 | Right uppercut |
| 5 | Left hook to the body |
| 6 | Right hook to the body |
| 7 | Jab to the head |
| 8 | Jab to the body |

Numbering follows the Catskill / peek-a-boo system as it's commonly published. Gyms vary. If yours numbers differently, load your own combinations and treat the chart as labels rather than gospel.

## Playing it

**VR (Meta Quest, Wolvic):** open the page, tap Enter VR, punch the lit number. Punches are detected from controller position and velocity, so a real punch registers and a slow reach does not. By default the app also checks which hand you threw with — a `2` thrown with the left hand ends the round. Southpaws can flip that in Settings.

**Mobile / desktop:** tap the lit number on the bag. Keys `1`–`8` work on desktop.

Settings include calling the number aloud, hiding the printed numbers so you work off the voice alone, and turning off the hand check.

## Drills

Nine built in, from `7-2` up to an eight-punch string, including the peek-a-boo `7-7-5-1-2-1`. You can type your own with any digits 1–8; custom combinations last for the session.

## Sound

Everything is synthesised at runtime with the Web Audio API — no audio files ship with this. Each punch layers glove-through-air, a leather thud, a surface crack, a sub thump, and the swivel chain rattling afterward. Body shots come out duller, uppercuts heavier, jabs snappier.

## Running it

WebXR needs a secure origin. `file://` and `content://` will load the page and refuse the VR session — Meta Quest Browser reports this as an unsupported session configuration.

GitHub Pages: Settings → Pages → deploy from `main`, root. **Pages does not serve private repositories on the free plan**, so the repo must be public or the account on a paid plan.

Locally over https:

```
npx serve -p 8000
npx cloudflared tunnel --url http://localhost:8000
```

Open the printed `https://…trycloudflare.com` URL in the headset.

## Roadmap

- **Recorded combinations with per-punch timing.** Record your own string and set the gap before each punch independently, so the `1` in a `1-2` can come faster than the `2`. Each step is a bare number today; the shape it wants is `{n, gap}`.
- Punch-detection tuning per user — the velocity threshold is currently fixed at 1.1 m/s.
- Hand tracking as an alternative to controllers.

## Built with

three.js r128, WebXR, Web Audio API. One file, about 46 kB.
