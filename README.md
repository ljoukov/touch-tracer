# Touch Tracer

Touch Tracer is a fullscreen, touch-focused image viewer built with SvelteKit.

It is designed for use as an iPhone home screen web app where pinch-to-zoom and accidental viewport scaling should be disabled, and the displayed image should always remain fully visible within the safe area.

## Features

- Fullscreen image display with `object-fit: contain` so the whole image is always visible.
- Safe-area aware layout using `env(safe-area-inset-*)`.
- Touch gesture blocking to prevent pinch zoom and double-tap zoom behavior.
- Works without `dvh` units (uses `window.innerHeight` fallback for iOS 15 compatibility).
- Drag-and-drop and file picker image loading.
- Empty state UI: `Select image / drop image here`.

## Usage

1. Open the app.
2. Tap the dropzone to choose an image, or drag and drop an image file.
3. The image is shown fullscreen and centered, preserving aspect ratio.
4. Tap `Select image` in the corner to replace the current image.

## Development

Install dependencies:

```sh
npm install
```

Run dev server:

```sh
npm run dev
```

Run type checks:

```sh
npm run check
```

Build production output:

```sh
npm run build
```

Preview production build locally:

```sh
npm run preview
```

## Deploy (Cloudflare Workers Assets)

The project uses `@sveltejs/adapter-auto` and deploys via Wrangler static assets:

1. Build:

```sh
npm run build
```

2. Deploy:

```sh
npx wrangler deploy
```

`npm run build` creates `dist/` by combining:

- `.svelte-kit/output/client` (hashed JS/CSS assets)
- `.svelte-kit/output/prerendered/pages` (prerendered HTML)
