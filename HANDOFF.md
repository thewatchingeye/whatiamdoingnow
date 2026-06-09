# what i am doing now — Project Handoff
## For Claude Code / VS Code continuation

---

## What this is

A browser-based performance instrument for a piece by composer James Saunders (Bath Spa University). Players ask an AI conductor for instructions during a live musical/theatrical performance, complete them, and report back. The AI shapes instructions in response to what performers do, tracking the arc of the piece over time.

The piece is titled **"what i am doing now / James Saunders / 2026"**.

Intended save location: `/Users/j.saunders/Library/CloudStorage/OneDrive-BathSpaUniversity/Documents/Scores/__in progress/what i am doing now`

---

## Current state

A fully functional **standalone single-file HTML prototype** (`what-i-am-doing-now.html`) with:
- Full UI (setup screen + performance screen)
- Anthropic API integration (claude-opus-4-5) for AI conductor
- Web Speech API for continuous hands-free speech recognition (no button presses during performance)
- Browser speech synthesis (TTS) for spoken AI responses
- Six performance context presets (also as separate `.json` files)
- Nine voice styles
- Task completion tracking logic
- `localStorage` persistence for API key
- Multi-player architecture stubbed (room codes, player state object) — not yet networked

---

## File structure

```
what i am doing now/
  what-i-am-doing-now.html       ← main app (standalone, works offline)
  HANDOFF.md                     ← this file
  contexts/
    open-durational.json
    stillness.json
    everyday-actions.json
    task-resistance.json
    sound-listening.json
    collective.json
```

---

## Visual design spec

- **Background**: white (`#fff`)
- **Font**: DIN Alternate (`'DIN Alternate', 'DIN Alternative', 'DIN Next', 'Barlow', 'Helvetica Neue', Arial, sans-serif`)
- **Title** (right-aligned, three lines):
  - `what i am doing now` — `#F09837`
  - `/ James Saunders` — `#427386`
  - `/ 2026` — `#89AFBB`
- **Type weights**: 300 (body), 400 (labels/buttons)
- **Colour palette**: `--black: #111`, `--mid: #888`, `--light: #bbb`, `--lighter: #e8e8e8`, `--red: #d0392b`
- Setup screen: max-width 580px, centred
- Performance screen: full viewport, instruction text large and centred (`clamp(24px, 5vw, 44px)`), weight 300
- Status indicator: small animated dot below instruction (pulses when listening, spins when thinking, rapid pulse when speaking)
- Log strip at bottom of performance screen, max-height 200px

---

## Key technical decisions

### show/hide screens
Use `element.style.display = 'flex'` / `'none'` directly — **do not use classList.add/remove** with screen toggling. A previous bug (`classList.add('')`) crashed the app. The two screens are `#screen-setup` and `#screen-perf`.

### API calls
Direct browser-to-Anthropic calls using:
```javascript
fetch('https://api.anthropic.com/v1/messages', {
  headers: {
    'x-api-key': key,
    'anthropic-version': '2023-06-01',
    'anthropic-dangerous-direct-browser-access': 'true',  // required for browser
  }
})
```
Model: `claude-opus-4-5`, `max_tokens: 180`

### API key storage
Saved to `localStorage` under key `widn_api_key`. Loaded on init, with a "Forget key" button to clear it.

### Continuous listening loop
After each AI response is spoken, `startListeningLoop()` is called automatically. It opens a `SpeechRecognition` instance, waits for speech, sends the transcript to `callConductor()`, which calls the API, then calls `speakText()`, which when done calls `startListeningLoop()` again. The cycle is:

```
initialGreeting() → callConductor() → speakText() → startListeningLoop()
                         ↑                                    |
                         └────────────────────────────────────┘
```

Guard flags: `S.listening`, `S.speaking`, `S.thinking` — the loop checks all three before starting.

---

## The system prompt (performance logic)

The system prompt encodes four conversational states:

1. **New instruction** — when performer asks "what should I do now?" or any variation → give ONE instruction
2. **Completion report** — performer says what they did → acknowledge, optionally give new instruction or wait
3. **New request without completion** — performer asks for new instruction before reporting completion of previous → check in, negotiate (simplify, retry, find equivalent)
4. **Progress report** — performer says "what I am doing now is…" → advise on how to develop/deepen/complicate the current task, do NOT give a new instruction

The app also does client-side detection of completion reports and new-instruction requests to annotate messages before sending:
```javascript
const isAskingForNew = /what (should i|do i|shall i|can i|next|now)|(what next|give me|what do i do|...)/i.test(userMessage);
const isCompletionReport = /^(i (did|have|finished|completed|...))/i.test(userMessage.trim());
```

State tracked: `S.lastInstruction` (text of last instruction given), `S.lastInstructionCompleted` (boolean).

---

## Voice styles (9)

| Value | Description |
|-------|-------------|
| `poetic` | Oblique, evocative, image-based language |
| `terse` | ≤15 words, imperative only |
| `process` | Step-by-step, specific about body/space/time |
| `question` | Questions only, no direct commands |
| `objective` | Automated telephone menu tone, disinterested |
| `enthusiastic` | Relentlessly complimentary, supportive of everything |
| `critical` | Always finds fault, undermines confidence |
| `taskmaster` | Blunt, results-only, ≤20 words |
| `quirky` | Non-sequitur instructions, no rationale |

---

## Performance context files (JSON schema)

```json
{
  "title": "Human-readable name",
  "description": "One-sentence summary shown in UI",
  "context": "Full context text passed to AI system prompt",
  "suggestedVoice": "one of the 9 voice style values"
}
```

The six presets are also embedded in the HTML as the `PRESETS` array (so the file works standalone without a server). When a server is available, these should be fetched from `/contexts/*.json` instead.

---

## Conversation summary (what was built and when)

**Session 1 — initial build**
- Designed system from scratch: single-player prototype with multi-player architecture stubbed
- Core loop: player speaks → AI responds → spoken aloud → repeat
- Room code system (5-char alphanumeric) for future multi-player joining
- Initial dark aesthetic (black background)

**Session 2 — redesign**
- Complete visual redesign: white background, DIN Alternate font
- Hands-free continuous listening (no buttons during performance)
- Three-line right-aligned title with brand colours

**Session 3 — bug fix**
- Fixed `classList.add('')` crash (empty token in DOMTokenList)
- Replaced all show/hide class toggling with `style.display` manipulation

**Session 4 — title colours**
- Applied three-colour title: `#F09837` / `#427386` / `#89AFBB`
- Applied to both setup screen and performance bar

**Session 5 — performance logic + context files + voice styles**
- Encoded task-tracking behaviour into system prompt (4 states)
- Client-side completion/request detection with message annotation
- Added context file system: preset dropdown, file upload, manual entry tabs
- Six context `.json` files created
- Added 5 new voice styles (objective, enthusiastic, critical, taskmaster, quirky)

**Session 6 — API key persistence**
- `localStorage` saves key on successful test
- Auto-loads on page open
- "Forget key" button to clear

---

## Next steps / known gaps

- **Multi-player networking**: the `S.players` object and room code architecture are in place. Next step is a WebSocket relay server (Node.js + `ws`, or Supabase Realtime) so multiple devices on the same room code share state. Each client broadcasts `{room, name, lastAction}` on every completion; all clients listen and update their local `S.players`.
- **Server-side API proxy**: when hosted publicly, move the Anthropic API call to a server endpoint (`POST /api/conduct`) so the key is never exposed in the browser. The fetch in `callConductor()` would change from `https://api.anthropic.com/v1/messages` to `/api/conduct` and the key moves to a `.env` file.
- **Context files from server**: replace the embedded `PRESETS` array with `fetch('/contexts/index.json')` on load.
- **Better TTS**: the current implementation uses browser speech synthesis. OpenAI TTS or ElevenLabs would give better voice quality. The `speakText()` function is self-contained and can be swapped without touching anything else.
- **Mobile mic permissions**: iOS Safari requires the page to be served over HTTPS for microphone access. Fine for local dev but required for deployment.
- **Per-player context**: the room code architecture could allow each player to load a different context, or all share one set centrally.
