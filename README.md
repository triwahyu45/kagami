# KAGAMI — 3D VRoid Character Engineering & Cinema Rig

AI Agent Skill for **Antigravity**, **Claude Code**, and **OpenAI Codex**. Unites VRoid Studio binary Protobuf/UV/BrushHair customization (`.vroid`/`.vroidcustomitem`) with Three.js film cinematography grammar and VRM 0.0 vs VRM 1.0 anatomical bone polarity & joint clamp guardrails.

---

## Quick Install (Claude Code / Codex / Antigravity)

Clone this skill into your agent's global or project skills directory:

| Agent | Global Install Path | Project-Local Path |
| :--- | :--- | :--- |
| **Antigravity** | `~/.gemini/config/skills/KAGAMI` | `.agents/skills/KAGAMI` |
| **Claude Code** | `~/.claude/skills/KAGAMI` | `.claude/skills/KAGAMI` |
| **OpenAI Codex** | `~/.codex/skills/KAGAMI` | `.agents/skills/KAGAMI` |

```bash
# Example (Antigravity / Claude Code / Codex — pick your path):
git clone https://github.com/triwahyu45/KAGAMI.git ~/.claude/skills/KAGAMI
```

---

## Requirements & Dependencies (For Agents)

- **Runtime / Tools**: **Python 3.10+** (for Protobuf/ZIP binary `.vroid` edits) and/or **Three.js** (for WebGL VRM scenes); **VRoid Studio 2.14.0** for live MCP editing
- **MCP Servers**: Required for live VRoid Studio editing & `.vrm` export: [`ningyo`](https://github.com/triwahyu45/NINGYO) (`vroid-studio` MCP at `http://127.0.0.1:17891/mcp`)
- **Related Skills**: None required
- **API Keys / Secrets**: None

```json
{
  "mcpServers": {
    "vroid-studio": {
      "url": "http://127.0.0.1:17891/mcp"
    }
  }
}
```

---

## Included Files

- `SKILL.md` — Character & cinema rig workflow + multi-angle visual gate
- `references/vroid-protobuf-and-uv.md` — `v1model/data.bin` Protobuf, `BrushHair`, & UV islands
- `references/vroid-closeup-audit.md` — `vroid-studio` MCP camera calibration & audit
- `references/cinematic-camera-and-vrm-clamps.md` — Film camera grammar & VRM 0.0/1.0 bone clamps

## When to Trigger / Search Keywords

`kagami`, `vroid`, `.vroid`, `.vroidcustomitem`, `.vrm`, `threejs vrm`, `vrm bone clamp`, `dolly zoom`, `brushhair`, `uv island`
