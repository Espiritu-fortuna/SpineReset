# SpineReset Build Task

Build a complete single-file PWA webapp called **SpineReset** — a corrective exercise guided flow app.

## Context
Modelled after the RepForge workout timer app at /home/ubuntu/repforge-git/. Use identical design language: black/gold, Bebas Neue + Barlow Condensed fonts, ring timers, mobile-first, max-width 480px.

## Files to Create
1. `/home/ubuntu/spinereset/index.html` — complete app, all CSS and JS inline
2. `/home/ubuntu/spinereset/sw.js` — service worker for offline/PWA
3. `/home/ubuntu/spinereset/manifest.json` — PWA manifest

## Design System (match RepForge exactly)
```
--gold: #D4AF37
--purple: #A855F7   (right-side modifications accent)
--bg: #0a0a0a
--surface: #131313
--surface2: #1c1c1c
--text: #f0ede8
--muted: #777
--danger: #e53935
--font-d: Bebas Neue
--font-b: Barlow
--font-c: Barlow Condensed
```
Google Fonts: `https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Barlow:wght@300;400;600&family=Barlow+Condensed:wght@400;600;700`

## App Architecture

### Views (same pattern as RepForge)
1. **#home** — day selector
2. **#session** — active exercise flow
3. **#complete** — session done screen

### Home Screen
- Title "SPINERESET" gold Bebas Neue large
- Subtitle "CORRECTIVE EXERCISE · 5-DAY PROGRAM"
- 5 day cards (Day 1-5) — tap to select, shows day title
- Week badge (Week 1, 2, 3...) stored in localStorage key `sr_week`
- START SESSION button

Day titles:
- Day 1: UPPER CORRECTIVE + CORE
- Day 2: LOWER CORRECTIVE + MOBILITY
- Day 3: UPPER STRENGTH + ASYMMETRY
- Day 4: CORE + MOBILITY
- Day 5: FULL INTEGRATION

### Session Flow
Phases run in sequence:
1. Warm-Up exercises (always same 3)
2. Day exercises (in order per schedule)
3. Right-side modifier phases (purple) — run after each exercise that has a rightMod
4. Complete screen

### Warm-Up (every session)
- A: Diaphragmatic Breathing + Posterior Pelvic Tilt — 10 breaths, 4s in/6-8s out — countdown timer
- B: Chin Tucks — 10 reps, 3s hold each — rep counter
- C: Thoracic Cat-Cow — 10 cycles — rep counter, cue: "ONLY mid-back moves"

### Exercise Data

Each exercise object:
```js
{
  id: number,
  name: string,
  muscles: string,
  sets: number,
  reps: string,          // e.g. "8-10" or "12-15 per side" or null for holds
  holdSec: number|null,  // hold duration per rep
  restSec: number,
  cue: string,
  rightMod: string|null, // null = no right-side phase
  rightExtraSets: number // how many extra sets on right (1 or 2)
}
```

**Exercise 1: Prone Y-T-W Raises**
- muscles: "Lower/mid trapezius, serratus anterior"
- sets: 3, reps: "8-10 each (Y, T, W)", holdSec: 3, restSec: 60
- cue: "Face down, thumbs up. Y=130deg, T=90deg, W=goalpost. Squeeze blades DOWN. 3-second hold at top. Never shrug."
- rightMod: "Do 2 EXTRA REPS with RIGHT arm only in each position (Y, T, W). Left arm stays standard."

**Exercise 2: Wall Slides (Wall Angels)**
- muscles: "Serratus anterior, lower trapezius, thoracic extensors"
- sets: 3, reps: "10-12", restSec: 45
- cue: "Back flat on wall — head, glutes, upper back touching. Goalpost to Y shape, 3s up / 3s down. Wrists and elbows stay on wall."
- rightMod: null

**Exercise 3: Deep Cervical Flexor Chin Tuck Hold**
- muscles: "Deep neck flexors"
- sets: 3, reps: "10 reps", holdSec: 10, restSec: 30
- cue: "Lying on back, towel under neck. Tiny nod — like holding a grape with chin. No visible surface neck strain."
- rightMod: "After EACH bilateral hold, add a gentle RIGHT head rotation while keeping the chin tuck. Rotate only to mild tension — never to pain."

**Exercise 4: Dumbbell Single-Arm Prone Row**
- muscles: "Rhomboids, mid/lower trapezius, posterior deltoid, rotator cuff"
- sets: 3, reps: "10-12 per side", restSec: 45
- cue: "Retract scapula FIRST, then bend elbow. Row to hip, squeeze blade back and down 2s. Lower 3-4s (slow eccentric). Full protraction at bottom. Start 5 kg."
- rightMod: "Do 4 SETS on RIGHT side (1 extra set). Left stays at 3 sets. Focus on engaging the right lower trapezius."

**Exercise 5: Dead Bug**
- muscles: "Transversus abdominis, obliques — anti-extension core"
- sets: 3, reps: "8-10 per side", restSec: 45
- cue: "Back FLAT on floor — never arching. Extend opposite arm and leg. Exhale on extend. If back arches = too far. Regression: extend legs only, arms point up."
- rightMod: null

**Exercise 6: Doorway Pectoral Stretch**
- muscles: "Pectoralis major/minor"
- sets: 2, reps: "30s per side, 2 heights", restSec: 0
- cue: "Forearms on door frame, step through. No lower back arching. Do shoulder level height AND above-shoulder height."
- rightMod: "After bilateral stretch, do 2 x 30s EXTRA on RIGHT side only: face the frame, right forearm on frame, rotate body AWAY from right arm."

**Exercise 7: Glute Bridge with Posterior Pelvic Tilt**
- muscles: "Gluteus maximus — counters anterior pelvic tilt"
- sets: 3, reps: "12-15", holdSec: 3, restSec: 45
- cue: "Flatten back first (tuck tailbone), THEN lift hips. Squeeze glutes hard. 3-second hold at top. No hyperextension. Drive through heels."
- rightMod: null

**Exercise 8: Half-Kneeling Hip Flexor Stretch**
- muscles: "Iliopsoas, rectus femoris"
- sets: 2, reps: "30s per side", restSec: 0
- cue: "TUCK TAILBONE and squeeze back-leg glute. Shift hips forward — do NOT lean torso. Raise same-side arm overhead for deeper psoas stretch."
- rightMod: null

**Exercise 9: Side-Lying Dumbbell External Rotation**
- muscles: "Infraspinatus, teres minor (rotator cuff external rotators)"
- sets: 3, reps: "12-15 per side", restSec: 45
- cue: "Lie on left side, right elbow pinned to hip at 90 degrees. Rotate forearm toward ceiling. 2s hold top, 3s lower. Small motion, no body rolling. Start 1-2 kg."
- rightMod: "Do 4 SETS on RIGHT side (1 extra). Left stays at 3 sets."

**Exercise 10: Upper Trap and Levator Stretch**
- muscles: "Upper trapezius, levator scapulae"
- sets: 2, reps: "30s per side, 2 positions", restSec: 0
- cue: "Trapezius: ear toward opposite shoulder. Levator: look toward opposite armpit. Hold each position 30 seconds."
- rightMod: "Add 1 EXTRA SET on LEFT upper trapezius (it sits higher) and RIGHT levator scapulae (this frees restricted right rotation)."

**Exercise 11: Thoracic Extension over Roller**
- muscles: "Thoracic spine extension mobility"
- sets: 2, reps: "8 reps at 2-3 spinal levels", holdSec: 5, restSec: 0
- cue: "Roller under mid-back. Hands behind head. Extend OVER roller. Core engaged, ribs down. 5-second hold at end-range."
- rightMod: null

**Exercise 12: Dumbbell Farmers Carry**
- muscles: "Postural muscles, core, grip strength"
- sets: 3, reps: "30-45 second walk", restSec: 60
- cue: "Chin tucked, shoulders packed down and back, core braced, no rib flare, neutral pelvis. Walk tall. 8-12 kg per hand."
- rightMod: null

**Exercise 13: Bird Dog**
- muscles: "Multifidus, transversus abdominis, glutes"
- sets: 3, reps: "8 per side", holdSec: 5, restSec: 0
- cue: "Quadruped. Brace core first. Extend opposite arm and leg. ZERO trunk rotation or pelvic shift. 5-second hold."
- rightMod: null

**Exercise 14: Push-Up Plus**
- muscles: "Serratus anterior — targets right scapular winging"
- sets: 3, reps: "8-12", restSec: 45
- cue: "At top of push-up, push floor AWAY to fully protract shoulder blades. Hold 2 seconds. The plus is the extra push at top. Regression: from knees or wall push-up plus."
- rightMod: "Focus on feeling RIGHT serratus anterior engage (muscle under right armpit). Push harder with right hand at top."

### Side Plank (Day 4 special)
- id: 15
- muscles: "Lateral core, quadratus lumborum"
- sets: 3 (left) / 4 (right — 1 extra set on right side)
- reps: "20-30s per side"
- restSec: 0
- cue: "Start from KNEES (not feet) to reduce lumbar load. Hips stacked, straight line. No sagging."
- LOCKED until week >= 3. If locked: show locked card, do extra Bird Dog set instead.
- rightMod: "Do 1 EXTRA SET on RIGHT side (4 sets right, 3 sets left)."

### Day Schedules
```
Day 1: [1, 2, 3, 5, 6, 10]
Day 2: [7, 8, 11, 13, 5, 10]
Day 3: [4, 1, 14, 9, 6]
Day 4: [5, 13, 15, 11, 8, 10, 3]  // 15 = Side Plank (locked if week < 3)
Day 5: [1, 4, 14, 12, 7, 5, 6]
```

### Session UI Components

**Exercise card header:**
- Phase badge: "WARM-UP" / "EXERCISE 3 OF 6" / "RIGHT SIDE +" (purple when right-side phase)
- Exercise name: large Bebas Neue, gold (purple during right-side phase)
- Muscle targets: small muted text
- Form cue: surface card, scrollable

**Set tracker:**
- Row of circles (filled = done, current = glowing gold ring, upcoming = muted outline)
- "SET 2 OF 3" label

**Timer/counter area:**
- For timed/hold: SVG ring timer (same as RepForge) with countdown seconds in center
- For reps: large rep count display, TAP TO COUNT button at bottom OR manual DONE SET button
- For stretches: timer countdown per side

**Controls:**
- Pause/Resume button (circle, center)
- Skip exercise button (ghost, top right)
- DONE SET button (gold, full width) — marks set complete, starts rest timer OR moves to next set

**Rest timer:**
- Full screen pulse: "REST" badge, countdown ring in muted gold
- "Skip Rest" small button

**Up Next panel (bottom):**
- Shows next 2 exercises coming up

**Right-side modification phase:**
- Purple color scheme instead of gold
- Badge: "RIGHT SIDE +" in purple
- Shows the rightMod instruction text prominently
- Separate set tracking

### Complete Screen
- Large "SESSION DONE" gold
- Stats: total exercises, total sets, time elapsed
- Key reminders section (3-4 bullet points, muted text)
- "Back to Days" button
- If Day 5 completed: "Complete Week N → Week N+1" button (increments sr_week)

### localStorage
- `sr_week`: current week number, default 1
- `sr_history`: array of {day, date, duration, sets} objects
- `sr_selected_day`: last selected day

### Audio (Web Speech API + AudioContext beeps)
Same system as RepForge:
- Announce exercise name when starting
- "Rest" spoken when rest begins
- "Begin" spoken when exercise starts
- Beep on set complete
- Respect audioMode setting (voice / beep / off)

### PWA Files

**manifest.json:**
```json
{
  "name": "SpineReset",
  "short_name": "SpineReset",
  "description": "Corrective exercise guided flow — 5-day weekly program",
  "start_url": "./",
  "display": "standalone",
  "background_color": "#0a0a0a",
  "theme_color": "#0a0a0a",
  "icons": [
    {"src": "icons/icon-192.png", "sizes": "192x192", "type": "image/png"},
    {"src": "icons/icon-512.png", "sizes": "512x512", "type": "image/png"}
  ]
}
```

**sw.js:** Cache-first strategy, cache all app files on install.

### Apple PWA meta tags (in head):
```html
<meta name="apple-mobile-web-app-capable" content="yes"/>
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent"/>
<meta name="apple-mobile-web-app-title" content="SpineReset"/>
```

## Reference Files
Study /home/ubuntu/repforge-git/index.html and /home/ubuntu/repforge-git/app.js for:
- Timer ring SVG pattern (circle with strokeDasharray/strokeDashoffset animation)
- View transition pattern (opacity fade between views)
- Phase badge styling
- Up-next list styling
- Controls layout
- Complete screen layout
- History panel
- CSS variables and typography scale

## Important Notes
- Single file for index.html — all CSS and JS inline, no external dependencies except Google Fonts
- The app must work offline after first load (service worker)
- Mobile-first, optimized for use during exercise (large touch targets, readable at arm's length)
- Right-side modifications are separate exercise phases, not just a note — they have their own set tracking
- Do NOT copy RepForge exercise data. Only copy structural/visual patterns.

## Git Instructions
After creating all files:
```
cd /home/ubuntu/spinereset
git add -A
git commit -m "feat: initial SpineReset app — 5-day corrective exercise flow PWA"
git push origin main
```

Then run: `openclaw system event --text "Done: SpineReset webapp built and pushed to GitHub" --mode now`
