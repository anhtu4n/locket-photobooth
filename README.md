# Photobooth 4 Shots — GitHub Raw JavaScript

`photobooth4-locket.js` is a portable **four-photo image composer**. This source file does not by itself install a filter in the official Locket camera. The host/integration has to execute the JavaScript and feed it four images, one per capture.

## GitHub Raw

1. Create a public GitHub repository.
2. Upload `photobooth4-locket.js` to its root.
3. Open the file and click **Raw**. Its address looks like:
   `https://raw.githubusercontent.com/USERNAME/REPO/main/photobooth4-locket.js`
4. Supply that Raw URL to a host that supports executing JavaScript with Canvas image APIs. **Do not upload the `.js` file as a `.cube` LUT.**

## Integration — 4 taps, 1 PNG

```js
// When your host has loaded photobooth4-locket.js:
const filter = globalThis.Photobooth4Filter;

// In your host's camera capture callback (called 4 times):
const state = await filter.onCapture(capturedImage);
if (state.complete) {
  const pngBlob = state.result.blob; // final PNG with 4 photos
  // Pass pngBlob to your host's normal image preview/send handler.
}
// When user starts a new sequence:
filter.onReset();
```

Use a frame format accepted by Canvas: `Blob`, data URL, `<img>`, `<video>`, `<canvas>`, or `ImageBitmap`.

The default layout is `strip` (four photos stacked vertically). To choose a square-ish 2x2 layout that may fit Locket better:

```js
globalThis.Photobooth4Filter.setLayout('grid');
```

To apply your existing `.cube` to each of the four photos:

```js
const cubeText = await (await fetch('https://example.com/your-lut.cube')).text();
globalThis.Photobooth4Filter.setCube(cubeText);
```

Or create your own independent session with both options:

```js
const session = Photobooth4.create({ layout: 'grid', cubeText });
for (const image of fourCapturedImages) await session.addFrame(image);
const pngBlob = session.getResult().blob;
```

## Demo

`demo.html` lets you test four camera shots or select four photos, then download the merged PNG. On GitHub Pages it needs an HTTPS site for camera permission. The file chooser works without camera access.

## Limitations / compatibility

- A raw GitHub URL only hosts source code; it does not provide camera hooks or install Locket filters.
- A `.cube` file contains color transform data and cannot count captures or combine frames.
- Requires Canvas / image decoding APIs. Many request/response rewrite script engines do not support these APIs.
- Without a documented interface from your Locket integration, there is no way to guarantee automatic insertion as a selectable camera filter. Map the host's capture callback to `onCapture` and its outgoing-image callback to the resulting PNG Blob.
- The source does not bypass app protections, intercept traffic, or transmit photos.

License: MIT. Use at your own risk, and only on camera/photo workflows you are authorized to modify.
