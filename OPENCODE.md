# Sage Protocol — OpenCode Plugin

Sage is a decentralized protocol for **collaborative prompt and knowledge curation**. It connects AI agents, developers, and communities in a feedback loop that improves prompts over time: authors publish → agents use → RLM captures feedback → scores update → governance curates → reputation accrues → better prompts surface.

This OpenCode plugin captures prompt-response pairs for RLM feedback and provides inline skill suggestions during coding sessions.

---

## Install

```bash
mkdir -p ~/.config/opencode/plugin/@sage-protocol
cp -r sage-plugin ~/.config/opencode/plugin/@sage-protocol/
```

Add to your `opencode.json`:

```json
{
  "plugin": ["@sage-protocol/sage-plugin"],
  "mcp": {
    "sage": {
      "type": "local",
      "command": ["sage", "mcp", "start"],
      "enabled": true
    }
  }
}
```

Or run `sage init --opencode` to configure automatically.

---

## What It Does

- **Prompt Capture** — silently records prompt-response pairs with session metadata (model, tokens, cost)
- **Inline Suggestions** — debounced skill and prompt suggestions injected into the OpenCode TUI
- **RLM Feedback** — tracks whether suggestions were accepted, steered, or rejected within a 30-second correlation window
- **Session Tracking** — maintains session and model context across streaming responses

---

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `SAGE_BIN` | `sage` | Path to the sage binary |
| `SAGE_SUGGEST_LIMIT` | `3` | Max suggestions per request |
| `SAGE_SUGGEST_DEBOUNCE_MS` | `800` | Debounce delay for TUI suggestions |
| `SAGE_SUGGEST_PROVISION` | `1` | Set `0` to skip MCP provisioning |
| `SAGE_RLM_FEEDBACK` | `1` | Set `0` to disable RLM feedback tracking |
| `SAGE_PLUGIN_DRY_RUN` | `0` | Set `1` to disable spawning sage (for tests) |

---

## Semantic Search (qmd)

Use `qmd` for semantic search over project knowledge. **Prefer `qmd query` over grep/find for discovery and exploratory search.**

```bash
# Hybrid search (recommended)
qmd query "governance architecture" -n 5

# Keyword search (fast, exact terms)
qmd search "LibraryRegistry manifest"

# Vector search (conceptual similarity)
qmd vsearch "how does voting delegation work"

# Check indexed collections
qmd status

# Re-index after updates
qmd embed
```

Use grep/find only for exact symbol matching or known file paths.

## Behavior Loop

This plugin participates in the full Sage agent loop:

**1. Search** — `sage search "<q>" --search-type skills --scope both`, `sage suggest skill "<prompt>"` (RLM-boosted). MCP: `search_prompts()`, `search_skills()`, `builder_recommend()`.

**2. Select & Group** — Group skills into libraries, sequence with behavior prompts:
```bash
sage library create "my-workflow" && sage library skill add ./skills/step1 -l "my-workflow"
```

**3. Execute** — `get_prompt(key, expand: ["behaviorPlan", "behaviorSkillBodies"])` generates a chained plan. `use_skill(key)` activates + auto-provisions MCP servers.

**4. Auto-improve** — This plugin handles step 4 automatically: captures prompt-response pairs, tracks accept/steer/reject within 30s correlation window, and feeds confidence scores back into suggestion ranking.

**5. Earn** — `sage governance proposals vote <id> --for`, `sage bounty list <dao>`, `sage library push`.

### Architecture Rule

CLI/SDK → Worker REST API (`api.sageprotocol.io`) → Subgraph. Never query subgraph directly.

## Requirements

- Sage CLI on PATH (or set `SAGE_BIN`)
- Bun v1.3+
- OpenCode
