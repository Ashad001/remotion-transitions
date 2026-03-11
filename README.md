# remotion-transitions

A Claude Code skill plugin for building **custom scene transitions in Remotion** — covers the `TransitionPresentation` API from scratch, animation math, timing patterns, and a catalog of production-ready examples to learn from or drop in directly.

## Transitions

| Name | Effect | Best for |
|---|---|---|
| **Striped Slam** | Alternating brand-color bars slam in from both sides, then retract | Opening cuts, high-impact moments |
| **Zoom Punch** | Outgoing scene retreats; incoming punches in with cubic ease | Mid-sequence energy |
| **Diagonal Reveal** | Dark panel sweeps left→right with skewed edge and glowing accent line | Cinematic reveals |
| **Emerald Burst** | Sharp radial flash peaks at the cut, dissolves in 20% of duration | Emotional or product moments |
| **Vertical Shutter** | N vertical panels close over the old scene, then snap open | Graphic, stop-motion feel |
| **Glitch Slam** | Horizontal shake decay + RGB-offset strip tears on exit | Final cuts, CTAs |

## Installation

Once listed on the marketplace:
```
/plugin install remotion-transitions
```

Until then, clone and install locally:
```bash
git clone https://github.com/ashad001/remotion-transitions
```

Then add the skill files from `skills/remotion-transitions/` to your local `~/.claude/skills/` directory.

## Skill triggers

This skill auto-activates when you ask Claude about:
- Custom Remotion transitions or `TransitionPresentation`
- "cool transitions", "high-energy transitions", "cinematic transitions"
- "striped transitions", "glitch transitions", "zoom punch", "shutter"
- Frame budget math, stagger formulas, spring configs for video

## References

| File | Covers |
|---|---|
| [`SKILL.md`](skills/remotion-transitions/SKILL.md) | Entry point, golden rules, quick reference |
| [`custom-transition-pattern.md`](skills/remotion-transitions/references/custom-transition-pattern.md) | `TransitionPresentation` scaffold, `linearTiming` vs `springTiming`, stagger formula, common gotchas |
| [`transition-catalog.md`](skills/remotion-transitions/references/transition-catalog.md) | Full TypeScript source for all 6 transitions with customization tips |
| [`animation-math.md`](skills/remotion-transitions/references/animation-math.md) | Easing functions, decaying oscillation, spring configs, `clamp` pattern, frame budget formula, responsive scaling |

## Requirements

- `remotion` ≥ 4.0
- `@remotion/transitions` ≥ 4.0
- React 18+
- TypeScript (recommended)

## License

MIT
