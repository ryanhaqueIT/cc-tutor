# CC Tutor — Learn Claude Code with an AI Buddy

A macOS menu bar companion app that helps you learn [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — the agentic coding CLI from Anthropic. It lives next to your cursor, can see your screen, talks to you, and points at things. Like having a real tutor sitting next to you as you learn.

Based on [Clicky](https://github.com/farzaa/clicky) by [@farzatv](https://x.com/farzatv), modified to be a dedicated Claude Code learning companion.

## Quick Start with Claude Code

> **Requirements before you begin:**
> - macOS 14.2+ with Xcode 15+ installed
> - Node.js 18+
> - A [Cloudflare](https://cloudflare.com) account (free tier works) — run `npx wrangler login` if you haven't already
> - API keys ready for: [Anthropic](https://console.anthropic.com/settings/keys), [AssemblyAI](https://www.assemblyai.com/app/keys), [ElevenLabs](https://elevenlabs.io/app/keys)
> - An ElevenLabs voice ID (find one at [ElevenLabs Voice Library](https://elevenlabs.io/voice-library) or use the default `kPzsL2i3teMYv0FxEYQ6`)

Once you have those, open Claude Code in the cloned repo directory and paste this:

```
Hi Claude.

Read the README.md for full setup instructions. I want to get CC Tutor running locally on my Mac.

Help me set up everything:
1. Install the Cloudflare Worker dependencies
2. Create the worker/.dev.vars file with my API keys (I'll provide them when you ask)
3. Update the proxy URLs in the Swift source to point to http://localhost:8787
4. Walk me through starting the local worker and opening the Xcode project

I'll handle the Xcode signing and macOS permissions myself.
```

Claude Code will read this file, walk you through providing your API keys, patch the source files, and get you ready to build.

## Manual Setup

### Prerequisites

- **macOS 14.2+** (required for ScreenCaptureKit)
- **Xcode 15+** (Swift/SwiftUI build toolchain)
- **Node.js 18+** (for the Cloudflare Worker proxy)
- **Cloudflare account** (free tier works — [sign up here](https://cloudflare.com))
- **API keys** for three services:
  - [Anthropic](https://console.anthropic.com/settings/keys) — powers the Claude AI responses
  - [AssemblyAI](https://www.assemblyai.com/app/keys) — powers real-time voice transcription
  - [ElevenLabs](https://elevenlabs.io/app/keys) — powers text-to-speech voice output
- **ElevenLabs voice ID** — the voice the tutor speaks with. The default in `wrangler.toml` is `kPzsL2i3teMYv0FxEYQ6`. Browse voices at [ElevenLabs Voice Library](https://elevenlabs.io/voice-library).

### Step 1: Install Worker Dependencies

```bash
cd worker
npm install
```

### Step 2: Configure API Keys for Local Development

Create a file at `worker/.dev.vars` with your API keys:

```
ANTHROPIC_API_KEY=sk-ant-your-key-here
ASSEMBLYAI_API_KEY=your-key-here
ELEVENLABS_API_KEY=your-key-here
ELEVENLABS_VOICE_ID=kPzsL2i3teMYv0FxEYQ6
```

> **Important:** This file is gitignored and never committed. These keys stay on your machine.

### Step 3: Update Proxy URLs in Swift Source

The app has a placeholder URL that must be replaced with your worker address. For local development, use `http://localhost:8787`.

There are exactly **2 files** with the placeholder:

**File 1: `leanring-buddy/CompanionManager.swift`** — line ~73

Find:
```swift
private static let workerBaseURL = "https://your-worker-name.your-subdomain.workers.dev"
```
Replace with:
```swift
private static let workerBaseURL = "http://localhost:8787"
```

**File 2: `leanring-buddy/AssemblyAIStreamingTranscriptionProvider.swift`** — line ~22

Find:
```swift
private static let tokenProxyURL = "https://your-worker-name.your-subdomain.workers.dev/transcribe-token"
```
Replace with:
```swift
private static let tokenProxyURL = "http://localhost:8787/transcribe-token"
```

The placeholder string to search for in both files is: `your-worker-name.your-subdomain.workers.dev`

### Step 4: Start the Local Worker

```bash
cd worker
npx wrangler dev
```

This starts a local server at `http://localhost:8787`. Keep this terminal open while using the app.

> **Note:** You must be logged into Cloudflare CLI. If you haven't before, run `npx wrangler login` first — this opens a browser for OAuth. This is an interactive command that Claude Code cannot run for you.

### Step 5: Open in Xcode and Build

```bash
open leanring-buddy.xcodeproj
```

In Xcode (these steps must be done manually — Claude Code cannot do them):

1. Select the **`leanring-buddy`** scheme in the scheme picker (top center of Xcode)
2. Go to **Signing & Capabilities** and set your **Team** (your Apple ID / personal team is fine)
3. Press **Cmd + R** to build and run

> **Do NOT build from the terminal with `xcodebuild`** — it invalidates macOS TCC permissions and the app will need to re-request screen recording, accessibility, etc.

### Step 6: Grant macOS Permissions

The app appears in your **menu bar** (top-right of screen), not the dock. Click the blue triangle icon to open the panel. It will walk you through granting four permissions:

| Permission | What it's for | How to grant |
|---|---|---|
| **Microphone** | Push-to-talk voice capture | System prompt appears automatically |
| **Accessibility** | Global keyboard shortcut (Ctrl+Option) | System Settings > Privacy & Security > Accessibility |
| **Screen Recording** | Taking screenshots when you press the hotkey | System Settings > Privacy & Security > Screen Recording — **requires app restart after granting** |
| **Screen Content** | ScreenCaptureKit access for multi-monitor capture | Click "Grant" in the panel — picker appears |

### Step 7: Start Learning

Hold **Control + Option** to talk. Ask anything about Claude Code. Release the keys when you're done speaking. The tutor will see your screen, respond with voice, and point at relevant things.

## Deploying the Worker (Optional)

For local dev, `wrangler dev` is all you need. If you want to deploy so the app works without a local terminal:

```bash
cd worker
npx wrangler login    # if not already logged in — interactive, opens browser
npx wrangler secret put ANTHROPIC_API_KEY      # interactive prompt — paste your key
npx wrangler secret put ASSEMBLYAI_API_KEY     # interactive prompt — paste your key
npx wrangler secret put ELEVENLABS_API_KEY     # interactive prompt — paste your key
npx wrangler deploy
```

> **Note:** The `wrangler secret put` commands are interactive (they prompt you to paste each key). Claude Code cannot run these — you must run them yourself in a terminal.

After deploying, Wrangler prints your worker URL (like `https://cc-tutor-proxy.your-account.workers.dev`). Update the two Swift files above to use that URL instead of `http://localhost:8787`.

To change the voice, edit `ELEVENLABS_VOICE_ID` in `worker/wrangler.toml` and redeploy.

## Guide / Topic System

The panel has a **Guide** toggle (next to the Model picker) that switches the tutor between different instruction sets:

| Guide | What it does |
|---|---|
| **General** | Broad Claude Code expertise. Ask anything about slash commands, CLAUDE.md, MCP, hooks, etc. |
| **CBA Setup** | Step-by-step walkthrough for setting up Claude Code at CBA. Guides engineers through installation, API key config, first session, CLAUDE.md setup, permissions, IDE integration, and team conventions. |

When a guide is active, the tutor sees the user's screen and knows what step they're on. It skips completed steps, helps with errors, and adapts to where the user is. Switching guides clears conversation history so context doesn't bleed across topics.

### Customizing the CBA Setup Guide

Edit the CBA setup instructions in `leanring-buddy/CompanionManager.swift`. Search for `case "cba-setup":` — that block contains the full step-by-step prompt. Replace the placeholder steps with your organization's actual process (internal proxy URLs, approved MCP servers, team CLAUDE.md conventions, specific tooling, etc.).

### Adding New Guides

To add a new guide (e.g., "Security Review", "PR Workflow", "New Engineer Onboarding"):

1. **Add a topic entry** in `CompanionManager.swift` — find `availableTopics` and add a new `(id:, label:)` tuple
2. **Add the prompt** in `CompanionManager.swift` — add a new `case` in `topicSpecificPromptBlock(for:)` with the step-by-step instructions
3. **Add the button** in `CompanionPanelView.swift` — find `topicPickerRow` and add a `topicOptionButton(label:, topicID:)` call

The prompt format is plain text. Write the instructions as if you're briefing a knowledgeable tutor — tell it the steps, the context, what to watch for on screen, and any constraints. Claude handles the rest (adapting to where the user is, referencing their screen, pointing at things).

## Project Structure

```
leanring-buddy/                              # Swift source (the typo is intentional/legacy)
  CompanionManager.swift                       # Central state machine, system prompts, topic/guide system
  CompanionPanelView.swift                     # Menu bar panel UI (settings, permissions, model + guide pickers)
  ClaudeAPI.swift                              # Claude streaming SSE client
  ElevenLabsTTSClient.swift                    # ElevenLabs text-to-speech playback
  OverlayWindow.swift                          # Full-screen transparent cursor overlay + pointing
  AssemblyAIStreamingTranscriptionProvider.swift # Real-time voice transcription via websocket
  BuddyDictationManager.swift                 # Push-to-talk audio capture pipeline
  GlobalPushToTalkShortcutMonitor.swift        # System-wide Ctrl+Option keyboard shortcut
  CompanionScreenCaptureUtility.swift          # Multi-monitor screenshot capture
  MenuBarPanelManager.swift                    # NSStatusItem + floating panel lifecycle
  DesignSystem.swift                           # Colors, corner radii, button styles
  ClickyAnalytics.swift                        # PostHog analytics (from original Clicky)
  Info.plist                                   # App config (LSUIElement, transcription provider)
worker/                                      # Cloudflare Worker API proxy
  src/index.ts                                 # Three routes: /chat, /tts, /transcribe-token
  wrangler.toml                                # Worker config + ElevenLabs voice ID
  .dev.vars                                    # Local dev API keys (you create this, gitignored)
AGENTS.md                                    # Full architecture doc for AI coding agents
CLAUDE.md                                    # Symlink to AGENTS.md
```

## Architecture

Menu bar app (no dock icon, `LSUIElement=true`) with two `NSPanel` windows — one for the control panel dropdown, one for the full-screen transparent cursor overlay.

**Voice pipeline:** Push-to-talk (Ctrl+Option via CGEvent tap) captures audio with AVAudioEngine, streams PCM16 over a websocket to AssemblyAI for real-time transcription.

**AI pipeline:** Transcript + screenshots of all displays (via ScreenCaptureKit, 1280px JPEG) are sent to Claude via SSE streaming. The system prompt contains deep Claude Code expertise covering slash commands, CLAUDE.md files, MCP servers, hooks, permissions, settings, IDE integrations, context management, git workflows, plan mode, and more. A topic/guide picker appends topic-specific instruction blocks to the base prompt (e.g., the CBA Setup guide adds a 7-step walkthrough). Claude's response is spoken aloud via ElevenLabs TTS.

**Pointing:** Claude can embed `[POINT:x,y:label:screenN]` tags in responses. The overlay parses these, maps screenshot-pixel coordinates to display-point coordinates across multiple monitors, and animates the blue cursor along a bezier arc to the target.

**API proxy:** All three external APIs (Anthropic, AssemblyAI, ElevenLabs) are proxied through a Cloudflare Worker so API keys never ship in the app binary.

## Troubleshooting

| Problem | Fix |
|---|---|
| App doesn't appear after building | It's in the **menu bar** (top-right), not the dock. Look for a small blue triangle icon. |
| "Socket is not connected" errors | Usually means the Cloudflare Worker isn't running. Make sure `npx wrangler dev` is active. |
| Push-to-talk doesn't work | Check that Accessibility permission is granted in System Settings. The app needs it for the global Ctrl+Option shortcut. |
| Screen capture fails | Grant Screen Recording permission, then **quit and reopen the app**. macOS requires a restart for this permission. |
| TTS doesn't play audio | Check your ElevenLabs API key and that you have credits remaining. The app falls back to macOS system voice if ElevenLabs fails. |
| Worker returns 500 errors | Check `worker/.dev.vars` has all four keys set correctly. Run `npx wrangler dev` and watch the terminal for error details. |
| Build fails in Xcode | Make sure you've set a signing team under Signing & Capabilities. Personal team (free Apple ID) works fine. |

## Credits

Built on top of [Clicky](https://github.com/farzaa/clicky) by [Farza](https://x.com/farzatv). Original project open-sourced for hacking and building.
