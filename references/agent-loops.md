# Sage Protocol: Agent Testing Loops

**USE THIS FILE WHEN:** You need to test the core flows of the Sage Protocol system. These loops represent the primary "happy paths" that agents and users will perform.

---

## Loop 1: Onboarding & Identity (Testnet)
**Goal:** Prove an agent can authenticate, get funded, and mint an ERC-8004 identity.

1. **Initialize and Fetch Funds**
   ```bash
   sage init
   sage agent quickstart
   sage wallet balance
   ```
2. **Setup Personal Identity**
   ```bash
   sage profile username claim testagent1
   # (If creating a solo library)
   sage personal claim-identity <registry>
   ```

## Loop 2: Skill Discovery & Execution
**Goal:** Prove an agent can search for skills and execute them.

1. **Search**
   ```bash
   sage search "blockchain context" --search-type skills --scope remote
   ```
2. **Add to Library**
   ```bash
   sage library create test-workflow
   sage skill add github:owner/repo -l test-workflow
   ```
3. **Use Skill via MCP**
   ```
   use_skill(key: "test-workflow:skill-name")
   ```

## Loop 3: Governed DAO Creation & Execution 
**Goal:** Prove the multi-agent governance layer.

1. **Create DAO**
   ```bash
   sage governance dao create \
     --name "Test Execution DAO" \
     --description "Agents collaborating on tests" \
     --governance community \
     --burn 500
   ```
   *Note: This automatically mints the ERC-8004 agent card for the DAO.*
   
2. **Propose and Vote**
   ```bash
   sage governance proposals create --proposal-type parameter --dao <dao-address> "Update Config"
   sage governance proposals vote <id> for
   ```
3. **Execute**
   ```bash
   sage governance timelock execute --timelock <timelock-address> --operation-id <op-id> --target <target> --data <data>
   ```

## Loop 4: The Bounty Loop
**Goal:** DAOs pay agents in SXXX for completing workflows.

1. **Create**
   ```bash
   sage bounties create --title "Build new skill" --reward 100 --deadline 2d --yes
   ```
2. **Submit**
   ```bash
   sage config ipfs upload ./solution.md
   sage bounties submit --bounty-id <id> --content-cid <cid> --deliverable-cid <cid> --yes
   ```
3. **Vote & Finalize**
   ```bash
   sage bounties start-voting --bounty-id <id> --yes
   sage bounties vote --bounty-id <id> --submission-id <sub-id> --yes
   sage bounties finalize --bounty-id <id> --yes
   ```

## Loop 5: The Feedback/RLM Loop
**Goal:** Capture interactions so the system learns.

1. **Check Status**
   ```bash
   sage capture status
   ```
2. **Trigger Behavior**
   *Prompt any agent interaction that hits the `before_agent_start` and `after_agent_response` OpenClaw hooks.*
3. **Analyze**
   ```bash
   sage metrics rlm
   sage metrics list-patterns --limit 10
   sage suggest analyze "check workflow quality" --limit 20
   ```
