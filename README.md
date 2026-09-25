# Is This Email Safe? 🛡️

A **Windows desktop app** that checks whether an email looks like spam or phishing — or looks fine. Paste an email in, press one big button, and get a clear answer in plain language.

- ✅ Probably safe
- 🔍 Needs caution
- ⚠️ Suspicious
- 🚨 Likely phishing

Everything runs **locally on your computer**. No account, no tracking, no uploads.

Languages: **English** and **Nederlands** (Dutch), with an easy system for adding more.

---

## Features

- **One-screen email checker** with huge text, big buttons, high contrast
- **“Check clipboard” one-click flow**: copy the email (or screenshot it), open the app, press one button — no pasting needed
- **Screenshot checking**: paste (Ctrl+V), drag & drop, or choose a picture — text is read on your computer with built-in OCR (English + Dutch), never uploaded
- **Two input modes**: fill in Sender / Subject / Content separately, or paste the entire email at once (with automatic sender/subject detection)
- **Real rule-based analysis engine** (40+ checks across 12 categories): sender, subject, message content, links, urgency, sensitive-info requests, payment requests, impersonation, suspicious domains, homograph/look-alike alphabets, parcel/vishing/extortion/BEC scams, scam language, formatting
- **Optional Mistral AI second opinion**: off by default; add your own API key in Settings and press the button on a result for a plain-language AI verdict. The key never leaves your computer except in calls you trigger yourself
- **Transparent scoring** (0–100) mapped to 4 friendly levels; exact score stays in the background
- **“Why?” explanations** in plain, non-technical language
- **Per-link analysis**: domain, verdict, and reason (shorteners, IP URLs, lookalikes, punycode, mismatched link text, suspicious TLDs, …)
- **Open `.eml` files** directly from Windows
- **Multi-language UI** (English + Dutch), persisted between launches
- **Accessibility**: Normal / Large / Extra-large text, Light / Dark / Follow-Windows themes, keyboard navigation, visible focus rings, screen-reader labels, never color-only
- **Help screen** written for elderly users + prominent safety rule
- **Settings & About screens**, all settings persist
- **Secure Electron setup**: `contextIsolation`, no `nodeIntegration`, minimal preload bridge, no remote content, email content never rendered as HTML, links never auto-open

## Project structure

```
MailAntispam/
├── package.json                  # scripts + electron-builder (Windows NSIS) config
├── src/
│   ├── main/
│   │   ├── main.js               # Electron main process (window, IPC, file dialog, clipboard)
│   │   ├── store.js              # settings persistence (%APPDATA%/…/settings.json, incl. Mistral key)
│   │   ├── emlParser.js          # dependency-free .eml parser
│   │   ├── ocr.js                # local screenshot OCR (tesseract.js, eng+nld, lazy worker)
│   │   └── ai.js                 # optional Mistral second opinion (prompt, parsing, API call)
│   ├── preload/
│   │   └── preload.js            # minimal contextBridge: settings, version, open file
│   └── renderer/
│       ├── index.html            # all screens (check / help / settings / about)
│       ├── styles.css            # themes, text sizes, high-contrast friendly UI
│       ├── renderer.js           # UI logic (analysis runs locally here)
│       ├── analyzer.js           # rule-based spam/phishing engine (no deps)
│       ├── i18n.js               # locale loader + DOM applier
│       └── locales/
│           ├── en.json           # English strings (all UI + all warning texts)
│           └── nl.json           # Dutch strings
├── samples/                      # test emails (legit, newsletter, EN phish, NL phish, .eml)
├── tests/run-tests.js            # smoke tests (npm test)
├── assets/
│   ├── icon.svg                  # app icon source
│   └── tessdata/                 # bundled OCR language data (eng + nld, offline)
└── dist/                         # build output (created by electron-builder)
```

## Requirements (for building only)

- **Node.js 18+** and npm (only needed to build — the installed app needs nothing extra)
- Windows 10/11 for the installer (building the installer also works from Windows)

Check your Node version:

```powershell
node --version
npm --version
```

## Install & run in development

```powershell
cd C:\Users\Mikas\Downloads\MailAntispam
npm install
npm start
```

`npm start` opens the app in development mode. No command line is needed by end users — that is only for developers.

## Run the tests

```powershell
npm test
```

This checks that a clearly legitimate email scores “safe”, clearly scammy samples score high, the `.eml` parser works, and the English/Dutch locale files contain the same keys.

## Build the Windows installer

```powershell
npm run build:win
```

The installer is created in `dist/`, e.g. `dist/Is-This-Email-Safe-Setup-1.0.0.exe`. Double-click it to install — no Python, Node, or other tools required on the user's PC.

Other scripts:

| Script | What it does |
|---|---|
| `npm run dist` | Build installer for the current OS |
| `npm run pack` | Pack without creating an installer (folder in `dist/`) |

## Privacy

- Local analysis and screenshot reading happen **locally**. Pasted emails and pictures never leave the computer.
- The optional Mistral second opinion sends the email text to Mistral's API **only when you press the button**, and only if you enabled it and saved a key. It is off by default.
- The Mistral API key is stored in the local settings file on your computer only.
- No analytics, no tracking, no accounts.
- Links found in emails are **never opened automatically**; they are shown as text with a Copy button.

## Optional Mistral AI second opinion

1. Create a free account at https://console.mistral.ai/ and copy an API key (the app's Settings screen has a “Where do I get a key?” button that opens this page).
2. Open **Settings** in the app, turn on **“Use Mistral AI for a second opinion”**, and paste the key. It is saved automatically on your computer.
3. Check any email, then press **“Get AI second opinion”** on the result card.

Notes:

- Default model is `mistral-small-latest`. Advanced users can type any other model ID into the **Model** field in Settings (e.g. `mistral-medium-latest`, `mistral-large-latest`, `open-mistral-7b`). The name is strictly validated and falls back to the default if it looks wrong.
- The key (and model choice) is only ever used by the main process; the app UI code never sees the key.
- Without a key, or offline, the button explains what is missing in plain language. The local check always works without any key.

## Screenshot checking (OCR)

- Press **Win+Shift+S**, select the suspicious email, then press **“Check clipboard”** in the app — or paste the screenshot with Ctrl+V, drag it onto the picture box, or choose an image file.
- Reading happens with tesseract.js using the bundled `assets/tessdata/eng.traineddata` + `nld.traineddata` files, so it works fully offline. First read takes a few seconds while the reader starts; later reads take ~1 second.
- If the picture is blurry or tiny, the app says so and suggests a sharper screenshot.

## How the scoring works

Each triggered rule adds points. Examples:

| Signal | Points (approx.) |
|---|---|
| Asks for your password | 20 |
| Sender domain imitates a brand (e.g. `paypa1-secure.com`) | 20 |
| Link points to an IP address / punycode trick | 18–20 |
| Link text says one site, goes to another | 16–18 |
| Free-mail sender (`gmail.com`) + claims to be a bank | 16 |
| Fake “no-reply” with a random code from a free mailbox | 16 |
| Random letter/number jumble in the sender address | 6–12 |
| Account-closure threat, prize promise, SMS-code request | 13–15 |
| Reply-To differs from sender, urgency language | 10–12 |
| Generic greeting, long/random links, ALL-CAPS subject | 5–8 |

Bands:

- **0–20** → Probably safe ✅
- **21–45** → Needs caution 🔍
- **46–70** → Suspicious ⚠️
- **71–100** → Likely phishing 🚨

Small bonuses apply when signals come from many different categories (scams combine tricks). Single tiny signals are clamped so normal emails stay “safe”. The exact number is deliberately de-emphasized in the UI — the friendly label and the “Why?” reasons are what matter.

To tune: edit weights in `src/renderer/analyzer.js` (`pushWarning(...)` calls) and add/adjust keyword lists in the `KW` object (both English and Dutch keywords live side by side so detection works in either language). Conditional scams (parcel, vishing, extortion, instant-payment, CEO fraud) have their own guarded blocks after link extraction so ordinary messages are not flagged.

## How to extend the screenshot / AI parts

- `src/main/ocr.js`: change OCR languages by editing `createWorker(['eng', 'nld'], …)` and adding the matching `.traineddata` file to `assets/tessdata/` (download from https://github.com/tesseract-ocr/tessdata_fast). Image types/sizes are validated in `isImageBuffer` and `recognizeImage`.
- `src/main/ai.js`: `buildPrompt` shapes the request, `parseOpinion` defensively parses the answer, `requestOpinion` performs the HTTPS call with a timeout, `sanitizeModel` validates a custom model ID. Pure helpers are covered by `npm test` without any network access.

## How to add another language

1. Copy `src/renderer/locales/en.json` to e.g. `src/renderer/locales/fr.json`.
2. Translate every string. Keep `{placeholders}` like `{domain}` intact.
3. Add the language code to `SUPPORTED` in `src/renderer/i18n.js`.
4. Add an `<option>` to both language selectors in `src/renderer/index.html` (`#langSelect` and `#settingsLang`).
5. Run `npm test` — it verifies all locale files contain identical keys.

The analyzer's detection keywords (`KW` in `analyzer.js`) are language-independent; consider adding keywords for the new language there too.

## How to extend the analysis engine

`src/renderer/analyzer.js` is a dependency-free module exposing `analyzeEmail({sender, subject, body, fullText})`.

- Add keywords to the `KW` object for new scam phrases (include both languages you support).
- Add a rule with `pushWarning(warnings, category, points, key, params)`, then add the `key` text to **both** locale files under `warnings.*`.
- Link checks live in the `for (const url of urls)` loop; add a check, push a reason id, and add its text to the locales.
- Verify with `npm test` and the `samples/` emails.

## Safety & disclaimer

The app gives an **automated assessment, not a guarantee**. It says “looks safe / looks suspicious”, never “this is 100% safe”. When in doubt: don't click, don't reply, and contact the company through a channel you already trust.
