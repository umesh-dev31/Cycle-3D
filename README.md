# Cycle-3D

A 3D product showcase built with React, Three.js, and GSAP. A single GLB model is rendered persistently on screen while scroll drives the camera through a sequence of cinematic angles, revealing different views of the product across each section. Three bike variants can be switched via a carousel that animates the canvas out, swaps the model, and slides the new one back in.

---

## Live Demo

[https://github.com/umesh-dev31/Cycle-3D](https://github.com/umesh-dev31/Cycle-3D)

---

## Tech Stack

| Tool | Purpose |
|---|---|
| Vite + React 19 | Build tooling and UI framework |
| Three.js | WebGL rendering engine |
| @react-three/fiber | React renderer for Three.js |
| @react-three/drei | Helpers: useGLTF, Environment, PerspectiveCamera |
| GSAP + ScrollTrigger | Scroll-driven camera interpolation and carousel transitions |
| Tailwind CSS v4 | Utility-first styling via the Vite plugin |

---

## Project Structure

```
index.html
src/
  App.jsx                  Root component — variants, carousel, all sections
  Scene.jsx                Fixed canvas with the GLB model and scroll camera
  useScrollCamera.js       GSAP ScrollTrigger -> camera keyframe interpolation
  useSectionInView.js      IntersectionObserver hook for section fade-ins
  BikeCarouselSection.jsx  Hero section UI: headline, CTAs, stats bar, dots
  Navbar.jsx               Fixed nav with dynamic accent color per variant
  Preloader.jsx            SVG progress ring loader tracking real page load
  LikeWidget.jsx           Floating like interaction widget
  DebugCameraPanel.jsx     Dev tool: slider panel for tuning camera keyframes
  index.css                @theme tokens, global reset, body gradient
public/
  models/
    bike.glb
    bike-vortex.glb
    bike-spectra.glb
  images/
    rock-pedestal.png
```

---

## Getting Started

**Prerequisites:** Node.js 18 or later.

```bash
# Clone the repository
git clone https://github.com/umesh-dev31/Cycle-3D.git
cd Cycle-3D

# Install dependencies
npm install

# Start the dev server
npm run dev
```

Open `http://localhost:5173` in your browser.

---

## How It Works

### Layer Composition

The site relies on a strict z-index stack to make the 3D model appear to float inside the real page:

```
z-0    body::before        Fixed radial gradient acting as the scene floor
z-1    HeroBackdrop        Color glow, rock pedestal PNG, and ghost text
z-2    Scene / Canvas      The 3D model (alpha: true, transparent background)
z-10   HTML sections       All scroll sections, stats, copy, and CTAs
z-30   Navbar              Fixed top navigation
z-100  Preloader           Loading screen
```

The canvas is `position: fixed` with a transparent WebGL background, so the model composites directly into the page rather than sitting inside a box.

### Scroll-Driven Camera

`useScrollCamera.js` defines one camera keyframe per scroll section. GSAP ScrollTrigger maps the page scroll progress (0 to 1) across those keyframes, and the `CameraRig` component inside `useFrame` lerps the actual Three.js camera toward the current interpolated target each frame. This produces a smooth, scrubbed cinematic feel rather than a hard jump between angles.

```js
// One entry per section — position, look-at target, field of view
const CAMERA_KEYFRAMES = [
  { position: [3.75, 0.50, 1.75], target: [0.05, 0.55, 0.05], fov: 29.0 },
  { position: [0.60, 0.50, 10.00], target: [0.05, 0.55, 0.10], fov: 13.5 },
  // ...
];
```

### GLB Auto-Normalization

Stock and AI-generated GLB files arrive at arbitrary scales and origins. `Scene.jsx` normalizes every loaded model by its height against a fixed target value (`TARGET_HEIGHT = 1.1`) and re-centers it at the world origin. This ensures the camera keyframes frame each variant consistently without per-model adjustment.

The scene also resets the model to identity transforms before measuring, because `useGLTF` caches scenes by URL — without the reset, revisiting a variant would measure the already-scaled geometry and produce a different scale each time.

### Product Variant Carousel

Switching between the three bikes is a pure DOM-level animation. GSAP slides the `Scene` wrapper div out to one side, React swaps the `modelPath` prop while it is off-screen, then GSAP slides the wrapper back in from the opposite side. The camera system is not involved in the transition.

`Scene` is wrapped in `forwardRef` so `App.jsx` can hold a ref to the wrapper div and animate it without needing to reach into the canvas internals.

### Section Entrance Animations

`useSectionInView.js` wraps the browser `IntersectionObserver` API in a React hook. A `Fade` component uses it to slide and fade content in from the left, right, or bottom when a section enters the viewport — independent of the continuous scroll-camera motion.

---

## Debug Mode

Add `?debug=true` to the URL to enable the camera debug panel. This replaces the scroll-driven camera with a set of sliders for position, target, and FOV. Scroll to a section, dial in the angle with the sliders, and copy the values into `CAMERA_KEYFRAMES` in `useScrollCamera.js`.

```
http://localhost:5173/?debug=true
```

---

## Scripts

```bash
npm run dev       # Development server with HMR
npm run build     # Production build to dist/
npm run preview   # Preview the production build locally
npm run lint      # Run oxlint
```

---

## Adding a New Bike Variant

1. Export your model as a `.glb` file and place it in `public/models/`.
2. Add an entry to the `BIKES` array in `App.jsx`:
   ```js
   { name: 'Nova', model: '/models/bike-nova.glb', color: '#a855f7', glow: '#6b21a8' }
   ```
3. Add a `useGLTF.preload` call at the bottom of `Scene.jsx`:
   ```js
   useGLTF.preload('/models/bike-nova.glb');
   ```
4. Use `?debug=true` to dial in any per-variant camera adjustments if needed.

---

## Dependencies

```json
{
  "@react-three/drei": "^10.7.8",
  "@react-three/fiber": "^9.7.0",
  "gsap": "^3.15.0",
  "react": "^19.2.8",
  "react-dom": "^19.2.8",
  "three": "^0.185.1"
}
```

---

## License

MIT
