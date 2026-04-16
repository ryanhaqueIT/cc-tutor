# CC Tutor — Learn Claude Code with an AI Buddy

A macOS menu bar companion app that helps you learn [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — the agentic coding CLI from Anthropic. It lives next to your cursor, can see your screen, talk to you, and point at things. Like having a real tutor sitting next to you as you learn.

Based on [Clicky](https://github.com/farzaa/clicky) by [@farzatv](https://x.com/farzatv), modified to be a dedicated Claude Code learning companion.

## What it does

- **Sees your screen** — press Control+Option, ask a question about Claude Code, and the tutor sees what you're looking at. If you have a terminal with Claude Code running, it'll reference the specific output, errors, or tool calls on screen.
- **Talks back** — responses are spoken aloud via text-to-speech, so you can keep your eyes on the code.
- **Points at things** — a blue cursor overlay can fly to and highlight specific UI elements, terminal lines, or buttons it's referencing.
- **Knows Claude Code deeply** — slash commands, CLAUDE.md files, MCP servers, hooks, permissions, settings, IDE integrations, context management, the agentic loop, and more.

## Topics you can ask about

- Getting started with Claude Code (installation, API keys, first session)
- Slash commands (`/init`, `/commit`, `/review`, `/compact`, `/clear`, `/model`, etc.)
- CLAUDE.md files and project-level instructions
- The permission system (auto-allow, settings.json)
- MCP servers and how to extend Claude Code
- Hooks for automating behaviors
- IDE integrations (VS Code, JetBrains)
- Context management and compaction
- Git workflows with Claude Code
- Multi-file editing and large refactors
- Plan mode and extended thinking
- Memory and task tracking
- Any coding question — especially "how would I use Claude Code to solve this?"

## Setup

### Prerequisites

- macOS 14.2+ (for ScreenCaptureKit)
- Xcode 15+
- Node.js 18+ (for the Cloudflare Worker)
- A [Cloudflare](https://cloudflare.com) account (free tier works)
- API keys for: [Anthropic](https://console.anthropic.com), [AssemblyAI](https://www.assemblyai.com), [ElevenLabs](https://elevenlabs.io)

### 1. Set up the Cloudflare Worker

The Worker is a tiny proxy that holds your API keys. The app talks to the Worker, the Worker talks to the APIs.

```bash
cd worker
npm install
```

Add your secrets:

```bash
npx wrangler secret put ANTHROPIC_API_KEY
npx wrangler secret put ASSEMBLYAI_API_KEY
npx wrangler secret put ELEVENLABS_API_KEY
```

Set the ElevenLabs voice ID in `wrangler.toml`:

```toml
[vars]
ELEVENLABS_VOICE_ID = "your-voice-id-here"
```

Deploy:

```bash
npx wrangler deploy
```

Copy the URL it gives you (like `https://your-worker-name.your-subdomain.workers.dev`).

### 2. Run the Worker locally (for development)

```bash
cd worker
npx wrangler dev
```

Create a `.dev.vars` file in the `worker/` directory:

```
ANTHROPIC_API_KEY=sk-ant-...
ASSEMBLYAI_API_KEY=...
ELEVENLABS_API_KEY=...
ELEVENLABS_VOICE_ID=...
```

Then update the proxy URLs in the Swift code to point to `http://localhost:8787`. Grep for `your-worker-name` to find them.

### 3. Update the proxy URLs in the app

```bash
grep -r "your-worker-name" leanring-buddy/
```

Replace with your Worker URL in:
- `CompanionManager.swift` — Claude chat + ElevenLabs TTS
- `AssemblyAIStreamingTranscriptionProvider.swift` — AssemblyAI token endpoint

### 4. Open in Xcode and run

```bash
open leanring-buddy.xcodeproj
```

In Xcode:
1. Select the `leanring-buddy` scheme
2. Set your signing team under Signing & Capabilities
3. Hit **Cmd + R** to build and run

The app appears in your menu bar (not the dock). Click the icon to open the panel, grant permissions, and you're ready to learn.

### Permissions needed

- **Microphone** — for push-to-talk voice capture
- **Accessibility** — for the global keyboard shortcut (Control + Option)
- **Screen Recording** — for taking screenshots when you use the hotkey
- **Screen Content** — for ScreenCaptureKit access

## Architecture

Menu bar app (no dock icon) with two `NSPanel` windows — one for the control panel dropdown, one for the full-screen transparent cursor overlay. Push-to-talk streams audio over a websocket to AssemblyAI, sends the transcript + screenshot to Claude via streaming SSE, and plays the response through ElevenLabs TTS. Claude is prompted with deep Claude Code expertise and can embed `[POINT:x,y:label:screenN]` tags to make the cursor fly to specific UI elements. All APIs are proxied through a Cloudflare Worker.

## Credits

Built on top of [Clicky](https://github.com/farzaa/clicky) by [Farza](https://x.com/farzatv). Original project open-sourced for hacking and building.
