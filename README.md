# LiveKit Voice Agent (Beginner Friendly)

A real-time conversational voice AI built with LiveKit Agents (Python) and LiveKit Cloud.

You can talk to this agent from the terminal, browser, or phone.

## What This Project Uses

This agent is built with a standard STT → LLM → TTS voice pipeline powered by LiveKit Inference.

- **Speech-to-Text:** AssemblyAI Universal Streaming
- **Language Model:** Gemini 2.5 Flash Lite
- **Text-to-Speech:** Inworld TTS (voice: Craig)
- **Voice Activity Detection:** Silero
- **Turn Detection:** Multilingual model

All models run through LiveKit's managed inference layer.

## Prerequisites

- Python 3.10 – 3.13
- uv package manager
- A free LiveKit Cloud account

---

## LiveKit Cloud Setup

### Install CLI

**Windows:**
```bash
winget install LiveKit.LiveKitCLI
```

### Authenticate

```bash
lk cloud auth
```

This opens a browser where you log in to LiveKit Cloud and select/create a project.

After authentication, run the following command to automatically create a `.env.local` file with the correct credentials:

```bash
lk app env -w
```

Creates `.env.local` with:
- `LIVEKIT_URL`
- `LIVEKIT_API_KEY`
- `LIVEKIT_API_SECRET`
- `NEXT_PUBLIC_LIVEKIT_URL`

[View credentials sheet](https://docs.google.com/spreadsheets/d/1eSq3vo-d6L5JbJQ3kL0ZpKUtZajnyGS-OVdglcB5bzQ/edit?usp=sharing)

### Deploy

**First time:**
```bash
lk agent create
```

This command will prompt you to select or confirm:
- The project on LiveKit Cloud
- The local project
- The TOML configuration file
- The environment file

Upon successful creation, it will automatically create `livekit.toml` with project subdomain and agent ID, and ask if you want to view logs (recommended to check build status). Example output:
```
Build completed - You can view build logs later with `lk agent logs --log-type=build`
Tailing runtime logs...safe to exit at any time
Waiting for deployment to start...
```

**Updates:**
```bash
lk agent deploy
```

As you make changes locally, run this command to sync them to the deployed agent. It only syncs changes; no need to run `lk agent create` again.

---

## Local Development

### Install Dependencies

```bash
uv sync
```

### Download Model Files

```bash
uv run agent.py download-files
```

### Run the Agent

**Console mode (local terminal):**
```bash
uv run agent.py console
```

**Dev mode (connects to LiveKit Cloud):**
```bash
uv run agent.py dev
```

**Production mode:**
```bash
uv run agent.py start
```

---

## Agent Behavior

- Greets users when they connect
- Responds to natural speech in real time
- Supports interruptions and turn-taking
- Room lifecycle is managed by LiveKit Cloud

## Cost Analysis

<img width="805" height="577" alt="image" src="https://github.com/user-attachments/assets/1ac7b1d8-2fcb-4c1f-8d16-518420f42d74" />
