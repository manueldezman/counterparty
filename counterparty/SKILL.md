---
name: counterparty
description: >
  Evaluates wallets, smart contracts, protocols, and agents before economic interactions.
  Use this skill when an agent needs to assess trust, risk, and the safest next action
  before sending funds, swapping assets, interacting with a contract, depositing into
  a protocol, staking, or delegating work to another agent.
version: 0.1.0
requires:
  anyBins:
    - cast
---

# Counterparty

## Purpose

Counterparty is a reusable trust-assessment and decision-routing skill for the Pharos AI Agent ecosystem.

Vision: Enable every Pharos agent to assess trust before taking economic action.

Tagline: Assess trust before value moves.

Counterparty evaluates wallets, smart contracts, protocols, and agents before economic interactions. It does not execute transactions directly. It returns a trust assessment and recommends the safest downstream skill to call next; the agent should then call that downstream skill to build, confirm, and execute the transaction when the recommendation allows it.

## When to Use This Skill

Use Counterparty before:

- Sending funds or tokens to a wallet.
- Swapping assets through a route, pool, or protocol.
- Staking or depositing into a protocol.
- Calling a smart contract.
- Delegating work to another agent.
- Choosing between direct transfer, escrow, identity verification, or no action.

## When Not to Use This Skill

Do not use Counterparty to:

- Execute transactions.
- Sign messages or approvals.
- Fetch swap quotes as the primary task.
- Deploy contracts.
- Replace formal audits, legal review, compliance review, or user consent.
- Produce a trust score without enough evidence or uncertainty disclosure.

## Input Expectations

Expect enough context to evaluate the intended interaction.

```json
{
  "target": "0x123...",
  "targetType": "wallet | contract | protocol | agent",
  "action": "payment | swap | stake | deposit | contract_call | delegation",
  "amount": "100",
  "asset": "PHRS"
}
```

`amount` and `asset` may be omitted when they do not apply to the action. If the target, target type, or intended action is missing, request the missing context before scoring.

Optional context can include chain, contract address, function selector, protocol name, previous interactions, source verification, audit status, admin controls, known identity, user risk tolerance, and available onchain evidence.

## Output Format

Always return machine-readable JSON. Do not wrap the final answer in prose when returning the assessment.

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

Required fields:

- `target`: wallet, contract, protocol, or agent identifier.
- `targetType`: `wallet`, `contract`, `protocol`, or `agent`.
- `action`: intended economic action.
- `trustScore`: integer from `0` to `100`.
- `riskLevel`: `low`, `medium`, `elevated`, `high`, or `critical`.
- `recommendation`: `proceed`, `proceed_with_caution`, `request_additional_verification`, `use_alternative_path`, or `reject`.
- `reasons`: non-empty list of evidence-based reasons.
- `nextAction`: structured downstream routing object.

## Workflow

1. Parse the target, target type, intended action, amount, asset, and chain context.
2. Gather available trust signals with tools such as `cast` or trusted Pharos data sources when available.
3. Score the target using the model in `references/trust-model.md`.
4. Reduce confidence when evidence is missing, ambiguous, stale, or contradictory.
5. Map the score to recommendation and risk level using `assets/risk-rules.json`.
6. Select the safest downstream route using `references/pharos-routing.md`.
7. Return only the structured JSON assessment.
8. After returning a safe `nextAction`, the agent may call the downstream skill to perform the action, including confirmation, signing, submission, and status tracking.

## Rules

- Do not execute transactions inside Counterparty.
- Do not sign messages, grant approvals, or submit contract calls.
- Do not hallucinate trust when data is missing.
- If uncertain, return `request_additional_verification`.
- Always return reasons.
- Always return a structured `nextAction`.
- Prefer escrow or identity verification when funds are material and trust is incomplete.
- Use `none` with `type: "halt"` for rejected or critical-risk actions.
- Keep the skill focused on trust assessment and decision routing.

## Recommendation Mapping

Use this mapping exactly:

- `80-100`: `proceed`, risk level `low`
- `60-79`: `proceed_with_caution`, risk level `medium`
- `40-59`: `request_additional_verification`, risk level `elevated`
- `20-39`: `use_alternative_path`, risk level `high`
- `0-19`: `reject`, risk level `critical`

## Example Requests and Responses

Request:

```text
Assess whether I should send 100 PHRS to 0x123...
```

Response:

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

Request:

```text
Should I call this newly deployed contract with an unknown admin?
```

Response:

```json
{
  "target": "0xabc...",
  "targetType": "contract",
  "action": "contract_call",
  "trustScore": 37,
  "riskLevel": "high",
  "recommendation": "use_alternative_path",
  "reasons": [
    "contract is newly deployed",
    "admin controls are unclear",
    "source verification or usage history is missing"
  ],
  "nextAction": {
    "skill": "escrow",
    "type": "protected_interaction",
    "requiresUserConfirmation": true
  }
}
```

Request:

```text
Evaluate whether this agent can receive delegated treasury work.
```

Response:

```json
{
  "target": "agent:treasury-runner",
  "targetType": "agent",
  "action": "delegation",
  "trustScore": 55,
  "riskLevel": "elevated",
  "recommendation": "request_additional_verification",
  "reasons": [
    "agent has some historical activity",
    "reliability evidence is incomplete",
    "delegation would authorize economically sensitive work"
  ],
  "nextAction": {
    "skill": "identity_verification",
    "type": "agent_attestation",
    "requiresUserConfirmation": true
  }
}
```

## Routing Behavior to Downstream Pharos Skills

Route recommendations to downstream skills only after the trust assessment is complete.

- Payment: route to `send_transaction` when safe.
- Swap: route to `swap` when the route or protocol is acceptable.
- Stake or deposit: route to `stake` or `contract_interaction` when protocol risk is acceptable.
- Contract call: route to `contract_interaction` when safe and require confirmation when risk is non-low.
- Delegation: route to the relevant agent skill only when trust is sufficient; otherwise route to `identity_verification`.
- Elevated uncertainty: route to `identity_verification` or `escrow`.
- High risk: route to `escrow` or `none`.
- Critical risk: route to `none` with `type: "halt"`.

When `recommendation` is `proceed` or `proceed_with_caution`, the calling agent should pass `nextAction` to the named downstream skill. That downstream skill is responsible for transaction preparation, user confirmation, signing, submission, and receipt/status reporting.

See `references/pharos-routing.md` for examples.
