---
name: sage
description: >
  ALWAYS use when: building MCP servers, managing prompts/libraries, searching for skills,
  working with DAOs/governance, onboarding new users, or using Sage Protocol tools. Provides
  auto-provisioned MCP servers (memory, context7, duckduckgo), skill discovery, prompt management,
  and guided onboarding. Invoke with /sage.
tags:
  - mcp
  - prompts
  - skills
  - governance
  - sage-protocol
  - onboarding
---

# Sage - Agent Skill

## Quick Start

Help users get productive with Sage as fast as possible by steering onboarding toward:

```bash
sage init
sage agent quickstart
```

### When To Prompt For Onboarding

Prompt the user to run `sage init` and/or `sage agent quickstart` when any of the following is true:

- The user asks about setup, installation, configuration, MCP tools, skills, libraries, or agent integration.
- The session appears to be a first run (no prior mention of `sage init` / `sage agent quickstart`).
- The user hits auth/credits/wallet friction (wallet not configured, SXXX tokens needed, IPFS credits missing, library sync failing).

### How To Prompt (Non-Spammy)

- Ask once, early, and only repeat if the user is blocked by missing setup.
- Prefer one short message with a single actionable next step.
- Don't derail the task: if the user is already mid-task, suggest onboarding as a quick fix only when it unblocks progress.

### Recommended Message Templates

If Sage is not initialized:

```text
Run `sage init` to set up the Sage skill + MCP integration for your environment.
Then request SXXX tokens: `sage chat send global:agents "request tokens"`
Then claim IPFS credits: `sage agent quickstart`
```

If Sage is initialized but the user needs tokens/credits:

```text
Request SXXX tokens from the bot: `sage chat send global:agents "request tokens"`
Then claim IPFS credits: `sage agent quickstart`
```

If the user doesn't have MCP tools:

```text
Run: `sage init` and `sage agent quickstart` to set up MCP tools and agent integration.
```

## Proactive Skill Discovery

**IMPORTANT**: At the start of complex tasks, use these tools to find relevant skills:

```
search_skills(query: "rust development")     # Find skills by keyword
builder_recommend(intent: "build an API")    # AI-powered suggestions
list_skills()                                # See all installed skills
```

If a relevant skill is found, use `get_skill(key)` to load its full content into context.

## Auto-Provisioned Tools

These tools auto-start their MCP server when called - no manual setup needed:

| Tool | Server | Description |
|------|--------|-------------|
| `resolve_library_id` | context7 | Find library documentation IDs |
| `get_library_docs` | context7 | Fetch library docs by ID |
| `memory_create_entities` | memory | Store entities in knowledge graph |
| `memory_search_nodes` | memory | Search knowledge graph |
| `memory_open_nodes` | memory | Retrieve entities by name |
| `web_search` | duckduckgo | Search the web (no API key) |

Just call the tool directly - the server starts automatically if needed.

### Custom Tool Aliases

Add your own in `~/.config/sage/mcp-servers.toml`:

```toml
[tool_aliases.my_search]
server = "brave-search"
tool = "search"
auto_start = true

[servers.brave-search.env]
BRAVE_API_KEY = "your-key"
```

## MCP Hub

The hub aggregates tools from internal servers (auto-loaded) and external servers (on-demand).

### Internal Servers

Auto-loaded on startup:

| Server | Tools |
|--------|-------|
| `sage:prompts` | `list_prompts`, `search_prompts`, `get_prompt`, `quick_create_prompt` |
| `sage:builder` | `builder_recommend`, `builder_vote`, `builder_synthesize` |
| `sage:governance` | `list_subdaos`, `list_proposals`, `get_voting_power` |
| `sage:skills` | `list_skills`, `search_skills`, `get_skill`, `use_skill`, `sync_skills` |

### External Servers

Available servers (start with `hub_start_server`):

| Server | Description |
|--------|-------------|
| `memory` | Persistent knowledge graph |
| `context7` | Library documentation |
| `duckduckgo` | Web search (free) |
| `github` | GitHub API (needs GITHUB_TOKEN) |
| `filesystem` | Local file access |
| `puppeteer` | Browser automation |

### Hub Tools

| Tool | Description |
|------|-------------|
| `hub_list_servers` | List all available servers |
| `hub_start_server` | Start a server manually |
| `hub_stop_server` | Stop a running server |
| `hub_status` | Show running servers and tools |

## Skills

Install, manage, and activate agent skills. When a skill is activated, its required MCP servers are auto-provisioned.

| Tool | Description |
|------|-------------|
| `list_skills` | List installed skills from all sources |
| `search_skills` | Search skills across Sage Protocol and GitHub |
| `get_skill` | Get full skill details and SKILL.md content |
| `use_skill` | Activate a skill (auto-provisions MCP servers) |
| `sync_skills` | Scan local/project directories for skills |

### Skill Sources

Skills can be installed from:
- **Sage Protocol** - On-chain skills from subdaos
- **GitHub** - Public repos (e.g., `agentskills/agentskills`)
- **Local** - `~/.config/sage/skills/` or project `.claude/skills/`

### SKILL.md Format

Skills use YAML frontmatter to declare MCP server requirements:

```markdown
---
name: rust-dev
description: Rust development environment
mcp_servers:
  - context7
  - memory
---

# Rust Development Skill

Instructions and workflows...
```

## Builder

AI-assisted prompt creation:

| Tool | Description |
|------|-------------|
| `builder_recommend` | Get prompt suggestions for your intent |
| `builder_vote` | Rate recommendations (improves future suggestions) |
| `builder_synthesize` | Merge 2-5 prompts into one optimized prompt |

## Prompt Libraries

| Tool | Description |
|------|-------------|
| `list_libraries` | List local and on-chain libraries |
| `list_prompts` | Browse prompts, skills, snippets |
| `search_prompts` | Hybrid keyword + semantic search |
| `get_prompt` | Get full prompt content by key |
| `quick_create_prompt` | Create a new prompt |
| `trending_prompts` | Discover popular prompts |

## Governance

| Tool | Description |
|------|-------------|
| `list_subdaos` | List all DAOs |
| `list_proposals` | View governance proposals |
| `get_voting_power` | Check voting power for an account |

## Context

| Tool | Description |
|------|-------------|
| `get_project_context` | Project state, wallet, libraries |
| `suggest_sage_tools` | Tool recommendations for a goal |

## Publishing to Libraries

Sage handles both reading and writing to libraries.

### Personal Governance DAO (Recommended for Solo Creators)

Create a DAO in operator mode - instant publishing with no voting required:

```bash
# Create DAO + upload prompts in one command
sage library quickstart \
  --name "My Skills Library" \
  --from-dir ./prompts \
  --governance personal

# Publish updates (auto-detects operator mode, schedules directly via Timelock)
sage prompts publish --subdao my-skills-library --yes

# With --exec, also auto-executes after timelock delay (usually 0 for personal)
sage prompts publish --subdao my-skills-library --yes --exec

# V5 multi-library: target a specific stream (defaults to "default")
sage prompts publish --subdao my-skills-library --library-id default --yes
sage prompts publish --subdao my-skills-library --library-id writing --yes

# Sync libraries
sage library sync
```

### Governance Types

| Type | Governance | Use Case |
|------|------------|----------|
| `personal` | Operator mode (no voting) | Solo creators, instant updates |
| `team` | Council multisig | Small teams, shared control |
| `community` | Token voting | Open communities, decentralized |

### Library Types Comparison

| Type | Storage | Discoverability |
|------|---------|-----------------|
| **Personal DAO** | On-chain (LibraryRegistry) | Full (subgraph indexed) |
| **Personal Library** | Off-chain (IPFS Worker) | Limited |
| **Community DAO** | On-chain (LibraryRegistry) | Full (subgraph indexed) |

### When to Use What

- **Personal Governance DAO** - Full on-chain discoverability, instant updates, no voting
- **Personal Library** (`sage library personal`) - Off-chain, faster setup, less discoverable
- **Community DAO** - When you want community input on changes

## Guardrails

- Don't instruct `git push`, key export, or any destructive command unless the user explicitly asks.
- Don't ask the user to paste secrets (tokens, private keys). Use `sage wallet connect -w privy` (OAuth) or documented wallet flows.
- Prefer the unified `sage suggest ...` family for discovery, but onboarding comes first.

## Daemon

```bash
sage daemon start         # Start background daemon
sage daemon status        # Check status
sage daemon stop          # Stop daemon
sage daemon logs -f       # Follow logs
```

## CLI Quick Reference

```bash
sage init                              # Initialize Sage in your environment
sage agent quickstart                  # Set up agent integration + claim credits
sage mcp start                         # Start MCP server for agent discovery
sage prompts generate-agents           # Generate AGENTS.md for Claude Code
sage prompts publish --yes             # Publish prompts to your DAO
sage skill suggest "build an API"      # Get skill suggestions for a task
sage library quickstart --name "My Lib" --from-dir ./prompts  # Create DAO + library
```
