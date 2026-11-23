---
name: xinyu
description: >-
  Official XinYu AI (@xinyuai/cli & @xinyuai/mcp-server) multimodal story-canvas workflow for AI agents.
  Use when generating images (Nano Banana Pro, GPT Image, SeeDream, Wan, Grok Imagine),
  native 4K / 30s videos (Seedance 2.5 draft-to-final, Seedance 2.0 multimodal ref, Kling O3/4.0, Wan, MiniMax),
  audio/TTS/music (ElevenLabs, Seed Audio), decomposing image layers, upscaling/rembg/outpainting,
  or inspecting, connecting, and tidying story-canvas nodes from the terminal or MCP.
---

# XinYu AI CLI & MCP Skill (`xinyu`)

Drive **XinYu AI** (`https://xinyuai.app`) directly from the command line or via its built-in MCP server (`xinyu mcp`) using `@xinyuai/cli`. Generate images, videos, audio, and text, decompose layers, upscale/edit media, and manage visual story-canvas nodes and reference edges with structured `--json` output.

## 1. Prerequisites & Installation

- **Node.js >= 18**
- **XinYu AI API Key** (`xys_live_...`): Sign in at `https://xinyuai.app` → top-right avatar → **Settings** → **Developer API Keys** → **Create key** (tick **Read-only key** if the agent should only inspect canvases/jobs without spending Xins).

```bash
# Install globally (exposes `xinyu`)
npm install -g @xinyuai/cli

# Verify installation
xinyu --version
```

Authenticate once locally (saves to `~/.xinyu/credential.json` with `chmod 600`) or pass `XINYU_API_KEY`:

```bash
xinyu login --key xys_live_xxxx --base-url https://xinyuai.app
xinyu whoami
xinyu balance --json
```

---

## 2. Critical Agent Guardrails (Read Before Running)

1. **Always pass `--json` for programmatic parsing**: Progress logs go to `stderr`; clean JSON goes to `stdout` (safe to pipe into `jq`).
2. **Never re-run a generation command on local wait timeout**:
   - Jobs run **asynchronously on the server** and deduct Xins at submission (auto-refunded if server-side generation fails).
   - If `xinyu generate ...` hits `--timeout`, the job is **still running**. Recover it with:
     ```bash
     xinyu job wait <jobId> --timeout 1800 --json
     # Or if you lost the jobId:
     xinyu job list --status RUNNING --json
     ```
3. **Every media generation requires a canvas (`--project <id>`)**:
   - Image, video, audio, edit, enhance, rembg, layers, and outpaint commands all require `--project <id>` (only `generate text` works without a project).
   - List existing canvases with `xinyu project list --json` or create one with `xinyu project create --name "..." --json`.
4. **Media inputs must be hosted URLs, never local file paths**:
   - Upload local reference images/videos/audio first via `xinyu upload ./file.png --json`, or reuse existing canvas assets from `xinyu project assets <projectId> --json`.
   - When using XinYu Presets (`xinyu presets`), import the preset first with `xinyu preset-import <presetUrl> --json` before passing it as a generation reference.
5. **Check live model IDs and image pricing before batch runs**:
   - Model catalog changes over time: run `xinyu model list --type IMAGE --json` or `xinyu model list --type VIDEO --json` as the source of truth for model IDs and supported resolutions/durations.
   - Estimate image cost before spending credits: `xinyu price --model <model> --size 2K --count 1 --json`.
   - For **Seedance 2.5**, prefer `--draft` first (fast, low-cost 480p preview), then promote approved shots to 1080p with `xinyu generate final <draftJobId> --project <id> --json`.
6. **Confirm destructive deletes**:
   - `xinyu node delete` is irreversible; ask the user before passing `--confirm-finished` or `--confirm-running`.

---

## 3. Command Reference

### Account, Models & Pricing

```bash
xinyu whoami
xinyu balance --json
xinyu model list [--type IMAGE|VIDEO|AUDIO] --json
xinyu price --model <modelId> [--size 1K|2K|4K] [--count <n>] [--refs <n>] --json
xinyu presets [--group clothing|pose] [--category <c>] [--angle <a>] [-q <query>] --json
xinyu preset-import <presetUrl> [--project <id>] [--place] --json
```

### Projects (Canvases) & Asset Upload

```bash
xinyu project list [--search <text>] [--limit <n>] --json
xinyu project create --name "<title>" [--description "<desc>"] --json
xinyu project read <projectId> [--limit <n>] --json
xinyu project assets <projectId> [-q <query>] [--type IMAGE|VIDEO|AUDIO] --json
xinyu project arrange <projectId> [--columns <n>] [--gap <n>] [--visibility live|quiet] --json
xinyu project delete <projectId> --json   # empty canvases only
xinyu upload <localFilePath> [--project <id>] [--place] [--visibility live|quiet] --json
```

### Multimodal Generation

*(Shorthand: `generate` = `gen`, `project` = `canvas`. All canvas-mutating commands accept `--title <text> --x <n> --y <n> --no-place-on-canvas --visibility live|quiet`.)*

```bash
# Text-to-Image / Image-to-Image
xinyu gen image --prompt "<prompt>" --project <id> \
  [--model <m>] [--aspect 16:9] [--size 2K] [--ref <url...>] [--count <n>] \
  [--no-wait] [--timeout 300] --json

# Text / Image / Multimodal-Reference to Video
xinyu gen video --prompt "<prompt>" --project <id> \
  [--model <m>] [--duration 5] [--resolution 1080p] [--audio] \
  [--start-image <url>] [--end-image <url>] [--element-image <url...>] \
  [--ref-video <url...>] [--ref-audio <url...>] [--draft] \
  [--no-wait] [--timeout 2400] --json

# Render Seedance 2.5 480p draft into 1080p final
xinyu gen final <draftJobId> --project <id> [--resolution 1080p] [--no-wait] --json

# Speech / Music / SFX (ElevenLabs TTS or Seed Audio)
xinyu gen audio --text "<text>" --project <id> [--model <m>] [--voice <v>] --json

# Multimodal Text LLM (no --project needed)
xinyu gen text --prompt "<prompt>" [--model <m>] [--media <url...>] --json
```

### Media Editing, Layer Decomposition & Enhancement

```bash
# Topaz Upscale & Face Enhancement
xinyu gen enhance --image <url> --project <id> [--upscale-factor 2] [--no-face-enhancement] --json

# Transparent Background Removal
xinyu gen rembg --image <url> --project <id> [--mask] --json

# Decompose Image into Editable Layers
xinyu gen layers --image <url> --project <id> [--max <n>] [--size 1K|2K] [--text] --json

# Outpaint (Expand Borders in Pixels)
xinyu gen outpaint --image <url> --project <id> [--top <n>] [--right <n>] [--bottom <n>] [--left <n>] --json

# Instruction-based Image / Video Editing
xinyu gen image-edit --prompt "<edit instruction>" --image <url> --project <id> [--model <m>] [--mode <m>] --json
xinyu gen video-edit --prompt "<edit instruction>" --video <url> --project <id> [--duration <s>] [--keep-audio] --json
xinyu gen video-action --action <upscale|extend> --source-task-id <id> --project <id> [--extend-times <n>] --json
```

### Canvas Node Topology & Job Recovery

```bash
# Canvas nodes (personal canvases)
xinyu node rename <nodeId> --project <id> --title "<new title>"
xinyu node connect <sourceNodeId> <targetNodeId> --project <id>
xinyu node disconnect <sourceNodeId> <targetNodeId> --project <id>
xinyu node set-params <nodeId> --project <id> --set model=seedance-2-5 duration=10
xinyu node delete <nodeId...> --project <id> [--confirm-finished] [--confirm-running]

# Async job polling
xinyu job get <jobId> --json
xinyu job wait <jobId> [--timeout 600] --json
xinyu job list [--status RUNNING|SUCCEEDED|FAILED] [--project <id>] [--limit 20] --json
```

---

## 4. End-to-End Storyboard-to-Video Example

```bash
# 1. Create a clean story canvas
PID=$(xinyu project create --name "Product Teaser" --json | jq -r '.id')

# 2. Check available models & estimate keyframe cost
xinyu model list --type IMAGE --json | jq '.[0:5]'
xinyu price --model nano-banana-2.1 --size 2K --count 1 --json

# 3. Generate keyframe image on canvas
KF_URL=$(xinyu gen image \
  --prompt "cinematic product shot on wet obsidian stone, dramatic rim light" \
  --model nano-banana-2.1 --aspect 16:9 --size 2K \
  --title "Shot 01 Keyframe" --project "$PID" --json | jq -r '.assets[0]')

# 4. Animate keyframe with Seedance 2.5 draft first, then promote to 1080p final
DRAFT_JOB=$(xinyu gen video \
  --prompt "slow dolly-in as water droplets ripple across the stone surface" \
  --model seedance-2-5 --start-image "$KF_URL" --duration 5 --draft \
  --title "Shot 01 Draft" --project "$PID" --no-wait --json | jq -r '.jobId')

xinyu job wait "$DRAFT_JOB" --timeout 1200 --json
xinyu gen final "$DRAFT_JOB" --project "$PID" --title "Shot 01 Final 1080p" --json

# 5. Tidy canvas layout
xinyu project arrange "$PID" --columns 4 --json
```

---

## 5. Built-in MCP Server (`xinyu mcp`)

If the host agent supports MCP, you can also attach XinYu AI as a native stdio MCP server (`xinyu_*` tools):

```json
{
  "mcpServers": {
    "xinyu": {
      "command": "npx",
      "args": ["-y", "@xinyuai/mcp-server"],
      "env": {
        "XINYU_API_KEY": "xys_live_xxxxxxxxxxxxxxxx"
      }
    }
  }
}
```
