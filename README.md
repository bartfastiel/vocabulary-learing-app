# Vocabulary Learning App

A modular, framework-free vocabulary trainer built with native Web Components. No build step, no dependencies — just HTML, CSS, and ES Modules served from a static file server.

---

## Features

- **Multi-mode quiz** — 10 question/answer combinations (text, image, audio)
- **Mini-games** — 8 games to spend and earn points
- **Multi-profile system** — independent progress, avatar, and settings per user
- **Avatar builder** — fully custom avatars with layered inline SVG artwork
- **Background themes** — 8 selectable color themes
- **Points & streaks** — accumulated score with all-time streak record
- **Custom vocabulary editor** — add your own lesson sets in the browser
- **Onboarding tutorial** — first-run help overlay

---

## Running Locally

No build step needed. Serve from the project root (required for ES module imports and `fetch` calls):

```sh
# Python
python -m http.server 8080

# Node
npx serve .
```

Then open `http://localhost:8080`.

---

## Architecture

**No frameworks, no build step, no global stylesheets.** Every component is a native Web Component with Shadow DOM and scoped CSS.

### Entry point

`index.html` loads `core/app-shell.js` as the sole `<script type="module">`. Everything else is imported transitively.

### Component tree

```
<app-shell>               core/app-shell.js        — root; profiles, themes, points display
  <vocab-help>            core/help-overlay.js      — onboarding tutorial
  <avatar-builder>        core/avatar-builder.js    — avatar creator
  <vocab-editor>          vocab/vocab-editor.js     — custom lesson editor
  <vocab-trainer>         vocab/vocab.js            — quiz orchestrator
    <vocab-question-*>    vocab/question/
    <vocab-answer-*>      vocab/answer/
  <game-lobby>            game/game-lobby.js        — game launcher
    <rocket-game>         game/rocket-game.js
    <flappy-game>         game/flappy-game.js
    … (8 games total)
```

### Shared module

`core/audio.js` is the only shared module used across components:

```js
playSound("ding" | "buzz")   // feedback sounds
playVoice(englishWord)        // plays a random pre-generated voice clip
```

### Data flow

1. `vocab-trainer` fetches `vocab/vocab.json` and picks a random word.
2. It selects a random `{ question, answer }` pair from the static `MODES` array. Image modes are filtered out if `word.allowImage` is false; multiple-choice modes are filtered if the lesson has fewer than 4 words.
3. It creates the question and answer components dynamically, sets `.word` on the question and `.data` on the answer.
4. The answer component fires `CustomEvent("answered", { detail: { correct } })`.
5. `vocab-trainer` advances to the next round and delegates score changes to the `PointsManager` instance passed in via `trainer.points`.

---

## Folder Structure

```
vocabulary-learning-app/
├── index.html
├── core/
│   ├── app-shell.js          # root component; profiles, themes, UI chrome
│   ├── audio.js              # shared sound + voice playback
│   ├── avatar-builder.js     # avatar editor (all artwork inline SVG)
│   ├── help-overlay.js       # onboarding tutorial overlay
│   └── profiles.js           # profile CRUD + localStorage snapshots
├── vocab/
│   ├── vocab.js              # orchestrator: modes, flow, component wiring
│   ├── vocab.json            # built-in lessons
│   ├── vocab-editor.js       # custom lesson editor component
│   ├── points.js             # PointsManager: points, streaks, highscores
│   ├── question/
│   │   ├── question-wordgerman.js
│   │   ├── question-wordenglish.js
│   │   ├── question-voiceenglish.js
│   │   └── question-image.js
│   └── answer/
│       ├── answer-choosewordenglish.js
│       ├── answer-choosewordgerman.js
│       ├── answer-choosevoiceenglish.js
│       ├── answer-typewordenglish.js
│       ├── answer-chooseimage.js
│       └── elements/
│           └── next-button.js
├── game/
│   ├── game-lobby.js         # game launcher (lists all games)
│   ├── rocket-game.js        # coin shooter (spend points, no earn)
│   ├── flappy-game.js        # flappy bird clone (earn up to 20 pts)
│   ├── reaction-game.js      # hit targets (earn up to 10 pts)
│   ├── memory-game.js        # emoji pair matching (earn up to 12 pts)
│   ├── jump-game.js          # endless runner (earn up to 15 pts)
│   ├── snake-game.js         # classic snake (earn up to 15 pts)
│   ├── breakout-game.js      # block breaker (earn up to 15 pts)
│   └── catcher-game.js       # catch stars, dodge bombs (costs 2, earn up to 10 pts)
└── assets/
    ├── audio/
    │   ├── ding.mp3           # correct-answer sound
    │   ├── buzz.mp3           # wrong-answer sound
    │   └── voice/             # per-word TTS clips
    │       └── <word>_<voice>.mp3
    └── img/                   # vocabulary images (DALL·E)
```

---

## Quiz Modes

All 10 active question/answer combinations:

| Question | Answer |
|---|---|
| German word | Choose English word |
| German word | Type English word |
| English word | Choose German word |
| English word | Choose image |
| English audio | Choose English word |
| English audio | Type English word |
| English audio | Choose audio (compare voices) |
| English audio | Choose image |
| Image | Choose English word |
| Image | Type English word |

---

## Mini-Games

Games are unlocked after earning at least 1 point. Each game costs 1 point to play (Catcher costs 2).

| Game | Max earn |
|---|---|
| Rocket | — (spend only) |
| Flappy Bird | 20 pts |
| Reaction | 10 pts |
| Memory | 12 pts |
| Endless Run | 15 pts |
| Snake | 15 pts |
| Breakout | 15 pts |
| Catcher | 10 pts |

---

## Profile System

Each profile stores independently:
- Points total and all-time streak record
- Avatar design and unlock status
- Background theme
- User role (`student` / `teacher` / `developer`)
- Custom vocabulary sets and game highscores

**Roles:**
- **Student** — default; full access to quiz and games
- **Teacher** — quiz only; games disabled
- **Developer** — unlimited points (shown as ∞); for testing

Switch profiles by clicking the avatar in the top-left corner.

---

## Background Themes

8 themes selectable per profile: `dark` (default), `ocean`, `purple`, `forest`, `sunset`, `rose`, `gold`, `ice`.

---

## vocab.json Structure

Each top-level entry is a lesson:

```json
{ "name": "Lesson 1", "words": [
    { "de": "der Hund", "en": "dog", "allowImage": true, "en_info": "also: canine" }
]}
```

| Field | Required | Description |
|---|---|---|
| `de` | yes | German word/phrase |
| `en` | yes | English translation |
| `allowImage` | no | Enable image-based modes for this word |
| `en_info` | no | Extra hint shown below the question |

Users can also add custom lessons via the in-app editor (✏️ button).

---

## Adding a New Question/Answer Mode

1. Create `vocab/question/question-<name>.js` — a Web Component with a `set word(vocab)` setter that renders the prompt.
2. Create `vocab/answer/answer-<name>.js` — a Web Component with a `set data({word, vocabulary, …})` setter that renders choices and dispatches `CustomEvent("answered", { detail: { correct } })` on interaction.
3. Import both at the top of `vocab/vocab.js`.
4. Add one or more `{ question, answer }` entries to the `MODES` array in `vocab/vocab.js`.

---

## Adding Avatar Artwork

All artwork is inline SVG inside `core/avatar-builder.js`. Layers (bottom → top): `background`, `face`, `hair`, `eyes`, `mouth`, `glasses`, `accessory`. Index 0 of `glasses` and `accessory` means "none".

Add an entry to the relevant array in the `LAYERS` constant:

```js
{ label: "My Style", svg: `<rect .../>` }
// Optional: { label: "Premium", cost: 1, svg: `...` }  — costs 1 point to unlock
```

No other changes needed.

---

## Audio Assets

Voice clips are pre-generated with OpenAI TTS. Filename convention:

```
assets/audio/voice/<normalized_word>_<voice>.mp3
```

where the word is lowercased with non-alphanumeric characters replaced by `_`, and voice is one of `alloy`, `ash`, `coral`, `nova`, `onyx`.

Generation scripts use an API key from `env.sh` (see `env.sh.template`). Logs are stored in `log/` (gitignored).

---

## LocalStorage Keys

| Key | Description |
|---|---|
| `allProfiles` | JSON array of all profile objects |
| `activeProfileId` | ID of the currently active profile |
| `points` | Current point total |
| `streakRecord` | All-time best streak |
| `userRole` | Current user role |
| `appBg` | Selected theme key |
| `avatarSelection` | `{ background, face, hair, eyes, mouth, glasses, accessory }` indices |
| `avatarUnlocked` | Array of unlocked premium avatar item indices |
| `vocabHelpSeen` | Set after onboarding tutorial is dismissed |
| `customVocab` | JSON array of user-created lesson sets |
| `gameHighscores` | JSON object with per-game highscores |

---

## Code Style Notes

- **Shadow DOM everywhere** — each component owns its HTML, CSS, and JS
- **No global state** — components communicate via properties, setters, and custom events
- **Inline CSS** — styles live inside each component's `shadowRoot.innerHTML`
- **Explicit imports** — all modules are listed explicitly; no dynamic discovery
- **Plain vanilla JS** — no TypeScript, no JSX, no transpilation

---

## Attributions

- `buzz.mp3` — LorenzoTheGreat (CC BY 3.0)
- `ding.mp3` — timgormly (CC0)
- Images in `assets/img/` — DALL·E
- Voice clips — OpenAI Text-to-Speech
