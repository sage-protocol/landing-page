# Sage MCP Tool Reference

Complete reference for all MCP tools available through the Sage MCP server.

---

## Internal Servers (Always Loaded)

### sage:prompts

| Tool | Parameters | Description |
|------|-----------|-------------|
| `list_prompts` | `kind?` (prompt/skill/snippet), `source?` (local/library/synced), `limit?` | Browse prompts |
| `search_prompts` | `query`, `source?` (local/onchain/all), `limit?` | Hybrid keyword + semantic search |
| `get_prompt` | `key`, `library?` | Get full prompt content by key |
| `quick_create_prompt` | `name`, `content`, `description?`, `mcp_servers?` | Create a new prompt |
| `trending_prompts` | `window?` (day/week/month), `category?`, `limit?` | Popular prompts |
| `list_libraries` | `source?` (local/onchain/all) | List prompt libraries |

### sage:builder

| Tool | Parameters | Description |
|------|-----------|-------------|
| `builder_recommend` | `intent`, `limit?`, `session_id?` | AI-powered prompt suggestions |
| `builder_vote` | `cid`, `vote` (1/-1), `session_id?` | Rate a recommendation |
| `builder_synthesize` | `cids` (JSON array), `user_intent`, `session_id?` | Merge 2-5 prompts |

### sage:skills

| Tool | Parameters | Description |
|------|-----------|-------------|
| `list_skills` | `source?` (sage/github/local/all), `limit?` | List installed skills |
| `search_skills` | `query`, `source?` (installed/sage/github/all), `limit?` | Search for skills |
| `get_skill` | `key` | Get full skill details + SKILL.md content |
| `use_skill` | `key` | Activate a skill (auto-provisions MCP servers) |
| `sync_skills` | `source?` (local/project/all) | Scan directories for skills |

### sage:governance

| Tool | Parameters | Description |
|------|-----------|-------------|
| `list_subdaos` | none | List all DAOs |
| `list_proposals` | `state?` (pending/active/succeeded/defeated/executed), `subdao?` | View proposals |
| `get_voting_power` | `account` | Voting power breakdown with NFT multipliers |

### sage:chat

| Tool | Parameters | Description |
|------|-----------|-------------|
| `chat_list_rooms` | `type?` (global/cid/lib/dao/all), `limit?` | List active chat rooms |
| `chat_history` | `room_id`, `limit?`, `before?` | Get message history |
| `chat_send` | `room_id`, `content`, `reply_to?` | Send a message |
| `chat_watch` | `room_id`, `action?` (watch/unwatch) | Subscribe to room |
| `chat_watched` | none | Watched rooms with unread counts |

### sage:library-sync

| Tool | Parameters | Description |
|------|-----------|-------------|
| `sync_library_stream` | `dao` (required), `libraryId?` (default: "soul") | Fetch a DAO's soul/library document from IPFS, security-scan it, and write to `~/.local/share/sage/souls/<dao>-<libraryId>.md`. Blocks on HIGH/CRITICAL security findings. |

### sage:rlm

| Tool | Parameters | Description |
|------|-----------|-------------|
| `rlm_stats` | none | System statistics (analyses, patterns, costs, model state) |
| `rlm_analyze_captures` | `goal`, `session_id?`, `max_depth?` (default 3), `use_local_model?` | Analyze captures for patterns (up to 120s) |
| `rlm_list_patterns` | `pattern_type?` (best-practice/anti-pattern/tool-usage), `min_confidence?`, `session_id?`, `limit?` | List discovered patterns |

---

## External Servers (Start with `hub_start_server`)

### memory

| Tool | Parameters | Description |
|------|-----------|-------------|
| `memory_create_entities` | `entities` (JSON array) | Store entities in knowledge graph |
| `memory_search_nodes` | `query` | Search knowledge graph |
| `memory_open_nodes` | `names` (JSON array) | Retrieve entities by name |
| `memory_read_graph` | none | Read entire graph |
| `memory_add_observations` | `observations` (JSON array) | Add facts to entities |
| `memory_delete_entities` | `entityNames` (JSON array) | Delete entities |
| `memory_create_relations` | `relations` (JSON array) | Link entities |
| `memory_delete_relations` | `relations` (JSON array) | Remove links |
| `memory_delete_observations` | `deletions` (JSON array) | Remove facts |

### context7

| Tool | Parameters | Description |
|------|-----------|-------------|
| `resolve_library_id` | `libraryName` | Find Context7-compatible library ID |
| `get_library_docs` | `context7CompatibleLibraryID`, `topic?`, `tokens?` | Fetch library documentation |

### github (needs GITHUB_TOKEN)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `github_list_issues` | `owner`, `repo`, `state?`, `labels?` | List repo issues |
| `github_get_issue` | `owner`, `repo`, `issue_number` | Get issue details |
| `github_create_issue` | `owner`, `repo`, `title`, `body?`, `labels?` | Create issue |
| `github_search_issues` | `q`, `sort?`, `order?` | Search issues/PRs |
| `github_search_code` | `q`, `sort?`, `order?` | Search code |
| `github_get_file_contents` | `owner`, `repo`, `path`, `ref?` | Read file from repo |
| `github_create_pull_request` | `owner`, `repo`, `title`, `head`, `base`, `body?` | Create PR |

### brave (needs BRAVE_API_KEY)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `brave_web_search` | `query`, `count?`, `freshness?` | Web search |
| `brave_news_search` | `query`, `count?`, `freshness?` | News search |
| `brave_video_search` | `query`, `count?` | Video search |
| `brave_local_search` | `query`, `count?` | Local business search |
| `brave_summarizer_search` | `key` | AI summary of search results |

### duckduckgo (no API key needed)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `web_search` | `query`, `count?` | Free web search |

---

## Hub Management Tools

| Tool | Parameters | Description |
|------|-----------|-------------|
| `hub_list_servers` | none | List all available external servers |
| `hub_start_server` | `server_id`, `session_id?` | Start a server |
| `hub_stop_server` | `server_id`, `session_id?` | Stop a server |
| `hub_status` | none | Show running servers and tools |
| `hub_session_list` | `server_id?` | List active sessions |

---

## Context & Discovery

| Tool | Parameters | Description |
|------|-----------|-------------|
| `get_project_context` | none | Project state, wallet, libraries |
| `suggest_sage_tools` | `goal`, `stage?` | Tool recommendations for a goal |
