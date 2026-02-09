# RLM — Reinforcement Learning from Memory

Sage captures your prompt-response pairs and uses them to discover workflow patterns, optimize future suggestions, and improve skill recommendations. The system runs locally via the sage daemon.

---

## How It Works

1. **Capture** — IDE plugins and CLI hooks record prompt-response pairs with session metadata (model, tokens, cost, source IDE)
2. **Store** — The daemon stores captures in a local metrics database
3. **Analyze** — On demand, the RLM service analyzes captures to discover patterns
4. **Apply** — Discovered patterns boost skill suggestions and prompt recommendations

```
IDE/Agent → sage capture hook → Daemon → Local DB
                                          ↓
                              rlm_analyze_captures
                                          ↓
                              Patterns (best-practice, anti-pattern, tool-usage)
                                          ↓
                              Improved suggestions via search_skills / builder_recommend
```

---

## Source Detection

Captures automatically detect which IDE/agent is active:

| Source | Detection |
|--------|-----------|
| `droid` | `DROID` or `DROID_VERSION` env var |
| `openclaw` | `OPENCLAW` env var |
| `claude-code` | `CLAUDE_CODE` or `CLAUDE_DESKTOP` env var |
| `opencode` | `OPENCODE` env var |
| `cursor` | `CURSOR` env var |
| `sage` | Default fallback |

Override with `SAGE_SOURCE=<name>`.

---

## CLI Commands

```bash
# Status and stats
sage capture status              # Capture health + counts
sage capture list                # Recent captures
sage capture list --source droid # Filter by source
sage capture summary             # Pattern summary
sage capture summary --top-n 10  # Top N patterns

# Data management
sage capture export              # Export captures
sage capture export --include-responses  # Include full responses
sage capture prune --older-than 30d      # Clean old captures
sage capture prune --older-than 30d --yes  # Skip confirmation

# Sync captures from external sources
sage capture sync                # Sync all
sage capture sync --source opencode  # Sync specific source
sage capture sync --dry-run      # Preview without syncing

# Hook integration (called by IDE plugins)
sage capture hook prompt         # Capture current prompt
sage capture hook response       # Capture current response
```

---

## MCP Tools

| Tool | Description |
|------|-------------|
| `rlm_stats` | System statistics: total analyses, pattern count, model state, costs |
| `rlm_analyze_captures` | Analyze captures to discover patterns (can call LLM APIs, takes up to 120s) |
| `rlm_list_patterns` | List discovered patterns with confidence scores |

### rlm_stats

Returns: total analyses, local model uses vs API uses, avg execution time, cost, patterns discovered, unique sessions, embedding state.

### rlm_analyze_captures

Parameters:
- `goal` (required): optimization target, e.g., "optimize REST API prompt"
- `session_id` (optional): analyze specific session
- `max_depth` (default: 3): recursion depth for pattern mining
- `use_local_model` (default: false): use local ONNX model instead of API

### rlm_list_patterns

Parameters:
- `pattern_type` (optional): `best-practice`, `anti-pattern`, `tool-usage`
- `min_confidence` (default: 0.5): minimum confidence threshold
- `session_id` (optional): filter by session
- `limit` (default: 20): max results

---

## Pattern Types

| Type | Description |
|------|-------------|
| `best-practice` | Workflows that consistently produce good results |
| `anti-pattern` | Patterns that lead to errors or wasted effort |
| `tool-usage` | How tools are effectively combined |

---

## Daemon Integration

RLM requires the sage daemon running:

```bash
sage daemon start         # Start background daemon
sage daemon status        # Check status
sage daemon logs -f       # Follow logs
```

The daemon provides:
- **Capture storage** — local SQLite metrics database
- **RLM aggregator** — pattern mining and analysis
- **Embedding service** — local ONNX model for similarity matching
- **P2P sync** — share patterns across instances (optional)

---

## IDE Plugin Integration

### OpenClaw

The OpenClaw plugin (`@sage-protocol/openclaw-sage`) automatically:
- Captures prompts/responses via `before_agent_start` hook
- Injects context and skill suggestions
- Tracks RLM feedback (accepted/rejected suggestions)

### OpenCode

The OpenCode plugin (`@sage-protocol/sage-plugin`) automatically:
- Records prompt-response pairs with session metadata
- Provides debounced inline skill suggestions in the TUI
- Tracks RLM feedback within a 30-second correlation window

Configuration:

| Variable | Default | Description |
|----------|---------|-------------|
| `SAGE_RLM_FEEDBACK` | `1` | Enable/disable RLM feedback tracking |
| `SAGE_SUGGEST_LIMIT` | `3` | Max suggestions per prompt |
| `SAGE_SUGGEST_DEBOUNCE_MS` | `800` | Debounce delay for suggestions |

### Claude Code / Factory Droid

Captures happen via session hooks configured by `sage init`:
- `UserPromptSubmit` → `sage capture hook prompt`
- `Stop` → `sage capture hook response`

---

## Local ONNX Model

For privacy-sensitive or offline analysis, RLM can use a local ONNX embedding model instead of API calls:

```
rlm_analyze_captures(goal: "optimize my workflow", use_local_model: true)
```

The daemon downloads and caches the model on first use. Check state with `rlm_stats` → `embedding_state` and `model_load_time_ms`.
