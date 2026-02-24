# 🩺 CallQA — Automated Voice Testing for Conversational Phone Agents

An end-to-end QA harness that makes real outbound calls to a voice AI agent, simulates diverse patient scenarios, transcribes conversations, and surfaces bugs in agent behavior — fully automated and reproducible.

> ✅ Built for repeatable testing: same scenarios → consistent runs → easier debugging  
> ✅ Configurable to any authorized phone agent or test line via environment variable

---

## Why This Project Exists

Voice agents are notoriously hard to test:
- Manual call testing doesn't scale
- Regressions are easy to miss
- "It sounded fine" isn't actionable

CallQA turns voice-agent testing into a structured, automated pipeline:

**Scenario → Call → Recording → Transcription → Report**

---

## Tech Stack

| Layer | Technology |
|---|---|
| Voice Calls | [Twilio](https://www.twilio.com/) — outbound calls + recording |
| Call Scripting | TwiML (Twilio Markup Language) |
| Speech-to-Text | [Groq Whisper](https://console.groq.com/) (`whisper-large-v3-turbo`) |
| Bug Analysis | Python — heuristic pattern matching on transcripts |
| Storage | JSON transcripts + MP3 recordings |
| Language | Python 3.12 |

---

## Architecture
```
Scenario Definition
       ↓
TwiML Script Builder     (pre-scripted patient dialogue with tuned pauses)
       ↓
Twilio Outbound Call     (real call to live voice agent)
       ↓
Audio Recording          (full conversation captured server-side)
       ↓
Groq Whisper STT         (audio → structured transcript)
       ↓
Heuristic Bug Analyzer   (pattern matching for failures, loops, bad phrasing)
       ↓
JSON Transcript + BUG_REPORT.md
```

### Design Decision: Pre-Scripted vs. Interactive

The alternative — a fully interactive bot using real-time webhooks, live transcription, and LLM-generated responses — introduces significant infrastructure overhead (webhook servers, ngrok, latency handling) without meaningfully improving bug detection quality for most scenarios.

The pre-scripted approach with calibrated pauses (18s initial, 10–14s between turns) is reliable, reproducible, and finds the same critical agent failures a real-time system would — at a fraction of the complexity. **The goal is surfacing bugs, not simulating perfect conversations.**

An interactive patient mode (LLM-driven responses) is scaffolded in `src/bot.py` as a future enhancement.

---

## Project Structure
```
├── src/
│   ├── bot.py              # (Roadmap) interactive LLM-driven patient
│   ├── call_handler.py     # Twilio lifecycle: call → record → transcribe → save
│   └── scenarios.py        # 10 patient persona + goal definitions
├── transcripts/            # JSON transcripts + MP3 recordings per call
├── main.py                 # Entry point — run one or all scenarios
├── analyze_bugs.py         # Transcript analysis → BUG_REPORT.md
├── requirements.txt
├── .env.example
├── architecture.md         # Extended design notes
└── BUG_REPORT.md           # Full findings with transcript evidence
```

---

## Setup

### Prerequisites

- Python 3.9+
- [Twilio account](https://www.twilio.com/) — add credits to call unverified numbers
- [Groq account](https://console.groq.com/) — free tier is sufficient

### Installation
```bash
git clone https://github.com/sherurox/pretty-good-ai-voice-bot
cd pretty-good-ai-voice-bot
python3 -m venv venv && source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
```

Edit `.env`:
```env
TWILIO_ACCOUNT_SID=your_twilio_sid
TWILIO_AUTH_TOKEN=your_twilio_token
TWILIO_PHONE_NUMBER=your_twilio_number
GROQ_API_KEY=your_groq_api_key
TARGET_PHONE_NUMBER=+1xxxxxxxxxx   # your authorized test line
```

---

## Usage
```bash
# Run a single scenario (e.g. scenario #1)
python main.py 1

# Run all 10 scenarios (~30–35 min)
python main.py all

# Analyze transcripts and generate bug report
python analyze_bugs.py
```

Artifacts are written to `transcripts/` after each run.

---

## Test Scenarios (10 included)

1. Simple Appointment Scheduling
2. Medication Refill
3. Appointment Rescheduling
4. Office Hours Inquiry
5. Insurance Question
6. Urgent Appointment
7. Cancellation
8. Confused Patient
9. Billing Question
10. Wrong Number

You can extend scenarios in `src/scenarios.py` and add corresponding scripted message flows in `src/call_handler.py → _build_conversation_script()`.

---

## Output Format

Each call produces two artifacts:
```
transcripts/
├── call_20260216_143022.json          # Structured transcript + metadata
└── call_20260216_143022_recording.mp3 # Raw audio
```

The JSON includes: `call_id`, `timestamp`, `scenario_id`, `persona`, `goal`, `conversation[]` (patient script + full transcription + best-effort speaker parsing).

---

## Customization

**Adjust timing/pauses** → `src/call_handler.py → _create_twiml_script()`

**Add new scenarios** → `src/scenarios.py` for metadata, `src/call_handler.py` for the scripted message sequence

---

## Roadmap

- **Interactive patient mode** — real-time agent response triggers LLM-generated patient reply (scaffolded in `src/bot.py`)
- **Speaker diarization** — cleaner patient vs. agent separation in transcripts
- **Richer evaluation** — sentiment analysis, task-completion scoring, rubric grading
- **CI-friendly dry runs** — prerecorded audio replays instead of live calls

---

## Important

> Only call numbers you own or have **explicit permission** to test. Do not store or commit real patient data — use synthetic scenarios only.

---

## License

MIT

---

## Author

**Shreyas Khandale**  
Built with iterative development support from Claude (Anthropic).
