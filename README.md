# Counterparty

## Introduction

Skills are reusable instruction packages that help AI agents perform specialized work with consistent workflows, domain rules, and bundled references. Counterparty is a Pharos ecosystem skill for assessing trust before economic actions.

Counterparty evaluates wallets, smart contracts, protocols, and agents before payments, swaps, staking, protocol deposits, contract calls, or delegation. It does not execute transactions. It returns a trust assessment, explains the risk, and recommends the safest downstream skill to call next.

Tagline: Assess trust before value moves.

## Installation Command

Install the skill by copying the `counterparty/` folder into the skills directory for your agent environment.

### OpenClaw

```bash
cp -R counterparty ~/.openclaw/skills/
```

### Claude Code

```bash
cp -R counterparty ~/.claude/skills/
```

### Codex

```bash
cp -R counterparty ~/.codex/skills/
```

For project-level usage, keep the `counterparty/` folder in the project root so the agent can load it with the project context.

## Skill Guidance

### 1. Skill File Storage Path

**OpenClaw**

- Global skills directory: `~/.openclaw/skills/`
- Skills installed via clawhub are stored globally.
- Project-specific skills can be placed in the project root.

**Claude Code**

- Download skills directory: `~/.claude/skills/`
- Claude Code loads skills from global and project-level directories.
- Each skill is a folder containing `SKILL.md` with frontmatter metadata.

**Codex**

- Download skills directory: `~/.codex/skills/`
- Codex reads skill definitions from the skills directory.
- Project-level skills override global skills with the same name.

### 2. Verify Skill Installation in Terminal

**OpenClaw**

```bash
openclaw skills list
```

If `counterparty` appears in the list with a green checkmark, it is installed and ready to use.

**Claude Code**

```text
/skills
```

Type `/skills` in Claude Code to see loaded skills. Installed skills also appear in the system reminder when a new session starts.

**Codex**

```text
/skills
```

Type `/skills` in Codex to list available skills. Codex loads skills when a session starts, so begin a new session after installing.

### 3. How an Agent Uses Counterparty

**Step A: Write Your Prompt**

Describe the intended economic action naturally.

```text
You: Assess whether I should send 100 PHRS to 0x123... before making the payment.
```

**Step B: Agent Selects Skill**

The agent matches the task to Counterparty and announces the skill.

```text
Agent: Using counterparty to assess trust, risk, and the safest next action.
```

**Step C: Skill Returns a Route**

Counterparty evaluates available trust signals and returns structured JSON with a score, recommendation, reasons, and downstream routing.

```json
{
  "target": "0x123...",
  "targetType": "wallet",
  "action": "payment",
  "trustScore": 82,
  "riskLevel": "low",
  "recommendation": "proceed",
  "reasons": [
    "counterparty has consistent activity",
    "no suspicious interaction patterns detected"
  ],
  "nextAction": {
    "skill": "send_transaction",
    "type": "native_token_transfer",
    "to": "0x123...",
    "amount": "100",
    "asset": "PHRS"
  }
}
```

**Step D: Downstream Skill Executes the Transaction**

If the recommendation allows the action, the agent passes `nextAction` to the named downstream skill. For this example, the agent calls `send_transaction`, then that skill handles user confirmation, signing, submission, and transaction status.

```text
Agent: Trust check passed. Calling send_transaction to execute the 100 PHRS transfer after confirmation.
```
