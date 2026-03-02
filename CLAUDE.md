# cmd-find

A Chrome extension that hijacks Cmd+F (Ctrl+F on Windows) and replaces the native find bar with an AI-powered semantic search overlay. Users bring their own API keys — no backend, no database, no proxying.

## What this is

- Pure client-side Chrome extension (Manifest V3)
- Intercepts Cmd+F on any page, renders a custom overlay
- Extracts page text, sends it + the user's query to their chosen AI provider
- Streams the response back into the overlay
- Stores API keys and preferences in `chrome.storage.sync`

## Tech stack

- Vanilla JS (no frameworks in the content script — keep it lightweight)
- Manifest V3
- Web Crypto API is available if needed
- No build step unless absolutely necessary — keep it simple enough to load unpacked

## Supported providers

At launch, support exactly these three. Nothing else.

- **OpenAI** — use `gpt-4o-mini` as default, streaming via fetch
- **Anthropic** — use `claude-haiku-4-5-20251001` as default, streaming via fetch
- **Google** — use `gemini-2.5-flash` as default, streaming via fetch

Users select their provider in the options page and paste their API key. They can also optionally override the default model with any valid model string.

## Project structure

```
cmd-find/
├── CLAUDE.md
├── manifest.json
├── content.js          # keydown hijack, overlay injection, page text extraction, API calls
├── options.html        # settings page: provider select, API key input, optional model override
├── options.js
├── overlay.css         # styles for the overlay UI
└── icons/              # 16, 48, 128px icons (placeholder PNGs are fine)
```

## Overlay UI

- shadcn/ui-inspired aesthetic: clean, minimal, dark mode, rounded corners, subtle borders
- Triggered by Cmd+F (Mac) or Ctrl+F (Windows/Linux)
- Dismissed by Escape or clicking outside
- Input at the top, streaming response renders below
- Show a subtle spinner while waiting for first token
- Do not show raw JSON or error objects to the user — catch errors and show a clean message

## Page text extraction

- Use `document.body.innerText` as the source
- Trim and truncate to ~12,000 words before sending to avoid token limit issues
- Strip excessive whitespace and newlines

## API call behavior

- Always stream responses — do not wait for the full completion
- Read the API key from `chrome.storage.sync` before each call
- If no API key is set, show a message directing the user to the options page with a direct link
- Never log API keys to the console

## Options page

- Provider dropdown: OpenAI / Anthropic / Google
- API key input (type="password")
- Optional model override text input with placeholder showing the default model for the selected provider
- Save button — persist to `chrome.storage.sync`
- Minimal, clean UI consistent with the overlay aesthetic

## Conventions

- No TypeScript, no React, no bundler — plain HTML/CSS/JS only
- Comment non-obvious logic but don't over-comment
- Prefer `async/await` over promise chains
- Keep content.js focused — if it gets over ~300 lines, flag it for refactoring
- Do not introduce any dependencies without flagging it first

## Things to avoid

- No backend, no server, no database
- No analytics, no telemetry, no network calls except directly to the AI provider APIs
- Do not store API keys anywhere except `chrome.storage.sync`
- Do not use `eval()` or `innerHTML` with unsanitized content (CSP compliance)
- Do not ask to scaffold a build system — load unpacked is fine for now
