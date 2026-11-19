# VoxGem

Italian text-to-speech in the browser, using the Gemini model `gemini-3.8-flash-tts`.
Voices: Sofia (a patient Italian teacher) and, in dialogue mode, Omri.

## Run

Open `index.html` in a browser, or serve the folder:

    npx serve .

Then click **API Key**, paste your Gemini key (from https://aistudio.google.com/apikey) and generate.

## Notes

- No backend. Your key is sent only to Google's Gemini API. Anyone using a hosted copy of this page uses their own key and pays for their own usage.
- By default the key is forgotten when the tab closes. Tick "Remember on this device" to keep it.
- Voices, style, mode, speed and your draft script are auto-saved in the browser.
- Audio is not stored. Use **Download WAV** to keep it.
- Dialogue scripts use one line per turn: `Sofia: ...` / `Omri: ...`.
