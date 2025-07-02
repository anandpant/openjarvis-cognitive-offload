You are Claude CLI and your mission is to autonomously create a fully self‑contained, Dockerized voice-to‑LLM assistant using the **OpenAI Agents Python SDK (voice workflow)**. You must build, test, and self‑verify a working MVP, with no manual intervention required after prompt initiation. Follow these detailed requirements precisely.

---

## 🧱 Project Setup

1. **Initialize project**  
   - Create root folder `voice_agent_mvp/`  
   - Initialize a Git repo inside it

2. **Dependencies**  
   - Ensure Python 3.11+  
   - Install required Python packages via `pip`:  
     ```
     openai-agents[voice]
     sounddevice
     numpy
     python-dotenv
     ```
   - Create `.env` file with `OPENAI_API_KEY=…`

---

## 🧠 Agent & Tools

Implement in `agent_workflow.py`:

- Define a Python function tool `@function_tool def add_task(task: str) -> dict:`  
  - Appends a new task record to `data/task_queue.json`  
  - Each record includes `"id"`, `"task"`, `"timestamp"`, `"status": "pending"`, `"source": "voice"`
  - Returns confirmation

- Define `@function_tool def ask_question(query: str) -> dict:`  
  - Logs the question in `data/task_queue.json` with `"type": "question"`  
  - Returns acknowledgment

- Create `Agent(name="Assistant", tools=[add_task, ask_question], model="gpt-4o", instructions="You are a voice agent that listens for tasks or questions and either creates tasks or logs questions.")`

---

## 🗣 Voice Pipeline

In `main.py`:

- Import:
  ```python
  from agents.voice import VoicePipeline, SingleAgentVoiceWorkflow, AudioInput
  from agent_workflow import Assistant
  import sounddevice as sd, numpy as np

	•	Configure VoicePipeline with SingleAgentVoiceWorkflow(Assistant) and default STT/TTS.
	•	Use sounddevice.InputStream to read from system mic (DJI Mic Mini), capture audio chunks until ~1.5s silence, concatenate into a numpy buffer.
	•	Wrap the buffer into AudioInput(buffer=..., frame_rate=..., channels=1), and call await pipeline.run(audio_input).
	•	Stream output: play TTS audio using sounddevice.OutputStream.
	•	Log lifecycle events and any VoiceStreamEventError to logs/debug.log.

⸻

📦 Storage & Logging
	•	Create folder data/ and logs/
	•	Initialize task_queue.json as an empty array []
	•	In main.py, after every agent turn, append JSON tasks/questions via tool calls.
	•	Log exceptions and errors in logs/debug.log.

⸻

🐳 Dockerization

Create Dockerfile:

FROM python:3.11-slim
RUN apt-get update && apt-get install -y ffmpeg libsndfile1
WORKDIR /app
COPY . .
RUN pip install --no-cache-dir openai-agents[voice] sounddevice numpy python-dotenv
ENV OPENAI_API_KEY=${OPENAI_API_KEY}
CMD ["python", "main.py"]

Ensure audio device access in container (mention --device /dev/snd in README).

⸻

🧪 Self‑Verification

At the end of the build process, run automated test:
	•	Play a 3‑sec .wav under test/sample.wav containing speech: simulate via AudioInput(buffer=…), bypassing mic.
	•	Verify that a task or question entry appears in task_queue.json.
	•	Assert that status: "pending" is set.
	•	Print test summary: “✅ Test passed — X entries added.”

⸻

✅ Success Criteria

On running:

docker build -t voice-agent .
docker run --env OPENAI_API_KEY=$KEY --device /dev/snd voice-agent

	•	You should speak into mic → agent classifies and logs tasks/questions to task_queue.json
	•	You hear a TTS acknowledgement
	•	No errors occur
	•	Test simulation writes expected entry

⸻

📄 README.md

Include clear instructions:
	•	How to set up .env
	•	How to build/run Docker
	•	Test steps and audio device flags

⸻

Build fully, test instantly, report status. Once done, exit cleanly.

