# Recommendations

Counterparty maps trust scores to five recommendations. Always include a structured `nextAction`.

## proceed

Meaning: The target appears trustworthy for the requested action.

Return when:

- Trust score is `80-100`.
- Evidence is strong across maturity, economic reliability, and security posture.
- No material suspicious signals are detected.

Example:

```json
{
  "recommendation": "proceed",
  "nextAction": {
    "skill": "send_transaction",
    "type": "native_token_transfer",
    "to": "0x123...",
    "amount": "100",
    "asset": "PHRS"
  }
}
```

## proceed_with_caution

Meaning: The action may continue, but the agent should preserve user confirmation and risk visibility.

Return when:

- Trust score is `60-79`.
- The target has mostly positive evidence with minor uncertainty or limited history.
- The action is routine or low-to-moderate value.

Example:

```json
{
  "recommendation": "proceed_with_caution",
  "nextAction": {
    "skill": "contract_interaction",
    "type": "guarded_contract_call",
    "requiresUserConfirmation": true
  }
}
```

## request_additional_verification

Meaning: More identity, contract, protocol, or agent evidence is needed before value moves.

Return when:

- Trust score is `40-59`.
- Key evidence is missing, uncertain, stale, or contradictory.
- The target may be legitimate but cannot be trusted confidently.

Example:

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

## use_alternative_path

Meaning: The direct action is too risky; use a safer mechanism.

Return when:

- Trust score is `20-39`.
- The target has weak maturity, weak reliability, or meaningful security concerns.
- The user may still need a protected way to proceed.

Example:

```json
{
  "recommendation": "use_alternative_path",
  "nextAction": {
    "skill": "escrow",
    "type": "protected_payment",
    "requiresUserConfirmation": true
  }
}
```

## reject

Meaning: The action should not proceed.

Return when:

- Trust score is `0-19`.
- The target has critical suspicious indicators.
- The action would expose the user or agent to unacceptable risk.

Example:

```json
{
  "recommendation": "reject",
  "nextAction": {
    "skill": "none",
    "type": "halt"
  }
}
```
