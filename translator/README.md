> **Original code by [CodingNepal](https://www.codingnepalweb.com/language-translator-app-html-css-javascript/)** — this project is an extended version of their Language Translator tutorial. More from the creator: [codingnepalweb.com](https://www.codingnepalweb.com)

# Language Translator

A lightweight, single-file web app that translates text between 20 languages. It runs entirely in the browser — no build step, no framework, no installation. Open `translator.html` and start translating.

## What it does

- Translates text between any two of the 20 supported languages
- Reads text aloud in the selected language (text-to-speech)
- Copies the original or translated text to the clipboard
- Swaps the source and target languages (and their text) in one click
- Re-translates automatically whenever you change a language
- Works on desktop and mobile with a responsive layout

## Supported languages

| | | | |
|---|---|---|---|
| Chinese | Croatian | Dutch | English (American) |
| English (British) | French | German | Greek |
| Hindi | Hungarian | Italian | Japanese |
| Korean | Norwegian | Portuguese | Russian |
| Spanish | Swedish | Turkish | Vietnamese |

## Getting started

1. Download `translator.html`.
2. Open it in any modern browser (Chrome, Edge, Firefox, Safari).
3. Type or paste text in the left box, pick the two languages, and press **Translate Text**.

An internet connection is required, since translations come from an online service and the icons and font are loaded from a CDN.

## How it works

Everything lives in one HTML file, organised in three parts:

| Part | Role |
|---|---|
| **HTML** | Two text areas (input and output), two language dropdowns, icon controls, the translate button and a reminder message |
| **CSS** | Light blue theme, flexbox layout, responsive rules for screens under 660px |
| **JavaScript** | Fills the dropdowns, sends translation requests, and handles the icon actions |

**Translation flow**

1. The app checks the input. If it is empty, a reminder appears under the button.
2. If both languages are the same, the text is copied across unchanged.
3. If the pair is American ↔ British English, a built-in word list converts spellings and common vocabulary (for example *color/colour*, *elevator/lift*) without calling the API.
4. Otherwise, the text is sent to the free [MyMemory Translation API](https://mymemory.translated.net/doc/spec.php):

   ```
   https://api.mymemory.translated.net/get?q=<text>&langpair=<from>|<to>
   ```

5. The best match is shown in the output box.

**Long text:** MyMemory limits each request to about 500 characters. The app splits long input into sentence-sized pieces (keeping line breaks), translates them one after another and joins the results.

**Out-of-date requests:** each translation is numbered, so if you change a language or start a new translation mid-request, the older result is discarded rather than overwriting the new one.

## Key functions

| Function | What it does |
|---|---|
| `translate(manual)` | Main entry point: validates input, picks the right translation path, and shows the result or an error |
| `translateText()` / `translateChunk()` | Split text into pieces and call the MyMemory API for each one |
| `chunkLine()` | Splits a line into sentences and packs them into pieces under the request limit |
| `swapEnglish()` | Converts between American and British English locally |
| Swap button | Exchanges the languages and the contents of both boxes |
| Copy icons | Copy the input or output text (with a fallback for older browsers) |
| Speaker icons | Read the text aloud using the browser's built-in speech synthesis |
| Language dropdowns | Changing either one re-runs the translation if there is text |
| `Ctrl/Cmd + Enter` | Keyboard shortcut for translating |

## Customising

- **Add a language:** add an entry to the `countries` object near the top of the script, e.g. `"pl-PL": "Polish"`. Codes follow the MyMemory format (`language-COUNTRY`).
- **Change the default pair:** edit the `defaultCode` line in the dropdown-filling loop (currently English (British) → German).
- **Change the colours:** edit the `--bg` (page background) and `--accent` / `--accent-hover` (button) variables at the top of the stylesheet.

## Limitations

- The free MyMemory service has a daily usage limit, so heavy use can be rate-limited.
- Machine translation can be inaccurate, especially for idioms or specialised text. It is not suitable for legal or medical content.
- The American ↔ British converter covers common words only, not every spelling rule.
- Text-to-speech depends on the voices installed in the user's browser and operating system, so some languages may have no voice available.

## Credits

- Original app concept and code: [CodingNepal](https://www.codingnepalweb.com/language-translator-app-html-css-javascript/)
- Translations: [MyMemory](https://mymemory.translated.net/)
- Icons: [Font Awesome](https://fontawesome.com/) · Font: [Poppins](https://fonts.google.com/specimen/Poppins)
