# LiveKit Voice Agent (Beginner Friendly)

A real-time conversational voice AI built with LiveKit Agents (Python) and LiveKit Cloud.

You can talk to this agent from the terminal, browser, or phone.

## What This Project Uses

This agent uses Google's Gemini Live (realtime) model for low-latency voice interactions via LiveKit's Google plugin.

- **Realtime LLM + Audio:** Gemini Live Realtime (native audio)
- **Voice Activity Detection:** Silero (local VAD)
- **Turn Detection:** Multilingual model (when needed)

Note: Gemini Live provides native audio output (no separate TTS). The agent uses LiveKit's RealtimeModel to handle speech generation and turn detection.

## Prerequisites

- Python 3.10 – 3.13
- uv package manager
- A free LiveKit Cloud account (note: allows only one free deployment)

Additionally, for Gemini Live you need a Google API key for the Gemini Live API (environment variable `GOOGLE_API_KEY`) or Vertex AI credentials if using Vertex.

Environment variables

- For local development, put `GOOGLE_API_KEY` in `.env.local` (this repo loads it via `load_dotenv(".env.local")`).
- For Cloud deployments, set `GOOGLE_API_KEY` in the LiveKit Cloud agent's environment (dashboard or `lk agent env set GOOGLE_API_KEY=...`).

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

After successful deployment, test your deployed agent at [Agents Playground](https://agents-playground.livekit.io/).

To view your projects and agents, visit [LiveKit Cloud](https://cloud.livekit.io).

For general documentation, see [LiveKit Docs](https://docs.livekit.io/intro/overview/).

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

Realtime usage (summary)

- Realtime models output native audio (no separate TTS). Use `google.realtime.RealtimeModel`.
- To speak first, call `session.generate_reply(...)` once after `session.start()`.
- Let the model emit farewells and then call the `end_conversation` tool; the tool must only shut down the session (no speech).
- Never call `session.say()` or `generate_reply()` inside a tool; avoid `allow_interruptions=False` with realtime models.

Note on limitation

Gemini Realtime may choose to call a tool without speaking a goodbye. This is expected behavior of the API. Your current implementation follows the recommended pattern (speech from the model, tools for control), which is the correct and safest approach for low-latency realtime agents.

## Cost Analysis

<img width="805" height="577" alt="image" src="https://github.com/user-attachments/assets/1ac7b1d8-2fcb-4c1f-8d16-518420f42d74" />

For pricing details, see [LiveKit Pricing](https://livekit.io/pricing) and [Inference Pricing](https://livekit.io/pricing/inference).

## LiveKit CLI Project Management

Use `lk project` commands to manage LiveKit projects via CLI. For full details, see [LiveKit Docs: Project Management](https://docs.livekit.io/intro/basics/cli/projects).

### Key Commands

- `lk cloud auth`: Authenticate and link LiveKit Cloud projects.
- `lk project add`: Add a new project manually.
- `lk project list`: List configured projects.
- `lk project remove`: Remove a project.
- `lk project set-default`: Set default project.

 
