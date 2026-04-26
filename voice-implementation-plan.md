# Zoe's Princess Academy Voice Implementation Plan

This is a later-wiring plan only. The current app uses `speechSynthesis` through `speak()`, and that behavior should remain intact until MP3 playback is deliberately added.

## Current audio-related flow in `index.html`

- `window.onload = init` boots the app.
- `init()` calls `updateStarDisplay()`, `navigate('hub')`, registers first-click audio unlock listeners, and loads speech synthesis voices.
- `unlockAudio()` primes `speechSynthesis` after first body click/touch.
- `speak(text, rate, pitch)` cancels current speech and speaks the given text using `SpeechSynthesisUtterance`.
- `navigate(view, params)` clears the app container, cancels speech synthesis, updates `state`, then routes to `renderHub`, `renderGradeSelect`, `renderElaHub`, `renderActivity`, or `renderProgress`.
- `renderHub()` currently speaks `Hello Zoe, I am ${princess.name}`.
- `renderGradeSelect()` currently speaks either the selected persona greeting plus grade prompt or just the grade prompt.
- `renderElaHub()` currently speaks the ELA category prompt.
- `renderActivity()` currently speaks item or activity instructions depending on `data.type`.
- `showFeedback(isPositive)` picks from `feedbackPool`, speaks the feedback, increments stars on correct answers, and fires confetti.
- `window.handleQuizAnswer(selected, target)` calls `showFeedback(true)` and advances after correct answers; wrong answers stay on the item.
- `window.nextActivity()` advances or shows a round-complete toast when wrapping.
- `renderProgress()` currently speaks the current star count.

## Recommended folder structure

```text
assets/
  audio/
    EN/
      app_boot_01.mp3
      hub_welcome_01.mp3
      ...
      curriculum/
        ela/
        math/
        science/
    ES/
      app_boot_01.mp3
      hub_welcome_01.mp3
      ...
      curriculum/
        ela/
        math/
        science/
```

Use the flat files in `voice-file-map.json` for shared UI/navigation lines. Use `assets/audio/{lang}/curriculum/...` later for generated item-specific recordings from existing curriculum values.

## Trigger plan by Voice ID

| Voice ID | Existing function that should call it | Interrupt or wait | Automatic or user click | Implementation notes |
|---|---|---|---|---|
| APP_BOOT_01 | `init()` | Wait | User click only | Browser autoplay will usually block this on first load. Queue it after `unlockAudio()` or use only as a first-gesture welcome. |
| APP_AUDIO_UNLOCK_01 | `unlockAudio()` | Wait | User click | Optional very short cue after the first trusted gesture. |
| HUB_WELCOME_01 | `renderHub()` | Interrupt current route audio | Automatic after click-unlocked | Equivalent to current `speak(introText)` fallback. |
| HUB_PROMPT_01 | `renderHub()` | Wait after welcome | Automatic after click-unlocked | Keep optional; avoid stacking with long princess greeting. |
| HUB_RANDOM_PRINCESS_DYNAMIC | `renderHub()` | Interrupt on route | Automatic after click-unlocked | Later can resolve by `state.princess.name`; fallback to current speech synthesis. |
| SUBJECT_READING_01 | Reading card inline handler before `navigate('gradeSelect', ...)` | Interrupt | User click | Best wired by replacing inline onclick with a helper such as `selectSubject('ela','belle')` later. |
| SUBJECT_MATH_01 | Math card inline handler | Interrupt | User click | Same helper pattern as Reading. |
| SUBJECT_SCIENCE_01 | Science card inline handler | Interrupt | User click | Same helper pattern as Reading. |
| SUBJECT_SPANISH_01 | Spanish card inline handler with `toggleLanguage('es')` | Interrupt | User click | Play after `state.lang` is set to `es`; avoid duplicate language-switch line. |
| GRADE_SELECT_01 | `renderGradeSelect(container, fromHub)` | Interrupt route audio | Automatic after click-unlocked | Preserve existing persona greeting by falling back to speech synthesis or adding persona-specific files later. |
| GRADE_KINDERGARTEN_01 | Kindergarten grade button inline handler | Interrupt | User click | Later helper can play this before routing to `elaHub` or `activity`. |
| GRADE_FIRST_01 | 1st grade button inline handler | Interrupt | User click | Same as Kindergarten. |
| GRADE_SECOND_01 | 2nd grade button inline handler | Interrupt | User click | Same as Kindergarten. |
| ELA_CATEGORY_PROMPT_01 | `renderElaHub()` | Interrupt route audio | Automatic after click-unlocked | MP3 equivalent of `speak(t.elaHubTitle)`. |
| ELA_LETTERS_INTRO_01 | Letters category button inline handler | Interrupt | User click | Play before `state.elaCategory` changes or just after route. |
| ELA_MATCH_INTRO_01 | Match category button inline handler | Interrupt | User click | Play before item audio only if it does not make the first item feel slow. |
| ELA_BLEND_INTRO_01 | Blend category button inline handler or first blend `renderActivity()` | Interrupt | User click or automatic after category click | Mirrors existing blend instruction. |
| ELA_SENTENCES_INTRO_01 | Sentences category button inline handler or first sentence `renderActivity()` | Interrupt | User click or automatic after category click | Mirrors existing sentence instruction. |
| ELA_SIGHT_INTRO_01 | Sight category button inline handler | Interrupt | User click | Sight uses flashcard renderer; item audio remains dynamic. |
| ACTIVITY_INTRO_FLASHCARD_01 | `renderActivity()` flashcard branch | Wait or skip | Automatic after click-unlocked | Use only on first flashcard in a round; otherwise use item-specific `data.audio`. |
| ACTIVITY_INTRO_QUIZ_01 | `renderActivity()` quiz branch | Wait or skip | Automatic after click-unlocked | Optional before `data.audio`; likely first quiz only. |
| ACTIVITY_INTRO_BLEND_01 | `renderActivity()` blend branch | Interrupt | Automatic after click-unlocked | MP3 equivalent of `speak("Tap the sounds, then read the word!")`. |
| ACTIVITY_INTRO_SENTENCE_01 | `renderActivity()` sentence branch | Interrupt | Automatic after click-unlocked | MP3 equivalent of current sentence instruction. |
| ACTIVITY_INTRO_MATH_01 | `renderActivity()` when `state.subject === 'math'` | Wait or skip | Automatic after grade click | Optional first-item-only subject intro. |
| ACTIVITY_INTRO_SCIENCE_01 | `renderActivity()` when `state.subject === 'science'` | Wait or skip | Automatic after grade click | Optional first-item-only subject intro. |
| CURRICULUM_ITEM_AUDIO_DYNAMIC | Flashcard card/listen, quiz card/load, sentence listen | Interrupt | Automatic on item load and user click on repeat/listen | Derive per-item MP3 later from `data.audio`; keep `speak(data.audio)` as fallback when MP3 is missing. |
| BLEND_SOUND_DYNAMIC | Blend sound button handlers | Interrupt | User click | Derive from `data.phonemes[i]`; do not queue, because phoneme taps should feel immediate. |
| BLEND_WORD_DYNAMIC | Blend word listen handler | Interrupt | User click | Derive from `data.item`; keep speech synthesis fallback. |
| SENTENCE_WORD_DYNAMIC | Sentence word button handlers | Interrupt | User click | Derive from cleaned word; keep speech synthesis fallback for any missing file. |
| FLASHCARD_LISTEN_HINT_01 | Flashcard branch | Wait | Automatic first-use only | Optional tutorial hint; should not play on every card. |
| QUIZ_REPEAT_HINT_01 | Quiz card repeat handler | Wait | User click | Optional if the child taps the question card after initial playback. |
| CORRECT_PRAISE_01 | `showFeedback(true)` | Interrupt | User click result | Randomize within correct praise lines instead of current `feedbackPool.good` when MP3s exist. |
| CORRECT_PRAISE_02 | `showFeedback(true)` | Interrupt | User click result | Good when star count increments. |
| CORRECT_PRAISE_03 | `showFeedback(true)` | Interrupt | User click result | Prefer for ELA if later subject-aware praise is added. |
| CORRECT_PRAISE_04 | `showFeedback(true)` | Interrupt | User click result | All-subject fallback. |
| WRONG_ENCOURAGE_01 | `showFeedback(false)` | Interrupt | User click result | Randomize within wrong-answer lines; never auto-advance. |
| WRONG_ENCOURAGE_02 | `showFeedback(false)` | Interrupt, then optionally replay item | User click result | Good candidate to follow with current item audio after a short delay. |
| WRONG_ENCOURAGE_03 | `showFeedback(false)` | Interrupt | User click result | Non-shaming retry. |
| WRONG_ENCOURAGE_04 | `showFeedback(false)` | Interrupt | User click result | Short retry. |
| NEXT_ACTIVITY_01 | `nextActivity()` non-wrapped path | Interrupt if user tapped Next | User click or correct-answer timeout | On correct-answer timeout, consider skipping to avoid praise overlap. |
| NEXT_ACTIVITY_02 | `nextActivity()` non-wrapped path | Interrupt if user tapped Next | User click or correct-answer timeout | Alternate transition line. |
| ROUND_COMPLETE_01 | `nextActivity()` wrapped path before/with toast | Interrupt | User click or correct-answer timeout | Matches current round-complete toast. |
| PROGRESS_OPEN_01 | `renderProgress()` | Interrupt route audio | Automatic after click-unlocked | Play before or instead of star-count dynamic line. |
| PROGRESS_EMPTY_01 | `renderProgress()` when `currentCount === 0` | Interrupt | Automatic after click-unlocked | Use instead of `PROGRESS_STARS_DYNAMIC` when no stars exist. |
| PROGRESS_STARS_DYNAMIC | `renderProgress()` | Wait after intro | Automatic after click-unlocked | Later generate numeric variants or preserve current `speak(`${currentCount} stars!`)`. |
| LANGUAGE_SWITCH_TO_SPANISH_01 | `toggleLanguage(forceLang)` after `state.lang` update | Interrupt | User click | Do not play if Spanish subject selection immediately plays `SUBJECT_SPANISH_01`. |
| LANGUAGE_SWITCH_TO_ENGLISH_01 | `toggleLanguage(forceLang)` after `state.lang` update | Interrupt | User click | English-only target line; `es` is intentionally `null` in the map. |
| BACK_TO_HUB_01 | Header dog and Back controls that call `navigate('hub')` | Interrupt | User click | Optional; use sparingly to avoid noisy navigation. |
| MORE_ACTIVITIES_SOON_01 | `renderActivity()` missing dataset fallback | Interrupt | Automatic after click-unlocked | Current fallback is only visible text. |
| IMAGE_UNAVAILABLE_01 | `loadPrincessImage()` after local and Base64 image fail | Wait or skip | Automatic only after click-unlocked | Non-critical; should not cover lesson audio. |
| SESSION_CELEBRATION_01 | Future streak/star milestone | Interrupt after feedback | Automatic after correct answer | Not wired now; reserved for later star/streak logic. |

## Autoplay and queueing guidance

- MP3 playback on first load will be blocked by most browsers unless started after a user gesture.
- Keep `unlockAudio()` as the first trusted gesture hook, but do not remove the current speech synthesis priming until MP3 behavior is tested on iPad/Safari.
- Use immediate interrupting playback for user-tapped listen, sound, word, answer, and navigation choices.
- Use wait/queue behavior for optional intros and hints, especially before curriculum item audio.
- Avoid playing both a category intro and the first item audio if the combined result feels slow for a child.

## Preserving `speechSynthesis` during later wiring

1. Add a small MP3 helper that attempts to play a Voice ID or resolved curriculum file.
2. If the MP3 is missing, blocked, or errors, call the existing `speak()` with the same text the app already uses.
3. Do not remove `speak()`, `unlockAudio()`, `availableVoices`, `window.currentUtterance`, or `speechSynthesis.onvoiceschanged`.
4. Do not change curriculum `audio` values during MP3 wiring; treat them as the source text for dynamic recordings.
5. Keep `navigate()` canceling current `speechSynthesis`; later add equivalent MP3 cancellation in the same place.
6. Test Safari/iPad behavior separately because the existing code was designed around gesture-gated audio.
