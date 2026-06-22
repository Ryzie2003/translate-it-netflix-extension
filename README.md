# TranslateIt

TranslateIt is an experimental Chrome extension for adding translated subtitle choices to Netflix. It reads the subtitles currently rendered by the Netflix player, sends them to a local DeepL-powered service, and displays the translated text in a custom overlay.

The extension is currently named **ExtraSubs** in `public/manifest.json`.

## Prototype status

This repository is a development prototype, not a production-ready browser extension. The core subtitle observation and translation flow is implemented, but the popup enable/disable flow and several edge cases still need work.

The implementation depends on Netflix DOM structure and CSS class names, which can change without notice.

## How it works

1. A Manifest V3 content script watches for Netflix's audio and subtitle menu.
2. The script adds a custom subtitle column containing DeepL target languages.
3. Selecting a custom language starts observing the active Netflix subtitle container.
4. Subtitle text is posted to a local Express endpoint.
5. The translated response is rendered over the video while the original subtitle layer is hidden.

## Tech stack

- Chrome Extension Manifest V3
- React 19 and TypeScript for the extension popup
- Vite for the extension build
- Plain JavaScript content and background scripts
- Express for the local translation service
- DeepL API through `deepl-node`

## Requirements

- Node.js and npm
- Google Chrome or another Chromium browser
- A [DeepL API](https://www.deepl.com/pro-api) authentication key
- A Netflix account and a title with subtitles enabled

DeepL usage may incur charges depending on your API plan.

## Setup

Install the extension and backend dependencies:

```bash
git clone https://github.com/Ryzie2003/TranslateIt.git
cd TranslateIt
npm install
npm install --prefix deepl-backend
```

Create a `.env` file in the repository root:

```dotenv
DEEPL_API_KEY=your_deepl_api_key
PORT=4032
```

Port `4032` is required by the current content script. If you change it, update the URL in `public/content.js` as well.

Build the extension:

```bash
npm run build
```

Start the local translation service from the repository root:

```bash
node deepl-backend/server.js
```

The service exposes:

```text
POST http://localhost:4032/translate
Content-Type: application/json

{
  "text": "Subtitle text",
  "target_lang": "ES"
}
```

## Load the extension in Chrome

1. Open `chrome://extensions`.
2. Enable **Developer mode**.
3. Select **Load unpacked**.
4. Choose the generated `dist` directory.
5. Open Netflix, play a title, and open its audio and subtitle menu.
6. Keep the local translation service running while using custom subtitles.

After source changes, run `npm run build` again and reload the extension from `chrome://extensions`.

## Available commands

```bash
npm run dev      # Run the Vite popup development server
npm run build    # Type-check and build the extension
npm run lint     # Run ESLint
npm run preview  # Preview the popup build
```

## Project structure

- `public/manifest.json` — Chrome extension manifest.
- `public/content.js` — Netflix menu integration, subtitle observation, translation requests, and overlay rendering.
- `public/background.js` — extension lifecycle and popup-message handling.
- `public/content.css` — selected-state and subtitle-visibility styles.
- `src/App.tsx` — popup enable/disable interface.
- `deepl-backend/server.js` — local Express translation endpoint.

## Current limitations

- The extension has not been prepared for Chrome Web Store distribution.
- The local backend allows cross-origin requests broadly and should not be exposed publicly as written.
- Translation requests are not cached, debounced, retried, or rate-limited.
- Errors from the backend or DeepL API are not surfaced in the Netflix interface.
- The enable/disable state is not persisted and is not fully connected to content-script lifecycle.
- Netflix UI selectors are hard-coded and may break after Netflix interface updates.
- Only the local development backend URL is supported.
