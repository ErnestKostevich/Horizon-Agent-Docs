---
title: Voice and wake word
description: Hands-free Horizon — wake words per persona, four TTS providers, continuous Talk Mode, and troubleshooting.
lastUpdated: 2026-05-20
---

# Voice and wake word

Horizon listens for its name in the background. Say "Horizon" (or
"Джарвис" if your active persona is Jarvis), wait for the chime, then
talk normally — it transcribes the rest as a command. Reply audio comes
back through your speakers using whichever TTS engine you've configured.

The whole pipeline is local except for the transcription call (Groq
Whisper or Deepgram) and the optional cloud TTS (ElevenLabs / OpenAI).
The wake-word matcher itself runs entirely in the renderer with no API
involvement.

## Required setup

You need three things before voice works:

1. **Microphone permission** — granted on first launch via the OS
   permission prompt. On macOS, also enable Horizon under System
   Settings → Privacy → Microphone.
2. **A transcription API key** — Groq is the cheapest and recommended:
   sign up at `console.groq.com`, copy the key into Settings →
   Connections → Groq, or run
   ```bash
   horizon connect provider --groq <KEY>
   ```
   Deepgram works too if you already have a key.
3. **(Optional) A TTS provider key** — ElevenLabs for the best voices,
   OpenAI for the cheapest, or none at all (system TTS is free).

Once those are in place, voice features unlock in the GUI. Toggle Wake
Mode with the microphone chip in the composer bar.

> **Why Groq instead of Web Speech API?** Google's Web Speech API throws
> a `network error` inside Electron — a confirmed upstream bug that
> hasn't been fixed in years. Horizon uses MediaRecorder + a Whisper
> endpoint instead, which works reliably across all three platforms.

## Wake words per persona

Horizon ships with one wake-word set, but the post-wake greeting depends
on which persona is active. The matcher accepts the persona's name plus
common mistranscriptions (Whisper is good but not perfect on a single
word).

| Persona | Listens for | First reply (EN) | First reply (RU) |
|---|---|---|---|
| Jarvis | "Horizon", "Jarvis" | At your service, Sir. | К вашим услугам, Сэр. |
| Friday | "Horizon" | Hey! Listening! | Привет! Слушаю! |
| Alfred | "Horizon" | How may I assist? | Чем могу быть полезен? |
| Sage | "Horizon" | Speak, I listen. | Говори, я внемлю. |
| Pixel | "Horizon" | Yoo, here I am! | Йоу, тут я! |

The greeting is picked at random from a list of four to five per persona.
See [Personas](../concepts/personas.md) for the full set.

### Recognised variants

The matcher is intentionally generous because Whisper sometimes
mistranscribes the wake word:

```
horizon · horizan · horison · jarvis ·
hey horizon · ok horizon · hello horizon ·
горизонт · хорайзон · харизон · джарвис ·
эй горизонт · окей горизонт
```

Fuzzy matching (Levenshtein distance ≤ 2) catches near-misses like
"horizen" or "горизонти".

## Continuous Talk Mode

In Talk Mode the wake word is only needed once. After the agent finishes
replying, the microphone reopens automatically for ~10 s — keep talking
and Horizon keeps responding, no re-trigger required. Stay silent and it
falls back to wake-word listening.

```bash
horizon set wakeTalkMode true
```

Or in the GUI: Settings → Voice → Continuous Talk Mode.

Combined with TTS, this makes Horizon feel like a phone call. Useful for
cooking, driving (with a phone mount and the PWA), or any flow where
your hands aren't free.

## Press-to-talk

If you don't want background listening, two manual modes are always
available:

- **Voice button** — bottom of the composer. Click once to start
  recording, click again to stop and send. The transcription appears
  in the input, edit if needed, hit Enter.
- **Hold-to-Dictate** — hold the dictate button; release to insert the
  transcription at the cursor without sending. Same UX as macOS
  dictation.

Both use the same Groq pipeline as the wake-word loop.

## TTS providers

Horizon supports four TTS backends. Pick one in Settings → Voice → TTS
Provider; you can change at any time without restart.

### System TTS (default)

Free, offline, instant. Uses the OS's built-in speech synthesis —
SAPI on Windows, NSSpeechSynthesizer on macOS, espeak-ng on Linux.
Quality varies by OS; macOS is the best of the three out of the box.

No setup required.

### ElevenLabs

Highest quality, paid, ~50–200 ms latency. Best for production demos
and accessibility.

```bash
horizon connect provider --elevenlabs <KEY>
```

Default voice id is `pNInz6obpgDQGcFmaJgB` ("Adam"). Override with
`elevenLabsVoice` in settings.

### OpenAI TTS

Good quality, cheap, ~300–800 ms latency. The `onyx` voice is the
default; `nova`, `shimmer`, `echo`, `fable`, `alloy` are the
alternatives.

```bash
horizon connect provider --openai <KEY>
```

(If you already use OpenAI as your chat provider the same key works.)

### Kokoro (local)

Free, offline, runs entirely in the browser via `kokoro.js`. Loads
from CDN on first use (~100 MB cache). Quality is between system TTS
and OpenAI, latency is ~500 ms once the model is loaded.

Enable: Settings → Voice → TTS Provider → Kokoro. No key needed.

## Per-persona voice preferences

You can override the TTS provider and voice on a per-persona basis. The
Inspector → Persona panel exposes:

- `voicePreset` — ElevenLabs voice id, OpenAI voice name, or system
  voice name
- `ttsProvider` — `elevenlabs | openai | system | kokoro`

For example, set Jarvis to a deep ElevenLabs voice and Pixel to OpenAI's
`shimmer`. The active persona's preferences override the global default.

## Echo prevention

When Horizon is speaking through your laptop speakers, the microphone
hears its own voice and would otherwise trigger the wake word again —
the infamous "infinite loop where Alexa wakes up Alexa" problem.

Horizon avoids it with three measures:

1. **Wake pause during TTS** — while `isSpeaking` is true, the wake loop
   stops recording entirely. No audio in, no false trigger.
2. **Echo cooldown** — 1.5 s after TTS ends, the matcher still rejects
   matches. By then any speaker echo has dissipated.
3. **Hallucination filter** — Whisper occasionally invents words for
   silence ("thank you", "subscribe", "music playing"). Those are on a
   denylist and never count as wake.

If you use a headset, none of this matters because there's no acoustic
loop in the first place.

## Hands-free workflow example

You're cooking, your hands are wet:

```
You: "Horizon"
Horizon: "Yes, Sir?"
You: "How many grams of butter in a stick?"
Horizon: "113 grams, or 4 ounces. About half a cup."
[Talk Mode reopens the mic]
You: "What temperature do I roast brussels sprouts at?"
Horizon: "200°C / 400°F for 25 minutes. Toss halfway."
[10s of silence → Talk Mode ends, back to wake-word listening]
```

No clicks, no typing, no screen contact. Combine with the PWA on your
phone (`horizon serve --enable-pwa` → connect from phone) and your
hands really don't need to leave the dough.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Wake mode toggle does nothing | Groq key not set | Settings → Connections → Groq → paste key |
| "Wake mode requires a Groq key" toast | Same as above | Same as above |
| Mic icon greyed out | OS permission denied | macOS: System Settings → Privacy → Microphone → enable Horizon. Windows: Settings → Privacy → Microphone |
| Says "Heard: ..." but doesn't fire | Wake word wasn't the first word | Disable Strict Mode (Settings → Voice → Strict Mode) OR start your sentence with "Horizon ..." |
| Triggers on background TV | Volume threshold too low | Settings → Voice → Volume Threshold → raise to 12-15 |
| Misses soft "Horizon" | Volume threshold too high | Lower to 5-6 |
| Says "Listening..." but never replies | Transcription failing silently | Open DevTools → Console; look for `[wake] transcribe error`. Check Groq quota at console.groq.com |
| Whisper transcribes garbage | Mic gain too low | `horizon doctor --voice` → see peak dB. Should be 100-200 / 255 |
| Robotic / clipped TTS | System TTS voice not installed | Settings → Voice → TTS Provider → switch to ElevenLabs or OpenAI |
| Voice replies cut off | Reply > 500 chars (system TTS soft cap) | Switch to ElevenLabs / OpenAI (handle longer text) |
| Wake word fires from Horizon's own reply | Echo cancellation off | Use a headset, or Settings → Voice → Echo Cooldown → raise to 2500 |

### Diagnostic commands

```bash
# Test mic: records 3 s, plays back, prints peak dB
horizon doctor --voice

# Show last wake-word debug record (what Whisper heard, what matched)
horizon doctor --wake

# Force a transcription test
horizon transcribe ./sample.wav
```

The GUI Settings → Voice panel has both buttons too ("Test Microphone"
and "Test Wake Word"), with a coloured peak-volume meter so you can see
your mic in real time.

## Related

- [Personas](../concepts/personas.md) — wake responses and voice
  presets per persona
- [Computer use](./computer-use.md) — wake then "open Chrome and..."
- [CLI reference](../cli-reference.md) — `horizon transcribe`,
  `horizon doctor --voice`
