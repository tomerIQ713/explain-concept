# Explain Concept — Design Spec

## Overview
A single-file HTML tool that explains code/concepts via OpenRouter's free models. No server, no build step. Open in a browser and go.

## Architecture
- **Single file:** `index.html` — self-contained (CSS in `<style>`, JS in `<script>`)
- **CDN dependency:** `marked.js` for markdown rendering (loaded from CDN on first use, cached by browser)
- **Storage:** `localStorage` for API key, theme preference, and explanation history
- **API:** OpenRouter API (`https://openrouter.ai/api/v1/chat/completions`) — OpenAI-compatible

## Default Model
`deepseek/deepseek-v4-flash:free` — best free model for structured explanations with strong reasoning. Configurable in settings.

## System Prompt
A crafted prompt that forces the model to return exactly 5 labeled sections:

```
You are a patient tutor who explains concepts from scratch. Given the following concept/code and the learner's experience level (Beginner/Intermediate/Advanced), produce a structured explanation with exactly these 5 sections:

1. **Plain English** — What it is in simple terms
2. **How It Works** — The mechanics under the hood
3. **Real-World Analogy** — A relatable metaphor
4. **Code Example** — A clean, minimal example (if applicable)
5. **Common Mistakes** — What beginners get wrong

Match the depth to the experience level. Be accurate but never condescending.
```

## UI Layout
```
+------------------------------------------+
| 🧠 Explain Concept        [🌙] [⚙️]     |
+------------------------------------------+
| Model: deepseek/deepseek-v4-flash:free    |
+------------------------------------------+
| [Beginner ▼]                              |
+------------------------------------------+
| Paste your code or concept here...        |
|                                           |
| [Explain]                                 |
+------------------------------------------+
| (Loading spinner while waiting)           |
+------------------------------------------+
| # Plain English                           |
| ...                                       |
|                                           |
| # How It Works                            |
| ...                                       |
|                                           |
| # Real-World Analogy                      |
| ...                                       |
|                                           |
| # Code Example                            |
| ...                                       |
|                                           |
| # Common Mistakes                         |
| ...                                       |
+------------------------------------------+
| History sidebar (toggleable)              |
| - Previous explanations (click to reopen) |
+------------------------------------------+
```

## Sections
1. **Header** — Logo/title + theme toggle + settings gear
2. **Experience Level** — Dropdown: Beginner / Intermediate / Advanced
3. **Input** — Textarea (auto-resizing)
4. **Explain Button** — Sends request, shows loading state, handles errors
5. **Result** — 5 markdown-rendered sections
6. **Settings Modal** — API key input + model override field
7. **History Panel** — Slide-out sidebar with past explanations (stored in localStorage)

## Error Handling
- No API key → show a prompt to add it in settings
- API error → display the error message inline, suggest checking key/model
- Rate limit (429) → show "Rate limited — wait a moment and retry"
- Network error → show "Check your internet connection"

## States
| State | UI |
|---|---|
| Empty | Textarea placeholder text, disabled button |
| Loading | Spinner/disabled button/text fades |
| Success | Rendered markdown sections |
| Error | Red error box with message |
| No API key | Settings modal auto-opens on first Explain click |

## History
- Max 20 entries stored in localStorage
- Slide-out sidebar panel
- Click entry to re-render that explanation
