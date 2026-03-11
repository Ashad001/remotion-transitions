# Animation Math for Transitions

Reference for the easing formulas, spring configs, and interpolation patterns used throughout the transition catalog.

---

## Easing Functions (Pure JS, no imports needed)

These produce `0→1` outputs from `0→1` inputs. Apply AFTER normalizing progress.

```ts
// Cubic ease-out (most common — fast start, decelerates into target)
const easeOut3 = (p: number) => 1 - Math.pow(1 - p, 3);

// Cubic ease-in (slow start — builds momentum)
const easeIn3 = (p: number) => Math.pow(p, 3);

// Cubic ease-in-out (symmetric S-curve — feels "cinematic")
const easeInOut3 = (p: number) =>
  p < 0.5 ? 4 * p * p * p : 1 - Math.pow(-2 * p + 2, 3) / 2;

// Ease-out power (variable exponent — higher n = more snap)
const easeOutN = (p: number, n: number) => 1 - Math.pow(1 - p, n);
// n=2: gentle   n=3: standard   n=4: snappy   n=6: very snappy

// Power ease-in (fractional = fast start, n<1)
const powerIn = (p: number, n: number) => Math.pow(p, n);
// n=0.5: very fast initial drop  n=0.6: medium  n=2: slow
```

### Usage in transitions
```ts
const pe = 1 - Math.pow(1 - presentationProgress, 3); // ease-out cubic
const bx = interpolate(pe, [0, 1], [-12, 116]);        // then map to position
```

---

## Stagger Patterns

### Forward stagger (first element leads)
```ts
const STAGGER = 0.3; // 30% of total duration used as stagger window
const delay = (i / N) * STAGGER;
const p = Math.max(0, Math.min(1, (progress - delay) / (1 - delay)));
```

### Reverse stagger (last element leads)
```ts
const revDelay = ((N - 1 - i) / N) * STAGGER;
const p = Math.max(0, Math.min(1, (progress - revDelay) / (1 - revDelay)));
```

### Bidirectional stagger (center leads, edges follow)
```ts
const center = (N - 1) / 2;
const dist = Math.abs(i - center) / center; // 0 at center, 1 at edges
const delay = dist * STAGGER;
```

---

## Decaying Oscillation (Shake Effect)

Used in `glitchSlam` for the horizontal shake on the exiting scene:

```ts
// sin(progress * π * frequency) × amplitude × decay
const shake = Math.sin(presentationProgress * Math.PI * 12)  // 6 full cycles
            * 30                                               // px amplitude
            * Math.pow(1 - presentationProgress, 1.5);        // decay curve
// Result: lots of shake at start, quickly calms
```

Tuning:
- `* Math.PI * N` → N/2 full oscillation cycles (12 = 6 cycles)
- `amplitude` → max pixel displacement
- Exponent `1.5` → higher = faster decay, lower = longer tail

---

## Spring Configs

All from `{ spring } from "remotion"` or `springTiming` from `@remotion/transitions`.

```ts
// Bouncy entrance (UI elements, cards)
{ damping: 10, stiffness: 380 }  // fast initial + strong overshoot

// Snappy with subtle bounce
{ damping: 14, stiffness: 340 }  // card slam-in with slight settle

// Critically damped — no bounce (smooth arrive)
{ damping: 200 }                 // instant settle, no ring

// Medium — one bounce
{ damping: 12, stiffness: 300 }
```

### Spring as progress driver
```ts
const s = spring({ frame: frame - delay, fps, config: { damping: 10, stiffness: 380 } });
const y = interpolate(s, [0, 1], [startY, 0]); // map spring to position
const opacity = s; // spring output already 0→1
```

---

## Clamp Pattern

Always use this to prevent `interpolate` from extrapolating beyond keyframes:

```ts
const clamp = {
  extrapolateLeft:  "clamp" as const,
  extrapolateRight: "clamp" as const,
};

// Usage
const x = interpolate(frame, [0, 30], [100, 0], clamp);
// At frame -5: returns 100 (not 116)
// At frame 60: returns 0 (not -100)
```

---

## `mixBlendMode` for Overlay Strips

Used in glitch strips to let underlying scene show through:

```ts
style={{ mixBlendMode: "screen" }}
// "screen": light blending — darker pixels become transparent
// Good for glowing color overlays on dark backgrounds

style={{ mixBlendMode: "multiply" }}
// "multiply": darkens — good for ink/shadow effects on light backgrounds

style={{ mixBlendMode: "overlay" }}
// "overlay": contrast boost — preserves highlights and shadows
```

---

## Frame Budget Formula

```
Total output frames = Σ(all scene durationInFrames) - Σ(all transition durationInFrames)

Example (7 scenes, 6 transitions):
  Scenes:      135 + 120 + 150 + 180 + 120 + 150 + 150 = 1005f
  Transitions: 50 + 35 + 40 + 40 + 35 + 30             =  230f
  Output:      1005 - 230                               =  775f = 25.8s @ 30fps
```

### Breathing room rule
Give each scene at least `transition_duration + 30` frames of unique content so
the overlap (where both scenes are visible) doesn't cut important moments.

---

## Responsive Scaling

For 9:16 Instagram Reels (1080×1920 base):

```ts
const { width, height } = useVideoConfig();
const s  = height / 1920;              // vertical scale
const sw = width  / 1080;              // horizontal scale
const sf = Math.sqrt(s * sw);          // geometric mean for font/icon sizes
const r  = (v: number) => Math.round(v * s);   // vertical dimensions
const rs = (v: number) => Math.round(v * sf);  // font sizes and icons
```

Safe zones (avoid notch / caption overlap):
```ts
const SAFE_TOP    = r(288);  // 15% of 1920 = 288px
const SAFE_BOTTOM = r(384);  // 20% of 1920 = 384px
```
