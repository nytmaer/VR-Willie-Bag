# HANDOFF — VR-Willie-Bag

**Repo:** github.com/nytmaer/VR-Willie-Bag (private until launch)
**Owner:** nytmaer
**State:** v1 prototype, feature-complete on flat screen, **never successfully entered a VR session yet**
**Date of handoff:** September 2026

---

## 1. What this is

A WebXR heavy bag that teaches the Cus D'Amato punch numbering system. One number lights up on the bag; you throw that punch. Wrong punch ends the round. **There is no timer, ever** — infinite time between punches is the core design rule, not a placeholder.

This is explicitly **not** a rhythm game. Any change that introduces time pressure, a scrolling track, or a "keep up" mechanic is out of scope for v1 and should be raised with the owner before it's written.

The numbering:

| # | Punch | Hand (orthodox) |
|---|-------|-----------------|
| 1 | Left hook to the head | L |
| 2 | Right hand to the head | R |
| 3 | Left uppercut | L |
| 4 | Right uppercut | R |
| 5 | Left hook to the body | L |
| 6 | Right hook to the body | R |
| 7 | Jab to the head | L |
| 8 | Jab to the body | L |

Southpaw setting mirrors the hand column only, not the numbers.

---

## 2. Repo contents

```
index.html    the entire application, ~32 kB, no build step
README.md     public-facing description
```

That's it. There is no package.json, no bundler, no node_modules. `index.html` loads three.js **r128 from cdnjs** via a single `<script>` tag and does everything else inline.

---

## 3. Architecture map

`index.html` is organised into ten numbered sections. Search for the comment banners (`/* === 1. THE SYSTEM === */` etc.).

| § | Contents | Notes |
|---|----------|-------|
| 1 | `PUNCH`, `ZONES`, `DRILLS` | The whole data model. `PUNCH[n]` holds label, hand, score value, weight (drives sound + swing force), body flag. `ZONES` places each number on the bag by `theta` (degrees around the bag, + is the puncher's right) and `y` (height in bag-local metres). |
| 2 | `S` | Global mutable state: sequence, index, score, streak, alive, settings. |
| 3 | Audio | Fully procedural Web Audio. `noise()` and `tone()` are the primitives; `sfxHit(n)` layers five voices per punch. No audio files anywhere — keep it that way. |
| 4 | Scene | Room, lights, bag geometry, zone decals, the 3D readout panel, floating score pops. |
| 5 | Swing physics | Spring-damper on a ceiling pivot. `impulse(dir, force)` on hit, `stepSwing(dt)` each frame. |
| 6 | Game logic | `loadCombo`, `correct`, `fail`, `punch(n, worldPos, normal, hand)`. **`punch()` is the single entry point for every input path** — pointer, keyboard, and VR all funnel through it. Add new input methods by calling it, not by duplicating logic. |
| 7 | Pointer input | Raycast from camera. Drag rotates the view; tap without drag throws. Passes `hand = null` so the hand check is skipped on touch. |
| 8 | VR input | Two `getControllerGrip()` gloves. Per-frame velocity, proximity to zone centre, and direction-of-travel gate. `armed` flag debounces so one punch can't register twice. |
| 9 | UI wiring | DOM panel, drills list, custom combo parser, settings, the VR button and its diagnostics. |
| 10 | Loop | `setAnimationLoop`. |

**Geometry constants** live at the top of §4: `PIVOT_Y = 3.1`, `BAG_Z = -0.9`, `BAG_CY = 1.24`, `R = 0.28`, `HALF = 0.70`. The bag front faces +Z; the player stands at the origin looking down −Z.

---

## 4. Hard constraints

Do not break these without asking:

- **Single file.** No build step, no bundler, no npm. Someone must be able to download `index.html` and open it.
- **three.js r128, from cdnjs.** Pinned deliberately. r128 has no `CapsuleGeometry` — the bag is a cylinder plus two hemispheres for that reason. If you upgrade, upgrade on purpose and test on the headset.
- **No audio files.** All sound is synthesised at runtime.
- **No `localStorage` or `sessionStorage`.** Saved combos are session-only by design; the file has to survive being opened from odd origins.
- **No timer, no time pressure.** See §1.
- Every input path calls `punch()`.

---

## 5. Known issues

**P0 — VR has never run.** The owner hit `The specified session configuration is not supported` in Meta Quest Browser. Root cause is almost certainly the origin: the file was opened from `content://…fileprovider/…`, and Quest Browser only grants WebXR on a secure origin. The button now retries with a bare `requestSession` and reports `isSecureContext` on failure. **This is unverified.** Nothing in §8 has been exercised by a human.

**P1 — Mobile HUD collides with the bag.** The `#callrow` combo chips render over the lower zone row (5 / 8 / 6) on a headset-browser-sized viewport. Confirmed in a screenshot from the Quest 2D browser. The chips are also redundant with the 3D panel — consider hiding the DOM chips entirely when the 3D panel is in view.

**P2 — Reference space fallback is unguarded.** If the first `requestSession` fails and the bare retry succeeds, §9 switches to `'local'` reference space. `local` puts the origin at the headset rather than the floor, so the bag will sit roughly 1.6 m too high. Either offset the rig when falling back, or detect the space type and shift `camRig.position.y`.

**P3 — Punch detection thresholds are guesses.** `speed > 1.1` m/s, zone hit radius `0.135`, re-arm radius `0.17`, direction gate `vel · normal < -0.35`. Nobody has thrown a punch at this yet. Expect to tune all four.

**P4 — `speechSynthesis` inside an immersive session is untested** on Quest Browser. If it's silent during a session, fall back to a synthesised spoken-number substitute or a distinct tone per number.

**P5 — Zone decals are flat discs floating 18 mm off a curved surface.** Acceptable head-on, visibly wrong at grazing angles. Fix by building each zone as a curved cylinder sector, or by baking the numbers into a single wrap texture on the bag body and keeping invisible collider discs for hit testing.

---

## 6. Task list, in order

### T1 — Get it into VR and verify (blocking everything else)
Serve over https (GitHub Pages once the repo is public, or `npx serve` + `npx cloudflared tunnel`). Enter VR on a Quest. Confirm: bag sits at the right height, gloves appear, a real punch registers, a slow reach does not, the wrong-number fail fires, the 3D panel is readable.
**Done when:** a full `7-7-5-1-2-1` can be completed in VR and a deliberate wrong punch ends the round.

### T2 — Tune punch detection
Expose the four constants from P3 as named consts at the top of §8 with comments. Add a hidden debug readout (peak velocity of the last punch, distance at trigger) toggled by a query param such as `?debug=1`. Then tune against real punches.
**Done when:** ten intended punches register ten times, and guarding, resetting, or reaching to point does not trigger a fail.

### T3 — Fix the mobile/2D HUD overlap (P1)
**Done when:** on a 1024×640 viewport no HUD element overlaps any zone, and the lit number is legible without scrolling.

### T4 — Per-punch timing recorder
This is the owner's stated v2 feature. Current model: a combo is `[7,7,5,1,2,1]`, bare numbers. Target model:

```js
// a step
{ n: 7, gap: 260 }   // gap = ms of intended delay BEFORE this punch
```

Three pieces:

1. **Record mode.** Owner throws the combo at the bag; the app captures the interval before each punch and writes it into `gap`. No failing during a recording — record mode is capture only.
2. **Speed control.** A global rate multiplier (0.25×–2×) applied to every `gap`, **plus** per-step multipliers so one punch can be sped up independently of its neighbours. The owner's example: in a recorded `1-2`, make the `1` faster than the `2`.
3. **Playback.** The called number arrives on the recorded schedule. Crucially, **the timing drives the calling, not the failing.** Throwing late must not end the round in v1 — only throwing the *wrong number* ends the round. If enforced timing windows are wanted later, that's a separate mode and a separate conversation with the owner.

Migrate existing drills by treating a bare number as `{n, gap: 0}`. Keep the parser accepting plain `7-7-5-1-2-1` strings.
**Done when:** the owner can record a combo, scrub one punch faster than the rest, and play it back with the voice calling on the recorded rhythm.

### T5 — Reference space fallback (P2)

### T6 — Hand tracking as an alternative to controllers
`hand-tracking` is already in `optionalFeatures`. Track the wrist or middle-finger metacarpal joint and reuse the same velocity gate. Falls back to controllers when unavailable.

### T7 — Curved zone decals (P5)

---

## 7. Testing matrix

| Surface | What to check |
|---------|---------------|
| Meta Quest Browser, immersive VR | Everything in T1. Primary target. |
| Meta Quest Browser, 2D | Layout, tap-to-punch, the Enter VR button's diagnostics |
| Desktop Chrome | Keys 1–8, drag-to-look, audio |
| Mobile Safari / Chrome | Tap targets, safe-area insets, no timer-related layout jump |

---

## 8. Working agreement

Game-feel rules — the fail condition, the absence of a timer, the numbering itself, scoring values — are the owner's calls. Bring proposals rather than committing changes to them. Architecture, performance, and the shape of the code are yours.

The owner ships working builds over specs. If a task in §6 is ambiguous, build the smallest working version and show it rather than writing a design doc about it.

---

## 9. Open questions for the owner

1. Should hitting bare leather (no zone) stay a harmless whiff, or count as a wrong punch and end the round? Currently harmless.
2. Should the hand check default on or off for first-time users? Currently on in VR.
3. Does the "long eight" drill match how you actually want the numbers strung, or is it filler?
4. Sound: is the current bag thud close to what you want, or should it pull from the Kung Foley layer approach instead of synthesising in-file?
