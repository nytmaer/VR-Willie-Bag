# TESTING — VR-Willie-Bag

Three layers, in order of how much doubt actually lives in each.

1. **Logic self-test** — `?test`. Headless, runs anywhere, catches rule regressions. Green today.
2. **Detection metrics** — `?debug=1`. The real unknown (P3). Needs a Quest; the readout turns "does it feel right" into numbers.
3. **Surface + on-device pass** — the manual checklist below, per HANDOFF §7.

No build step, no framework, no npm — same constraint as the app. The self-test lives inside `index.html` behind a query flag and never runs in normal play.

---

## 1. Logic self-test — `?test`

Open `index.html?test` in any browser. A full-screen report lists each check PASS/FAIL; the same summary prints to the console (`[willie test] N/N passed`). No headset, no server needed — a local file works.

Covered (all pure, no DOM/three.js state):

| Check | Why it matters |
|-------|----------------|
| 8 numbers + 8 zones defined | Guards against a half-edited data model |
| `8` (jab body) is a **lead** hand | The one number where even ≠ rear — easy to get wrong |
| `2` is a rear hand | Baseline orthodox sanity |
| `neededHand` orthodox: `2`→R, `8`→L | Core of the hand check |
| `neededHand` southpaw: `2`→L, `8`→R | The mirror flips the hand, not the number (HANDOFF §1) |
| parser: `7-7-5-1-2-1`, mixed separators, junk stripped, empty | Custom-combo input can't crash or admit `9`/`0` |

**Add a case** by dropping an `eq(name, got, want)` or `ok(name, cond)` line into `runTests()` (§12). Keep the tested logic in a pure function (like `neededHand` / `parseCombo`) so the test exercises the real code, not a copy.

**Done when:** `?test` reports all green after any change to numbering, the hand check, or the parser.

---

## 2. Detection metrics — `?debug=1`

Open the app with `?debug=1` and enter VR. A bottom-left overlay shows, live:

- **fps** and whether the session is `[VR]` or `[flat]`
- the four **thresholds** currently in force (`DET` at the top of §8)
- **peak speed** per glove since its last punch (m/s)
- **last hit** — hand, number, trigger speed, and distance-at-trigger in cm

The four knobs, all in `DET` (§8), are guesses until real punches move them:

| Const | Now | Question the metrics answer |
|-------|-----|------------------------------|
| `speed` | 1.1 m/s | Does a **jab** clear it as easily as a **hook**? Watch peak speed for both. |
| `hitR` | 0.135 m | Are real punches triggering, or landing just outside? Watch distance-at-trigger. |
| `rearmR` | 0.17 m | Does one punch ever register twice? Does the re-arm ever strand you? |
| `dir` | −0.35 | How many clean punches get rejected for angle? (No trigger despite speed + proximity.) |

Record into the table below. Throw **10 intended** punches and note how many register; then deliberately do the four things that must **not** register.

```
Punch type   intended→registered   peak m/s (min–max)   dist at trigger (cm)
jab   (7)     __ / 10               ___ – ___            ___
hook  (1)     __ / 10               ___ – ___            ___
upper (3)     __ / 10               ___ – ___            ___
body  (5)     __ / 10               ___ – ___            ___

Must NOT fire:   guard held up [ ]   slow reach-to-point [ ]   reset between punches [ ]   double-count [ ]
```

**Done when (T2):** ten intended punches register ten times, and none of the four false-positive cases fire.

---

## 3. On-device / surface checklist — HANDOFF §7

### VR (Meta Quest Browser, immersive) — primary target, T1
Serve over **https** (GitHub Pages once public, or `npx serve` + `npx cloudflared tunnel`). `file://` / `content://` will refuse the session.

- [ ] Session starts; bag sits at roughly chest height (not floating ~1.6 m high — that's the P2 `local` fallback)
- [ ] **Gloves** show as fist + wrist/forearm, red left / blue right, forearm pointing back toward the elbow (not forward through the bag)
- [ ] A real punch registers; a slow reach does **not**
- [ ] Wrong **number** ends the round; the 3D panel reads `OUT`
- [ ] Wrong **hand** ends the round (hand check on, orthodox)
- [ ] **Reset works in-headset:** after a fail, the panel says "pull either trigger to run it again" and a trigger restarts the combo — no need to remove the headset *(the bug that prompted this pass)*
- [ ] A full `7-7-5-1-2-1` can be completed in VR
- [ ] fps holds 72 (or 90) — check the `?debug=1` readout; watch for hitches when zones change state

### VR (Meta Quest Browser, 2D tab)
- [ ] Layout readable; tap-to-punch works; the Enter VR button's diagnostics make sense
- [ ] Known issue **P1**: the `#callrow` chips overlap the lower zone row (`5·8·6`). Not yet fixed — confirm it's no worse.

### Desktop Chrome
- [ ] Keys `1`–`8` throw; drag rotates the view; tap-without-drag throws
- [ ] Audio plays after the first interaction (gesture-gated)
- [ ] Hand check is **skipped** (pointer/keys pass `hand=null`) — wrong-hand can't fail here by design

### Mobile Safari / Chrome
- [ ] Tap targets hit; safe-area insets respected; no timer-related layout jump

---

## Quick reference

| URL | Does |
|-----|------|
| `index.html`          | Normal play |
| `index.html?test`     | Run the logic self-test, show a PASS/FAIL report |
| `index.html?debug=1`  | Live detection-metrics overlay (peak speed, trigger distance, fps) |
