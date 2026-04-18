# Svenska Study Portal — Exercise Drills Design

**Date:** 2026-04-18
**Status:** In progress (sections 1-6 approved, sections 7-8 pending)
**Exam date:** 2026-04-27 (written + oral, equal weight)
**Scope:** Build on existing ch 7-9 data (284 cards). Ch 11-13 added later.

---

## Design Decisions

- **Typing over clicking** — forces production, triggers the "generation effect" (better retention when you produce vs. recognize)
- **Elaborative feedback always available** — explanation cards shown automatically on wrong answers, available via "Why?" button on correct answers
- **Color-coded word parts** — consistent visual language across all exercises so the brain pattern-matches colors before reading text
- **All Swedish content verified** through Lexin/Folkets Lexikon + Rivstart textbook/workbook. Zero fabrication.
- **Mobile-first** — same warm design language as existing flashcard app

---

## Section 1: Dashboard Home Screen

Current portal opens straight into flashcards. With 9 exercise types, we need a home screen.

### Layout
- Same warm design: off-white `#F4EDE0`, Fraunces + Inter fonts
- Chapter selector at top (same buttons as now)
- Grid of exercise tool cards below
- Each card shows: icon/emoji, exercise name, short description, mastery indicator (% from localStorage)

### Tool Cards (9 total)
1. **Flashcards** — existing deck (links to current flashcard view)
2. **Verb Conjugation** — "Drill all 5 verb forms"
3. **Noun Forms** — "Drill singular/plural, obestämd/bestämd"
4. **Adjective Forms** — "Grundform, en/ett/plural, komparativ, superlativ"
5. **Prepositions** — "i, på, om, med, för, till in context"
6. **Particles / Partikelverb** — "Match the right particle to the verb"
7. **Pronouns** — "någon/ingen, den här/det här, min/mitt/mina"
8. **Sentence Builder** — "Construct full Swedish sentences"
9. **Chapter Review** — "Multiple choice quiz mixing everything"

### Mastery Tracking
- Each exercise saves scores to localStorage per chapter
- Dashboard card shows a simple progress ring or percentage

---

## Section 2: Bottom Navigation Bar

Fixed to bottom of screen, always visible.

### 4 Tabs
- **Home** — dashboard with all exercise tool cards
- **Chapter** — shows current chapter number (e.g. "Ch 7"), tapping opens chapter selector
- **Progress** — overview of mastery across all exercise types for current chapter
- **Settings** — dark mode toggle, TTS speed, reset progress

### Behavior
- Active tab highlighted with warm accent color
- Chapter number visible at a glance without tapping
- On desktop: bottom nav stays (mobile-first priority)

---

## Section 3: Shared Feedback System

Applies to all drill exercises (sections 4-9).

### When You Answer Wrong
- Your answer shown in **red with strikethrough**
- Correct answer shown in **green, large, Fraunces font**
- Explanation card appears automatically below with:
  - **Rule badge** at top in relevant type color (e.g. verb red, noun blue/green)
  - **Pattern line** — the rule shown visually with color-coded endings
  - **Why line** — plain English explanation
  - **Tip** (when relevant) — a memory hook
- Max 4-5 lines. Dense but scannable, never a wall of text.

### When You Answer Right
- Your answer shown in **green with checkmark**
- Subtle **"Why?" button** appears
- Tapping opens the same explanation card
- Can skip and keep drilling fast

### Word Part Color System (consistent across ALL exercises)

| Part | Color | Hex | What it highlights |
|---|---|---|---|
| Stem | dark charcoal | (base text) | Root that doesn't change: `köp`, `ring`, `stor` |
| Ending / suffix | bright teal | `#0A818A` | Part that changes: köp**a**, köp**er**, köp**te** |
| Particle | deep purple | `#7028C8` | Separate word: ladda **ner**, koppla **av** |
| Preposition | magenta | `#B8338A` | Connecting word: **i** Malmö, **på** måndag |
| Komparativ ending | orange | `#B56008` | stor → stör**re**, billig → billig**are** |
| Superlativ ending | deep red | `#C03018` | stor → stör**st**, billig → billig**ast** |
| Plural ending | blue | `#2065B8` | lampa → lamp**or**, stol → stol**ar** |
| Bestämd ending | green | `#1F8A4A` | lampa → lamp**an** → lamp**orna** |
| Gender marker (en/ett) | small badge | — | **en** stol, **ett** bord |

Key words in rule text are **bold + colored**. Pattern lines show all forms with each ending in its distinct color. Concrete examples beyond just the current word. Icons as quick visual anchors.

### Explanation Card Type Colors (border/accent)
- Verb: red `#C03018`
- en-noun: blue `#2065B8`
- ett-noun: green `#1F8A4A`
- Adjective: amber `#B8720A`
- Preposition: magenta `#B8338A`
- Particle: purple `#7028C8`
- Pronoun: purple `#5E35B1`

---

## Section 4: Verb Conjugation Drill

### Flow
1. See verb in **infinitive** + English meaning + verb group badge (e.g. Grupp 1)
2. **5 input fields:** Imperativ, Infinitiv (pre-filled), Presens, Preteritum, Supinum
3. Type the 4 missing forms
4. Hit **Check** (or Enter)
5. Each field turns green/red individually
6. Wrong fields get explanation card with color-coded pattern
7. "Why?" button on correct fields
8. **Next verb** button

### Example Explanation (wrong answer)
> ~~köpde~~ → **köpte**
>
> Grupp 2b — stem ends in **-p** (voiceless consonant)
>
> Voiceless endings (**-k, -p, -t, -s, -x**) → preteritum gets **-te** (not -de)
>
> köp**a** → köp**er** → köp**te** → köp**t**

### Data Source
All verbs from ch 7-9 flashcard JSON. Each verb already has forms stored.

### Filtering
- By chapter (7, 8, 9)
- By verb group (1, 2a, 2b, 3, 4/special)
- "Weakest first" — verbs you've gotten wrong most appear first

### Scoring
Per verb — all 4 fields correct = full mark. Tracks which *specific forms* you struggle with (e.g. nail presens but mix up preteritum vs supinum).

---

## Section 5: Noun Form Drill

### Flow
1. See noun with **en/ett badge** + English meaning + noun group badge
2. **4 input fields:** Obestämd singular (pre-filled), Bestämd singular, Obestämd plural, Bestämd plural
3. Type 3 missing forms
4. Check → green/red per field
5. Explanation card shows noun group pattern with color-coded endings

### Example Explanation (wrong answer)
> ~~lamporen~~ → **lamporna**
>
> Grupp 1 — en-words ending in **-a**
>
> lamp**a** → lamp**an** → lamp**or** → lamp**orna**
>
> (stem dark, -a/-an in green for bestämd, -or/-orna in blue for plural)
>
> All Grupp 1 nouns: drop the -a, add **-or** for plural.

### Filtering
By chapter, by noun group (1-7), by en/ett, weakest first.

---

## Section 6: Adjective Form Drill

### Flow
1. See adjective in **en-form (grundform)** + English meaning
2. **4 input fields:** Ett-form, Plural/bestämd, Komparativ, Superlativ
3. Check → green/red per field
4. Explanation card with color-coded endings

### Example Explanation (wrong answer)
> ~~billigre~~ → **billigare**
>
> Regular adjective (long word, 3+ syllables)
>
> billig → billig**t** → billig**a** → billig**are** → billig**ast**
>
> Long adjectives add **-are / -ast**. Short adjectives change vowel: stor → st**ö**r**re** → st**ö**r**st**

### Special Cases
- Irregular adjectives (bra → bättre → bäst) get **Oregelbunden** badge
- Adjectives that don't compare (svensk, gravid) get a note explaining why

### Filtering
By chapter, by regular/irregular, weakest first.

### Flashcard Fix
Adjective flashcards currently show grundform, komparativ, superlativ but are missing **plural/bestämd form**. Add these rows to adjective card backs:

| Form | Example | Color |
|---|---|---|
| Grundform (en) | stor | type color (amber) |
| Grundform (ett) | stort | teal |
| Plural / bestämd | stora | blue |
| Komparativ | större | orange |
| Superlativ | störst | red |

---

## Section 7: Preposition Drill

### Flow
1. See **Swedish sentence with blank** where preposition goes + English translation
2. Example: "Jag bor ___ Malmö" *(I live in Malmö)*
3. Type the preposition
4. Check → green/red
5. Explanation card shows preposition rule category with more examples

### Example Explanation (wrong answer)
> "Jag bor ___ Malmö" — ~~på~~ → **i**
>
> **Plats: städer & länder** — use **i** for cities, countries, regions
>
> **i** Malmö, **i** Sverige, **i** Skåne
>
> Use **på** for: islands (**på** Gotland), streets (**på** Storgatan), workplaces (**på** jobbet)

### Preposition Categories (from Rivstart ch 7-9)
- Place: i / på / vid / mittemot / mellan
- Direction: till / från / hem / hemma
- Time: i / på / om / för...sedan
- With verbs: tycker **om**, pratar **med**, tittar **på**, drömmer **om**
- Fixed expressions: på morgonen, i helgen, till fots

### Data Source
Sentences from textbook/workbook examples and existing flashcard data. Every sentence verified against source material.

### Filtering
By chapter, by preposition category (place / time / verb-fixed), weakest first.

---

## Section 8: Particles / Partikelverb Drill

### Mode 1: Fill in the Particle
1. See verb + English meaning + sentence with particle missing
2. Example: "Jag ska ___ ___ appen nu." *(download the app)*
3. Type: **ladda ner**
4. Check → green/red

### Example Explanation (wrong answer)
> ~~ladda upp~~ → **ladda ner**
>
> **Partikelverb** — the particle changes the meaning completely
>
> ladda **ner** = download (down)
> ladda **upp** = upload (up)
>
> In main clauses the particle separates from the verb:
> Jag **laddar** **ner** appen.

### Mode 2: Particle Placement (Word Order)
1. See verb + particle + jumbled sentence words
2. Type the full sentence with correct word order
3. Tests the separation rule — main clauses vs bisatser

### Example
> Put in order: *han / koppla av / brukar / i helgen*
>
> **Han brukar koppla av i helgen.** *(infinitive — particle stays together)*
> vs.
> **Han kopplar av i helgen.** *(presens — particle separates)*

### Partikelverb from ch 9
ladda ner, checka in, koppla av, gå upp, gå av, hälsa på, komma ihåg — all verified from textbook.

### Filtering
By chapter, weakest first.

---

## Section 9: Pronoun Drill

### Flow
1. See sentence with blank + English translation
2. Example: "Har du ___ husdjur?" *(Do you have any pets?)*
3. Type: **något**
4. Check → green/red

### Pronoun Categories
- **någon/något/några** — matches noun gender
- **ingen/inget/inga** — negative, matches noun gender
- **Double negation rule:** inte + någon OR ingen, never both
- **Demonstrativa:** den här / det här / de här
- **Possessiva:** min / mitt / mina
- **Reflexiva:** sig / sin / sitt / sina

### Example Explanation (wrong answer)
> ~~någon~~ → **något**
>
> **Någon/något/några** — matches the noun's gender
>
> **någon** + en-word: någon bok
> **något** + ett-word: något husdjur
> **några** + plural: några böcker
>
> Here: husdjur is **ett** husdjur → **något**

### Double Negation Rule (from ch7-131)
> Swedish doesn't double-negate.
>
> Jag har **inga** pengar. *(ingen + noun)*
> Jag har **inte några** pengar. *(inte + någon)*
> ~~Jag har inte inga pengar.~~ *(never both)*

### Filtering
By chapter, by pronoun type, weakest first.

---

## Section 10: Flashcard Improvements

### Revised Weighted Shuffle (more aggressive SRS)

| Rating | Label | Weight | Frequency |
|---|---|---|---|
| 0 | unrated | 7 | Highest — never seen |
| 1 | nope | 6 | Very frequent |
| 2 | okay | 4 | Frequent |
| 3 | good | 2 | Moderate |
| 4 | great | 1 | Rare |
| 5 | got it | 0.5 | Very rare — mastered |

Gap between "nope" and "got it" is now 12x instead of 6x.

### Adjective Card Back — Add Plural Form
Add 5 rows instead of 3:

| Form | Color |
|---|---|
| Grundform (en) | amber (type color) |
| Grundform (ett) | teal |
| Plural / bestämd | blue |
| Komparativ | orange |
| Superlativ | red |

---

## Sections Still Pending

- **Sentence Builder** — construct full Swedish sentences from English prompts (design TBD)
- **Chapter Review** — multiple choice quiz mixing all grammar types (design TBD)

---

## Verification Requirements

1. **Primary source:** Rivstart textbook + workbook — grammar rules from course material
2. **Lexin + Folkets Lexikon:** every word form verified through `lexin` CLI
3. **verify_cards.py pipeline:** extended to cover verb forms and adjective forms
4. **Swedish language accuracy skill:** loaded before writing any Swedish content
5. **Zero fabrication:** if Lexin doesn't have a word, flag it for manual verification

---

## Tech Stack

- Pure static HTML/JS/CSS — same as existing portal
- Single-page app with view switching (dashboard → drill → back)
- All data from existing chapter JSON files
- localStorage for all progress/scores/settings
- No framework, no build step
- Mobile-first, works on any browser
