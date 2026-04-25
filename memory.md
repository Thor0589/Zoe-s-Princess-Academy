# Zoe's Princess Academy Memory

## 2026-04-24 Handoff

### Current Status
- Active project folder is `/Users/fernandoceja/Documents/AI-Projects/Zoe’s Princess Academy`.
- Main app file is `Zoe’s Princess Academy.html`.
- Local preview server was started from this folder on `http://localhost:8766/Zoe%E2%80%99s%20Princess%20Academy.html`.
- Verified current server response: HTML returns `200 text/html`.
- Verified app script parses successfully.
- Verified all 11 princess image paths exist under `assets/`.
- Verified all 11 princess records still include embedded Base64 fallback images for restricted environments.

### Completed Work
- Normalized app branding to `Zoe's Princess Academy` in the browser title and visible home title.
- Rebuilt the home screen to be compact and closer to the reference design:
  - small book/magic icon above the title,
  - title and subtitle near the top,
  - random princess greeting moved into a small pill,
  - no large princess intro card,
  - CSS Grid subject layout with two half-width cards followed by two full-width cards on wide screens.
- Kept the single-file HTML app structure.
- Kept local `assets/` image loading for normal development.
- Kept automatic Base64 fallback loading for iOS Shortcuts or other restricted `file://` environments.
- Confirmed key subject-card assets are served successfully from the local server.

### Key Decisions
- Created this project-local `memory.md` because no local memory file existed after the rename; the AI-Projects parent memory was not updated because this was a project-specific handoff.
- Kept the renamed folder and file with the curly apostrophe: `Zoe’s Princess Academy`.
- Left local asset paths as clean relative paths like `assets/belle.jpg`.
- Used Base64 only as fallback data inside the princess data structure, not as the primary image source.

### Blockers
- No functional app blockers found in the current project state.
- The old `localhost:8765` server points at the pre-rename setup and should not be used for the renamed project.
- Browser automation in this thread lost its runtime after the folder rename because the original working directory disappeared; terminal and HTTP checks were used for final verification.

### Next Intended Step
- Use the new local preview URL on port `8766` or restart VS Code Live Preview from `/Users/fernandoceja/Documents/AI-Projects/Zoe’s Princess Academy`.
- If testing iOS Shortcuts next, open/copy `Zoe’s Princess Academy.html` and confirm local images fall back to embedded Base64 when Shortcuts blocks asset access.

## 2026-04-25 Security Fix

### XSS Fixes Applied to index.html
Applied fixes from security audit (equivalent to claude/init-project-review-lCuch branch):
- `quiz` type: `onclick="speak('${data.audio}')"` → `onclick='speak(${jsString(data.audio)})'`
- `quiz` type: `onclick="handleQuizAnswer('${opt}', '${data.a}')"` → `onclick='handleQuizAnswer(${jsString(opt)}, ${jsString(data.a)})'`
- `blend` type: `onclick="speak('${data.phonemes[i]}')"` → `onclick='speak(${jsString(data.phonemes[i])})'`
- `blend` type: `onclick="speak('${data.item}')"` → `onclick='speak(${jsString(data.item)})'`

Pattern: All data values in inline onclick handlers now go through the existing `jsString()` sanitizer (consistent with the already-correct `sentence` type).

## 2026-04-25 Button/Navigation Audit & Fix

### Root Cause
Every ELA curriculum array contained exactly **1 item**. `nextActivity()` computed `(0 + 1) % 1 = 0`, kept `activityIndex` at 0, and re-rendered the same card. The activity re-appeared (with pop animation) but content was identical — looked dead.

A secondary XSS fix was also missed: the flashcard **Listen** button used `onclick="speak('${data.audio}')"` instead of `jsString()`.

### Buttons Checked
- Hub: subject cards (Reading ELA, Math, Science, Spanish/language toggle) ✓
- Grade select: K / Grade 1 / Grade 2 / Back ✓
- ELA Hub: Letters, Match, Blend, Sentences, Sight Words, Back ✓
- Activity: Next (all types), Back, Listen/Speak, Quiz answers, clickable card ✓
- Progress: Back-to-hub ✓
- Header: logo→hub, star counter→progress, language toggle ✓

### Files Changed
- `index.html` — four changes:
  1. Curriculum expanded: all 30 ELA arrays (en+es × 3 grades × 5 categories) now have 5–6 items; math/science (en+es × 3 grades) expanded to 4–5 items.
  2. `renderActivity`: added optional-chaining crash guard (`?.`) and `Math.min` clamp on activityIndex; renders friendly fallback if dataset is missing.
  3. `nextActivity`: added optional-chaining guard, empty-list fallback to elaHub/gradeSelect, wrap-around toast ("🌟 Round complete!") shown only when `list.length > 1`.
  4. Flashcard Listen button XSS fix: `onclick="speak('${data.audio}')"` → `onclick='speak(${jsString(data.audio)})'`.

### Remaining Content Gaps
- None blocking. Each path has ≥ 4 activities. Content is starter-level; real expansion would add 10–20 items per category.
- No `spanish` subject key — the Spanish hub button switches language to `es` and routes to `ela`. Working as designed; no separate subject needed.

### Next Recommended Improvement
Add a **progress indicator** (e.g., "2 / 5 ✦") to the activity header so the child knows how many cards remain before the round completes.

## 2026-04-25 Full Curriculum Expansion

### Summary
Replaced the starter curriculum (≤6 items per array) with a complete multi-grade curriculum across all subjects and both languages.

### Items Added per Subject
| Subject | Grade | EN items | ES items |
|---------|-------|----------|----------|
| ELA – letters  | K/G1/G2 | 26/20/20 | 25/20/20 |
| ELA – match    | K/G1/G2 | 20/20/20 | 20/20/20 |
| ELA – blend    | K/G1/G2 | 20/20/20 | 20/20/20 |
| ELA – sentences| K/G1/G2 | 15/15/15 | 15/15/15 |
| ELA – sight    | K/G1/G2 | 20/20/20 | 20/20/20 |
| Math           | K/G1/G2 | 25/25/25 | 25/25/25 |
| Science+Social | K/G1/G2 | 25/25/25 | 25/25/25 |

**Grand total: 881 curriculum items**

### Content Highlights
- ELA K: Full A–Z alphabet, Dolch Pre-Primer sight words, 20 CVC blends
- ELA G1: Digraphs (SH/CH/TH/WH/CK/NG), vowel teams, compound words, opposites
- ELA G2: Vowel teams (AI/OA/EE/IGH/OW), prefixes/suffixes, grammar, multisyllabic blends
- Math K: Counting, shapes, addition/subtraction within 5, patterns, odd/even
- Math G1: Operations within 20, place value (tens/ones), time, fractions intro, measurement
- Math G2: Operations within 1000, multiplication (2s/5s/10s), area/perimeter, fractions, 3D shapes, division
- Science K: 5 senses, seasons, living/non-living, community helpers, push/pull, life cycles
- Science G1: Life cycles, food chains, earth materials, light/sound, geography, US landmarks
- Science G2: Water cycle, habitats, solar system, states of matter, simple machines, fossils, US history
- ES ELA: Full Spanish alphabet (incl. Ñ), syllable blending, diphthongs, prefixes/suffixes in Spanish
- ES Math/Science: Spanish translations of all EN content, culturally aligned

### Areas for Future Expansion
- Add 3rd grade (g3) tier across all subjects
- Add more sentence types (question/exclamation) for comprehension
- Add audio recordings for Spanish accented characters (á, é, í, ó, ú)
- Add story-based reading passages (multi-sentence comprehension)
- Add geometry and data/graphing for G2 math
