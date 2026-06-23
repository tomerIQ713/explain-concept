# Explain Concept Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file HTML tool that explains code/concepts via OpenRouter's DeepSeek V4 Flash (free model).

**Architecture:** Single `index.html` file with embedded CSS and JS. No server, no build step. API key in localStorage. OpenRouter API called directly from the browser (CORS-friendly). Markdown rendered via `marked.js` CDN.

**Tech Stack:** Vanilla HTML/CSS/JS, `marked.js` (CDN), OpenRouter API (OpenAI-compatible)

## Global Constraints
- Single file: `index.html` — no external files, no build step
- Default model: `deepseek/deepseek-v4-flash:free`
- All persistent state in `localStorage` only
- API key never sent anywhere except OpenRouter
- Dark theme by default
- Max history: 20 entries

---

### Task 1: HTML Skeleton + CSS Layout + Theme System

**Files:**
- Create: `index.html` (full file)

**Interfaces:**
- Produces: Full HTML skeleton with CSS variables, header, form placeholders, modal overlay, history panel structure, and script tag placeholder

- [ ] **Step 1: Write the complete HTML file with embedded CSS**

Write `index.html` with:
- Document structure with `<!DOCTYPE html>` and viewport meta
- `marked.js` CDN script tag
- CSS custom properties for dark/light themes
- Body layout: container → header → model badge → form → result area
- Header with title and icon buttons (theme toggle, history, settings)
- Form: experience level select, concept textarea, explain button
- Result area div
- Settings modal overlay with API key input, model select dropdown, save button
- History panel (slide-out sidebar)
- Empty script section with comments for JS

- [ ] **Step 2: Open in browser and verify**

Open `index.html` in browser. Confirm:
- Layout renders correctly
- No console errors about missing JS (expected — JS not yet wired)
- Dark theme is default

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add HTML skeleton and CSS layout with theme system"
```

---

### Task 2: Settings Modal Logic + localStorage

**Files:**
- Modify: `index.html` (insert settings JS logic)

**Interfaces:**
- Produces: `getSettings()` → `{ apiKey: string, model: string }`
- Consumes: settings modal HTML from Task 1

- [ ] **Step 1: Write settings management code**

Insert into the script section:
- `STORAGE_KEYS` constants object
- `getSettings()` function reading from localStorage
- `saveSettings(apiKey, model)` function writing to localStorage
- Event listeners for settings button, modal overlay click-to-close, save button
- Populate form fields from stored values on modal open

- [ ] **Step 2: Test in browser**

Open `index.html`, click ⚙️, enter a fake API key `sk-or-test`, pick a different model, save. Close and reopen settings — values should persist.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add settings modal with localStorage persistence"
```

---

### Task 3: API Integration + Explain Button

**Files:**
- Modify: `index.html` (add API call + explain button logic)

**Interfaces:**
- Consumes: `getSettings()` → `{ apiKey, model }` from Task 2
- Produces: `explainConcept(concept, level)` → `Promise<markdownString>`

- [ ] **Step 1: Write the API call function**

Insert `explainConcept(concept, level)`:
- Read API key and model from settings
- Construct system prompt with experience level
- POST to `https://openrouter.ai/api/v1/chat/completions`
- Handle 429, 4xx, 5xx, and network errors
- Return response content

- [ ] **Step 2: Write explain button handler**

- Read text from textarea, level from select
- Disable button, show "Thinking..." with spinner
- Call `explainConcept()`
- On success: hand markdown to render function (stub for now — log to console)
- On error: show error box
- Re-enable button

- [ ] **Step 3: Test with a real API key**

Open in browser, enter a valid OpenRouter API key in settings, type "what is a closure in JavaScript", click Explain. Check browser network tab for successful API call.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add OpenRouter API integration with explain button"
```

---

### Task 4: Result Rendering (Markdown → 5 Sections)

**Files:**
- Modify: `index.html` (add render logic)

**Interfaces:**
- Consumes: `marked` global (CDN-loaded)
- Produces: `renderResult(markdown, query, level)` → renders DOM

- [ ] **Step 1: Write `renderResult()` function**

```javascript
function escapeHtml(str) {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
}

function renderResult(markdown, query, level) {
  const html = marked.parse(markdown, { breaks: true });
  resultArea.innerHTML = `
    <div style="font-size:0.8rem;color:var(--text-muted);margin-bottom:8px;">
      Level: ${level} · ${new Date().toLocaleTimeString()}
    </div>
    <div class="result-content">${html}</div>
  `;
}
```

- [ ] **Step 2: Wire `renderResult` into explain handler**

Replace the console.log stub from Task 3 with `renderResult(markdown, concept, level)`.

- [ ] **Step 3: Add result section CSS styles**

Add styles for `.result-section`, `.result-content`, code blocks within results.

- [ ] **Step 4: Test in browser**

Run an explanation. Confirm markdown renders with proper headings, code blocks, and lists.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add markdown result rendering with marked.js"
```

---

### Task 5: History Management

**Files:**
- Modify: `index.html` (add history logic)

**Interfaces:**
- Produces: `getHistory()`, `addToHistory(query, level, markdown)`, `renderHistory()`
- Consumes: `renderResult(markdown, query, level)` from Task 4

- [ ] **Step 1: Write history functions**

```javascript
const MAX_HISTORY = 20;

function getHistory() {
  try {
    return JSON.parse(localStorage.getItem(STORAGE_KEYS.HISTORY) || '[]');
  } catch { return []; }
}

function addToHistory(query, level, markdown) {
  const history = getHistory();
  history.unshift({ query, level, markdown, timestamp: Date.now() });
  if (history.length > MAX_HISTORY) history.length = MAX_HISTORY;
  localStorage.setItem(STORAGE_KEYS.HISTORY, JSON.stringify(history));
  renderHistory();
}

function renderHistory() {
  const list = document.getElementById('historyList');
  const history = getHistory();
  if (!history.length) {
    list.innerHTML = '<div style="color:var(--text-muted);font-size:0.85rem;">No explanations yet.</div>';
    return;
  }
  list.innerHTML = history.map((item, i) => `
    <div class="history-item" data-index="${i}">
      <div class="query">${escapeHtml(truncate(item.query, 60))}</div>
      <div class="meta">${item.level} · ${new Date(item.timestamp).toLocaleDateString()}</div>
    </div>
  `).join('');

  list.querySelectorAll('.history-item').forEach(el => {
    el.addEventListener('click', () => {
      const idx = parseInt(el.dataset.index);
      const item = getHistory()[idx];
      if (item) {
        renderResult(item.markdown, item.query, item.level);
      }
    });
  });
}

function truncate(str, n) {
  return str.length > n ? str.slice(0, n) + '...' : str;
}
```

- [ ] **Step 2: Wire history into explain handler**

Call `addToHistory(concept, level, markdown)` after successful render in the explain button handler.

- [ ] **Step 3: Wire history toggle button**

Add event listener for history toggle button to open/close panel.

- [ ] **Step 4: Test in browser**

Run 2-3 explanations. Open history sidebar. Click old entry — it should re-render.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add explanation history with localStorage persistence"
```

---

### Task 6: Theme Toggle + Polish

**Files:**
- Modify: `index.html` (add theme toggle + init logic)

- [ ] **Step 1: Write theme toggle functions + init**

```javascript
function getTheme() {
  return localStorage.getItem(STORAGE_KEYS.THEME) || 'dark';
}

function setTheme(theme) {
  document.documentElement.className = theme === 'light' ? 'light' : '';
  localStorage.setItem(STORAGE_KEYS.THEME, theme);
  document.getElementById('themeToggle').textContent = theme === 'light' ? '☀️' : '🌙';
}

// Init
setTheme(getTheme());
renderHistory();
```

- [ ] **Step 2: Add model badge update function + wire to settings**

```javascript
function renderModelBadge() {
  const { model } = getSettings();
  const modelLabel = document.getElementById('modelSelect').selectedOptions[0]?.text || model;
  document.getElementById('modelBadge').textContent = `Model: ${modelLabel}`;
}
```

Call `renderModelBadge()` after settings save.

- [ ] **Step 3: Final verification**

Test the complete flow:
1. Open `index.html` — dark theme by default ✅
2. Toggle theme — persists on refresh ✅
3. Enter API key in settings ✅
4. Select experience level ✅
5. Paste concept, click Explain — loading state ✅
6. Results render with 5 sections ✅
7. History sidebar stores entries ✅
8. Click history entry — re-renders ✅
9. Settings modal saves and restores ✅
10. Error states display correctly ✅

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add theme toggle, model badge, and final polish"
```
