# 汉字 Flashcard App — Developer Guide

A single-file Chinese character flashcard app. No build tools, no server needed.

---

## Deploying Locally

**Quickest:** Double-click `chinese-flashcards-1500.html` to open in any browser.
Fonts load from Google Fonts (internet needed first time; cached after).

**Local server (recommended for development):**
```bash
python3 -m http.server 8080
# then open: http://localhost:8080/chinese-flashcards-1500.html
```

**VS Code:** Right-click → "Open with Live Server"

---

## How It Works

Everything is in one HTML file with three sections:

| Section | Purpose |
|---------|---------|
| `<style>` | All CSS — card flip animation, dark mode, layout |
| `<body>` | HTML structure |
| `<script>` | Character data (`DATA` array) + all app logic |

**Progress** is saved in `localStorage` under `hsk1500-learned-v1`. Persists across sessions automatically.

---

## Tier Ranges

| Button | Characters |
|--------|-----------|
| 1–100 | Core particles, pronouns, basic verbs |
| 101–300 | High-frequency content words |
| 301–600 | Common everyday vocabulary |
| 601–1000 | Intermediate vocabulary |
| 1001–1500 | Advanced written/literary vocabulary |

---

## Extending the App

### 1. Add more characters

Find `const DATA = [...]` in the `<script>` section. Each entry:

```js
{id:1501, h:"字", p:"zì", m:"character; word; handwriting",
 zh:"写汉字。", py:"Xiě Hànzì.", en:"Write Chinese characters."}
```

Fields: `id` (unique rank), `h` (Hanzi), `p` (Pinyin with tones),
`m` (English meaning), `zh` (Chinese example), `py` (Pinyin example), `en` (English example).

Then add a new tier button in HTML and range in JS:
```html
<button class="tier-btn" data-tier="6">1501–2000</button>
```
```js
const TIERS = { ..., 6:[1501,2000] };
```

### 2. Add audio pronunciation

Uses the built-in Web Speech API — no API key needed:

```js
function speak(text) {
  const u = new SpeechSynthesisUtterance(text);
  u.lang = 'zh-CN'; u.rate = 0.85;
  window.speechSynthesis.speak(u);
}
// Call in flipCard() when showing the back:
if (isFlipped && deck[idx]) speak(deck[idx].h);
```

### 3. Add stroke order diagrams

Free library — [Hanzi Writer](https://hanziwriter.org):

```html
<script src="https://cdn.jsdelivr.net/npm/hanzi-writer@3.5/dist/hanzi-writer.min.js"></script>
```
```js
// Add a container in card-back HTML: <div id="stroke-target"></div>
// Then in flipCard():
if (isFlipped && deck[idx]) {
  document.getElementById('stroke-target').innerHTML = '';
  HanziWriter.create('stroke-target', deck[idx].h, {
    width:100, height:100, strokeColor:'#B91C1C', showOutline:true
  }).animateCharacter();
}
```

### 4. Add a quiz mode

Show 4 meaning choices and check the answer:
```js
function renderQuiz() {
  const correct = deck[idx];
  const wrong = DATA.filter(c=>c.id!==correct.id)
    .sort(()=>Math.random()-.5).slice(0,3);
  const choices = [...wrong, correct].sort(()=>Math.random()-.5);
  // render buttons, check on click
}
```

### 5. Change accent color

Edit the CSS variables at the top of `<style>`:
```css
:root {
  --red: #1D6AB4;        /* change to any color */
  --red-light: #EFF6FF;
  --red-border: #BFDBFE;
}
```

### 6. Go fully offline (no Google Fonts)

Remove the `<link>` to Google Fonts and use system fonts:
```css
font-family: 'PingFang SC', 'Microsoft YaHei', 'Noto Serif CJK SC', serif;
```

---

## Data Sources

- Frequency order based on [Jun Da Modern Chinese Character Frequency List](http://lingua.mtsu.edu/chinese-computing/statistics/)
- Definitions cross-referenced with [CC-CEDICT](https://cc-cedict.org) (CC BY-SA 4.0)
- Stroke animation (optional) via [Hanzi Writer](https://hanziwriter.org) (MIT)
