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
