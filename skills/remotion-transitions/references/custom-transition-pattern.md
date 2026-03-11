# Custom Transition Pattern

## The `TransitionPresentation` API

```ts
import type { TransitionPresentation } from "@remotion/transitions";
import { TransitionSeries, linearTiming, springTiming } from "@remotion/transitions";
```

### Component Shape

Every transition is a factory function that returns `{ component, props }`:

```tsx
function myTransition(): TransitionPresentation<Record<string, never>> {
  const component = ({
    presentationProgress,   // 0 → 1 over the transition duration
    presentationDirection,  // "exiting" | "entering"
    children,               // the scene being wrapped (React.ReactNode)
    passedProps,            // typed to your generic — pass {} if none
  }: {
    presentationProgress: number;
    presentationDirection: "entering" | "exiting";
    children: React.ReactNode;
    passedProps: Record<string, never>;
  }) => {
    const entering = presentationDirection === "entering";
    // ... animate based on presentationProgress + direction
    return <AbsoluteFill>{children}</AbsoluteFill>;
  };

  return { component, props: {} };
}
```

### Instantiate at Module Level (Critical)

```ts
// ✅ Outside any component — stable reference, avoids re-mount
const MY_TRANSITION = myTransition();

// ❌ Inside a component — remounts on every render
function MyComp() {
  const t = myTransition(); // BAD
}
```

### Using in TransitionSeries

```tsx
export function MyComposition() {
  return (
    <TransitionSeries>
      <TransitionSeries.Sequence durationInFrames={120}>
        <SceneA />
      </TransitionSeries.Sequence>

      <TransitionSeries.Transition
        presentation={MY_TRANSITION}
        timing={linearTiming({ durationInFrames: 40 })}
      />

      <TransitionSeries.Sequence durationInFrames={120}>
        <SceneB />
      </TransitionSeries.Transition>
    </TransitionSeries>
  );
}
```

## Frame Budget Math

Total frames = Σ(scene durations) − Σ(transition durations)

```
// Example:
// 3 scenes × 120f = 360f
// 2 transitions × 30f = 60f removed
// Total rendered = 300f = 10s @ 30fps
```

## Timing Types

### `linearTiming` — Frame-perfect, dramatic transitions
```ts
linearTiming({ durationInFrames: 50 })
// Use for: striped slams, bursts, glitch effects — where frame-perfect control matters
```

### `springTiming` — Physics-based transitions
```ts
springTiming({ config: { damping: 200 }, durationInFrames: 35 })
// Use for: punchy zoom transitions, smooth reveals
// High damping (200) = no bounce, fast settle
// Low damping (10-14) = springy overshoot
```

## Interpolation Helpers

Always define this once at module level:

```ts
const clamp = {
  extrapolateLeft:  "clamp" as const,
  extrapolateRight: "clamp" as const,
};
```

Then use it everywhere:
```ts
const opacity = interpolate(presentationProgress, [0, 1], [0, 1], clamp);
const x = interpolate(presentationProgress, [0, 0.5, 1], [100, 10, 0], clamp);
```

## Stagger Formula

For N elements with individual stagger offsets:

```ts
const STAGGER = 0.3; // max 30% of total duration used for stagger delay

for (let i = 0; i < N; i++) {
  const delay = (i / N) * STAGGER;
  // Normalize progress for this element: starts later, reaches 1 at p=1
  const p = Math.max(0, Math.min(1, (presentationProgress - delay) / (1 - delay)));
  const eased = 1 - Math.pow(1 - p, 3); // cubic ease-out
  // Use `eased` for this element's position
}

// For reverse stagger (last element first):
const revDelay = ((N - 1 - i) / N) * STAGGER;
```

## Overlay Pattern

Most transitions render **on top of the children** using `pointerEvents: "none"`:

```tsx
return (
  <AbsoluteFill>
    {children}         {/* scene underneath */}
    <div style={{
      position: "absolute",
      inset: 0,
      background: "...",
      pointerEvents: "none",  // never block interaction
    }} />
  </AbsoluteFill>
);
```

## Common Gotchas

### `clockWipe` requires explicit dimensions
```ts
// ❌ Fails — computes SVG clip path without knowing canvas size
clockWipe()

// ✅ Pass width/height explicitly
clockWipe({ width: 1080, height: 1920 })
```

### Negative-width div trick (avoid it)
```ts
// ❌ At progress=0, right: 108% collapses the div (invisible)
<div style={{ inset: 0, right: `${100 - x}%` }} />

// ✅ Use left + right: 0 so the div can never collapse
<div style={{ top: 0, bottom: 0, left: `${x}%`, right: 0 }} />
```
