# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**kurotori** (黒枠トリミング) is a browser-based tool for automatically trimming black borders from images. It is a single-file, zero-dependency application — no build step, no package manager, no server required.

## Running the App

Open `index.html` directly in a browser:

```bash
open index.html          # macOS
xdg-open index.html      # Linux
```

Or serve it with any static file server:

```bash
python3 -m http.server 8000
npx serve .
```

There are no tests, no linting, and no CI configuration.

## Architecture

The entire application lives in `index.html` as a single self-contained file with three sections:

1. **`<style>`** — All CSS using CSS custom properties (`--accent`, `--bg`, etc.) for theming. No external stylesheet.
2. **`<body>`** — Static HTML structure; UI state is shown/hidden via `.active` class toggling.
3. **`<script>`** — All application logic in vanilla JS, no frameworks.

### Core Logic Flow

1. **File loading** (`loadFile`) — Validates MIME type and extension, reads via `FileReader`, draws to a hidden `<canvas id="canvasIn">` to enable pixel access.
2. **Crop detection** (`autoCrop`) — Scans pixel rows/columns using `getImageData`. A pixel is "dark" if all RGB channels are ≤ threshold value. Finds the first non-dark row/column from each edge. Applies padding and draws the cropped region to `<canvas id="canvasOut">`.
3. **Size optimization** — For PNG: generates blob and falls back to WebP if the result exceeds the original file size. For lossy formats: iteratively lowers quality (−5% per step, max 15 attempts) until the output is smaller than the original.
4. **Download** (`downloadResult`) — Creates a temporary `<a>` element with an `ObjectURL` blob and clicks it.

### Intentional Security Measures

These were explicitly implemented and must be preserved:

- **No `innerHTML`** for user-derived content — all dynamic text uses `textContent` or DOM construction (`setChipContent`, `showCanvasPreview`).
- **Filename sanitization** (`sanitizeFileName`) — strips path traversal characters, control characters, leading/trailing dots, and enforces a 200-char limit.
- **File type validation** (`isValidImageFile`) — checks both `file.type` (MIME prefix) and `file.name` extension against an allowlist.
- **Quality loop guard** (`MAX_QUALITY_ATTEMPTS = 15`) — prevents infinite loops in `autoFitLossy`.
