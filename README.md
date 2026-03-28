# Chess Adventure 🐴

An interactive chess learning app for kids — built as a single HTML file with no dependencies.

## Features

- **12 guided lessons** — pieces, movement, check, checkmate
- **Free play & vs Computer** modes
- **Move confirmation** — pieces show a ghost preview with a ✓ button before committing
- **Talk to Agent** — speak a move (e.g. "e2 to e4") and get an evaluation read back
- **Cell coordinates** — toggleable square labels (a1–h8) on the board
- **Move narration** — the app speaks tips, threats, and analysis after each move
- **Hint system** — suggests the best available move with an arrow
- **Pawn promotion**, **en passant**, **castling** all supported

## How to Run

Just open `chess.html` in any modern browser. No server, no install.

```
double-click chess.html
```

Works best in **Chrome** or **Edge** (required for the Talk-to-Agent microphone feature).

## Controls

| Button | Action |
|--------|--------|
| 🎤 | Talk to Agent — speak a move or ask for a hint |
| 🔢 | Toggle square coordinate labels |
| 💡 | Show a hint arrow |
| 🔊 | Toggle voice narration |
| New Game | Reset the board |

### Talking to the Agent
Click 🎤 then say:
- `"e2 to e4"` — evaluates that move
- `"e4"` — finds which piece(s) can go there
- `"hint"` / `"what should I do?"` — triggers the hint
- `"new game"` — resets the board

### Move Confirmation
1. Click a piece → green dots show legal moves
2. Click a destination → piece **ghosts** to that square with a **✓ Move?** button
3. Click **✓ Move?** to confirm, or click anywhere else to cancel and hear an evaluation

---

## Adding Your Own Voice

The app uses the browser's built-in **Web Speech Synthesis** API. There are two ways to replace it with your own voice.

---

### Option A — ElevenLabs Voice Clone (Recommended, easiest)

ElevenLabs lets you clone your voice from a 1–5 minute recording.

**Step 1 — Clone your voice**
1. Go to [elevenlabs.io](https://elevenlabs.io) and create a free account
2. Go to **Voices → Add a new voice → Instant Voice Cloning**
3. Record or upload 1–3 minutes of yourself speaking clearly
4. Name the voice (e.g. "My Chess Voice") and save it
5. Copy the **Voice ID** from the voice card

**Step 2 — Get an API key**
1. Go to your ElevenLabs profile → **API Keys**
2. Create a key and copy it

**Step 3 — Swap the `speak()` function in chess.html**

Find this function in `chess.html`:
```javascript
function speak(text){
  if(!G.soundOn)return;
  if(synth.speaking)synth.cancel();
  const u=new SpeechSynthesisUtterance(text);
  ...
  synth.speak(u);
}
```

Replace it with:
```javascript
const ELEVEN_API_KEY = 'YOUR_API_KEY_HERE';
const ELEVEN_VOICE_ID = 'YOUR_VOICE_ID_HERE';

async function speak(text){
  if(!G.soundOn)return;
  try{
    const res = await fetch(`https://api.elevenlabs.io/v1/text-to-speech/${ELEVEN_VOICE_ID}`,{
      method:'POST',
      headers:{
        'xi-api-key': ELEVEN_API_KEY,
        'Content-Type':'application/json'
      },
      body: JSON.stringify({
        text,
        model_id:'eleven_turbo_v2',
        voice_settings:{stability:0.5, similarity_boost:0.8}
      })
    });
    const blob = await res.blob();
    const url = URL.createObjectURL(blob);
    const audio = new Audio(url);
    audio.play();
    audio.onended = () => URL.revokeObjectURL(url);
  }catch(e){ console.error('ElevenLabs error',e); }
}
```

> **Free tier:** 10,000 characters/month — plenty for casual use.

---

### Option B — Pre-recorded Audio Files (No API, works offline)

Record yourself saying each phrase, save as MP3 files, and map them in the code.

**Step 1 — Record your audio**

Use any recorder (your phone, Audacity, Voice Memos on Mac). Record each phrase as a separate file:

| Filename | Say this |
|----------|----------|
| `audio/select.mp3` | (a short confirmation sound or "okay") |
| `audio/check.mp3` | "Check!" |
| `audio/welcome.mp3` | "Welcome to Chess Adventure! I am Knight the horse! ..." |
| ... | one file per `speak(...)` call in the code |

**Step 2 — Create an audio map**

Add this near the top of the `<script>` in chess.html:
```javascript
const AUDIO = {
  'Check!': 'audio/check.mp3',
  'Sound is on!': 'audio/sound_on.mp3',
  // add more mappings here
};

function speak(text){
  if(!G.soundOn)return;
  if(AUDIO[text]){
    new Audio(AUDIO[text]).play();
  } else {
    // fallback to browser voice for unmapped phrases
    const u = new SpeechSynthesisUtterance(text);
    synth.speak(u);
  }
}
```

> This is best for fixed short phrases. For dynamic text (move narration) Option A works much better.

---

### Option C — Browser Voice Selection (No recording needed)

If you just want a better-sounding built-in voice, you can pick a specific system voice by name.

In the existing `speak()` function, change the voice selection line:
```javascript
// Current:
const nice=voices.find(v=>v.name.includes('Samantha')||...);

// Change to your preferred voice name — run this in the browser console to see all available:
// window.speechSynthesis.getVoices().forEach(v => console.log(v.name))
const nice=voices.find(v=>v.name==='NAME_FROM_CONSOLE');
```

Common good voices by OS:
- **Windows:** `Microsoft Zira` · `Microsoft David` · `Microsoft Aria Online`
- **Mac:** `Samantha` · `Karen` · `Daniel`
- **Chrome (any OS):** `Google US English` · `Google UK English Female`
