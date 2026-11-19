# VoxGem

Italian text-to-speech in the browser, using the Gemini model `gemini-3.8-flash-tts`.
Voices: Sofia (a patient Italian teacher) and, in dialogue mode, Omri.

## Run

Open `index.html` in a browser, or serve the folder:

    npx serve .

Then click **API Key**, paste your Gemini key (from https://aistudio.google.com/apikey) and generate.

To install it as an app (PWA), serve it over `localhost` or HTTPS (e.g. GitHub Pages) and use the **Install App** button or the browser's install option. Opening the file straight from disk works, but can't be installed.

## Icons (`icons/`)

| File | Used for |
|---|---|
| `favicon-16.png`, `favicon-32.png`, `favicon-48.png` | Browser tab icons |
| `apple-touch-icon-180.png` | iPhone / iPad home screen |
| `icon-192.png`, `icon-512.png`, `icon-1024.png` | PWA install icons (+ header logo uses 192) |
| `icon-maskable-192.png`, `icon-maskable-512.png` | Android adaptive icons (logo on cream, safe-zone padded) |

`icon-512.png` and the maskable icons were generated from `icon-1024.png`.

The service worker loads from the network first, so updates show up on the next online launch; the cache is only an offline fallback. Gemini API calls are never cached.

## Notes

- No backend. Your key is sent only to Google's Gemini API. Anyone using a hosted copy of this page uses their own key and pays for their own usage.
- By default the key is forgotten when the tab closes. Tick "Remember on this device" to keep it.
- Voices, style, mode, speed and your draft script are auto-saved in the browser.
- Audio is not stored. Use **Download WAV** to keep it.
- Dialogue scripts use one line per turn: `Sofia: ...` / `Omri: ...`.
