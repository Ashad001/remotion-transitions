# Transition Catalog

Six production-tested transitions from Fyltr's Instagram Reel campaign. All use the `TransitionPresentation` API — see `custom-transition-pattern.md` for the scaffold.

Design tokens used: `DARK_BG = "#0a0a0a"`, `EMERALD = "#10b981"`. Replace with your own brand colors.

---

## 1. Striped Slam

**Effect:** N horizontal bars (alternating brand colors) slam in from left and right simultaneously, cover the old scene, then retract to reveal the new scene.
**Energy:** Maximum. Best for opening transitions.
**Timing:** `linearTiming({ durationInFrames: 50 })` — needs time for the slam + retract

```tsx
function stripedSlam(stripes = 8): TransitionPresentation<Record<string, never>> {
  const STRIPE_COLORS = [DARK_BG, EMERALD]; // alternate brand colors
  const component = ({
    presentationProgress,
    presentationDirection,
    children,
  }: {
    presentationProgress: number;
    presentationDirection: "entering" | "exiting";
    children: React.ReactNode;
    passedProps: Record<string, never>;
  }) => {
    const bars = Array.from({ length: stripes }, (_, i) => {
      const h = 100 / stripes;
      const color = STRIPE_COLORS[i % STRIPE_COLORS.length];
      const fromLeft = i % 2 === 0; // alternate directions
      const stagger = (i / stripes) * 0.3;
      const p = Math.max(0, Math.min(1, (presentationProgress - stagger) / (1 - stagger)));
      const pe = 1 - Math.pow(1 - p, 3); // cubic ease-out

      let x: number;
      if (presentationDirection === "exiting") {
        // Slam IN: bars come from outside → park at 0
        x = fromLeft
          ? interpolate(pe, [0, 1], [-112, 0])
          : interpolate(pe, [0, 1], [112, 0]);
      } else {
        // Retract: reverse stagger, go back where they came from
        const revStagger = ((stripes - 1 - i) / stripes) * 0.3;
        const rp = Math.max(0, Math.min(1, (presentationProgress - revStagger) / (1 - revStagger)));
        const rpe = 1 - Math.pow(1 - rp, 3);
        x = fromLeft
          ? interpolate(rpe, [0, 1], [0, -112])
          : interpolate(rpe, [0, 1], [0, 112]);
      }

      return (
        <div key={i} style={{
          position: "absolute",
          top: `${i * h}%`,
          left: 0,
          width: "112%",
          height: `${h + 0.4}%`, // +0.4% overlap prevents hairline gaps
          background: color,
          transform: `translateX(${x}%)`,
          pointerEvents: "none",
        }} />
      );
    });

    return <AbsoluteFill>{children}{bars}</AbsoluteFill>;
  };
  return { component, props: {} };
}

// Usage
const STRIPED_SLAM = stripedSlam(8);
// <TransitionSeries.Transition presentation={STRIPED_SLAM} timing={linearTiming({ durationInFrames: 50 })} />
```

**Customization tips:**
- More stripes (12-16) → tighter, more intense
- Larger stagger (0.4) → more cascade feel
- Use 3+ colors in `STRIPE_COLORS` for rainbow effect

---

## 2. Zoom Punch

**Effect:** Old scene zooms out and fades. New scene punches in from a slightly smaller scale with cubic ease-in-out.
**Energy:** Medium-high. Feels like a camera cut with intent.
**Timing:** `springTiming({ config: { damping: 200 }, durationInFrames: 35 })`

```tsx
function zoomPunch(): TransitionPresentation<Record<string, never>> {
  const component = ({
    presentationProgress,
    presentationDirection,
    children,
  }: {
    presentationProgress: number;
    presentationDirection: "entering" | "exiting";
    children: React.ReactNode;
    passedProps: Record<string, never>;
  }) => {
    const entering = presentationDirection === "entering";

    if (entering) {
      const p = presentationProgress;
      // Cubic ease-in-out for smooth punch
      const pe = p < 0.5 ? 4 * p * p * p : 1 - Math.pow(-2 * p + 2, 3) / 2;
      const scale = interpolate(pe, [0, 1], [0.86, 1]);
      return (
        <AbsoluteFill style={{ opacity: presentationProgress, transform: `scale(${scale})` }}>
          {children}
        </AbsoluteFill>
      );
    }

    // Exiting: zoom out slightly (zoom-out = scene "retreats")
    const scale = interpolate(presentationProgress, [0, 1], [1, 1.08]);
    return (
      <AbsoluteFill style={{ opacity: 1 - presentationProgress, transform: `scale(${scale})` }}>
        {children}
      </AbsoluteFill>
    );
  };
  return { component, props: {} };
}
```

**Customization tips:**
- Entering scale `[0.76, 1]` → more dramatic "smash" feel
- Exiting scale `[1, 1.15]` → more aggressive retreat
- Add `filter: blur(...)` on exiting for depth-of-field effect

---

## 3. Diagonal Reveal

**Effect:** A dark panel with a skewed right edge sweeps left→right across the screen, dragging an emerald accent line at the leading edge. Old scene fades beneath; new scene is revealed from the left.
**Energy:** Cinematic. Great mid-sequence for "reveal" moments.
**Timing:** `linearTiming({ durationInFrames: 40 })`

```tsx
function diagonalReveal(): TransitionPresentation<Record<string, never>> {
  const component = ({
    presentationProgress,
    presentationDirection,
    children,
  }: {
    presentationProgress: number;
    presentationDirection: "entering" | "exiting";
    children: React.ReactNode;
    passedProps: Record<string, never>;
  }) => {
    if (presentationDirection === "exiting") {
      return (
        <AbsoluteFill style={{ opacity: 1 - Math.pow(presentationProgress, 0.6) }}>
          {children}
        </AbsoluteFill>
      );
    }

    // Sweep boundary: starts off left (-12%) → sweeps past right edge (116%)
    const pe = 1 - Math.pow(1 - presentationProgress, 2.5); // ease-out power
    const bx = interpolate(pe, [0, 1], [-12, 116]);

    return (
      <AbsoluteFill>
        {children}

        {/* Dark cover: at bx=-12, left=-12% covers everything; at bx=116, collapses */}
        {/* CRITICAL: use left+right:0 NOT inset+right — negative width makes div invisible */}
        <div style={{
          position: "absolute",
          top: 0, bottom: 0,
          left: `${bx}%`,
          right: 0,
          background: DARK_BG,
          pointerEvents: "none",
        }} />

        {/* Skewed leading-edge softener */}
        <div style={{
          position: "absolute",
          top: "-10%", bottom: "-10%",
          left: `${bx - 7}%`,
          width: "10%",
          background: DARK_BG,
          transform: "skewX(-9deg)",
          transformOrigin: "top left",
          pointerEvents: "none",
        }} />

        {/* Accent line at the blade edge */}
        <div style={{
          position: "absolute",
          top: 0, bottom: 0,
          left: `${bx - 0.5}%`,
          width: 3,
          background: EMERALD,
          transform: "skewX(-9deg)",
          boxShadow: `0 0 14px ${EMERALD}, 0 0 32px rgba(16,185,129,0.45)`,
          pointerEvents: "none",
        }} />
      </AbsoluteFill>
    );
  };
  return { component, props: {} };
}
```

**Customization tips:**
- Change sweep direction: start `bx` at `[112, -16]` to sweep right→left
- Change `skewX` angle for sharper/softer blade edge
- Use white panel for "light burst" variant

---

## 4. Emerald Burst

**Effect:** Sharp radial flash erupts from screen center on the cut. Peaks instantly at `presentationProgress = 0`, clears by 20%. Entering scene fades in as the burst dissolves.
**Energy:** High impact. Best for emotional or reveal moments.
**Timing:** `linearTiming({ durationInFrames: 40 })`

```tsx
function emeraldBurst(): TransitionPresentation<Record<string, never>> {
  const component = ({
    presentationProgress,
    presentationDirection,
    children,
  }: {
    presentationProgress: number;
    presentationDirection: "entering" | "exiting";
    children: React.ReactNode;
    passedProps: Record<string, never>;
  }) => {
    const entering = presentationDirection === "entering";

    // Flash: enters at full opacity, clears fast; exits builds late
    const burstOpacity = entering
      ? interpolate(presentationProgress, [0, 0.2, 1], [1, 0, 0], clamp)
      : interpolate(presentationProgress, [0, 0.8, 1], [0, 0, 1], clamp);

    const sceneStyle = entering
      ? { opacity: interpolate(presentationProgress, [0, 0.25, 1], [0, 1, 1], clamp) }
      : { opacity: 1 - Math.pow(presentationProgress, 2) };

    return (
      <AbsoluteFill>
        <AbsoluteFill style={sceneStyle}>{children}</AbsoluteFill>
        <AbsoluteFill style={{
          background: `radial-gradient(circle at 50% 50%, #ffffff 0%, ${EMERALD} 18%, rgba(16,185,129,0.55) 40%, transparent 62%)`,
          opacity: burstOpacity,
          pointerEvents: "none",
        }} />
      </AbsoluteFill>
    );
  };
  return { component, props: {} };
}
```

**Customization tips:**
- White core + brand color → creates a camera flash effect
- Move gradient center: `circle at 50% 30%` for top-origin burst
- Adjust `[0, 0.2, 1]` → `[0, 0.1, 1]` for even sharper flash

---

## 5. Vertical Shutter

**Effect:** N vertical panels (venetian blind style) snap shut over the old scene, then snap open to reveal the new scene. Alternating brand colors per panel.
**Energy:** High. Very graphic, stop-motion feel.
**Timing:** `linearTiming({ durationInFrames: 35 })`

```tsx
function verticalShutter(panels = 7): TransitionPresentation<Record<string, never>> {
  const component = ({
    presentationProgress,
    presentationDirection,
    children,
  }: {
    presentationProgress: number;
    presentationDirection: "entering" | "exiting";
    children: React.ReactNode;
    passedProps: Record<string, never>;
  }) => {
    const w = 100 / panels;
    const shutters = Array.from({ length: panels }, (_, i) => {
      const stagger = (i / panels) * 0.25;
      const p = Math.max(0, Math.min(1, (presentationProgress - stagger) / (1 - stagger)));
      const pe = 1 - Math.pow(1 - p, 3);

      // Exiting: panels CLOSE (scaleX 0→1 from left origin)
      // Entering: panels OPEN (scaleX 1→0)
      const scaleX = presentationDirection === "exiting"
        ? interpolate(pe, [0, 1], [0, 1])
        : interpolate(pe, [0, 1], [1, 0]);

      const color = i % 2 === 0 ? DARK_BG : EMERALD;

      return (
        <div key={i} style={{
          position: "absolute",
          top: 0, bottom: 0,
          left: `${i * w}%`,
          width: `${w + 0.3}%`, // +0.3% overlap prevents gaps
          background: color,
          transform: `scaleX(${scaleX})`,
          transformOrigin: "left center",
          pointerEvents: "none",
        }} />
      );
    });

    return <AbsoluteFill>{children}{shutters}</AbsoluteFill>;
  };
  return { component, props: {} };
}
```

**Customization tips:**
- Horizontal shutter: use `top`/`height` instead of `left`/`width`, `scaleY` instead of `scaleX`, `transformOrigin: "top center"`
- More panels (12-16) → tighter, more frantic look
- `transformOrigin: "center center"` → panels collapse to middle instead of side

---

## 6. Glitch Slam

**Effect:** Old scene shakes horizontally with decaying amplitude while 4 colored RGB-offset horizontal strips tear across. New scene hard-pops in at 12% progress with a brief scale punch.
**Energy:** Maximum chaos. Best as final transition before CTA.
**Timing:** `linearTiming({ durationInFrames: 30 })`

```tsx
function glitchSlam(): TransitionPresentation<Record<string, never>> {
  const component = ({
    presentationProgress,
    presentationDirection,
    children,
  }: {
    presentationProgress: number;
    presentationDirection: "entering" | "exiting";
    children: React.ReactNode;
    passedProps: Record<string, never>;
  }) => {
    const entering = presentationDirection === "entering";

    if (entering) {
      // Hard pop-in at 12%, small scale punch
      const opacity = interpolate(presentationProgress, [0, 0.12, 1], [0, 1, 1], clamp);
      const scale = interpolate(presentationProgress, [0, 0.25, 1], [1.05, 1.01, 1], clamp);
      return (
        <AbsoluteFill style={{ opacity, transform: `scale(${scale})` }}>
          {children}
        </AbsoluteFill>
      );
    }

    // Exiting: shake + RGB strips
    // sin wave decays as (1 - progress)^1.5 → lots of shake early, calms fast
    const shake = Math.sin(presentationProgress * Math.PI * 12) * 30 * Math.pow(1 - presentationProgress, 1.5);
    const opacity = 1 - Math.pow(presentationProgress, 0.5);

    const STRIPS = [
      { top: "18%", h: "4%", dx: 28,  color: "rgba(239,68,68,0.55)"   }, // red
      { top: "37%", h: "2%", dx: -22, color: "rgba(16,185,129,0.55)"  }, // emerald
      { top: "58%", h: "5%", dx: 36,  color: "rgba(59,130,246,0.5)"   }, // blue
      { top: "76%", h: "2%", dx: -18, color: "rgba(255,255,255,0.35)" }, // white
    ];
    const stripOpacity = interpolate(presentationProgress, [0, 0.3, 0.8, 1], [0, 1, 0.6, 0], clamp);

    const strips = STRIPS.map((s, i) => (
      <div key={i} style={{
        position: "absolute",
        top: s.top, left: 0, right: 0,
        height: s.h,
        background: s.color,
        transform: `translateX(${s.dx * presentationProgress * 2}px)`,
        opacity: stripOpacity,
        pointerEvents: "none",
        mixBlendMode: "screen",
      }} />
    ));

    return (
      <AbsoluteFill style={{ opacity, transform: `translateX(${shake}px)` }}>
        {children}
        {strips}
      </AbsoluteFill>
    );
  };
  return { component, props: {} };
}
```

**Customization tips:**
- More `Math.PI * N` → faster shake frequency (more glitchy)
- Adjust `dx` values and add more strips for more visual chaos
- Add `filter: hue-rotate(...)` on the exiting scene for color corruption

---

## Full Scene Sequence (Reel Pattern)

```tsx
// Instantiate all at module level
const STRIPED_SLAM     = stripedSlam(8);
const ZOOM_PUNCH       = zoomPunch();
const DIAGONAL_REVEAL  = diagonalReveal();
const EMERALD_BURST    = emeraldBurst();
const VERTICAL_SHUTTER = verticalShutter(7);
const GLITCH_SLAM      = glitchSlam();

// Timings
const T_SLAM    = linearTiming({ durationInFrames: 50 });
const T_ZOOM    = springTiming({ config: { damping: 200 }, durationInFrames: 35 });
const T_DIAG    = linearTiming({ durationInFrames: 40 });
const T_BURST   = linearTiming({ durationInFrames: 40 });
const T_SHUTTER = linearTiming({ durationInFrames: 35 });
const T_GLITCH  = linearTiming({ durationInFrames: 30 });

// Total transition overhead: 50+35+40+40+35+30 = 230 frames removed
```
