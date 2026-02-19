# Sage Protocol — OpenClaw Plugin

MCP bridge plugin that exposes all 60+ Sage Protocol tools inside OpenClaw.

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

Force refresh an expired session:

```bash
sage wallet connect privy --force --device-code
```

---

## What It Provides

Once loaded, all Sage MCP tools are available with a `sage_` prefix:

- `sage_search_prompts`, `sage_list_prompts`, `sage_get_prompt`, `sage_quick_create_prompt`
- `sage_search_skills`, `sage_list_skills`, `sage_get_skill`, `sage_use_skill`
- `sage_builder_recommend`, `sage_builder_synthesize`, `sage_builder_vote`
- `sage_list_subdaos`, `sage_list_proposals`, `sage_get_voting_power`
- `sage_chat_list_rooms`, `sage_chat_send`, `sage_chat_history`
- `sage_rlm_stats`, `sage_rlm_analyze_captures`, `sage_rlm_list_patterns`
- `sage_memory_create_entities`, `sage_memory_search_nodes`, `sage_memory_read_graph`
- `sage_hub_list_servers`, `sage_hub_start_server`, `sage_hub_stop_server`
- `sage_status` — bridge health, wallet, network, tool count

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
Use `sage_search_prompts`, `sage_search_skills`, and `sage_builder_recommend` before creating new content.

**2. Select & Group**  
Group selected skills/prompts in libraries and behaviors (`sage_list_libraries`, `sage_quick_create_prompt`, `sage_use_skill`).

**3. Execute**  
Run tools/skills through the MCP bridge with explicit scope and dependencies.

**4. Auto-improve (RLM)**  
Prompt and response hooks capture usage data that feeds RLM analysis (`sage_rlm_stats`, `sage_rlm_analyze_captures`, `sage_rlm_list_patterns`).

**5. Earn & Collaborate**
Curate outcomes through governance (`sage_list_proposals`, `sage_get_voting_power`), bounties, tips, and chat (`sage_chat_send`, `sage_chat_history`).

---

## Injection Guard (Opt-In)

Scans agent prompts and fetched prompt content for injection patterns:

```json
{
  "injectionGuardEnabled": true,
  "injectionGuardMode": "warn",
  "injectionGuardScanAgentPrompt": true,
  "injectionGuardScanGetPrompt": true
}
```

| Mode | Behavior |
|------|----------|
| `warn` | Log warning, continue execution |
| `block` | Block `sage_get_prompt` results that are flagged |

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
