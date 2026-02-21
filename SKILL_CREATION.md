# Collaborative Skill Creation

How agents and users work together to turn workflows into publishable skills.

---

## When to Use This

**USE when:** you've completed meaningful, reusable work and want to package it as a Sage skill — or when RLM patterns suggest a recurring workflow that should be formalized.

**DON'T USE when:** the work is project-specific, trivial (under 10 lines), or the user hasn't expressed interest in publishing.

---

## The Skill Creation Pipeline

```
Discover → Draft → Test → Publish → Govern
   ↑                                    |
   └────────── RLM feedback ───────────┘
```

### Prompts vs. Skills vs. MCP Dependencies (Crucial Distinction)

Before creating content, you must understand the difference between the three layers of the Sage Ecosystem. **Agents often confuse these.**

1. **Prompts:** 
   - A raw string of text instructions (e.g., "Summarize this PR").
   - Can be used by humans or LLMs out of the box. 
   - Managed via `sage quick-create prompt`.

2. **Skills:** 
   - A structured `.md` document (like this one) that contains a Prompt *plus* execution routing logic (Frontmatter).
   - Skills tell an agent *when* to use the underlying instructions (`USE WHEN` / `DON'T USE WHEN`).
   - Managed via `sage skill add` or `sage library skill add`.

3. **MCP Dependencies (Execution Targets):** 
   - External servers (like GitHub, AWS, Playwright) that provide the actual *tools* the agent needs to execute the Skill.
   - **Crucial Rule:** A skill *can* define how to use a local CLI (like the `sage` CLI or `git`). However, if your Skill requires an agent to interact with an external API or service that *isn't* exposed via a local CLI (like searching a private GitHub repo or provisioning AWS resources), the Skill must *declare an MCP dependency* on that server so the Sage Hub can auto-provision it.

#### How to Add MCP Dependencies to a Skill Manifest

If your drafted Skill requires external tools, you **must** declare them in the YAML frontmatter under the `mcp_server_configs` block. 

When an agent loads this skill, the Sage Hub will automatically attempt to provision and connect these servers via the Portal.

```yaml
---
name: my-github-reviewer-skill
version: 1.0.0
description: >
  USE WHEN: the user asks to review a GitHub PR.
mcp_server_configs:
  github:
    source: 
      type: "npx"
      package: "@modelcontextprotocol/server-github"
    env_requirements:
      - name: "GITHUB_PERSONAL_ACCESS_TOKEN"
        description: "Requires repo read access"
  puppeteer:
    source:
      type: "npx"
      package: "@modelcontextprotocol/server-puppeteer"
---
```
*(If the user's daemon lacks the required `env_requirements`, it will drop into `PendingAuthorization` state and prompt the user via CLI to provide the keys securely).*

---

### 1. Discover: Identify Skill-Worthy Work

Skills emerge from three sources:

#### Source A: Post-Task Recognition

After completing substantial work, evaluate it:

```
Criteria:
- [ ] Reusable across projects (not hardcoded to one repo)
- [ ] Non-trivial (solves a real problem, not just boilerplate)
- [ ] Complete (tested, working, handles edge cases)
- [ ] Teachable (can be described in a prompt that another agent follows)

If 3+ checked → offer to package as skill
If 2 checked → note it, revisit after more instances
If 1 or fewer → skip
```

#### Source B: RLM Pattern Detection

Analyze captures for recurring workflows:

```bash
# Find best-practice patterns with high confidence
sage capture summary --top-n 10
```

Via MCP:
```
rlm_list_patterns(pattern_type: "best-practice", min_confidence: 0.7)
```

Look for patterns that appear 3+ times across sessions. These are strong skill candidates.

#### Source C: Bounty Requirements

Open bounties often describe exactly what skill is needed:

```bash
sage bounties list --subdao 0x<target-dao>
```

Match bounty descriptions to your capabilities. If you can build what they're asking for, the skill and the bounty submission are the same deliverable.

---

### 2. Draft: Write the Skill

A skill is a markdown file with YAML frontmatter. Create it locally:

```bash
mkdir -p ~/.sage/skills/my-new-skill
```

#### Skill Structure

```markdown
---
name: my-new-skill
version: 1.0.0
description: >
  One-line description of what this skill does.

  USE WHEN: specific trigger conditions.
  DON'T USE WHEN: conditions where this skill is wrong.
tags:
  - relevant
  - keywords
---

# Skill Name

Brief explanation of what this skill accomplishes and why it's useful.

## Quick Start

Minimal steps to use this skill immediately.

## Detailed Instructions

Step-by-step workflow with examples.

## Templates

Reusable output templates for consistent results.

## Negative Examples

What NOT to do — common mistakes and why they fail.
```

#### Writing Tips (from OpenAI Skills Best Practices)

1. **Routing logic in the description**: The `USE WHEN` / `DON'T USE WHEN` in frontmatter determines when agents load this skill. Be specific.

   ```yaml
   # Good
   description: >
     USE WHEN: deploying Solidity contracts to Base with Foundry.
     DON'T USE WHEN: writing frontend code, using Hardhat, or deploying to non-EVM chains.

   # Bad
   description: "A skill for blockchain development"
   ```

2. **Negative examples**: Tell the agent what NOT to do. These prevent the most common failures.

   ```markdown
   ## What NOT to Do

   ### Don't skip the dry-run
   ```bash
   # WRONG: Deploy directly
   forge script Deploy.s.sol --broadcast

   # RIGHT: Always dry-run first
   forge script Deploy.s.sol
   # Review output, then:
   forge script Deploy.s.sol --broadcast
   ```
   ```

3. **Templates inside the skill**: Provide output format templates so results are consistent.

   ```markdown
   ## Output Template

   When reporting deployment results, use this format:

   ```
   Deployment Complete
   - Contract: {name} at {address}
   - Network: {network} (chain ID: {chainId})
   - Gas used: {gas} ({cost} ETH)
   - Verified: {yes/no}
   ```
   ```

4. **Design for long runs**: If the skill involves multi-step workflows, include checkpoints.

   ```markdown
   ## Checkpoints

   After each step, verify before continuing:
   1. After compilation: check for warnings
   2. After dry-run: review simulated transactions
   3. After broadcast: verify on block explorer
   4. After verification: confirm source matches
   ```

---

### 3. Test: Validate the Skill

#### Local Testing

Install the skill locally and test it:

```bash
# Install from local path
sage skill add ./my-new-skill

# Verify it appears
sage skill list

# Test suggestion matching
sage skill suggest "the task this skill should match"

# Load and verify content
sage skill get my-new-skill
```

#### Agent Testing

Ask an agent to use the skill on a real task:

```
"Use the my-new-skill skill to [describe the task].
Follow the skill instructions exactly. Report any steps
that are unclear, missing, or wrong."
```

Fix issues and re-test until the skill produces consistent results across 3+ different inputs.

---

### 4. Publish: Push to Library

#### To Personal Library

```bash
# Add skill to your personal library
sage library skill add ./my-new-skill -l personal

# Push library to IPFS
sage library personal push
```

Via MCP:
```
library_skill_add(path: "./my-new-skill", library: "personal")
library_personal_push()
```

#### To DAO Library

Publishing to a DAO library goes through governance:

```bash
# Add skill to a named library
sage library skill add ./my-new-skill -l my-dao-lib

# Push to IPFS (returns CID)
sage library push my-dao-lib
# Output: Pushed to IPFS: bafkrei...

# Create governance proposal
sage library promote my-dao-lib \
  --dao 0x<your-dao> \
  --collection default \
  --yes
```

If you're the operator:
```bash
# Promote + auto-execute (skips voting for operator-mode DAOs)
sage library promote my-dao-lib \
  --dao 0x<your-dao> \
  --collection default \
  --yes --exec
```

#### One-Step Quickstart (New DAOs)

For users who don't have a DAO yet:

```bash
sage library quickstart \
  --name "My Prompt Library" \
  --from-dir ./my-skills-directory \
  --governance personal
```

This creates a DAO, uploads skills, and publishes in one command.

---

### 5. Govern: Iterate with Community

After publishing, the skill enters the governance cycle:

- **DAO members review** the skill content
- **Proposals** to update the library go through voting
- **Tips** from users who find the skill valuable go to the creator
- **RLM feedback** from agents using the skill improves scoring

Monitor your published skills:

```bash
# Check tips received
sage tips list --recipient 0x<your-address>

# Check your reputation
# Via API: GET /reputation/author/<your-address>

# See if your library was updated by governance
sage governance proposals list --dao 0x<your-dao>
```

---

## Collaborative Workflow (Agent + User)

The ideal flow when an agent identifies a skill opportunity:

```
Agent: "The CSV parser you just built could be a useful skill.
       It handles 5 formats and has proper error handling.
       Want me to draft a skill file?"

User: "Sure, go ahead."

Agent: [Creates skill markdown with frontmatter, instructions,
        templates, and negative examples]
      "Here's the draft. I've included:
       - Routing: USE WHEN parsing CSV/TSV/delimited files
       - 3 templates for different output formats
       - Negative examples for common encoding issues

       Want to review it before I publish to your library?"

User: "Looks good. Publish it."

Agent: [Installs locally, tests suggestion matching, pushes to library]
      "Published to your personal library. CID: bafkrei...

       If you want to share it with your DAO, I can create a
       governance proposal to add it to the default collection."
```

---

## Skill Quality Checklist

Before publishing, verify:

| Check | Requirement |
|-------|-------------|
| Frontmatter | `name`, `version`, `description` with USE WHEN/DON'T USE WHEN, `tags` |
| Quick Start | Can someone use this in under 2 minutes? |
| Instructions | Step-by-step, no assumed knowledge |
| Templates | At least one output template for consistent results |
| Negative Examples | At least 2 "what not to do" examples |
| Testing | Works on 3+ different inputs |
| Reusability | No hardcoded paths, credentials, or project-specific values |

---

## Integration with Other Docs

| Need | Reference |
|------|-----------|
| Proactive skill suggestions | [COMPANION.md](COMPANION.md) |
| RLM pattern analysis | [RLM.md](RLM.md) |
| Library and governance workflow | [SKILL.md](SKILL.md) |
| Token economics for tips | [ECONOMICS.md](ECONOMICS.md) |
| Full CLI commands | [references/cli-commands.md](references/cli-commands.md) |
| Full MCP tools | [references/mcp-tools.md](references/mcp-tools.md) |
