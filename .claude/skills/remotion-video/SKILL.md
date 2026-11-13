---
name: "remotion-video"
description: "Use when building motion graphics, animated ads, explainers or any video rendered from React code with Remotion, including Arabic/RTL layouts."
---

# Remotion video

Build programmatic videos in React with Remotion. Every frame is a function of the frame number, so the same code always renders the same video.

## Setup and commands

- Check for a Remotion project first: a `remotion` dependency in `package.json` or a `remotion.config.ts`. If none exists, scaffold one with `npx create-video@latest`, then install dependencies.
- Preview: `npx remotion studio`
- Render a video: `npx remotion render src/index.ts <CompositionId> out/<name>.mp4`
- Render one frame to check it: `npx remotion still <CompositionId> out/frame.png --frame=30`

## Core rules

1. Time is in frames. Read `useCurrentFrame()` and `useVideoConfig()` (`fps`, `durationInFrames`, `width`, `height`). Convert seconds to frames with `fps`.
2. Animate only with `interpolate`, `spring`, `Easing` and `random(seed)` from `remotion`. Do not use CSS transitions or keyframes, `setTimeout`, `requestAnimationFrame` or `Math.random()`, because they break deterministic rendering.
3. Register each video in `src/Root.tsx` with `<Composition id, component, durationInFrames, fps, width, height, defaultProps />`.
4. Structure scenes with `<Sequence from durationInFrames>` or `<Series>`. Use `<TransitionSeries>` from `@remotion/transitions` for transitions between scenes.
5. Put assets in `public/` and load them with `staticFile('name.png')`. Use `<Img>`, `<OffthreadVideo>` or `<Video>`, and `<Audio>`.
6. Load fonts explicitly. Use `@remotion/google-fonts` for Google fonts and `@remotion/fonts` (`loadFont`) for local or brand fonts. Never rely on a system fallback font.
7. For async data or assets, call `delayRender()` and release it with `continueRender(handle)`.

## Example

```tsx
import {AbsoluteFill, interpolate, spring, useCurrentFrame, useVideoConfig} from 'remotion';

export const Title: React.FC<{text: string}> = ({text}) => {
  const frame = useCurrentFrame();
  const {fps} = useVideoConfig();
  const scale = spring({frame, fps, config: {damping: 14, stiffness: 120}});
  const opacity = interpolate(frame, [0, 15], [0, 1], {extrapolateRight: 'clamp'});
  return (
    <AbsoluteFill style={{justifyContent: 'center', alignItems: 'center'}}>
      <h1 style={{transform: `scale(${scale})`, opacity, fontSize: 120}}>{text}</h1>
    </AbsoluteFill>
  );
};
```

## Arabic and RTL

- Set `direction: 'rtl'` on text containers and check alignment in still frames.
- Mirror horizontal layouts by hand. Nothing is mirrored automatically.
- Use a font with full Arabic glyph coverage and load it with `loadFont`.
- Decide between Arabic-Indic and Western digits from the brief, then keep them consistent.

## Design direction

- Aim for professional, distinctive work. Avoid default template looks.
- Motion needs a purpose: vary easing, stagger entrances with spring delays, and hold text on screen long enough to read.
- Keep important content inside safe margins (about 5% on each side).

## Brand assets

- For brand work, use the client's own logo and installed fonts from their brand folder.
- Never substitute default or fallback fonts such as DejaVu.

## Verification

- Render stills at the start, middle and end of each scene and inspect them for clipping, overlap and illegible text.
- Check the total duration with `durationInFrames / fps` before the final render.
- Confirm the output file plays and matches the requested resolution and frame rate.