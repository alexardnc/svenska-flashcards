# Dashboard + Bottom Nav + Verb Conjugation Drill — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a dashboard home screen with bottom nav and a verb conjugation drill exercise to the existing Svenska flashcard portal, without breaking the existing flashcard functionality.

**Architecture:** The existing 1255-line `index.html` is already large. We'll add a view-switching system: the dashboard is the new landing page, flashcards and verb drill are separate views within the same file. Views are shown/hidden via CSS (`display:none`). All state stays in localStorage. The verb drill reuses the existing chapter data (73 verbs across ch 7-9).

**Tech Stack:** Pure HTML/CSS/JS — no framework, no build step. Same as existing app.

**Spec:** `docs/specs/2026-04-18-exercise-drills-design.md`

---

## File Structure

All changes happen in one file:
- **Modify:** `index.html` — add dashboard view, bottom nav, verb drill view, and view-switching logic

Data files are read-only (already contain all verb data needed):
- **Read:** `data/chapter-07.json`, `data/chapter-08.json`, `data/chapter-09.json` — verb cards already embedded in the CHAPTERS object inside index.html

---

## Key Implementation Details

### View System
Three views, switched by showing/hiding:
- `#view-dashboard` — the new home screen (default landing page)
- `#view-flashcards` — the existing flashcard UI (moved into a container div)
- `#view-verb-drill` — the new verb conjugation exercise

Only one view is visible at a time. The bottom nav highlights the active tab. The header and chapter selector are shared across all views.

### Verb Data
Existing verb cards have 4 forms: `[infinitiv, presens, preteritum, supinum]`. The spec calls for 5 fields including imperativ. Imperativ is derived programmatically:
- Group 1 (infinitiv ends in -a): drop the -a → `prata` → `prata!` (imperativ = infinitiv for Group 1)
- Group 2 (infinitiv does NOT end in -a): imperativ = stem → `ringa` → `ring!`, `köpa` → `köp!`
- Group 3 (short verbs): imperativ = infinitiv → `bo` → `bo!`
- Group 4 (irregular): varies, but generally imperativ = stem → `skriva` → `skriv!`, `gå` → `gå!`

The verb group is parsed from the `note` field which contains strings like "Group 1 (-ar)", "Group 2a (-er, -de)", etc.

### Verb Group Colors & Explanations
Each verb group has a rule pattern stored as a JS object, used to generate the color-coded explanation cards. The explanation includes:
- Group badge (e.g. "Grupp 2b")
- Rule text with key terms bold+colored
- Pattern line showing all 5 forms with endings in teal (`#0A818A`)
- Additional examples from the same group

### Scoring
Verb drill scores are stored in localStorage key `svenska_ch${N}_verb_scores` as `{ "ch8-001": { imperativ: true, presens: false, preteritum: true, supinum: false }, ... }`. This enables "weakest first" sorting and per-form tracking.

---

### Task 1: Add View Container System

**Files:**
- Modify: `index.html` — wrap existing flashcard content in a view div, add dashboard and verb drill view containers

This task restructures the HTML to support multiple views without changing any existing functionality. The flashcard view should work exactly as before.

- [ ] **Step 1: Wrap existing `<main>` content in a view div**

Find the opening `<main>` tag (line 742) and the closing `</main>` tag (line 849). Wrap everything between them in a `<div id="view-flashcards">`. Add two new empty view containers before it. The `<main>` tag itself stays as the outer container.

```html
<main>
  <!-- DASHBOARD VIEW -->
  <div id="view-dashboard" class="view" style="display:none">
  </div>

  <!-- FLASHCARDS VIEW (existing content) -->
  <div id="view-flashcards" class="view">
    <!-- Chapter selector -->
    <div class="chapter-selector" id="chapterSelector"></div>
    <!-- ... all existing flashcard content stays here unchanged ... -->
  </div>

  <!-- VERB DRILL VIEW -->
  <div id="view-verb-drill" class="view" style="display:none">
  </div>
</main>
```

- [ ] **Step 2: Add view-switching CSS**

Add to the `<style>` section, after the existing CSS:

```css
/* ── VIEW SYSTEM ── */
.view { display: none; width: 100%; }
.view.active { display: flex; flex-direction: column; align-items: center; gap: 12px; }
```

Remove the `display:none` inline styles from the view divs and instead give `#view-flashcards` the `active` class by default:

```html
<div id="view-flashcards" class="view active">
```

- [ ] **Step 3: Add view-switching JS function**

Add to the `<script>` section, before the INIT block:

```javascript
// ── VIEW SWITCHING ──────────────────────────────────────────────────────────
let currentView = 'flashcards';

function switchView(viewName) {
  document.querySelectorAll('.view').forEach(v => v.classList.remove('active'));
  const target = document.getElementById('view-' + viewName);
  if (target) target.classList.add('active');
  currentView = viewName;
  localStorage.setItem('svenska_last_view', viewName);

  // Update bottom nav active state
  document.querySelectorAll('.bottom-nav-btn').forEach(b => {
    b.classList.toggle('active', b.dataset.view === viewName);
  });

  // Update header title based on view
  const titles = {
    dashboard: 'Svenska',
    flashcards: CHAPTERS[currentChapter].title,
    'verb-drill': 'Verb Drill'
  };
  document.getElementById('headerChapter').textContent = titles[viewName] || '';
}
```

- [ ] **Step 4: Verify flashcards still work**

Open `index.html` in a browser. The flashcard view should look and function exactly as before — chapter switching, card flipping, rating, filters all work. Nothing should be visually different yet.

- [ ] **Step 5: Commit**

```bash
cd ~/claude/svenska/portal-preview
git add index.html
git commit -m "Add view container system for dashboard and verb drill

Wrap existing flashcard content in a switchable view container.
Add empty dashboard and verb drill view containers.
Add switchView() function for toggling between views.
Existing flashcard functionality unchanged."
```

---

### Task 2: Add Bottom Navigation Bar

**Files:**
- Modify: `index.html` — add bottom nav HTML and CSS

- [ ] **Step 1: Add bottom nav CSS**

Add to the `<style>` section:

```css
/* ── BOTTOM NAV ── */
.bottom-nav {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 200;
  display: flex;
  justify-content: space-around;
  align-items: center;
  height: 64px;
  background: var(--card-bg);
  border-top: 1px solid var(--border);
  padding-bottom: env(safe-area-inset-bottom, 0);
  box-shadow: 0 -2px 16px rgba(28,23,16,0.08);
  transition: background 0.3s, border-color 0.3s;
}
.bottom-nav-btn {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  padding: 8px 12px;
  background: none;
  border: none;
  cursor: pointer;
  color: var(--subtext);
  font-family: 'Inter', sans-serif;
  font-size: 0.6rem;
  font-weight: 600;
  transition: color 0.15s;
  -webkit-tap-highlight-color: transparent;
  position: relative;
}
.bottom-nav-btn .nav-icon {
  font-size: 1.3rem;
  line-height: 1;
}
.bottom-nav-btn.active {
  color: #E07830;
}
.bottom-nav-btn.active::after {
  content: '';
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 24px;
  height: 3px;
  background: #E07830;
  border-radius: 0 0 3px 3px;
}
.bottom-nav-label {
  white-space: nowrap;
}
/* Chapter badge in nav */
.nav-chapter-badge {
  font-family: 'Fraunces', serif;
  font-weight: 900;
  font-size: 0.85rem;
  line-height: 1;
}
```

Also update the `<main>` padding-bottom to account for the nav bar:

```css
main {
  /* change padding-bottom from 130px to 84px since bottom nav takes 64px */
  padding: 18px 14px 84px;
}
```

- [ ] **Step 2: Add bottom nav HTML**

Add just before `</body>`, after the closing `</main>` tag but before `<script>`:

```html
<!-- Bottom Navigation -->
<nav class="bottom-nav" id="bottomNav">
  <button class="bottom-nav-btn active" data-view="dashboard" onclick="switchView('dashboard')">
    <span class="nav-icon">&#x1F3E0;</span>
    <span class="bottom-nav-label">Home</span>
  </button>
  <button class="bottom-nav-btn" data-view="flashcards" onclick="switchView('flashcards')">
    <span class="nav-icon">&#x1F4DA;</span>
    <span class="bottom-nav-label">Flashcards</span>
  </button>
  <button class="bottom-nav-btn" data-view="chapter" onclick="openChapterPicker()">
    <span class="nav-icon"><span class="nav-chapter-badge" id="navChapterNum">7</span></span>
    <span class="bottom-nav-label">Chapter</span>
  </button>
  <button class="bottom-nav-btn" data-view="settings" onclick="toggleSettingsPanel()">
    <span class="nav-icon">&#x2699;&#xFE0F;</span>
    <span class="bottom-nav-label">Settings</span>
  </button>
</nav>
```

- [ ] **Step 3: Add chapter picker and settings panel JS**

Add to the `<script>` section:

```javascript
// ── BOTTOM NAV HELPERS ──────────────────────────────────────────────────────
function openChapterPicker() {
  // Scroll to chapter selector and briefly highlight it
  const selector = document.getElementById('chapterSelector');
  if (currentView !== 'flashcards' && currentView !== 'verb-drill') {
    switchView('flashcards');
  }
  selector.scrollIntoView({ behavior: 'smooth', block: 'start' });
  selector.style.outline = '2px solid #E07830';
  setTimeout(() => selector.style.outline = '', 1500);
}

function toggleSettingsPanel() {
  // For now, just toggle dark mode (settings panel can be expanded later)
  document.getElementById('darkToggle').click();
}

function updateNavChapter() {
  const badge = document.getElementById('navChapterNum');
  if (badge) badge.textContent = currentChapter;
}
```

Also update `switchChapter()` to call `updateNavChapter()`:

```javascript
// Add at end of switchChapter() function:
updateNavChapter();
```

- [ ] **Step 4: Update INIT to start on dashboard and set default view**

In the INIT block at the bottom, change the starting view. Replace the existing init code so that:
1. The dashboard view is shown by default (not flashcards)
2. The saved last view is restored from localStorage
3. The nav chapter badge is updated

```javascript
// ── INIT ────────────────────────────────────────────────────────────────────
const savedChapter = Number(localStorage.getItem('svenska_last_chapter'));
if (savedChapter && CHAPTERS[savedChapter]) {
  currentChapter = savedChapter;
  CARDS_RAW = CHAPTERS[savedChapter].cards;
}

renderChapterSelector();
loadRatings();
updateFilterCounts();
buildDeck();
showCard();
updateNavChapter();

// Restore last view or default to dashboard
const savedView = localStorage.getItem('svenska_last_view') || 'dashboard';
switchView(savedView);
```

- [ ] **Step 5: Verify bottom nav works**

Open in browser. Bottom nav should be visible at bottom. Tapping Home/Flashcards should switch views. Chapter badge should show current chapter number. Dark mode toggle should still work via Settings.

- [ ] **Step 6: Commit**

```bash
cd ~/claude/svenska/portal-preview
git add index.html
git commit -m "Add fixed bottom navigation bar with 4 tabs

Home, Flashcards, Chapter, and Settings tabs.
Chapter tab shows current chapter number and scrolls to selector.
Active tab highlighted with orange accent.
Safe area inset for mobile devices."
```

---

### Task 3: Build Dashboard Home Screen

**Files:**
- Modify: `index.html` — add dashboard HTML, CSS, and mastery calculation logic

- [ ] **Step 1: Add dashboard CSS**

```css
/* ── DASHBOARD ── */
.dashboard-greeting {
  width: 100%;
  text-align: left;
  padding: 8px 0 4px;
}
.dashboard-greeting h2 {
  font-family: 'Fraunces', serif;
  font-size: 1.5rem;
  font-weight: 900;
  color: var(--text);
  line-height: 1.2;
}
.dashboard-greeting p {
  font-size: 0.85rem;
  color: var(--subtext);
  margin-top: 4px;
}
.dashboard-grid {
  width: 100%;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}
.tool-card {
  background: var(--card-bg);
  border: 2px solid var(--border);
  border-radius: 16px;
  padding: 18px 14px;
  cursor: pointer;
  transition: border-color 0.2s, transform 0.1s, box-shadow 0.2s;
  text-align: left;
  -webkit-tap-highlight-color: transparent;
  position: relative;
  overflow: hidden;
}
.tool-card:hover { border-color: var(--subtext); }
.tool-card:active { transform: scale(0.97); }
.tool-card.disabled {
  opacity: 0.45;
  cursor: not-allowed;
  pointer-events: none;
}
.tool-card-icon {
  font-size: 1.6rem;
  margin-bottom: 8px;
}
.tool-card-name {
  font-family: 'Fraunces', serif;
  font-weight: 700;
  font-size: 0.95rem;
  color: var(--text);
  line-height: 1.2;
}
.tool-card-desc {
  font-size: 0.7rem;
  color: var(--subtext);
  margin-top: 4px;
  line-height: 1.3;
}
.tool-card-mastery {
  margin-top: 10px;
  height: 4px;
  background: var(--progress-bg);
  border-radius: 4px;
  overflow: hidden;
}
.tool-card-mastery-fill {
  height: 100%;
  background: linear-gradient(90deg, #FF9F6A, #E05A35);
  border-radius: 4px;
  transition: width 0.3s;
}
.tool-card-coming {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 0.55rem;
  font-weight: 800;
  color: var(--subtext);
  text-transform: uppercase;
  letter-spacing: 0.08em;
  background: var(--bg2);
  padding: 2px 8px;
  border-radius: 8px;
}
```

- [ ] **Step 2: Add dashboard HTML**

Inside the `<div id="view-dashboard">`:

```html
<div id="view-dashboard" class="view">
  <!-- Chapter selector (shared) -->
  <div class="chapter-selector" id="dashChapterSelector"></div>

  <div class="dashboard-greeting">
    <h2>Plugga Svenska</h2>
    <p id="dashSubtitle">Kapitel 7 &middot; 132 cards</p>
  </div>

  <div class="dashboard-grid">
    <!-- Flashcards -->
    <div class="tool-card" onclick="switchView('flashcards')">
      <div class="tool-card-icon">&#x1F0CF;</div>
      <div class="tool-card-name">Flashcards</div>
      <div class="tool-card-desc">Swipe through vocabulary cards</div>
      <div class="tool-card-mastery"><div class="tool-card-mastery-fill" id="mastery-flashcards" style="width:0%"></div></div>
    </div>
    <!-- Verb Conjugation -->
    <div class="tool-card" onclick="switchView('verb-drill')">
      <div class="tool-card-icon">&#x1F525;</div>
      <div class="tool-card-name">Verb Drill</div>
      <div class="tool-card-desc">Drill all 5 verb forms</div>
      <div class="tool-card-mastery"><div class="tool-card-mastery-fill" id="mastery-verbs" style="width:0%"></div></div>
    </div>
    <!-- Noun Forms (coming soon) -->
    <div class="tool-card disabled">
      <div class="tool-card-coming">Coming soon</div>
      <div class="tool-card-icon">&#x1F4E6;</div>
      <div class="tool-card-name">Noun Forms</div>
      <div class="tool-card-desc">Obestämd &amp; bestämd, singular &amp; plural</div>
      <div class="tool-card-mastery"><div class="tool-card-mastery-fill" style="width:0%"></div></div>
    </div>
    <!-- Adjective Forms (coming soon) -->
    <div class="tool-card disabled">
      <div class="tool-card-coming">Coming soon</div>
      <div class="tool-card-icon">&#x1F308;</div>
      <div class="tool-card-name">Adjective Forms</div>
      <div class="tool-card-desc">En/ett/plural, komparativ, superlativ</div>
      <div class="tool-card-mastery"><div class="tool-card-mastery-fill" style="width:0%"></div></div>
    </div>
    <!-- Prepositions (coming soon) -->
    <div class="tool-card disabled">
      <div class="tool-card-coming">Coming soon</div>
      <div class="tool-card-icon">&#x1F4CD;</div>
      <div class="tool-card-name">Prepositions</div>
      <div class="tool-card-desc">i, på, om, med, för, till in context</div>
      <div class="tool-card-mastery"><div class="tool-card-mastery-fill" style="width:0%"></div></div>
    </div>
    <!-- Particles (coming soon) -->
    <div class="tool-card disabled">
      <div class="tool-card-coming">Coming soon</div>
      <div class="tool-card-icon">&#x1F9E9;</div>
      <div class="tool-card-name">Particles</div>
      <div class="tool-card-desc">Partikelverb: ladda ner, koppla av...</div>
      <div class="tool-card-mastery"><div class="tool-card-mastery-fill" style="width:0%"></div></div>
    </div>
    <!-- Pronouns (coming soon) -->
    <div class="tool-card disabled">
      <div class="tool-card-coming">Coming soon</div>
      <div class="tool-card-icon">&#x1F464;</div>
      <div class="tool-card-name">Pronouns</div>
      <div class="tool-card-desc">Någon/ingen, den här, min/mitt/mina</div>
      <div class="tool-card-mastery"><div class="tool-card-mastery-fill" style="width:0%"></div></div>
    </div>
    <!-- Sentence Builder (coming soon) -->
    <div class="tool-card disabled">
      <div class="tool-card-coming">Coming soon</div>
      <div class="tool-card-icon">&#x1F3D7;&#xFE0F;</div>
      <div class="tool-card-name">Sentence Builder</div>
      <div class="tool-card-desc">Construct full Swedish sentences</div>
      <div class="tool-card-mastery"><div class="tool-card-mastery-fill" style="width:0%"></div></div>
    </div>
    <!-- Chapter Review (coming soon) -->
    <div class="tool-card disabled" style="grid-column: span 2;">
      <div class="tool-card-coming">Coming soon</div>
      <div class="tool-card-icon">&#x1F3AF;</div>
      <div class="tool-card-name">Chapter Review</div>
      <div class="tool-card-desc">Multiple choice quiz mixing all grammar types</div>
      <div class="tool-card-mastery"><div class="tool-card-mastery-fill" style="width:0%"></div></div>
    </div>
  </div>
</div>
```

- [ ] **Step 3: Add dashboard JS for chapter selector and mastery**

```javascript
// ── DASHBOARD ───────────────────────────────────────────────────────────────
function renderDashChapterSelector() {
  const container = document.getElementById('dashChapterSelector');
  if (!container) return;
  container.innerHTML = '';
  Object.values(CHAPTERS).forEach(ch => {
    const btn = document.createElement('button');
    btn.className = 'chapter-btn' + (ch.num === currentChapter ? ' active' : '');
    btn.dataset.chapter = ch.num;
    btn.innerHTML = `
      <div class="chapter-btn-num">${ch.num}</div>
      <div class="chapter-btn-label">${ch.subtitle}</div>
      <div class="chapter-btn-count">${ch.cards.length} cards</div>
    `;
    btn.addEventListener('click', () => {
      switchChapter(ch.num);
      renderDashChapterSelector();
      updateDashboard();
    });
    container.appendChild(btn);
  });
}

function updateDashboard() {
  // Update subtitle
  const ch = CHAPTERS[currentChapter];
  const sub = document.getElementById('dashSubtitle');
  if (sub) sub.textContent = `Kapitel ${currentChapter} \u00B7 ${ch.cards.length} cards`;

  // Update flashcard mastery from existing ratings
  const ratings = JSON.parse(localStorage.getItem(getStorageKey()) || '{}');
  const cards = ch.cards;
  if (cards.length > 0) {
    const totalRating = cards.reduce((sum, c) => sum + (ratings[c.id] || 0), 0);
    const maxRating = cards.length * 5;
    const pct = Math.round((totalRating / maxRating) * 100);
    const bar = document.getElementById('mastery-flashcards');
    if (bar) bar.style.width = pct + '%';
  }

  // Update verb drill mastery
  const verbScores = JSON.parse(localStorage.getItem(`svenska_ch${currentChapter}_verb_scores`) || '{}');
  const verbs = ch.cards.filter(c => c.type === 'verb');
  if (verbs.length > 0) {
    const totalForms = verbs.length * 4; // 4 forms to type per verb
    let correct = 0;
    verbs.forEach(v => {
      const s = verbScores[v.id];
      if (s) {
        correct += Object.values(s).filter(Boolean).length;
      }
    });
    const pct = Math.round((correct / totalForms) * 100);
    const bar = document.getElementById('mastery-verbs');
    if (bar) bar.style.width = pct + '%';
  }
}
```

- [ ] **Step 4: Update switchView to render dashboard on entry**

Add to the `switchView()` function:

```javascript
// Inside switchView(), after setting currentView:
if (viewName === 'dashboard') {
  renderDashChapterSelector();
  updateDashboard();
}
```

- [ ] **Step 5: Verify dashboard displays correctly**

Open in browser. Dashboard should show as the home screen with all 9 tool cards in a 2-column grid. Flashcards and Verb Drill should be tappable. The rest should be greyed out with "Coming soon" badges. Chapter selector should work. Mastery bars should show 0% initially.

- [ ] **Step 6: Commit**

```bash
cd ~/claude/svenska/portal-preview
git add index.html
git commit -m "Add dashboard home screen with exercise tool cards

9 tool cards in 2-column grid. Flashcards and Verb Drill are active,
rest show 'Coming soon'. Each card has mastery progress bar.
Chapter selector and subtitle update per chapter.
Dashboard is now the default landing view."
```

---

### Task 4: Build Verb Drill — HTML Structure and CSS

**Files:**
- Modify: `index.html` — add verb drill view HTML and styling

- [ ] **Step 1: Add verb drill CSS**

```css
/* ── VERB DRILL ── */
.drill-header {
  width: 100%;
  text-align: center;
  padding: 8px 0;
}
.drill-verb-word {
  font-family: 'Fraunces', serif;
  font-weight: 900;
  font-size: 1.8rem;
  color: var(--c-verb);
  line-height: 1.2;
}
.drill-verb-english {
  font-size: 0.85rem;
  color: var(--subtext);
  margin-top: 4px;
}
.drill-verb-group {
  display: inline-block;
  font-size: 0.65rem;
  font-weight: 800;
  color: white;
  background: var(--c-verb);
  padding: 3px 10px;
  border-radius: 12px;
  margin-top: 8px;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}
.drill-progress {
  width: 100%;
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.72rem;
  color: var(--subtext);
  font-weight: 600;
}
.drill-progress-bar {
  flex: 1;
  height: 4px;
  background: var(--progress-bg);
  border-radius: 4px;
  overflow: hidden;
}
.drill-progress-fill {
  height: 100%;
  background: var(--c-verb);
  border-radius: 4px;
  transition: width 0.3s;
}

/* Input fields */
.drill-form-grid {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.drill-form-row {
  display: flex;
  align-items: center;
  gap: 10px;
}
.drill-form-label {
  width: 90px;
  font-size: 0.75rem;
  font-weight: 700;
  color: var(--subtext);
  text-align: right;
  flex-shrink: 0;
}
.drill-form-input {
  flex: 1;
  font-family: 'Fraunces', serif;
  font-size: 1.1rem;
  font-weight: 700;
  padding: 10px 14px;
  border: 2px solid var(--border);
  border-radius: 12px;
  background: var(--card-bg);
  color: var(--text);
  outline: none;
  transition: border-color 0.2s, background 0.2s;
}
.drill-form-input:focus {
  border-color: var(--c-verb);
}
.drill-form-input.correct {
  border-color: #1F8A4A;
  background: #1F8A4A12;
  color: #1F8A4A;
}
.drill-form-input.incorrect {
  border-color: #C03018;
  background: #C0301812;
  color: #C03018;
  text-decoration: line-through;
}
.drill-form-input.prefilled {
  background: var(--bg2);
  color: var(--subtext);
  border-style: dashed;
}
.drill-correct-answer {
  font-family: 'Fraunces', serif;
  font-weight: 900;
  font-size: 1rem;
  color: #1F8A4A;
  margin-top: 2px;
  display: none;
}
.drill-form-row.has-error .drill-correct-answer { display: block; }

/* Check and Next buttons */
.drill-btn-row {
  width: 100%;
  display: flex;
  gap: 10px;
  margin-top: 4px;
}
.drill-check-btn, .drill-next-btn {
  flex: 1;
  padding: 14px;
  border: none;
  border-radius: 14px;
  font-family: 'Inter', sans-serif;
  font-size: 0.9rem;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.15s, transform 0.1s;
  -webkit-tap-highlight-color: transparent;
}
.drill-check-btn {
  background: var(--c-verb);
  color: white;
}
.drill-check-btn:active { transform: scale(0.97); }
.drill-check-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}
.drill-next-btn {
  background: var(--bg2);
  color: var(--text);
  display: none;
}
.drill-next-btn:active { transform: scale(0.97); }

/* Why button */
.drill-why-btn {
  background: none;
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 4px 12px;
  font-size: 0.7rem;
  font-weight: 700;
  color: var(--subtext);
  cursor: pointer;
  margin-top: 4px;
  transition: border-color 0.15s;
}
.drill-why-btn:hover { border-color: var(--subtext); }

/* Explanation card */
.drill-explanation {
  width: 100%;
  border-radius: 14px;
  padding: 16px;
  margin-top: 8px;
  display: none;
  border-left: 4px solid var(--c-verb);
  background: var(--card-bg);
  box-shadow: var(--shadow);
}
.drill-explanation.visible { display: block; }
.drill-explanation .rule-badge {
  display: inline-block;
  font-size: 0.65rem;
  font-weight: 800;
  color: white;
  background: var(--c-verb);
  padding: 2px 10px;
  border-radius: 8px;
  margin-bottom: 8px;
}
.drill-explanation .pattern-line {
  font-family: 'Fraunces', serif;
  font-size: 1rem;
  font-weight: 700;
  color: var(--text);
  margin: 8px 0;
  line-height: 1.6;
}
.drill-explanation .pattern-line .stem { color: var(--text); }
.drill-explanation .pattern-line .ending { color: #0A818A; font-weight: 900; }
.drill-explanation .why-text {
  font-size: 0.8rem;
  color: var(--text2);
  line-height: 1.5;
}
.drill-explanation .why-text strong { color: var(--text); }
.drill-explanation .why-text .kw-verb { color: var(--c-verb); font-weight: 700; }
.drill-explanation .why-text .kw-ending { color: #0A818A; font-weight: 700; }
.drill-explanation .tip {
  font-size: 0.75rem;
  color: var(--subtext);
  margin-top: 8px;
  padding-top: 8px;
  border-top: 1px solid var(--border);
  font-style: italic;
}

/* Score summary at end */
.drill-summary {
  width: 100%;
  text-align: center;
  padding: 24px 0;
}
.drill-summary h2 {
  font-family: 'Fraunces', serif;
  font-size: 1.4rem;
  font-weight: 900;
  color: var(--text);
}
.drill-summary .score-big {
  font-family: 'Fraunces', serif;
  font-size: 2.5rem;
  font-weight: 900;
  color: var(--c-verb);
  margin: 8px 0;
}
.drill-summary .score-detail {
  font-size: 0.8rem;
  color: var(--subtext);
  margin: 4px 0;
}

/* Verb filter tabs */
.drill-filter {
  width: 100%;
  display: flex;
  gap: 5px;
  flex-wrap: wrap;
}
```

- [ ] **Step 2: Add verb drill HTML**

Inside `<div id="view-verb-drill">`:

```html
<div id="view-verb-drill" class="view">
  <!-- Chapter selector (shared) -->
  <div class="chapter-selector" id="verbChapterSelector"></div>

  <!-- Filter by verb group -->
  <div class="drill-filter" id="verbGroupFilter"></div>

  <!-- Progress -->
  <div class="drill-progress">
    <div class="drill-progress-bar">
      <div class="drill-progress-fill" id="verbDrillProgress" style="width:0%"></div>
    </div>
    <span id="verbDrillCount">0 / 0</span>
  </div>

  <!-- Drill card -->
  <div id="verbDrillCard" style="width:100%">
    <div class="drill-header">
      <div class="drill-verb-word" id="drillVerbWord"></div>
      <div class="drill-verb-english" id="drillVerbEnglish"></div>
      <div class="drill-verb-group" id="drillVerbGroup"></div>
    </div>

    <div class="drill-form-grid" id="drillFormGrid">
      <div class="drill-form-row" data-form="imperativ">
        <div class="drill-form-label">Imperativ</div>
        <div style="flex:1">
          <input class="drill-form-input" id="drillImperativ" autocomplete="off" spellcheck="false" autocapitalize="off">
          <div class="drill-correct-answer" id="correctImperativ"></div>
        </div>
      </div>
      <div class="drill-form-row" data-form="infinitiv">
        <div class="drill-form-label">Infinitiv</div>
        <div style="flex:1">
          <input class="drill-form-input prefilled" id="drillInfinitiv" readonly>
        </div>
      </div>
      <div class="drill-form-row" data-form="presens">
        <div class="drill-form-label">Presens</div>
        <div style="flex:1">
          <input class="drill-form-input" id="drillPresens" autocomplete="off" spellcheck="false" autocapitalize="off">
          <div class="drill-correct-answer" id="correctPresens"></div>
        </div>
      </div>
      <div class="drill-form-row" data-form="preteritum">
        <div class="drill-form-label">Preteritum</div>
        <div style="flex:1">
          <input class="drill-form-input" id="drillPreteritum" autocomplete="off" spellcheck="false" autocapitalize="off">
          <div class="drill-correct-answer" id="correctPreteritum"></div>
        </div>
      </div>
      <div class="drill-form-row" data-form="supinum">
        <div class="drill-form-label">Supinum</div>
        <div style="flex:1">
          <input class="drill-form-input" id="drillSupinum" autocomplete="off" spellcheck="false" autocapitalize="off">
          <div class="drill-correct-answer" id="correctSupinum"></div>
        </div>
      </div>
    </div>

    <div class="drill-btn-row">
      <button class="drill-check-btn" id="verbCheckBtn" onclick="checkVerbDrill()">Check</button>
      <button class="drill-next-btn" id="verbNextBtn" onclick="nextVerbDrill()">Next &rarr;</button>
    </div>

    <!-- Why button (shown after correct answers) -->
    <button class="drill-why-btn" id="verbWhyBtn" style="display:none" onclick="toggleVerbExplanation()">Why?</button>

    <!-- Explanation card -->
    <div class="drill-explanation" id="verbExplanation"></div>
  </div>

  <!-- Summary (shown at end of round) -->
  <div class="drill-summary" id="verbDrillSummary" style="display:none">
    <h2>Round Complete!</h2>
    <div class="score-big" id="verbScoreBig"></div>
    <div class="score-detail" id="verbScoreDetail"></div>
    <div class="score-detail" id="verbWeakestForms"></div>
    <button class="drill-check-btn" style="margin-top:16px;max-width:240px" onclick="startVerbDrill()">Drill again &rarr;</button>
    <button class="drill-next-btn" style="display:inline-block;margin-top:8px;max-width:240px" onclick="switchView('dashboard')">Back to Home</button>
  </div>
</div>
```

- [ ] **Step 3: Verify the HTML structure renders**

Open in browser. Navigate to verb drill view (via dashboard "Verb Drill" card or by manually calling `switchView('verb-drill')` in console). The layout should show: chapter selector, empty progress bar, verb word area, 5 input fields, and Check button. No JS logic yet — just verify the visual structure is correct.

- [ ] **Step 4: Commit**

```bash
cd ~/claude/svenska/portal-preview
git add index.html
git commit -m "Add verb drill view HTML structure and CSS

5 input fields (imperativ through supinum), check/next buttons,
explanation card, why button, progress bar, summary screen.
Color-coded correct/incorrect states on inputs.
No drill logic yet — visual structure only."
```

---

### Task 5: Build Verb Drill — Core Logic

**Files:**
- Modify: `index.html` — add verb drill JavaScript logic

- [ ] **Step 1: Add verb data extraction and imperativ derivation**

```javascript
// ── VERB DRILL LOGIC ────────────────────────────────────────────────────────

// Parse verb group from note field
function parseVerbGroup(note) {
  if (!note) return { group: '?', label: 'Unknown' };
  const m = note.match(/Group\s+(\d[ab]?)/i);
  if (!m) return { group: '?', label: 'Unknown' };
  return { group: m[1], label: 'Grupp ' + m[1] };
}

// Derive imperativ from infinitiv and verb group
function deriveImperativ(infinitiv, group) {
  // Group 1: ends in -a → imperativ = infinitiv (e.g. prata! / jobba!)
  if (group === '1') return infinitiv;
  // Group 2a/2b: drop -a → stem (e.g. ringa→ring!, köpa→köp!)
  if (group === '2a' || group === '2b' || group === '2') {
    return infinitiv.replace(/a$/, '');
  }
  // Group 3: imperativ = infinitiv (e.g. bo!, sy!)
  if (group === '3') return infinitiv;
  // Group 4 / special: drop -a if present (e.g. skriva→skriv!, gå→gå!)
  if (infinitiv.endsWith('a')) return infinitiv.slice(0, -1);
  return infinitiv;
}

// Get all verbs for current chapter, with computed imperativ
function getVerbsForDrill() {
  const ch = CHAPTERS[currentChapter];
  return ch.cards
    .filter(c => c.type === 'verb')
    .map(c => {
      const gInfo = parseVerbGroup(c.back.note);
      const forms = c.back.forms; // [infinitiv, presens, preteritum, supinum]
      return {
        id: c.id,
        infinitiv: forms[0],
        presens: forms[1],
        preteritum: forms[2],
        supinum: forms[3],
        imperativ: deriveImperativ(forms[0], gInfo.group),
        english: c.front,
        group: gInfo.group,
        groupLabel: gInfo.label,
        note: c.back.note,
        example: c.back.example,
        exampleTranslation: c.back.example_translation
      };
    });
}
```

- [ ] **Step 2: Add drill state management and start function**

```javascript
let verbDrillVerbs = [];
let verbDrillIndex = 0;
let verbDrillResults = []; // { id, imperativ: bool, presens: bool, ... }
let verbGroupFilter = 'all';

function startVerbDrill() {
  let verbs = getVerbsForDrill();
  if (verbs.length === 0) {
    alert('No verbs in this chapter!');
    return;
  }

  // Apply group filter
  if (verbGroupFilter !== 'all') {
    verbs = verbs.filter(v => v.group === verbGroupFilter);
    if (verbs.length === 0) {
      alert('No verbs in this group!');
      return;
    }
  }

  // Sort by weakest first (using saved scores)
  const scores = JSON.parse(localStorage.getItem(`svenska_ch${currentChapter}_verb_scores`) || '{}');
  verbs.sort((a, b) => {
    const sa = scores[a.id];
    const sb = scores[b.id];
    const scoreA = sa ? Object.values(sa).filter(Boolean).length : 0;
    const scoreB = sb ? Object.values(sb).filter(Boolean).length : 0;
    return scoreA - scoreB; // lowest score first
  });

  verbDrillVerbs = verbs;
  verbDrillIndex = 0;
  verbDrillResults = [];

  // Show drill card, hide summary
  document.getElementById('verbDrillCard').style.display = '';
  document.getElementById('verbDrillSummary').style.display = 'none';

  showVerbDrillCard();
}

function showVerbDrillCard() {
  const verb = verbDrillVerbs[verbDrillIndex];
  if (!verb) return;

  // Update header
  document.getElementById('drillVerbWord').textContent = verb.infinitiv;
  document.getElementById('drillVerbEnglish').textContent = verb.english;
  document.getElementById('drillVerbGroup').textContent = verb.groupLabel;

  // Update progress
  const pct = Math.round((verbDrillIndex / verbDrillVerbs.length) * 100);
  document.getElementById('verbDrillProgress').style.width = pct + '%';
  document.getElementById('verbDrillCount').textContent = `${verbDrillIndex + 1} / ${verbDrillVerbs.length}`;

  // Reset input fields
  document.getElementById('drillInfinitiv').value = verb.infinitiv;
  ['imperativ', 'presens', 'preteritum', 'supinum'].forEach(form => {
    const el = document.getElementById('drill' + form.charAt(0).toUpperCase() + form.slice(1));
    el.value = '';
    el.className = 'drill-form-input';
    el.disabled = false;
    const row = el.closest('.drill-form-row');
    row.classList.remove('has-error');
    const correctEl = document.getElementById('correct' + form.charAt(0).toUpperCase() + form.slice(1));
    if (correctEl) correctEl.style.display = 'none';
  });

  // Reset buttons
  document.getElementById('verbCheckBtn').style.display = '';
  document.getElementById('verbCheckBtn').disabled = false;
  document.getElementById('verbNextBtn').style.display = 'none';
  document.getElementById('verbWhyBtn').style.display = 'none';
  document.getElementById('verbExplanation').classList.remove('visible');

  // Focus first input
  document.getElementById('drillImperativ').focus();
}
```

- [ ] **Step 3: Add check answer logic**

```javascript
function checkVerbDrill() {
  const verb = verbDrillVerbs[verbDrillIndex];
  const formNames = ['imperativ', 'presens', 'preteritum', 'supinum'];
  const results = {};
  let allCorrect = true;
  let hasError = false;

  formNames.forEach(form => {
    const inputId = 'drill' + form.charAt(0).toUpperCase() + form.slice(1);
    const el = document.getElementById(inputId);
    const userAnswer = el.value.trim().toLowerCase();
    const correctAnswer = verb[form].toLowerCase();

    const isCorrect = userAnswer === correctAnswer;
    results[form] = isCorrect;

    el.disabled = true;

    if (isCorrect) {
      el.classList.add('correct');
    } else {
      allCorrect = false;
      hasError = true;
      el.classList.add('incorrect');
      const row = el.closest('.drill-form-row');
      row.classList.add('has-error');
      const correctEl = document.getElementById('correct' + form.charAt(0).toUpperCase() + form.slice(1));
      if (correctEl) {
        correctEl.textContent = verb[form];
        correctEl.style.display = 'block';
      }
    }
  });

  // Save result
  verbDrillResults.push({ id: verb.id, ...results });

  // Save to localStorage
  const scores = JSON.parse(localStorage.getItem(`svenska_ch${currentChapter}_verb_scores`) || '{}');
  scores[verb.id] = results;
  localStorage.setItem(`svenska_ch${currentChapter}_verb_scores`, JSON.stringify(scores));

  // Show/hide buttons
  document.getElementById('verbCheckBtn').style.display = 'none';
  document.getElementById('verbNextBtn').style.display = '';

  if (hasError) {
    // Auto-show explanation on error
    showVerbExplanation(verb);
  } else {
    // Show Why button for correct answers
    document.getElementById('verbWhyBtn').style.display = '';
  }
}

function nextVerbDrill() {
  verbDrillIndex++;
  if (verbDrillIndex >= verbDrillVerbs.length) {
    showVerbDrillSummary();
  } else {
    showVerbDrillCard();
  }
}
```

- [ ] **Step 4: Add Enter key to submit / advance**

```javascript
// Allow Enter key to Check or Next
document.getElementById('view-verb-drill').addEventListener('keydown', e => {
  if (e.key === 'Enter') {
    e.preventDefault();
    const checkBtn = document.getElementById('verbCheckBtn');
    const nextBtn = document.getElementById('verbNextBtn');
    if (checkBtn.style.display !== 'none' && !checkBtn.disabled) {
      checkVerbDrill();
    } else if (nextBtn.style.display !== 'none') {
      nextVerbDrill();
    }
  }
  // Tab to next input field
  if (e.key === 'Tab' && !e.shiftKey) {
    const inputs = ['drillImperativ', 'drillPresens', 'drillPreteritum', 'drillSupinum'];
    const currentIdx = inputs.indexOf(document.activeElement?.id);
    if (currentIdx >= 0 && currentIdx < inputs.length - 1) {
      e.preventDefault();
      document.getElementById(inputs[currentIdx + 1]).focus();
    }
  }
});
```

- [ ] **Step 5: Verify drill flow works end-to-end**

Open in browser. Go to dashboard → tap Verb Drill. Should see first verb with empty fields. Type answers, hit Check. Correct fields turn green, wrong turn red with correct answer shown below. Hit Next → advances to next verb. Enter key works. After last verb, should show summary (we'll build this in next step).

- [ ] **Step 6: Commit**

```bash
cd ~/claude/svenska/portal-preview
git add index.html
git commit -m "Add verb drill core logic — check answers and advance

Parse verb groups from note field, derive imperativ from infinitiv.
Check answers with green/red feedback per field.
Save scores per verb to localStorage.
Enter key submits, Tab navigates between fields.
Weakest-first sorting based on saved scores."
```

---

### Task 6: Build Verb Drill — Explanations and Summary

**Files:**
- Modify: `index.html` — add explanation card content generation and summary screen

- [ ] **Step 1: Add verb group rule definitions**

```javascript
// ── VERB GROUP RULES ────────────────────────────────────────────────────────
const VERB_GROUP_RULES = {
  '1': {
    label: 'Grupp 1',
    pattern: (stem) => `${stem}<span class="ending">a</span> → ${stem}<span class="ending">ar</span> → ${stem}<span class="ending">ade</span> → ${stem}<span class="ending">at</span>`,
    imperativRule: 'Group 1: imperativ = infinitiv (keep the -a)',
    presensRule: 'Group 1 verbs always end in <span class="kw-ending">-ar</span> in presens',
    preteritumRule: 'Group 1 verbs always end in <span class="kw-ending">-ade</span> in preteritum',
    supinumRule: 'Group 1 verbs always end in <span class="kw-ending">-at</span> in supinum',
    tip: 'The biggest group! If the infinitiv ends in -a and presens ends in -ar, it\'s Group 1.'
  },
  '2a': {
    label: 'Grupp 2a',
    pattern: (stem) => `${stem}<span class="ending">a</span> → ${stem}<span class="ending">er</span> → ${stem}<span class="ending">de</span> → ${stem}<span class="ending">t</span>`,
    imperativRule: 'Group 2: imperativ = stem (drop the -a)',
    presensRule: 'Group 2a verbs end in <span class="kw-ending">-er</span> in presens',
    preteritumRule: 'Group 2a: stem ends in a <strong>voiced</strong> consonant → <span class="kw-ending">-de</span>',
    supinumRule: 'Group 2a verbs end in <span class="kw-ending">-t</span> in supinum',
    tip: 'Voiced consonants: l, m, n, ng, r, v + all vowels. Think: you can hum them.'
  },
  '2b': {
    label: 'Grupp 2b',
    pattern: (stem) => `${stem}<span class="ending">a</span> → ${stem}<span class="ending">er</span> → ${stem}<span class="ending">te</span> → ${stem}<span class="ending">t</span>`,
    imperativRule: 'Group 2: imperativ = stem (drop the -a)',
    presensRule: 'Group 2b verbs end in <span class="kw-ending">-er</span> in presens',
    preteritumRule: 'Group 2b: stem ends in a <strong>voiceless</strong> consonant (<span class="kw-ending">k, p, t, s, x</span>) → <span class="kw-ending">-te</span> (not -de)',
    supinumRule: 'Group 2b verbs end in <span class="kw-ending">-t</span> in supinum',
    tip: 'Voiceless = you can\'t hum them: k, p, t, s, x. That\'s why it\'s -te not -de.'
  },
  '2': {
    label: 'Grupp 2',
    pattern: (stem) => `${stem}<span class="ending">a</span> → ${stem}<span class="ending">er</span> → ${stem}<span class="ending">de/te</span> → ${stem}<span class="ending">t</span>`,
    imperativRule: 'Group 2: imperativ = stem (drop the -a)',
    presensRule: 'Group 2 verbs end in <span class="kw-ending">-er</span> in presens',
    preteritumRule: 'Group 2: <span class="kw-ending">-de</span> after voiced consonant, <span class="kw-ending">-te</span> after voiceless (k,p,t,s,x)',
    supinumRule: 'Group 2 verbs end in <span class="kw-ending">-t</span> in supinum',
    tip: 'Check the last consonant of the stem to know if it\'s -de or -te.'
  },
  '3': {
    label: 'Grupp 3',
    pattern: (stem) => `${stem} → ${stem}<span class="ending">r</span> → ${stem}<span class="ending">dde</span> → ${stem}<span class="ending">tt</span>`,
    imperativRule: 'Group 3: imperativ = infinitiv (short verbs, one syllable)',
    presensRule: 'Group 3: just add <span class="kw-ending">-r</span> to the infinitiv',
    preteritumRule: 'Group 3 verbs end in <span class="kw-ending">-dde</span> in preteritum',
    supinumRule: 'Group 3 verbs end in <span class="kw-ending">-tt</span> in supinum',
    tip: 'Short verbs ending in a vowel: bo, sy, tro, nå, ske.'
  },
  '4': {
    label: 'Grupp 4 (special)',
    pattern: () => 'Irregular — each verb has its own pattern',
    imperativRule: 'Group 4: irregular, but usually imperativ = stem',
    presensRule: 'Group 4 verbs are <strong>irregular</strong> — you need to memorize each one',
    preteritumRule: 'Group 4 verbs have <strong>irregular</strong> preteritum — often a vowel change',
    supinumRule: 'Group 4 verbs have <strong>irregular</strong> supinum — often ends in <span class="kw-ending">-it</span> or <span class="kw-ending">-tt</span>',
    tip: 'Common Group 4 verbs: gå-gick-gått, skriva-skrev-skrivit, dricka-drack-druckit'
  },
  '?': {
    label: 'Unknown group',
    pattern: () => '',
    imperativRule: '', presensRule: '', preteritumRule: '', supinumRule: '',
    tip: ''
  }
};
```

- [ ] **Step 2: Add explanation card rendering**

```javascript
function showVerbExplanation(verb) {
  const el = document.getElementById('verbExplanation');
  const rules = VERB_GROUP_RULES[verb.group] || VERB_GROUP_RULES['?'];

  // Extract stem from infinitiv
  const stem = verb.infinitiv.endsWith('a') && verb.group !== '3'
    ? verb.infinitiv.slice(0, -1)
    : verb.infinitiv;

  // Build pattern with actual forms
  const patternHtml = `<span class="stem">${verb.imperativ}</span> → <span class="stem">${verb.infinitiv.replace(stem, stem)}</span>${verb.infinitiv !== stem ? `<span class="ending">${verb.infinitiv.slice(stem.length)}</span>` : ''} → ${stem}<span class="ending">${verb.presens.slice(stem.length)}</span> → ${stem}<span class="ending">${verb.preteritum.slice(stem.length) || verb.preteritum}</span> → ${stem}<span class="ending">${verb.supinum.slice(stem.length) || verb.supinum}</span>`;

  // Determine which forms were wrong to show relevant rules
  const lastResult = verbDrillResults[verbDrillResults.length - 1];
  let rulesHtml = '';

  const formRuleMap = {
    imperativ: rules.imperativRule,
    presens: rules.presensRule,
    preteritum: rules.preteritumRule,
    supinum: rules.supinumRule
  };

  // Show rules for wrong forms, or all rules if Why button was clicked
  const showAll = !lastResult || Object.values(lastResult).every(v => v === true || typeof v !== 'boolean');
  Object.entries(formRuleMap).forEach(([form, rule]) => {
    if (rule && (showAll || (lastResult && lastResult[form] === false))) {
      const wasWrong = lastResult && lastResult[form] === false;
      rulesHtml += `<div class="why-text" style="margin-top:6px">${wasWrong ? '&#x274C; ' : ''}${rule}</div>`;
    }
  });

  el.innerHTML = `
    <div class="rule-badge">${rules.label}</div>
    <div class="pattern-line">${patternHtml}</div>
    ${rulesHtml}
    ${rules.tip ? `<div class="tip">${rules.tip}</div>` : ''}
  `;
  el.classList.add('visible');
}

function toggleVerbExplanation() {
  const el = document.getElementById('verbExplanation');
  if (el.classList.contains('visible')) {
    el.classList.remove('visible');
  } else {
    showVerbExplanation(verbDrillVerbs[verbDrillIndex]);
  }
}
```

- [ ] **Step 3: Add summary screen**

```javascript
function showVerbDrillSummary() {
  document.getElementById('verbDrillCard').style.display = 'none';
  document.getElementById('verbDrillSummary').style.display = '';

  // Calculate scores
  const totalForms = verbDrillResults.length * 4;
  let correctForms = 0;
  const formTotals = { imperativ: 0, presens: 0, preteritum: 0, supinum: 0 };
  const formCorrect = { imperativ: 0, presens: 0, preteritum: 0, supinum: 0 };

  verbDrillResults.forEach(r => {
    ['imperativ', 'presens', 'preteritum', 'supinum'].forEach(f => {
      formTotals[f]++;
      if (r[f]) {
        correctForms++;
        formCorrect[f]++;
      }
    });
  });

  const pct = Math.round((correctForms / totalForms) * 100);
  document.getElementById('verbScoreBig').textContent = pct + '%';
  document.getElementById('verbScoreDetail').textContent = `${correctForms} / ${totalForms} forms correct across ${verbDrillResults.length} verbs`;

  // Find weakest form
  let weakest = null;
  let weakestPct = 100;
  Object.keys(formTotals).forEach(f => {
    const p = Math.round((formCorrect[f] / formTotals[f]) * 100);
    if (p < weakestPct) {
      weakestPct = p;
      weakest = f;
    }
  });

  if (weakest && weakestPct < 100) {
    document.getElementById('verbWeakestForms').textContent = `Weakest form: ${weakest} (${weakestPct}%)`;
  } else {
    document.getElementById('verbWeakestForms').textContent = 'All forms strong!';
  }

  // Update progress bar to 100%
  document.getElementById('verbDrillProgress').style.width = '100%';
}
```

- [ ] **Step 4: Add verb group filter tabs**

```javascript
function renderVerbGroupFilter() {
  const container = document.getElementById('verbGroupFilter');
  if (!container) return;

  const verbs = getVerbsForDrill();
  const groups = [...new Set(verbs.map(v => v.group))].sort();

  container.innerHTML = '';

  // "All" tab
  const allBtn = document.createElement('button');
  allBtn.className = 'filter-tab' + (verbGroupFilter === 'all' ? ' active' : '');
  allBtn.style.cssText = verbGroupFilter === 'all' ? 'background:var(--c-verb);border-color:var(--c-verb);color:white' : '';
  allBtn.textContent = `All (${verbs.length})`;
  allBtn.addEventListener('click', () => { verbGroupFilter = 'all'; renderVerbGroupFilter(); startVerbDrill(); });
  container.appendChild(allBtn);

  // Per-group tabs
  groups.forEach(g => {
    const count = verbs.filter(v => v.group === g).length;
    const btn = document.createElement('button');
    btn.className = 'filter-tab' + (verbGroupFilter === g ? ' active' : '');
    btn.style.cssText = verbGroupFilter === g ? 'background:var(--c-verb);border-color:var(--c-verb);color:white' : '';
    btn.textContent = `Grupp ${g} (${count})`;
    btn.addEventListener('click', () => { verbGroupFilter = g; renderVerbGroupFilter(); startVerbDrill(); });
    container.appendChild(btn);
  });
}
```

- [ ] **Step 5: Add verb drill chapter selector and integrate with switchView**

```javascript
function renderVerbChapterSelector() {
  const container = document.getElementById('verbChapterSelector');
  if (!container) return;
  container.innerHTML = '';
  Object.values(CHAPTERS).forEach(ch => {
    const verbCount = ch.cards.filter(c => c.type === 'verb').length;
    const btn = document.createElement('button');
    btn.className = 'chapter-btn' + (ch.num === currentChapter ? ' active' : '');
    btn.dataset.chapter = ch.num;
    btn.innerHTML = `
      <div class="chapter-btn-num">${ch.num}</div>
      <div class="chapter-btn-label">${ch.subtitle}</div>
      <div class="chapter-btn-count">${verbCount} verbs</div>
    `;
    btn.addEventListener('click', () => {
      switchChapter(ch.num);
      renderVerbChapterSelector();
      renderVerbGroupFilter();
      startVerbDrill();
    });
    container.appendChild(btn);
  });
}
```

Update `switchView()` to initialize verb drill when entering:

```javascript
// Add to switchView(), after the dashboard block:
if (viewName === 'verb-drill') {
  renderVerbChapterSelector();
  renderVerbGroupFilter();
  startVerbDrill();
}
```

- [ ] **Step 6: Verify full drill flow**

Open in browser. Go to Dashboard → Verb Drill:
1. Should show chapter selector with verb counts per chapter
2. Group filter tabs should appear (All, Grupp 1, Grupp 2a, etc.)
3. First verb appears with empty fields
4. Type answers → Check → correct/incorrect feedback with color-coded explanation
5. Click "Why?" on correct answers → explanation shows
6. Tab through fields, Enter to submit
7. After last verb → summary screen with score and weakest form
8. "Drill again" restarts, "Back to Home" returns to dashboard
9. Switch chapters → verb list updates

- [ ] **Step 7: Commit**

```bash
cd ~/claude/svenska/portal-preview
git add index.html
git commit -m "Add verb drill explanations, summary, and group filters

Color-coded explanation cards with rule badges, pattern lines,
and tips per verb group (1, 2a, 2b, 3, 4).
Summary screen shows % correct and weakest form.
Filter tabs per verb group with counts.
Chapter selector shows verb counts."
```

---

### Task 7: Final Integration and Deploy

**Files:**
- Modify: `index.html` — final polish and integration fixes

- [ ] **Step 1: Update page title**

```html
<title>Svenska · Study Portal</title>
```

- [ ] **Step 2: Ensure chapter switching syncs across all views**

Update `switchChapter()` to also refresh dashboard and verb drill state:

```javascript
// Add to end of switchChapter():
if (currentView === 'dashboard') {
  renderDashChapterSelector();
  updateDashboard();
} else if (currentView === 'verb-drill') {
  renderVerbChapterSelector();
  renderVerbGroupFilter();
  startVerbDrill();
}
```

- [ ] **Step 3: Test the complete flow end-to-end**

Full test checklist:
1. Open app → lands on dashboard
2. Dashboard shows 9 tool cards, correct chapter, correct mastery %
3. Tap Flashcards → flashcard view works exactly as before (flip, rate, filter, TTS)
4. Bottom nav → tap Home → back to dashboard
5. Tap Verb Drill → drill starts with first verb
6. Complete a full round → summary shows
7. Switch chapter in verb drill → new verbs load
8. Filter by verb group → correct verbs shown
9. Go back to dashboard → verb mastery bar updated
10. Dark mode works across all views
11. Refresh page → last view and chapter restored from localStorage
12. Mobile: bottom nav has safe area padding, all views scrollable

- [ ] **Step 4: Commit**

```bash
cd ~/claude/svenska/portal-preview
git add index.html
git commit -m "Final integration: sync chapter switching, update page title

Chapter switching now syncs across dashboard, flashcards, and verb
drill views. Page title updated to 'Svenska · Study Portal'."
```

- [ ] **Step 5: Deploy to Vercel**

```bash
cd ~/claude/svenska/portal-preview
git push
vercel --prod
```

Verify at https://portal-preview-tawny.vercel.app that:
- Dashboard loads as home screen
- Flashcards work as before
- Verb drill works end-to-end
- Bottom nav works on mobile

---

## Self-Review Checklist

- [x] **Spec coverage:** Sections 1 (dashboard), 2 (bottom nav), 3 (feedback system), 4 (verb drill) all have tasks. Section 10 (flashcard improvements) deliberately excluded per user request.
- [x] **Placeholder scan:** All tasks have complete code blocks. No TBD/TODO.
- [x] **Type consistency:** `switchView()`, `getVerbsForDrill()`, `VERB_GROUP_RULES`, `parseVerbGroup()`, `deriveImperativ()` — names consistent throughout.
- [x] **Note:** Verb data has 4 forms (no imperativ). Plan derives imperativ from infinitiv + group. The derivation logic is documented and covers all 4 groups + special cases.
- [x] **Note:** No TDD in this plan since the project is pure static HTML with no test framework. Testing is manual browser verification at each step.
