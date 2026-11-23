# XinYu AI — Official Agent Skills, CLI & MCP (`@xinyuai/cli`)

**One canvas, from spark to final cut — now native to your terminal and AI coding agents.**

[**XinYu AI (`xinyuai.app`)**](https://xinyuai.app) unifies frontier image, video, audio, and storyboard workflows onto a single visual node canvas under **one shared credit balance (Xins, never expires)** — featuring **Seedance 2.5** (native 4K, up to 30s, draft-to-final), **Seedance 2.0** (image + video + audio multimodal references), **Kling 4.0 / O3**, **Wan 3.0**, **Nano Banana Pro**, **GPT Image 2.6**, **SeeDream 5.0 Pro**, **ElevenLabs**, and **Seed Audio**.

This repository provides the official agent `SKILL.md` and MCP configuration templates for Claude Code, Cursor, Google Antigravity, Codex, Windsurf, Cline, OpenClaw, and CLI-Anything (`cli-hub`).

English · [简体中文](#简体中文说明)

---

## Quick Install

### 1. Install the Agent Skill (`SKILL.md`)

```bash
npx skills add xinyuai-official/xinyu-skills --skill xinyu -g -y
```

### 2. Install the CLI (`@xinyuai/cli`)

```bash
npm install -g @xinyuai/cli
xinyu login --key xys_live_xxxx
xinyu balance --json
```

> Get your API key at **[xinyuai.app](https://xinyuai.app)** → Avatar → **Settings** → **Developer API Keys** (supports **Read-only keys** for safe inspection).

### 3. Or Connect via MCP (`@xinyuai/mcp-server`)

Add to your Claude Desktop, Cursor (`.cursor/mcp.json`), Claude Code (`.mcp.json`), or Google Antigravity (`~/.gemini/config/mcp_config.json`):

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

---

## Documentation

- **CLI Guide**: [https://xinyuai.app/en/cli](https://xinyuai.app/en/cli) ([中文文档](https://xinyuai.app/zh/cli))
- **MCP Guide (36+ Tools)**: [https://xinyuai.app/en/mcp](https://xinyuai.app/en/mcp) ([中文文档](https://xinyuai.app/zh/mcp))
- **Canonical Agent Skill**: [`skills/xinyu/SKILL.md`](skills/xinyu/SKILL.md)

---

## 简体中文说明

本仓库提供 **心宇 AI（[xinyuai.app](https://xinyuai.app)）** 官方公开版 Agent 技能包（`SKILL.md`）与命令行 / MCP 配置模板：

1. **安装命令行工具**：`npm install -g @xinyuai/cli`（内置 `xinyu` 命令与 `xinyu mcp` 服务）
2. **安装 Agent Skill**：`npx skills add xinyuai-official/xinyu-skills --skill xinyu -g -y`
3. **核心能力**：支持终端 `--json` 管道化生成图像、4K/30秒视频（Seedance 2.5 样片转正片、Kling O3、Wan 等）、语音配乐、图层拆解、抠图扩图高清化，以及直接读写、连线与排版心宇故事画布节点。

## License

MIT
