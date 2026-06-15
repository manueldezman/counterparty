# Pharos Routing

Counterparty routes to downstream Pharos skills after scoring trust and risk. It does not execute the downstream action itself; the calling agent passes `nextAction` to the selected downstream skill, which handles confirmation, signing, submission, and status reporting.

## Payment

Use `send_transaction` only when the recommendation allows value transfer.

```json
{
  "recommendation": "proceed",
  "nextAction": {
    "skill": "send_transaction",
    "type": "native_token_transfer"
  }
}
```

After this route is returned, the agent calls `send_transaction` with the `nextAction` fields so that skill can execute the transfer.

For incomplete trust, route to verification or escrow.

```json
{
  "recommendation": "request_additional_verification",
  "nextAction": {
    "skill": "identity_verification",
    "type": "counterparty_verification",
    "requiresUserConfirmation": true
  }
}
```

## Swap

Use `swap` when the route, pool, contract, or protocol passes trust checks.

```json
{
  "recommendation": "proceed",
  "nextAction": {
    "skill": "swap"
  }
}
```

If the route uses a risky pool or contract, return a safer recommendation and request a different route.

```json
{
  "recommendation": "use_alternative_path",
  "nextAction": {
    "skill": "swap",
    "type": "find_alternative_route",
    "avoid": "0xabc..."
  }
}
```

## Stake or Deposit

Use `stake` for staking flows and `contract_interaction` for protocol deposits when trust is acceptable.

```json
{
  "recommendation": "proceed_with_caution",
  "nextAction": {
    "skill": "stake",
    "requiresUserConfirmation": true
  }
}
```

## Contract Call

Use `contract_interaction` for acceptable contract calls. Require explicit confirmation when risk is not low.

```json
{
  "recommendation": "proceed_with_caution",
  "nextAction": {
    "skill": "contract_interaction",
    "requiresUserConfirmation": true
  }
}
```

## Delegation

Use `identity_verification` when agent identity, reliability, or authority is not strong enough.

```json
{
  "recommendation": "request_additional_verification",
  "nextAction": {
    "skill": "identity_verification",
    "type": "agent_attestation",
    "requiresUserConfirmation": true
  }
}
```

## High Risk

Use `escrow` when the user still needs a protected route.

```json
{
  "recommendation": "use_alternative_path",
  "nextAction": {
    "skill": "escrow",
    "type": "protected_interaction",
    "requiresUserConfirmation": true
  }
}
```

## Critical Risk

Use `none` and halt the action.

```json
{
  "recommendation": "reject",
  "nextAction": {
    "skill": "none",
    "type": "halt"
  }
}
```
