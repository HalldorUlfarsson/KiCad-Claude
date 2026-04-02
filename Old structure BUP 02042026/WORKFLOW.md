# Project Workflow Guide

**Project:** KiCad-Claude Audio Amplifier  
**Last Updated:** 2026-03-31

---

## Toolchain

| Tool | Version | Purpose |
|---|---|---|
| KiCad | 10.0.0 | Schematic capture, PCB layout |
| LTspice | 26.0.1 (macOS 17.2.4) | Analog circuit simulation |
| Claude Desktop | Latest | AI design assistant |
| KiCad MCP Server | lamaalrajih/kicad-mcp | Connects Claude Desktop to KiCad |
| git | 2.39.5 | Version control |

---

## MCP Server Setup (macOS)

### Installation Location
```
~/kicad-mcp/
```

### Claude Desktop Config
File: `~/Library/Application Support/Claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "kicad": {
      "command": "/Users/hau/kicad-mcp/.venv/bin/python3",
      "args": ["/Users/hau/kicad-mcp/main.py"],
      "env": {
        "KICAD_SEARCH_PATHS": "~/Documents/KiCad"
      }
    }
  }
}
```

### Verifying Connection
In Claude Desktop, look for the tools/hammer icon near the message input. KiCad tools should be listed. If not, restart Claude Desktop fully (Cmd+Q).

### Reinstalling MCP Server
```bash
cd ~/kicad-mcp
brew install uv
make install
```

---

## Project File Location

```
/Users/hau/Library/CloudStorage/GoogleDrive-halldorion@gmail.com/My Drive/2026/KiCad-Claude/
```

---

## GitHub Repository

**URL:** https://github.com/HalldorUlfarsson/KiCad-Claude  
**Spec URL:** https://raw.githubusercontent.com/HalldorUlfarsson/KiCad-Claude/main/SYSTEM_SPEC.md

---

## Session Protocol

### Session Start
1. Open Claude Desktop → KiCad Audio Amplifier project
2. Claude auto-fetches spec and orients, or paste spec URL manually

### Session End
```bash
cd "/Users/hau/Library/CloudStorage/GoogleDrive-halldorion@gmail.com/My Drive/2026/KiCad-Claude"
git add .
git commit -m "Session YYYY-MM-DD — description"
git push
```

---

## Commit Script

Save as `commit.sh` in project root:

```bash
#!/bin/bash
cd "/Users/hau/Library/CloudStorage/GoogleDrive-halldorion@gmail.com/My Drive/2026/KiCad-Claude"
git add .
git commit -m "Session $(date '+%Y-%m-%d') — auto commit"
git push
echo "✅ Committed and pushed"
```

Make executable: `chmod +x commit.sh`

---

## Claude Desktop Project Instruction

```
You are assisting with an ongoing bespoke audio electronics design project.

AT THE START OF EVERY CONVERSATION:
1. Fetch and read the current system specification at:
   https://raw.githubusercontent.com/HalldorUlfarsson/KiCad-Claude/main/SYSTEM_SPEC.md
2. Summarise the current project state in 3-4 bullet points
3. Confirm the KiCad MCP connection is active by running list_projects
4. State what the open questions are from Section 7 of the spec
5. Ask what we are working on today

PROJECT CONTEXT:
- Single-channel audio amplifier: NE5532 buffer → TPA3251 Class D → 8Ω speaker
- Supply: 32V external brick, ±15V linear regulated for buffer
- Toolchain: KiCad 10 + LTspice + KiCad MCP server connected
- Repo: https://github.com/HalldorUlfarsson/KiCad-Claude
- KiCad path: /Users/hau/Library/CloudStorage/GoogleDrive-halldorion@gmail.com/My Drive/2026/KiCad-Claude

AT THE END OF EVERY CONVERSATION:
1. Summarise what was decided or completed this session
2. Update the open questions list
3. Remind the user to run the commit script
```

---

## Schematic Generation Workflow

1. Design agreed in chat
2. Claude generates `.kicad_sch` file → user downloads and places in project folder
3. User opens in KiCad to verify visually
4. Claude inspects via MCP
5. ERC run in KiCad, errors resolved
6. Commit to GitHub

---

## Human Verification Gates

- [ ] Spec sign-off — requirements agreed before schematic work
- [ ] Schematic review — ERC clean, biasing and decoupling checked
- [ ] Pre-layout simulation — LTspice confirms behaviour matches spec
- [ ] Pre-fab layout review — ground planes, trace widths, placement
