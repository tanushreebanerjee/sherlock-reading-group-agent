# 🔍 Sherlock — AI Reading Group Assistant

A single-file web app that joins your academic reading group as a knowledgeable peer. It listens to the conversation, has read the paper, and contributes when called or when it can genuinely help.

**No backend. No installation. Just open the HTML file in Chrome.**

---

## How It Works

### 1. Paper Ingestion (at setup)

You drop a PDF onto the setup screen. Sherlock uses **PDF.js** (runs entirely in your browser) to extract all the text from the paper. That text — up to ~60,000 characters — is stored in memory and injected into every AI prompt as context. Sherlock genuinely "has read" the paper in the sense that the full text is present in every API call.

### 2. Speech-to-Text (during the meeting)

Sherlock uses the browser's built-in **Web Speech API** (`SpeechRecognition`) — no external STT model, no API key needed. Chrome sends the microphone audio to Google's servers and returns transcribed text in real time. You get:

- **Interim results** — live, shown dimmed as people speak
- **Final results** — committed segments that drive all trigger logic

Two separate SR instances are used:
- **Main SR** — runs continuously throughout the meeting
- **Dictation SR** — one-shot, fires when you click 🎤 in the ask box, temporarily pauses the main SR to avoid mic conflicts

### 3. Participation Triggers

On every finalized speech segment, Sherlock checks three things in order:

| Trigger | How it fires |
|---|---|
| **Wake word** — `"Sherlock…"` | Regex `/\bsherlock\b/i` on transcript; waits 1.5 s then calls AI |
| **Uncertainty** | Phrases like *"I'm not sure"*, *"what does this mean"*, *"confused"* → AI responds if cooldown allows |
| **Silence** | If no speech for N seconds (30 s balanced, 15 s active), AI considers chipping in |

Sherlock can also respond to **manually typed or dictated questions** at any time via the ask box.

### 4. AI Response (the actual LLM call)

When a trigger fires, Sherlock assembles a prompt with:
- A **system message** describing its role (peer PhD student in Zwicker's group) + the **full paper text**
- A **user message** with the recent ~600 chars of transcript and trigger context

That goes to whichever AI provider you configured. If Sherlock has nothing valuable to add (silence trigger, no uncertainty), it can respond `NO_CONTRIBUTION` and stay quiet.

### 5. Output

Responses appear as cards in the transcript panel, labeled by trigger type. Optional TTS reads them aloud via the browser's **SpeechSynthesis API**.

---

## AI Providers

| Provider | Model | Cost | Notes |
|---|---|---|---|
| **Gemini** | `gemini-3.8-flash` | Free tier available | Recommended for getting started |
| **Groq** | `llama-4-maverick-17b` | Free tier | Very fast inference |
| **Anthropic** | `claude-opus-4-5` | Paid | Most capable |

Get your key:
- Gemini: [aistudio.google.com](https://aistudio.google.com) → Get API key
- Groq: [console.groq.com/keys](https://console.groq.com/keys)
- Anthropic: [console.anthropic.com](https://console.anthropic.com)

---

## Usage

### Local (recommended)

1. Download `reading-group-ai.html`
2. Open it directly in **Chrome** (Chrome required for Web Speech API)
3. Enter your API key and select provider
4. Drop this week's PDF onto the setup screen
5. Click **Start Meeting** and enable the mic

> **Why Chrome only?** The Web Speech API requires Chrome. Also, Anthropic's API requires the `anthropic-dangerous-allow-browser` header which Chrome allows from local files; other browsers may block it.

### GitHub Pages

This repo is deployed at: **[your-username.github.io/sherlock](https://your-username.github.io/sherlock)**

Open the page in Chrome, then use it exactly like the local file. Your API key is never sent anywhere except the AI provider you choose — it stays in your browser tab's memory and is gone when you close the tab.

---

## Sensitivity Modes

Click the **🎯** button during a meeting to cycle between:

| Mode | Icon | Behavior |
|---|---|---|
| **Quiet** | 🤐 | Manual questions only — never auto-interrupts |
| **Balanced** | 🎯 | Chimes in on uncertainty and long pauses (default) |
| **Active** | 🗣 | Speaks up frequently, shorter silence threshold |

---

## Privacy

- The paper PDF is processed entirely in your browser by PDF.js — it is never uploaded anywhere
- The extracted paper text and recent transcript snippets are sent to your chosen AI provider per request (same as any chat session with that provider)
- Your API key is stored only in the browser tab's JavaScript memory for the duration of the session

---

## Project Structure

```
sherlock/
├── index.html          # The entire app (renamed from reading-group-ai.html)
└── README.md
```

Single self-contained file — all CSS, JS, and HTML are inline. No build step, no dependencies to install, no `node_modules`.

---

## Lab

Built for the [Zwicker Lab](https://www.cs.umd.edu/~zwicker/) reading group at the University of Maryland, Department of Computer Science.
