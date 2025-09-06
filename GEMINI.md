# Talk Extension — Hands-free Voice with Gemini-CLI

This extension adds a Talk MCP server and a set of `/talk…` commands for **voice → transcript → assistant reply → TTS playback**. It’s designed for accessibility and fully hands-free workflows.

## Commands

- **/talk** — quick help (this doc in short form)
- **/talk:devices** — list input devices (with indexes you can select)
- **/talk:start [key=value …]** — start the capture/transcribe loop
- **/talk:status** — show loop state (running? device? last transcript?)
- **/talk:stop** — stop the loop
- **/talk:tts [voice=… model=… fmt=wav|mp3]** — synthesize the **last assistant reply** to audio (returns a path you can play)

### Common /talk:start keys

- `device_index` (int) — pick a mic from `/talk:devices`
- `language=…` — e.g. `en`, `fr`, `es`, `de`, `pt`, `ar`, `hi`, `zh`, `ja`
- `energy_gate` (float) — VAD threshold; higher = stricter start
- `end_sil_ms` (int) — trailing silence to end an utterance (e.g., `600`)
- `max_utter_ms` (int) — hard stop per utterance (e.g., `12000`)
- `model` — Whisper model (`tiny|base|small|medium|large` or CLI alias)
- `blocking` (bool) — keep the tool running until stop; usually `false`
- `delete_after` (bool) — remove temp audio after transcription
- `whisper_bin` — path to whisper CLI if using CLI mode
- `whisper_venv` — venv path for whisper CLI
- `poll_ms` (int) — internal loop polling interval

> Exact names may vary slightly with your MCP; these reflect the versions we’ve iterated on.

## Quick Start

1. `/talk:devices`  
2. `/talk:start device_index=0 language=en end_sil_ms=600 max_utter_ms=12000`  
3. Speak. The assistant will reply in text.  
4. Optional TTS: `/talk:tts voice=alloy fmt=mp3` → returns an audio file path.  
5. `/talk:stop`

## Multilingual Flow

- Input: choose `language=…` on `/talk:start`
- Output: pick a TTS voice/model:  
  `/talk:tts voice=alloy model=gpt-4o-mini-tts fmt=wav`

## Tips & Behavior

- The capture loop auto-segments speech using **energy gate** + **silence tail**.  
- The assistant should **reply concisely** after each utterance.  
- If TTS is available, `/talk:tts` will write an audio file (absolute path).  
- Your MCP should **log to STDERR only**; stdout is reserved for JSON-RPC.

## Platform Notes

- **macOS**: grant Microphone permission for your terminal (Settings → Privacy & Security → Microphone). For Bluetooth mics, check the device sample rate in Audio MIDI Setup.
- **Linux**: ensure PortAudio/ALSA access; user may need to be in `audio` group. On PipeWire, confirm the correct default source.
- **Windows**: choose `device_index` explicitly; verify the mic is “Default Communications Device”.

## Troubleshooting

- **No devices listed** → check mic permissions; try restarting the terminal after granting access.  
- **Hot mic / no segmentation** → raise `energy_gate` or `end_sil_ms`.  
- **Chops off words** → lower `energy_gate`, raise `end_sil_ms`, raise `max_utter_ms`.  
- **“Connection closed” on startup** → missing Python deps in the extension venv; (re)run your bootstrap script to install `pyaudio` (or platform alt), `whisper`/bindings, and TTS deps.  
- **JSON parse errors** → some tool printed to **stdout**. Ensure all logging uses **stderr**.

## Safety

- No shell is executed by these commands.  
- Audio files are stored in a temp folder; set `delete_after=true` if you want them removed automatically.

## Examples

- Pick a different mic and Spanish:
