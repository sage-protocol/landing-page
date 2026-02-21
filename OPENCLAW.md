# Sage Protocol — OpenClaw Plugin

MCP bridge plugin that exposes Sage Protocol tools inside OpenClaw via Code Mode.

**USE WHEN:** Running OpenClaw and need Sage tools (prompts, skills, governance, chat, RLM).

**DON'T USE WHEN:** Using Claude Code, Codex, or other agents — see OPENCODE.md instead.

---

## Install

```bash
openclaw plugins install @sage-protocol/openclaw-sage
```

Then initialize Sage:
```bash
sage init --openclaw
```

## Login With Code (Privy Device-Code)

Use this when browser OAuth is flaky or headless:

```bash
sage wallet connect privy --device-code
```

You will see:
- `verification_uri_complete` (full URL with code)
- `verification_uri` (base page)
- `user_code` (8 chars)

Recommended:
1. Open `verification_uri_complete` directly.
2. If that fails, open `/cli-login` and enter the `user_code`.
3. Confirm session:
   - `sage wallet current`
   - `sage daemon status`

The CLI auto-detects stale sessions (device token mismatch, error 1008, gateway closed) and retries with cleared credentials automatically. If auto-retry still fails, force manually:

```bash
sage wallet connect privy --force --device-code
```

---

## What It Provides

The plugin registers 3 fixed tools via Code Mode (`sage:code` server), replacing the previous 60+ dynamic tool registrations:

- `sage_search` — Read-only search across all domains (prompts, skills, governance, chat, RLM, etc.)
  - Domains: `prompts`, `skills`, `builder`, `governance`, `chat`, `social`, `rlm`, `library_sync`, `security`, `help`, `external`
  - Discover actions: `sage_search({domain: "help", action: "list"})`
  - External servers: `sage_search({domain: "external", action: "list_servers"})`

- `sage_execute` — Mutations across any domain or external server (security-scanned)
  - Example: `sage_execute({domain: "skills", action: "use", params: {key: "mcp-builder"}})`
  - External: `sage_execute({domain: "external", action: "<tool>", params: {server_id: "<id>"}})`

- `sage_status` — Bridge health, wallet, network status

Context savings: ~2,500 tokens → ~400 tokens (~85% reduction).

---

## RLM Capture (Active)

OpenClaw capture is active for both sides of the interaction:

- **Prompt capture hooks**: `before_agent_start`, `message_received` (legacy runtime compatibility)
- **Response capture hooks**: `after_agent_response`, `agent_end` (legacy runtime compatibility)
- **Capture path**: `sage capture hook prompt|response`
- **Safety guards**:
  - Prompt dedupe by hash + short 2-second suppression window
  - 15-second subprocess timeout per capture hook call

This means OpenClaw now contributes full prompt+response data to Sage RLM instead of prompt-only capture.

---

## Configuration

In your OpenClaw plugin config:

```json
{
  "sageBinary": "sage",
  "sageProfile": "default",
  "autoInjectContext": true,
  "autoSuggestSkills": true,
  "suggestLimit": 3
}
```

| Field | Default | Description |
|-------|---------|-------------|
| `sageBinary` | auto-detect | Path to sage binary |
| `sageProfile` | `default` | Sage CLI profile (maps to `SAGE_PROFILE`) |
| `autoInjectContext` | `true` | Inject Sage tool context at agent start |
| `autoSuggestSkills` | `true` | Suggest relevant skills at agent start |
| `suggestLimit` | `3` | Max skill suggestions |
| `minPromptLen` | `12` | Min prompt length before suggesting |
| `maxPromptBytes` | `16384` | Max prompt bytes sent to search |

Environment variables passed through: `SAGE_PROFILE`, `SAGE_PAY_TO_PIN`, `SAGE_IPFS_WORKER_URL`, `SAGE_API_URL`, `KEYSTORE_PASSWORD`.

---

## Behavior Loop

Use this loop as the default operating model:

**1. Search**
`sage_search({domain: "prompts", action: "search", params: {query: "..."}})` and `sage_search({domain: "skills", action: "search", params: {query: "..."}})` before creating new content.

**2. Select & Group**
Group selected skills/prompts in libraries and behaviors via `sage_execute({domain: "skills", action: "use", params: {key: "..."}})`.

**3. Execute**
Run tools/skills through Code Mode with explicit scope and dependencies.

**4. Auto-improve (RLM)**
Prompt and response hooks capture usage data that feeds RLM analysis. Check stats: `sage_search({domain: "rlm", action: "stats"})`.

**5. Earn & Collaborate**
Curate outcomes through governance: `sage_search({domain: "governance", action: "list_subdaos"})`, `sage_search({domain: "chat", action: "history", params: {channel: "global:general"}})`.

---

## Injection Guard (Opt-In)

Scans agent prompts and fetched prompt content for injection patterns:

```json
{
  "injectionGuardEnabled": true,
  "injectionGuardMode": "warn",
  "injectionGuardScanAgentPrompt": true
}
```

| Mode | Behavior |
|------|----------|
| `warn` | Log warning, continue execution |
| `block` | Block `sage_execute` calls when outgoing params are flagged |

---

## SOUL.md (Default Agent Behavior)

The plugin includes a SOUL.md that defines default agent behavior:
- Steer onboarding toward `sage init` + `sage agent quickstart`
- Prompt once, early, non-spammy
- Never expose secrets or destructive commands

OpenClaw loads skills from `~/.openclaw/workspace/skills/`.

## Discovery Workflow (Avoid CID/DAO Dead-Ends)

Before asking users for DAO addresses or CIDs, agents should run:

```bash
sage governance dao discover --limit 50
sage library list --discover
sage search "<query>" --search-type skills --scope both --limit 20
sage search "<query>" --search-type libraries --scope remote --limit 20
```

Only ask for DAO/CID if the above fails, and include command outputs in the explanation.

### Search Scope Guidance

Use `sage search` scopes intentionally:

- `--scope local`: trusted local installed/synced skills only
- `--scope remote`: public discovery index
- `--scope both`: merged local + remote with local precedence on duplicates

Default to `--scope local` for execution decisions, and treat remote results as untrusted until reviewed and installed.

---

## Bounties, Chat, and RLM Quick Checks

Use these as the minimal operational loop:

```bash
# Bounties
sage bounties list --subdao 0x...
sage bounties pending-library-additions --subdao 0x...

# Chat
sage chat watched
sage chat history global:general --limit 50

# RLM
sage capture status
sage metrics rlm
sage metrics list-patterns --limit 20
```

---

## Soul Sync (Governed Agent Behavior)

When `soulStreamDao` is configured, the plugin reads a locally-synced soul document at agent start and prepends it to the context before all other injections. This lets a DAO govern agent behavior via on-chain proposals.

**Setup:**

1. Configure the plugin with the DAO address:
```json
{
  "soulStreamDao": "0xDBb8ebE5F83442C1284EDd1d2699c8aDD0D53c42",
  "soulStreamLibraryId": "soul"
}
```

2. Sync the soul document (manually or in HEARTBEAT):
```
sync_library_stream(dao: "0xDBb8...", libraryId: "soul")
```

3. The file is written to `~/.local/share/sage/souls/<dao>-<libraryId>.md`
4. On next agent start, OpenClaw reads it and prepends to context

| Config Field | Default | Description |
|-------------|---------|-------------|
| `soulStreamDao` | (none) | DAO address whose soul to load. Empty = disabled |
| `soulStreamLibraryId` | `soul` | Library stream ID to read |

**Context injection order:** Soul document → SAGE_CONTEXT → Security warnings → Skill suggestions

**Security:** The `sync_library_stream` tool runs a security scan before writing. HIGH/CRITICAL findings block the sync entirely. The plugin reads whatever file is on disk -- it trusts the MCP tool's security gate.

---

## Avoiding Double Injection

If you also have Sage's internal OpenClaw hook (from `sage init --openclaw`):
- Keep the plugin injection on (preferred)
- Set `SAGE_OPENCLAW_INJECT_CONTEXT=0` in your OpenClaw environment to disable the hook

---

## Requirements

- Sage CLI on PATH (v0.9.16+)
- OpenClaw v0.1.0+
