# VoxGem — Build Plan

An Italian text-to-speech app that runs entirely in the browser. It uses the Gemini TTS model `gemini-3.8-flash-tts` to read Italian text in the voice of "Sofia", a patient Italian teacher. It can also read a two-person dialogue between Sofia and "Omri".

The look and structure follow the owner's earlier app, **WaveScrub**: one `index.html`, Tailwind CDN, WaveSurfer.js, and a dark/light theme. Copy its design and reuse its UI patterns. Don't copy its speech-to-text or saved-library features.

---

## 1. Constraints

- **No backend.** Plain static files: `index.html` plus optional PWA files. It must work from `localhost` or any static host such as GitHub Pages.
- **The user supplies the API key in the UI.** Never hard-code a key and never commit one. The key goes straight from the browser to Google's API.
- **One HTML file** for the app, with inline `<script>` and `<style>`, same as WaveScrub.
- **No audio storage in the app.** Generated audio lives in memory only, for playback and for the **Download WAV** button. There is no IndexedDB library.
- **Settings are auto-saved.** Voice choices, style instructions and mode go to `localStorage` on every change.
- **Language:** the UI is in English and all generated speech is Italian.

## 2. Configuration (one `CONFIG` object at the top of the script)

```js
const CONFIG = {
  MODEL_ID: 'gemini-3.8-flash-tts',
  API_BASE: 'https://generativelanguage.googleapis.com/v1beta/models',
  SAMPLE_RATE: 24000,   // Gemini TTS returns raw 16-bit PCM, mono, 24 kHz
  DEFAULT_VOICES: { sofia: 'Sulafat', omri: 'Achird' },   // starting choices; the user can change them
  DEFAULT_STYLE: 'Speak in a warm, patient, and encouraging tone, like a friendly Italian language teacher. Read at a slow, deliberate pace (around 0.8x normal speed) with crystal-clear pronunciation, enunciating every syllable and vowel distinctly, leaving natural pauses between turns.',
};
```

### Voice list (for the dropdowns)

These are Gemini's prebuilt voices. All of them speak Italian, since the voices are multilingual. The `style` text is Google's own one-word description. Gender is how each voice is commonly described, and is only there to help choose.

```js
const VOICES = [
  // Female-sounding
  { name: 'Sulafat',      style: 'Warm',       gender: 'F' },
  { name: 'Vindemiatrix', style: 'Gentle',     gender: 'F' },
  { name: 'Achernar',     style: 'Soft',       gender: 'F' },
  { name: 'Despina',      style: 'Smooth',     gender: 'F' },
  { name: 'Erinome',      style: 'Clear',      gender: 'F' },
  { name: 'Callirrhoe',   style: 'Easy-going', gender: 'F' },
  { name: 'Aoede',        style: 'Breezy',     gender: 'F' },
  { name: 'Kore',         style: 'Firm',       gender: 'F' },
  { name: 'Leda',         style: 'Youthful',   gender: 'F' },
  { name: 'Zephyr',       style: 'Bright',     gender: 'F' },
  { name: 'Autonoe',      style: 'Bright',     gender: 'F' },
  { name: 'Laomedeia',    style: 'Upbeat',     gender: 'F' },
  { name: 'Pulcherrima',  style: 'Forward',    gender: 'F' },
  { name: 'Gacrux',       style: 'Mature',     gender: 'F' },
  // Male-sounding
  { name: 'Achird',        style: 'Friendly',      gender: 'M' },
  { name: 'Iapetus',       style: 'Clear',         gender: 'M' },
  { name: 'Sadaltager',    style: 'Knowledgeable', gender: 'M' },
  { name: 'Umbriel',       style: 'Easy-going',    gender: 'M' },
  { name: 'Algieba',       style: 'Smooth',        gender: 'M' },
  { name: 'Charon',        style: 'Informative',   gender: 'M' },
  { name: 'Rasalgethi',    style: 'Informative',   gender: 'M' },
  { name: 'Schedar',       style: 'Even',          gender: 'M' },
  { name: 'Zubenelgenubi', style: 'Casual',        gender: 'M' },
  { name: 'Sadachbia',     style: 'Lively',        gender: 'M' },
  { name: 'Puck',          style: 'Upbeat',        gender: 'M' },
  { name: 'Fenrir',        style: 'Excitable',     gender: 'M' },
  { name: 'Orus',          style: 'Firm',          gender: 'M' },
  { name: 'Alnilam',       style: 'Firm',          gender: 'M' },
  { name: 'Enceladus',     style: 'Breathy',       gender: 'M' },
  { name: 'Algenib',       style: 'Gravelly',      gender: 'M' },
];
```

**Dropdown behavior:**
- Group the options with `<optgroup>` (Female / Male) and label each one `Name — Style`, for example `Sulafat — Warm`.
- The last option is **"Custom…"**. It shows a text input where the user types any voice name, for example a newer voice such as "Tutor 3" that isn't in this list. The custom value is auto-saved too.
- Next to each dropdown is a **▶ Preview** button. It generates a short Italian sample in that voice:
  - Sofia: "Ciao! Sono Sofia, la tua insegnante di italiano."
  - Omri: "Ciao, sono Omri. Piacere di conoscerti!"

  Cache each preview in memory by voice name so replaying doesn't call the API again. Previews count against the user's API usage, so say so in the button's tooltip.

## 3. Gemini TTS API

**Request:** `POST {API_BASE}/gemini-3.8-flash-tts:generateContent`
Headers: `Content-Type: application/json`, `x-goog-api-key: <user key>`. Use this header rather than putting the key in the URL, so the key doesn't end up in logs.

**Single speaker (Sofia only):**
```json
{
  "contents": [{ "parts": [{ "text": "<STYLE>:\n<italian text>" }] }],
  "generationConfig": {
    "responseModalities": ["AUDIO"],
    "speechConfig": {
      "voiceConfig": { "prebuiltVoiceConfig": { "voiceName": "<sofia voice>" } }
    }
  }
}
```

**Dialogue (Sofia + Omri), up to 2 speakers:**
```json
{
  "contents": [{ "parts": [{ "text": "<STYLE>:\nSofia: Ciao Omri, come stai?\nOmri: Sto bene, grazie!" }] }],
  "generationConfig": {
    "responseModalities": ["AUDIO"],
    "speechConfig": {
      "multiSpeakerVoiceConfig": {
        "speakerVoiceConfigs": [
          { "speaker": "Sofia", "voiceConfig": { "prebuiltVoiceConfig": { "voiceName": "<sofia voice>" } } },
          { "speaker": "Omri",  "voiceConfig": { "prebuiltVoiceConfig": { "voiceName": "<omri voice>" } } }
        ]
      }
    }
  }
}
```
The speaker names in the script text must match the `speaker` values exactly. If both speakers have the same voice, warn the user, because the two speakers will sound identical.

**Response:** `candidates[0].content.parts[0].inlineData.data` holds base64 raw PCM (s16le, mono, 24 kHz, mimeType like `audio/L16;codec=pcm;rate=24000`). The browser can't play this directly:
1. Decode the base64 into a `Uint8Array`.
2. Add a 44-byte **WAV header** (PCM, 1 channel, 16-bit, the sample rate from the mimeType or 24000 if missing) → `Blob` of type `audio/wav`.
3. Load that blob into WaveSurfer and keep it in a variable for the Download button. When new audio replaces it, revoke the old object URL.

**Errors:** show the HTTP status and Google's `error.message` in the message bar. Handle these cases specifically:
- 400 mentioning the voice: invalid voice name. Suggest picking another voice or fixing the custom name.
- 400/403 otherwise: invalid key or model.
- 429: quota exceeded.
- A response with no `inlineData`: often a safety block or an empty prompt.

## 4. Script format & parsing

- **Single mode:** the whole textarea is sent as Sofia's text.
- **Dialogue mode:** one line per turn, `Sofia: …` or `Omri: …`.
  - Before sending, normalize speaker prefixes so they match case-insensitively (`sofia:` becomes `Sofia:`).
  - Show a validation warning if a line has no known speaker prefix, but still allow sending.
- Small helper buttons above the textarea: **"+ Sofia"** and **"+ Omri"** insert the prefix at the start of a new line.
- **Auto-detect:** if dialogue mode is on but no `Omri:` line exists, fall back to the single-speaker request.
- The script text is also auto-saved to `localStorage` as an unsent draft, so a reload doesn't lose it.

## 5. Auto-saved settings

One `localStorage` key, `voxgem_settings`, holding JSON:
```js
{ mode: 'single'|'dialogue', sofiaVoice, omriVoice, style, speed, draft }
```
- Save on every `input`/`change` event, debounced by about 300 ms for the textareas.
- Load on startup and fall back to the `CONFIG` defaults for anything missing.
- Wrap all reads and writes in `try/catch`. If storage fails, the app still works with the defaults.
- The API key is stored **separately** under its own key, as described in section 6, and never inside `voxgem_settings`.
- Show a subtle "Saved" tick next to the settings heading after each save.

## 6. UI layout (reuse WaveScrub components and Tailwind classes)

**Header** (same sticky header as WaveScrub):
- Logo with a pulsing strip and the name `Vox<span class="text-blue-500">Gem</span>`. The accent color can stay blue or change to emerald/teal for an Italian feel; the executor decides.
- **API Key** button with a status dot, which opens the key drawer.
- Theme toggle. Also auto-save the theme choice.
- Optional PWA install button (phase 3).
- No "Saved" button, no library drawer, no session title badge.

**API key drawer** (adapted from WaveScrub):
- Password input with placeholder `AIza…` and a show/hide eye toggle.
- **"Remember on this device"** checkbox:
  - Checked: the key is stored in `localStorage`.
  - Unchecked (default): the key is stored in `sessionStorage` and disappears when the tab closes.
- Clear and Done buttons.
- Helper text: "Your key is stored only in this browser and is sent only to Google's Gemini API." Link to aistudio.google.com/apikey.
- Status dot: grey for no key, green when a key is set. Optional: a **"Test key"** button that calls `GET {API_BASE}` with the `x-goog-api-key` header (list models) and shows OK or error.

**Main card 1: Voices & Style** (all auto-saved)
- Mode switch, as a segmented control: **Sofia only** | **Dialogue (Sofia + Omri)**.
- **Sofia voice:** dropdown with a ▶ Preview button.
- **Omri voice:** dropdown with a ▶ Preview button. Shown only in Dialogue mode.
- Collapsible **"Style instructions"** section: a textarea prefilled with `DEFAULT_STYLE` and a "Reset to default" link.

**Main card 2: Script**
- "+ Sofia / + Omri" helper buttons, shown in Dialogue mode.
- Large textarea for the Italian script, with `lang="it"`, `spellcheck`, and a character counter.
- **Generate Audio** button, the primary blue button, with a spinner and a "Generating…" state (same pattern as WaveScrub's Transcribe button). Shortcut: Ctrl/Cmd+Enter.
- Message bar for errors and success (reuse `showSttMessage`, renamed to `showMessage`).

**Main card 3: Player** (shown after generation):
- Large time display (current time / duration), as in WaveScrub.
- WaveSurfer waveform with timeline. Keep zoom-slider, mouse-wheel and pinch zoom.
- Controls: Play/Pause, −2 s / +2 s skip buttons, **Replay from start**, and a speed select (0.5 / 0.75 / 0.85 / 1.0 / 1.25).
  - Speed changes playback with pitch preserved: `setPlaybackRate(rate, true)`.
- **Download WAV** button.
  - Filename: `voxgem-<first-4-words-slug>-<YYYYMMDD-HHMM>.wav`.
  - Next to it, an optional small filename input, prefilled with that name, that the user can edit before downloading.
- Note under the player: "Audio isn't kept after you leave the page. Download it to keep it."
- Optional phase 2: under the player, show the script that produced this audio and highlight the current turn using estimated timing (each turn's share of total characters × duration). Label it as approximate.

**Floating mini-player:** reuse it as-is, with Play/Pause, the time, −2 s/+2 s, and scroll-to-top.

**Keyboard shortcuts:** Space for play/pause (only when focus is not in a text field), ←/→ to skip 2 s, Ctrl/Cmd+Enter to generate.

**Leaving with unsaved audio:** if audio was generated but never downloaded, add a `beforeunload` warning.

## 7. What to drop from WaveScrub

The following go:
- The audio file upload/dropzone.
- **The entire IndexedDB library:** the saved drawer, home library grid, session cards, Save / Save As buttons, the rename modal and the title badge.
- All STT code: provider detection, Groq/xAI/OpenAI, `transcribe`, word-sync rendering, punctuation alignment, the edit-text mode, RTL detection.
- The language select.
- The ±10 ms/±100 ms nudges.
- The "JS Offline / Ready" badge. You can keep it if it shows "Ready" only after WaveSurfer loads.

## 8. Security notes

- Never put the key in a URL, in console logs or in the settings object.
- Escape all user text before inserting it into the page. Prefer `textContent` over `innerHTML`.
- Pin CDN versions, for example `wavesurfer.js@7.8.x` instead of `@7`.
- Add a `.gitignore` covering `.env` and `*.key`, just in case.
- Add a short note in the README: anyone using the hosted page uses their own key and pays for their own usage.

## 9. Build phases

**Phase 1 (MVP)**
1. Scaffold `index.html`: header, theme toggle and API key drawer with `localStorage`/`sessionStorage`.
2. Voices & Style card: Sofia dropdown with custom option, style instructions, and auto-saved settings.
3. Script card, then `generateSpeech()`: fetch, base64 → PCM → WAV blob.
4. Player card: WaveSurfer, play/pause, speed, Download WAV.
5. Error handling and loading states.

**Phase 2**
6. Dialogue mode with the Omri dropdown, multi-speaker config, the "+ Sofia / + Omri" helpers and validation.
7. Voice preview buttons with an in-memory cache.
8. Floating mini-player, keyboard shortcuts, zoom, and the `beforeunload` warning.
9. Optional: approximate turn highlighting.

**Phase 3 (optional)**
10. PWA: `manifest.webmanifest`, `sw.js` (cache only the app shell, never API responses) and icons.
11. Long scripts: if the text exceeds a safe length (for example ~4000 characters), split it at paragraph or turn boundaries. Generate the parts one after another and join the PCM before adding the WAV header.

## 10. Acceptance checklist

- [ ] With no key set, clicking Generate opens the key drawer and shows a message.
- [ ] Single-mode Italian text produces playable audio in the chosen Sofia voice at a slow, clear pace.
- [ ] A dialogue script produces audio with two distinct voices.
- [ ] Changing a voice, the style, the mode or the speed and then reloading keeps those choices. A custom voice name is kept too.
- [ ] Preview plays a short sample. Clicking it again for the same voice makes no new API call.
- [ ] The downloaded `.wav` plays in a normal media player.
- [ ] Nothing audio-related is stored in the browser: no IndexedDB, no audio in `localStorage`.
- [ ] The key survives a reload only if "Remember" was checked.
- [ ] Both light and dark mode work, and the layout works at 375 px phone width.
- [ ] No key appears in the network URL, the console or `voxgem_settings`.
