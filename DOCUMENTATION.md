# Image to Video (Preview) — Public API and Usage

This document describes the public APIs, global functions, expected HTML structure, and usage patterns for the Image-to-Video preview app contained in `README.md`.

## Overview
The app allows users to upload multiple images and preview them as a timed slideshow directly in the browser (no backend required). Images are displayed one after another in a single `<div>` using simple CSS and JavaScript, with a default delay of 1 second between images.

## Runtime Environment
- **Platform**: Modern web browsers
- **Dependencies**: None (vanilla HTML/CSS/JS)
- **I/O**: Uses the File API and `URL.createObjectURL` to preview local images securely

## Required HTML Structure
The script expects the following elements to exist with specific IDs:

```html
<input type="file" id="imageUpload" multiple accept="image/*">
<button onclick="startSlideshow()">Start Preview</button>
<div id="videoArea"></div>
```

- **`#imageUpload`**: File input used to select images. The `change` event is wired by the script to load and stage previews.
- **`startSlideshow()` button**: Triggers the slideshow. You can also call `startSlideshow()` programmatically.
- **`#videoArea`**: Container where image elements are dynamically injected.

Optional CSS used by the included sample page:
```css
#videoArea img {
  max-width: 80%;
  height: auto;
  display: none; /* images are hidden by default; the current one is shown */
}
```

## Global Variables
The following globals are created by the page’s script and considered part of the public surface in the browser global scope:

- **`images: File[]`**
  - Collected from `#imageUpload`. Each `File` is converted to an object URL for previewing.
- **`currentIndex: number`**
  - Zero-based index of the image currently being displayed.

## Functions (Public)

### `startSlideshow(): void`
- **Purpose**: Starts the slideshow from the first image. Validates that images have been selected.
- **Behavior**:
  - If no images are loaded, shows an alert: "Please upload some images first!"
  - Resets `currentIndex` to 0 and invokes `showImage()` to begin the timed sequence.
- **Side Effects**:
  - Reads and updates `images` and `currentIndex`.
  - Shows/hides image elements inside `#videoArea`.
- **Usage**:
  - Called by the sample page’s Start button via `onclick`.
  - Can be invoked programmatically after you populate `images` and render image elements.

### `showImage(): void`
- **Purpose**: Displays the current image and schedules the next display step after a fixed delay.
- **Behavior**:
  - Hides all images under `#videoArea`.
  - Shows the image whose ID matches `img{currentIndex}`.
  - Increments `currentIndex` and, if more images remain, schedules the next call using `setTimeout` (default 1000 ms).
- **Notes**: Considered part of the global scope in this page. Treat as an internal step function when integrating; prefer starting the flow with `startSlideshow()`.

## Events

### `#imageUpload` — `change`
- **Purpose**: Load and stage selected images for preview.
- **Behavior**:
  - Converts selected files into an array: `images = Array.from(input.files)`.
  - Clears `#videoArea` and appends an `<img>` element per file with IDs `img0`, `img1`, ...
  - Sets each image’s `src` to an object URL via `URL.createObjectURL(file)`.

## Usage Examples

### Basic Page Integration
Embed the required HTML and include the script. The following example mirrors the sample page structure and behavior.

```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Image to Video — Preview</title>
    <style>
      #videoArea img { max-width: 80%; height: auto; display: none; }
      body { font-family: Arial, sans-serif; text-align: center; padding: 20px; }
    </style>
  </head>
  <body>
    <h1>Image to Video (Preview)</h1>
    <input type="file" id="imageUpload" multiple accept="image/*" />
    <br />
    <button onclick="startSlideshow()">Start Preview</button>

    <div id="videoArea"></div>

    <script>
      let images = [];
      let currentIndex = 0;

      document.getElementById('imageUpload').addEventListener('change', (e) => {
        images = Array.from(e.target.files);
        const videoArea = document.getElementById('videoArea');
        videoArea.innerHTML = '';
        images.forEach((imgFile, index) => {
          const img = document.createElement('img');
          img.src = URL.createObjectURL(imgFile);
          img.id = 'img' + index;
          videoArea.appendChild(img);
        });
      });

      function startSlideshow() {
        if (images.length === 0) {
          alert('Please upload some images first!');
          return;
        }
        currentIndex = 0;
        showImage();
      }

      function showImage() {
        const total = images.length;
        for (let i = 0; i < total; i++) {
          document.getElementById('img' + i).style.display = 'none';
        }
        document.getElementById('img' + currentIndex).style.display = 'block';
        currentIndex++;
        if (currentIndex < total) {
          setTimeout(showImage, 1000);
        }
      }
    </script>
  </body>
</html>
```

### Programmatic Start
You can start the slideshow without the button, for example after validating form input:

```html
<script>
  // After files are selected (or injected), simply call:
  startSlideshow();
</script>
```

### Customizing the Delay
Adjust the slideshow speed by changing the timeout value inside `showImage()`:

```js
setTimeout(showImage, 2000); // 2 seconds per image
```

### Resetting or Replaying the Slideshow
To replay from the beginning after it completes, call:

```js
currentIndex = 0;
showImage();
```

## Error Handling and Edge Cases
- **No images selected**: `startSlideshow()` alerts the user and aborts.
- **Large files / many images**: Rendering many large images could be memory intensive. Consider downscaling or limiting file size.
- **Revoking object URLs**: For production, consider revoking created object URLs via `URL.revokeObjectURL()` when an image is no longer needed.

## Accessibility Notes
- Ensure the Start button is keyboard focusable and has accessible text.
- Consider adding ARIA live regions or status messages to indicate slideshow progress to screen readers.

## Extensibility Ideas
- Add play/pause/stop controls rather than relying on a single Start button.
- Support transitions (fade, slide) with CSS animations.
- Allow per-image duration, or user-configurable slideshow speed.
- Export as an actual video by leveraging canvas capture and the MediaRecorder API (requires additional logic and UX).

## API Summary
- **Globals**:
  - `images: File[]`
  - `currentIndex: number`
- **Functions**:
  - `startSlideshow(): void` — public entry point to start the preview
  - `showImage(): void` — display-and-advance step function used by the slideshow