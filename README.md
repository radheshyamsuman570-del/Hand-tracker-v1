# Hand Tracker

Real-time two-hand tracking (21 landmarks per hand) in the browser. MediaPipe `HandLandmarker`
for inference, WebGL for drawing, with rendering moved to a Web Worker + OffscreenCanvas when
supported. Plain HTML/CSS/ES modules: no build step, deploys as a static site.

## Structure
```
hand-tracker/
├── vercel.json            static config: output dir, camera permission policy, caching
├── package.json           test scripts only (no runtime dependencies)
├── test/core.test.mjs     unit tests (smoothing, quality governor, geometry, packing)
└── public/
    ├── index.html
    ├── css/styles.css
    └── js/
        ├── main.js            app loop, UI, lifecycle
        ├── config.js          all tunables + device profile
        ├── camera.js          getUserMedia, live resolution change, flip
        ├── tracker.js         MediaPipe wrapper, GPU->CPU fallback, hand slots
        ├── smoothing.js       One Euro filter
        ├── quality.js         FPS meter + adaptive quality governor
        ├── renderer.js        worker / main-thread renderer facade
        ├── render-worker.js   worker entry
        └── gl-core.js         WebGL core (one draw call per frame)
```

## Run locally
```
npm run serve        # http://localhost:8080  (localhost is allowed to use the camera)
npm test             # unit tests (Node 18+)
```
Test on a phone over HTTPS only (e.g. a Vercel preview URL); browsers block cameras on plain HTTP.

## Deploy to Vercel
1. Push this folder to a Git repo and import it in Vercel, or run `npx vercel --prod` here.
2. Leave framework preset as **Other**. `vercel.json` already sets no build/install command and
   `outputDirectory: public`.

## How it stays fast
- The `<video>` element is shown natively; frames are never copied to the overlay. Only 63 floats
  per hand go to the worker, as a transferred buffer.
- `detectForVideo` is synchronous and driven by `requestVideoFrameCallback`, so frames cannot queue.
- Geometry for bones, joints, glow and trails is batched into a single WebGL draw call.
- The governor lowers capture resolution and overlay DPR when FPS stays under target, and raises
  them again after sustained headroom (30 s lock after a downgrade prevents flapping).
- The camera is released while the tab is hidden.

## Notes
- MediaPipe's runtime and model load from jsDelivr / Google Storage on first visit. To self-host,
  see the comment at the top of `public/js/config.js`.
- Handedness slots follow MediaPipe's label; if the two hands cross, their colors may swap.
