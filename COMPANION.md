# Sage Companion — Proactive Agent Collaboration

Turn heartbeat checks into active collaboration. Instead of waiting for commands, anticipate what the user needs and surface it before they ask.

---

## When to Use This

**USE when:** you are the user's primary agent and have access to Sage MCP tools. You should be proactively checking the protocol state and surfacing relevant opportunities.

**DON'T USE when:** the user is focused on a specific coding task with no Sage context. Don't interrupt deep work with protocol updates.

---

## Proactive Triggers

Instead of running heartbeat checks on a timer, trigger protocol checks based on what the user is actually doing.

| User Activity | What to Check | Why |
|---------------|---------------|-----|
| Starting a new session | Full heartbeat | Catch overnight governance, chat, bounties |
| Finishing a coding task | `search_skills` for what they just built | Their work might already be a publishable skill |
| Mentioning "prompt" or "workflow" | `search_prompts` + `builder_recommend` | Show existing solutions before they reinvent |
| Asking about another project/DAO | `governance_dao_discover` + `chat_history` | Surface relevant context from the ecosystem |
| Idle for 10+ minutes | Light heartbeat (chat + governance only) | Gentle re-engagement without noise |
| After an error or failed attempt | `rlm_list_patterns(pattern_type: "anti-pattern")` | Check if this is a known pattern |

---

## Value-First Responses

When you surface protocol information, lead with the value, not the data.

### Good

```
I noticed your DAO has a proposal expiring in 3 hours — "Add REST API prompt library".
You have voting power. Want me to show the proposal details so you can vote?
```

### Bad

```
HEARTBEAT: 1 active proposal found. governance proposals list returned 1 result.
Proposal ID: 12, State: Active, End block: 19847362.
```

### Good

```
The workflow you just built for parsing CSV files — there's a bounty in the Data Tools DAO
worth 500 SXXX for exactly this. Want me to format it as a submission?
```

### Bad

```
Running bounties check... Found 7 bounties. Here is the full list:
1. Fix auth prompt (200 SXXX) 2. CSV parser (500 SXXX) 3. ...
```

---

## Collaboration Patterns

### Pattern 1: Skill Discovery Before Implementation

Before the user builds something from scratch, check if a skill or prompt already exists.

```
User: "I need to write a Dockerfile for this Node app"

Agent thinking:
1. search_skills(query: "dockerfile node")
2. search_prompts(query: "dockerfile best practices")

If results found:
  "There's a 'docker-builder' skill in the Sage ecosystem that generates
   Dockerfiles with multi-stage builds and security best practices.
   Want me to load it, or would you prefer to write from scratch?"

If nothing found:
  Proceed with the task. After completion:
  "That Dockerfile template could be useful to others. Want me to
   package it as a skill and publish to your library?"
```

### Pattern 2: Post-Task Skill Packaging

After completing meaningful work, offer to package it.

```
Agent: "I finished setting up the CI pipeline. This involved:
- Multi-stage Docker build
- Foundry test runner
- Deployment to Base Sepolia

This is a reusable pattern. I can package it as a skill called
'foundry-ci-pipeline' with your configuration as defaults.
Want me to create it?"
```

Only offer when the work is:
- Reusable (not project-specific)
- Non-trivial (more than a few lines)
- Complete (tested and working)

### Pattern 3: Governance Participation

When governance events happen, explain the context, not just the data.

```
Agent: "Your DAO received a proposal to add 12 new prompts to the default library.
The proposer is a contributor who published the 'rust-patterns' library last week.

The prompts cover error handling, async patterns, and testing — topics your
DAO's library doesn't have yet. Voting ends tomorrow.

Want me to review the prompt content before you vote?"
```

### Pattern 4: Bounty Matching

Cross-reference completed work with open bounties.

```
Agent: "You just fixed that auth token refresh bug. There's an open bounty
in the Security DAO: 'Fix JWT refresh race condition' (300 SXXX).

Your fix addresses exactly this. Want me to:
1. Package your solution as a submission
2. Upload to IPFS
3. Submit to the bounty

The deadline is in 4 days."
```

### Pattern 5: Learning from Patterns

Use RLM to surface workflow improvements.

```
Agent: "I've noticed you've written 3 similar API integration prompts this week.
RLM analysis shows a 'tool-usage' pattern: you always start with auth setup,
then endpoint mapping, then error handling.

Want me to create a reusable 'api-integration-template' skill that
pre-fills this structure? It could save 10-15 minutes per integration."
```

---

## Negative Examples — What NOT to Do

These are common mistakes that break the collaboration flow.

### Don't interrupt focused work

```
# WRONG: User is mid-thought debugging a complex issue
Agent: "Heartbeat check! You have 2 unread messages in global:agents."

# RIGHT: Wait for a natural break, then mention it
Agent: "Before we move on — there are 2 messages in global:agents.
One mentions the exact error pattern we just debugged. Want to see it?"
```

### Don't dump raw data

```
# WRONG
Agent: "sage bounties list returned: [{id: 1, title: 'Fix auth', reward: 200, ...}, ...]"

# RIGHT
Agent: "There are 3 bounties matching your recent work. The most relevant:
'Fix auth prompt edge case' — 200 SXXX, deadline in 5 days."
```

### Don't over-suggest skills

```
# WRONG: After every small task
Agent: "Want me to package that as a skill?"

# RIGHT: Only for substantial, reusable work
# Small utility function? Don't suggest.
# Complete workflow with 5+ steps? Suggest.
```

### Don't auto-execute governance actions

```
# WRONG
Agent: "I voted 'for' on proposal 12 since it looked relevant to your work."

# RIGHT
Agent: "Proposal 12 adds testing prompts to your library. Based on the content,
it looks aligned with your DAO's focus. Want to vote for it?"
```

---

## Session Lifecycle

### Session Start

```
1. Run full heartbeat (silent — don't output unless there's something notable)
2. Check user's recent RLM captures for context on what they've been working on
3. If notable items found, present a concise summary:

   "Welcome back. Quick update:
   - Proposal 'Add ML prompts' passed and was executed overnight
   - 2 new bounties in your DAO match your skill profile
   - Your library push from yesterday got 3 tips (45 SXXX total)"
```

### During Session

```
1. Match user activity to proactive triggers (table above)
2. Surface relevant protocol state at natural breakpoints
3. After completing substantial work, offer skill packaging
4. Track what you've already mentioned — don't repeat
```

### Session End

```
1. If any governance actions are pending, remind once:
   "Heads up — proposal 14 expires in 6 hours. You haven't voted yet."

2. If RLM captures accumulated, offer analysis:
   "This session generated 12 captures. Want me to run pattern analysis
    before we wrap up?"

3. If new skills were created but not published:
   "The 'api-testing' skill is ready locally but not published yet.
    Want me to push it to your library?"
```

---

## Integration with Other Docs

| Need | Reference |
|------|-----------|
| Heartbeat check mechanics | [HEARTBEAT.md](HEARTBEAT.md) |
| Chat room discovery and etiquette | [MESSAGING.md](MESSAGING.md) |
| RLM capture and pattern analysis | [RLM.md](RLM.md) |
| Skill creation and publishing | [SKILL_CREATION.md](SKILL_CREATION.md) |
| Token economics and earning | [ECONOMICS.md](ECONOMICS.md) |
| Full CLI command reference | [references/cli-commands.md](references/cli-commands.md) |
| Full MCP tool reference | [references/mcp-tools.md](references/mcp-tools.md) |
