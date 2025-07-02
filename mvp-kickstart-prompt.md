🛠️ Claude Prompt: MVP Voice-to-LLM Offload System

You are an autonomous CLI assistant responsible for implementing a minimal, production-ready MVP for a voice-to-LLM offload system. You will create a fully functional, Dockerized Python project that meets the following requirements.

---

# 🎯 GOAL

Build a system that:
- Listens continuously for speech using the DJI Mic Mini (USB audio input)
- Detects when real voice activity starts/stops using VAD (voice activity detection)
- When speech ends, transcribes the spoken input using `faster-whisper`
- Routes the transcribed message to a Gemini-compatible or OpenAI-compatible LLM
- Based on LLM response, stores a structured result in a local queue (JSON) and a human-readable log (Markdown)
- Handles errors gracefully and logs them to a separate file
- Dockerized for deployment and future integration into a Palantir OSDK compute module

---

# 📁 DIRECTORY STRUCTURE

Create the following:

voice_offload_mvp/
├── Dockerfile
├── requirements.txt
├── main.py
├── config.yaml
├── data/
│   ├── task_queue.json
│   ├── inbox.md
│   └── error.log
├── logs/
│   └── debug.log
├── models/
│   └── whisper model cache here (optional .gitignore)
├── utils/
│   ├── audio_capture.py
│   ├── transcription.py
│   ├── llm_router.py
│   ├── storage.py
│   └── logger.py
├── test/
│   └── sample_audio.wav (optional)
└── README.md

---

# 🔧 IMPLEMENTATION DETAILS

## 1. **Audio Input + Voice Detection**
- Use `pyaudio` to stream from the default system microphone (DJI Mic Mini)
- Use `webrtcvad` to detect voice activity
- When activity is detected, buffer audio until silence (≥1.5s)
- Save audio as `.wav` in a temp location

## 2. **Transcription**
- Use `faster-whisper` with a local model (small or base)
- Output plain text transcript from audio

## 3. **LLM Routing**
- Send transcript to an LLM API (preferably Gemini, fallback OpenAI)
- Prompt the LLM to classify the note into one of:
  - `todo` → store as task
  - `question` → flag for review
  - `reminder` → time-based
  - `unclassified` → needs manual triage
- Structure response as JSON

## 4. **Storage & Logs**
- Append structured task to `data/task_queue.json`
- Append human-readable entry to `data/inbox.md`
- Log errors to `data/error.log`
- Log debug steps to `logs/debug.log`

## 5. **Dockerization**
- Base image: `python:3.11-slim`
- Install `ffmpeg`, `faster-whisper`, `pyaudio`, and required deps
- Expose `/data` volume for persistent storage

---

# 🧪 TESTING
After setup, simulate with test audio file (`test/sample_audio.wav`) to verify:
- VAD correctly detects speech
- Whisper transcribes accurately
- LLM call succeeds
- Output saved to `.json` and `.md`
- Errors (e.g., bad audio) are gracefully handled

---

# 🛠️ CONFIGURATION

Create a `config.yaml` file with:
```yaml
vad:
  aggressiveness: 2
  silence_threshold: 1.5  # seconds

llm:
  provider: "gemini"  # or "openai"
  model: "gemini-pro"
  api_key: "ENV_GEMINI_API_KEY"

output:
  queue_path: "./data/task_queue.json"
  log_path: "./data/inbox.md"
  error_log: "./data/error.log"


⸻

✅ DONE STATE

After successful setup:
	•	I should be able to run:

docker build -t voice-offload .
docker run -v $(pwd)/data:/app/data voice-offload


	•	Speak into my DJI Mic Mini
	•	See new structured entries in data/task_queue.json and inbox.md
	•	All actions logged and traceable

⸻

💡 NOTES
	•	Don’t use a wake-word, just rely on continuous VAD
	•	Optimize for pleasant UX: no clunky interfaces, just speak and it logs
	•	Structure for future compute module integration (clean code, modular utils)
	•	Auto-download Whisper model on first run if missing
	•	Include README with setup, usage, config override instructions

⸻

Begin now. Self-verify each component before moving to the next. If any step fails (e.g., audio input not available in Docker), suggest fallback strategies. Your final output should be a fully working local prototype.
