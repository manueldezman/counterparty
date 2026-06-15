# Trust Model

Counterparty scores trust from `0` to `100` using three weighted categories:

| Category | Weight |
| --- | ---: |
| Counterparty Maturity | 30% |
| Economic Reliability | 40% |
| Security Posture | 30% |

The final score is the weighted sum of the category scores. Each category should be scored from `0` to `100` using available evidence. Missing, uncertain, stale, or contradictory data should reduce confidence and trigger a safer recommendation.

## Wallet Targets

### Counterparty Maturity

Assess whether the wallet appears established and consistently used.

- Wallet age.
- Activity consistency.
- Transaction history depth.
- Counterparty diversity.
- Repeated interaction with known legitimate entities.

### Economic Reliability

Assess whether the wallet has a history of reliable economic behavior.

- Successful completed transfers.
- Low failed-transaction rate.
- Repeated payments without dispute signals.
- Asset flow consistency with the requested action.
- Prior interaction history with the user or protocol.

### Security Posture

Assess whether the wallet shows suspicious or compromised behavior.

- Suspicious interaction patterns.
- Links to known malicious contracts or scam clusters.
- Recent abnormal activity.
- Unusual approval or draining behavior.
- Sanctioned, flagged, or high-risk counterparties when data is available.

## Contract and Protocol Targets

### Counterparty Maturity

Assess whether the contract or protocol is established.

- Contract age.
- Deployment history.
- Usage history.
- User adoption.
- Continuity of protocol operations.

### Economic Reliability

Assess whether the contract or protocol has handled value reliably.

- Total and recent usage.
- Successful deposits, withdrawals, swaps, stakes, or calls.
- Liquidity depth for economic actions.
- Incident history.
- Observable user retention or repeat usage.

### Security Posture

Assess technical and governance risk.

- Source verification.
- Security indicators such as audits, bug bounties, or public reviews.
- Upgradeability and admin risk.
- Privileged roles, pause controls, or withdrawal controls.
- Proxy patterns and implementation changes.
- Known exploit, phishing, or impersonation signals.

## Agent Targets

### Counterparty Maturity

Assess whether the agent has an established operating history.

- Historical activity.
- Usage frequency.
- Age of agent identity.
- Consistent task completion patterns.

### Economic Reliability

Assess whether the agent completes economically meaningful work reliably.

- Interaction history.
- Prior delegated-task outcomes.
- Reliability indicators.
- Failed, disputed, or abandoned work.
- Repeat usage by trusted users or protocols.

### Security Posture

Assess whether the agent can be trusted with the requested authority.

- Identity or attestation evidence.
- Permission scope requested.
- Access to funds, approvals, or sensitive workflows.
- Prior malicious, anomalous, or policy-violating behavior.
- Dependency, operator, or model risk when available.

## Uncertainty Handling

Missing data is a risk signal. Do not fill gaps with optimistic assumptions.

- If evidence is incomplete but not alarming, prefer `request_additional_verification`.
- If the action moves material value and trust is incomplete, prefer `escrow` or `identity_verification`.
- If suspicious indicators exist, reduce the score even when maturity signals are strong.
- If critical risk is detected, recommend `reject` and route to `none`.
